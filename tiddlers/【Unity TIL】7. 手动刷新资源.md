## 背景

最近的项目，Unity 版本是 2020.3，电脑是 mac，不清楚是哪个的原因，unity 经常崩溃。代码敲着敲着就崩溃，关闭了运行时编译代码还是崩，后来就直接关了 AutoRefrash，崩溃的问题暂时是没有了（之前在 win 上用 2018 从来没有这种情况）。

## 需求

每次进来手动 Command-R 一下，unity 总是会停上一小会，然后编译开始转圈，有时不清楚到底是没有修改，还是正在刷新，操作几下 unity，又崩了，所以想在刷新后，让 unity 显示自己在干嘛。

## 解决方案

做一个编辑器扩展，在执行后，开一个 EditorWindow，调用 AssetDatabase.Refresh，然后在编译完成前，显示在干嘛。

```csharp
using UnityEditor;
using UnityEngine;

public class ManualRefresh : EditorWindow 
{
    [MenuItem("Tools/Manual Refresh %&r", false, 0)]
    public static void Open()
    {
        var window = GetWindow<ManualRefresh>();
        window.titleContent = new GUIContent("手动刷新");
        window.minSize = new Vector2(200, 50);
        window.maxSize = window.minSize;

        if (EditorApplication.isPlaying) {
            EditorApplication.isPlaying = false;
        }

        AssetDatabase.Refresh();
        Debug.Log("开始刷新资源");
    }

    private void OnGUI()
    {
        EditorGUILayout.Space();
        EditorGUILayout.Space();

        if (EditorUtility.scriptCompilationFailed)
        {
            Debug.Log("编译错误");
            Close();
            return;
        }

        if (EditorApplication.isCompiling)
        {
            EditorGUILayout.LabelField("正在编译");
            return;
        }
        
        Close();
    }
}
```

## 参考链接

* https://www.cnblogs.com/zouqiang/p/12795019.html
* https://zhuanlan.zhihu.com/p/580760194
