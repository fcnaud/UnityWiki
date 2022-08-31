> 课程练习，[Shader Development from Scratch for Unity with Cg | Udemy](https://www.udemy.com/course/unity-shaders/) 关于光照模型这一小节的练习。

在 Surface Shader 中实现基本的光照模型。

## 内置光照模式

Surface Shader 有两个内置的光照模式，定义在 *Lighting.cginc* 中

* Lambert。diffuse lighting 漫反射
* BlinnPhong。specular lighting 高光反射

## 如何使用

通过 `progama` 指令

```glsl
#pragma surface surf Lambert
```

## 自定义光照模型

创建一个以 Lighting 为开头的函数，例如

```glsl
half4 LightingCustomLambert(SurfaceOutput s)
```

## Lambert

```glsl
        #pragma surface surf BasicLambert
        
        half4 LightingBasicLambert(SurfaceOutput s, half3 lightDir, half atten)
        {
            half NdotL = dot(s.Normal, lightDir);
        
            half4 c;
            c.rgb = s.Albedo * _LightColor0.rgb * (NdotL * atten);
            c.a = s.Alpha;
            return c;
        }
```

* 加入了光线入射角的影响

## BlinnPhong

```glsl
    #pragma surface surf BlinnPhong 
    
    half4 LightingBasicBlinnPhong(SurfaceOutput s, half3 lightDir, half3 viewDir, half atten)
    {
        half3 h = normalize(lightDir + viewDir);
        float nh = max(0, dot(s.Normal, h));
        float spec = pow(nh, 48.0);
        
        float diff = max(0, dot(s.Normal, lightDir));
        
        half4 c;
        c.rgb = (s.Albedo * _LightColor0.rgb * diff + _LightColor0.rgb * spec) * atten;
        c.a = s.Alpha;
        return c;
    }
```

* 加入和观察角度的影响
* 48 是因为 Unity 采用了这个数字

## Toon  

```glsl
     #pragma surface surf Toon 
     
     sampler2D _RampTex;
    
    float4 LightingToon(SurfaceOutput s, float3 lightDir, float atten)
    {
        float diff = dot(s.Normal, lightDir);
        float h = diff * 0.5 + 0.5;
        float3 ramp = tex2D(_RampTex, float2(h, 0)).rgb;
        
        float4 c;
        c.rgb = s.Albedo * _LightColor0.rgb * ramp;
        c.a = s.Alpha;
        return c;
    }
```

## 参考链接

* [Unity - Manual: Custom lighting models in Surface Shaders (unity3d.com)](https://docs.unity3d.com/Manual/SL-SurfaceShaderLighting.html)
* [Unity - Manual: Surface Shader lighting examples (unity3d.com)](https://docs.unity3d.com/Manual/SL-SurfaceShaderLightingExamples.html)
* [Shader Development from Scratch for Unity with Cg | Udemy](https://www.udemy.com/course/unity-shaders/)

