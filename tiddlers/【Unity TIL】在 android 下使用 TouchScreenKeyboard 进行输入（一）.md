一些情况下，可能会有这样的需求，只要调起键盘进行输入，不需要像 UGUI 中那样有一个显示的 Field，这时就直接使用 TouchScreenKeyboard 调动系统输入来处理。
## 如何使用
1. TouchScreenKeyboard.Open 调起 keyboard
2. 在 Update 中判断 keyboard 的状态，当不可见并处于 Done 状态时视为输入完成

## 遇到的问题
在 2018.4.19（我使用的版本）中，唤起 keyboard 后，在 keyboard 外点击，keyboard 收起，不能正确识别为取消输入，导致再次唤起 keyboard 失败，丢失焦点（切换应用到后台再切回来）会刷新。

## 如何解决
在 keyboard 收起后，同样会有一次焦点的切换，在 OnApplicationFocus 中进行判断，当获得焦点，keyboard 依然处于可视状态时，视为 cancel。

## 实例
```c#
using System;
using System.Collections.Generic;
using UnityEngine;
using UnityEngine.UI;

public class Test
{
    private string lastInput = String.Empty;
    private TouchScreenKeyboard keyboard;

    // 调起键盘
    public void OnChatClick()
    {
        // Open 根据需要修改具体的参数
        keyboard = TouchScreenKeyboard.Open(lastInput, TouchScreenKeyboardType.ASCIICapable, true, true);
    }

    // 处理输入
    public void DealInput(string content)
    {

    }

    // 检测输入完毕
    private void Update()
    {
        if (TouchScreenKeyboard.visible == false && keyboard != null)
        {
            if (keyboard.status == TouchScreenKeyboard.Status.Done)
            {
                DealInput(keyboard.text);
                lastInput = String.Empty;
                keyboard = null;
            }
        }
    }
    
    // 因为在 android 上点击键盘外边，会收起但是不会被判定为 cancel
    // 所以补一个键盘失去焦点的判断
    // 2018.4.19 是这样，后边不清楚有没有改
    private void OnApplicationFocus(bool hasFocus)
    {
        if (hasFocus)
        {
            if (keyboard != null && keyboard.status == TouchScreenKeyboard.Status.Visible)
            {
                keyboard.active = false;
                lastInput = keyboard.text;
                keyboard = null;
            }
        }
    }
}
```

## 参考链接
* [HoloLens开发手记 - Unity之Keyboard input 键盘输入 - msp的昌伟哥哥 - 博客园 (cnblogs.com)](https://www.cnblogs.com/mantgh/p/5680580.html)
* [Unity - Scripting API: TouchScreenKeyboard](https://docs.unity.cn/2021.3/Documentation/ScriptReference/TouchScreenKeyboard.html)
