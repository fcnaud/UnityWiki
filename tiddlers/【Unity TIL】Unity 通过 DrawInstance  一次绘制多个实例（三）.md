游戏开发中，可能会碰到一次绘制多个拥有相同 mesh 的物体，它们可能是位置，旋转等不同，或者是材质的某些参数不同，我们要做的就是配合 Unity 的渲染方式，尽可能地减少绘制的操作。

由于下文主要讨论的是 DrawInstance 和 MaterialPropertyBlock，所以其他的一下影响合批的内容没有讨论。

## 动态合批
绘制多个相同 Mesh，相同 Material 的物体时，打开 Material 的 Gpu Instance 选项，在满足顶点限制的条件下，会进行动态合批（动态合批需要打开）。

![image-20211109114411477](https://fcnaud-note.oss-cn-beijing.aliyuncs.com/note-images/2021/11/09/11-44-12-image-20211109114411477-43251b.png)

![image-20211109114322083](https://fcnaud-note.oss-cn-beijing.aliyuncs.com/note-images/2021/11/09/11-43-23-image-20211109114322083-6ba150.png)


## 不同 Material 会打断合批

修改 Material 的参数，这样会打断合批。并且 Unity 会为每一个物体创建一个 Material 的实例。

```c#
public class TestDrawInstance : MonoBehaviour
{
    public Transform[] Cubes;
    
    void Start()
    {
        foreach (var cube in Cubes)
        {
            cube.GetComponent<Renderer>().material.
                SetColor("_Color", new Color(
                Random.Range(0f, 1f),
                Random.Range(0f, 1f),
                Random.Range(0f, 1f)));
        }
    }
}
```

![image-20211109131033777](https://fcnaud-note.oss-cn-beijing.aliyuncs.com/note-images/2021/11/09/13-10-35-image-20211109131033777-d1100d.png)

![image-20211109131242879](https://fcnaud-note.oss-cn-beijing.aliyuncs.com/note-images/2021/11/09/13-12-45-image-20211109131242879-3702a0.png)

![image-20211109131316338](https://fcnaud-note.oss-cn-beijing.aliyuncs.com/note-images/2021/11/09/13-13-17-image-20211109131316338-a3b452.png)

![image-20211109131714801](https://fcnaud-note.oss-cn-beijing.aliyuncs.com/note-images/2021/11/09/13-17-16-image-20211109131714801-f88800.png)

## 通过 MaterialPropertyBlock 避免创建 Material 实例

这时可以借助 MaterialPropertyBlock 来设置不同的属性，但是这只会节省，创建多个 Material 的消耗，~~并不会影响到渲染，依然会打断合批。~~ 不会打断，满足动态合批的条件时，会进行合批。

``` c#
MaterialPropertyBlock propertyBlock = new MaterialPropertyBlock();
foreach (var cube in Cubes)
{
    // 设置不同的属性
    propertyBlock.SetColor("_Color", new Color(
        Random.Range(0f, 1f),
        Random.Range(0f, 1f),
        Random.Range(0f, 1f)));
    
    // 设置 PropertyBlock 替代设置 material 属性
    cube.GetComponent<Renderer>()
        .SetPropertyBlock(propertyBlock);
}
```

## DrawMeshInstanced 一次性绘制

直接使用 DrawMeshInstanced API 也可以达到合批的作用，但是无法设置不同的参数。同时需要注意一次 DrawInstance 调用最多绘制 1023 个物体的限制。

``` C#
public Transform[] Cubes;
private Material drawMat;
Mesh drawMesh;
Matrix4x4[] trsMat;

private void Update()
{
    // 使用 DrawInstance 绘制一次性绘制
    drawMat = Cubes[0].GetComponent<Renderer>().sharedMaterial;
    drawMesh = Cubes[0].GetComponent<MeshFilter>().sharedMesh;
    trsMat = new Matrix4x4[Cubes.Length];

    for (int i = 0; i < Cubes.Length; i++) {
        trsMat[i] = Matrix4x4.TRS(
            Cubes[i].position, 
            Cubes[i].rotation, 
            Cubes[i].lossyScale);
        Cubes[i].GetComponent<Renderer>().enabled = false;
    }
    
    Graphics.DrawMeshInstanced(
        drawMesh,
        0,
        drawMat,
        trsMat,
        Cubes.Length,
        null,
        ShadowCastingMode.Off,
        false
        );
}

```

![image-20211109133753364](https://fcnaud-note.oss-cn-beijing.aliyuncs.com/note-images/2021/11/09/13-37-54-image-20211109133753364-232d19.png)

### DrawMeshInstanced 配合 MaterialPropertyBlock 进行绘制

DrawMeshInstanced 和 MaterialPropertyBlock 同样是可以进行配合的，但是需要编写能处理 MaterialPropertyBlock 数据的 shader。

首先要 [编写支持 GPU Instancing 的 Shader](https://zhuanlan.zhihu.com/p/34499251)。以顶点片元 shader 为例，需要如下步骤

1. 开启 gpu instance。 `#pragma multi_compile_instancing`
2. 在结构体中添加 id 声明。`UNITY_VERTEX_INPUT_INSTANCE_ID`
3. 定义数据。
4. setup，在 vert/frag 中使用需要先 `UNITY_SETUP_INSTANCE_ID`。在 frag 中使用还要在 vert 中使用 `UNITY_TRANSFER_INSTANCE_ID`
5. 获取数据。`UNITY_ACCESS_INSTANCED_PROP`

shader sample

```glsl

Shader "Custom/TestDrawInstance"
{
    Properties
    {
        _MainTex ("Texture", 2D) = "white" {}
    }
    SubShader
    {
        Tags { "RenderType"="Opaque" }
        LOD 100

        Pass
        {
            CGPROGRAM
            #pragma vertex vert
            #pragma fragment frag
            // make fog work
            #pragma multi_compile_fog
            // 1. 开启 gpu instancing
            #pragma multi_compile_instancing

            #include "UnityCG.cginc"

            struct appdata
            {
                float4 vertex : POSITION;
                float2 uv : TEXCOORD0;
                // 2. 在数据中添加一个语义为 SV_InstanceID 的元素
                // 在 vertex 中使用
                UNITY_VERTEX_INPUT_INSTANCE_ID
            };

            struct v2f
            {
                float2 uv : TEXCOORD0;
                UNITY_FOG_COORDS(1)
                float4 vertex : SV_POSITION;
                
                // 2. 在数据中添加一个语义为 SV_InstanceID 的元素
                // 在 fragment 中使用
                UNITY_VERTEX_INPUT_INSTANCE_ID
            };

            // 3. 定义数据, name 为任意字符串, START 和 END 成对使用
            UNITY_INSTANCING_BUFFER_START(Props)
                // 定义一个实例属性
                UNITY_DEFINE_INSTANCED_PROP(float4, _Color)
            UNITY_INSTANCING_BUFFER_END(Props)

            sampler2D _MainTex;
            float4 _MainTex_ST;

            v2f vert (appdata v)
            {
                v2f o;
                
                // 4. setup
                UNITY_SETUP_INSTANCE_ID(v);
                // 在 fragment 中使用，需要在此进行设置
                UNITY_TRANSFER_INSTANCE_ID(v, o);
                
                o.vertex = UnityObjectToClipPos(v.vertex);
                o.uv = TRANSFORM_TEX(v.uv, _MainTex);
                UNITY_TRANSFER_FOG(o,o.vertex);
                return o;
            }

            fixed4 frag (v2f i) : SV_Target
            {
                // 4. setup
                UNITY_SETUP_INSTANCE_ID(i);
                // sample the texture
                fixed4 col = tex2D(_MainTex, i.uv);
                // 5. 获取实例数据
                col = UNITY_ACCESS_INSTANCED_PROP(Props, _Color);
                // apply fog
                UNITY_APPLY_FOG(i.fogCoord, col);
                return col;
            }
            ENDCG
        }
    }
}
```

使用代码控制绘制

```c#
public class TestDrawInstance : MonoBehaviour
{
    public Transform[] Cubes;
    private Material drawMat;
    private Mesh drawMesh;
    private Matrix4x4[] trsMat;
    private MaterialPropertyBlock instanceBlock;
    private Vector4[] colors;

    private void Start()
    {
        drawMat = Cubes[0].GetComponent<Renderer>().sharedMaterial;
        drawMesh = Cubes[0].GetComponent<MeshFilter>().sharedMesh;
        trsMat = new Matrix4x4[Cubes.Length];
        colors = new Vector4[Cubes.Length];

        instanceBlock = new MaterialPropertyBlock();
        for (int i = 0; i < Cubes.Length; i++)
        {
            colors[i] = new Vector4(
                Random.Range(0f, 1f),
                Random.Range(0f, 1f),
                Random.Range(0f, 1f),
                1);
            Cubes[i].GetComponent<Renderer>().enabled = false;
        }
        instanceBlock.SetVectorArray("_Color", colors);
    }

    private void Update()
    {
        for (int i = 0; i < Cubes.Length; i++) {
            trsMat[i] = Matrix4x4.TRS(
                Cubes[i].position, 
                Cubes[i].rotation, 
                Cubes[i].lossyScale);
        }
        
        Graphics.DrawMeshInstanced(
            drawMesh,
            0,
            drawMat,
            trsMat,
            Cubes.Length,
            instanceBlock,
            ShadowCastingMode.Off,
            false
            );
    }
}
```

## 参考链接

* [Material Property Block 使用](https://blog.csdn.net/liweizhao/article/details/81937590)
* [DrawMeshInstanced 配合 MeterialPropertyBlock 进行绘制](https://blog.csdn.net/le12380/article/details/104842394)
* [GPU Instancing](https://zhuanlan.zhihu.com/p/34499251)
