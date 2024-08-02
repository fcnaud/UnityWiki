> 原文链接：[Unity Text Mesh 穿透问题 - fcnaud - 博客园](https://www.cnblogs.com/fcnaud/p/18241815)

## 0. 问题
在 3D 场景中使用 TextMesh 的时候，字体无法被遮挡，永远在最上层。

![1e64e4def979eec5b617f9aeeac30a8f.png](https://fcnaud-note.oss-cn-beijing.aliyuncs.com/note-images/2024/06/07/16-11-04-1e64e4def979eec5b617f9aeeac30a8f.png)

虽然目前在场景中可以直接使用 TextMeshPro，但是实际开发的时候总会有各种各样的情况，可能是兼容老项目，也可能是想保持项目足够简单，不想引入 TextMeshPro。这里就只记录如何解决这一问题的方案。

## 1. 原因
主要是因为 TextMesh 使用的 Shader 里边有这样一句。
```hlsl
ZTEST Always
```
也就是永远通过深度检测，所以会一直处于最上层。

## 2. 解决方案
找到 TextMesh 默认使用的 shader，删除这一句，做一个替换。

## 3. 实际处理
### 3.1 shader
由于使用的 Unity 的内置 shader，所以无法直接修改，需要去 Unity 提供的下载地址找。[Download Archive (unity.com)](https://unity.com/releases/editor/archive) 下载相应的版本

![4c093fe492307657a1cec7abf5ce0cc0.png](https://fcnaud-note.oss-cn-beijing.aliyuncs.com/note-images/2024/06/07/16-09-39-4c093fe492307657a1cec7abf5ce0cc0.png)

找到其中的名为 `Font.shader` 的文件，shader 名为 `"GUI/Text Shader"`。
创建一个新的 shader 并删除 `ZTEST Always` 即可。

![b713f09bbc7db6d0f870fb68e6056ce9.png](https://fcnaud-note.oss-cn-beijing.aliyuncs.com/note-images/2024/06/07/16-09-56-b713f09bbc7db6d0f870fb68e6056ce9.png)
### 3.2 material
创建一个新的材质球，并设置 shader 为上一步创建的。替换掉默认使用的字体材质球后，你会发现，字体并不能正确显示。这是因为字体贴图无法和自定义的这个材质球自动关联，需要手动进行设置。(另外一个方式就是创建一个可编辑的字体，Create Editable Copy)
![00ccb040305b4f05c399ec3289000433.png](https://fcnaud-note.oss-cn-beijing.aliyuncs.com/note-images/2024/06/07/16-12-25-00ccb040305b4f05c399ec3289000433.png)

把字体文件的贴图拖到材质球的贴图框，就可以了。
![44775d18d07d48cf257db395d3788722.png](https://fcnaud-note.oss-cn-beijing.aliyuncs.com/note-images/2024/06/07/16-13-02-44775d18d07d48cf257db395d3788722.png)

![868fd4e02f24d2e2f2c4301964aa7ca5.png](https://fcnaud-note.oss-cn-beijing.aliyuncs.com/note-images/2024/06/07/16-13-09-868fd4e02f24d2e2f2c4301964aa7ca5.png)

最终结果

![d7c12c4616793215898c36b07efb9379.png](https://fcnaud-note.oss-cn-beijing.aliyuncs.com/note-images/2024/06/07/16-13-22-d7c12c4616793215898c36b07efb9379.png)



