---
title: GUI学习
published: 2026-07-15
pinned: false
description: Unity GUI学习 - IMGUI系统知识
tags: [Unity,学习,UI]
author: boluobao
draft: false
---

# GUI学习

## GUI概念

全称 即使模式游戏用户交互界面 (IMGUI) 是一个由代码驱动的UI系统  

## GUI作用

1.作为程序员的调试工具，创建游戏内调试工具  
2.为脚本组件创建自定义检视面板  
3.创建新的编辑器窗口和工具以拓展Unity本身（一般用作游戏内置工具）   

注意：不要用它为玩家制作UI功能  

## GUI的工作原理

在继承MonoBehaviour的脚本中，使用GUI函数绘制IMGUI   

```csharp
private void OnGUI()
{
    // 绘制IMGUI
}
```
注意：  
1.它每帧执行 相当于是用于专门绘制GUI界面的函数  
2.一般只在其中执行GUI相关界面绘制和操作逻辑 
3.该函数在OnDisable之前 LateUpdate之后执行  
4.只要是继承MonoBehaviour的脚本，都可以使用OnGUI函数绘制IMGUI   

## 文本和按钮控件

### GUI绘制的共同点

1.他们都是GUI公共类中提供的静态函数直接调用即可 
2.他们的参数都大同小异  
位置参数：Rect参数 xy位置wh尺寸 
显示文本：string参数    
图片信息：Texture参数   
综合信息：GUIContent参数    
自定义样式：GUIStyle参数    
3.每一种控件都有多种重载，都是各个参数的排列组合    
必备的参数内容是位置信息和显示信息  

### 文本控件

```csharp
public Texture tex;
public Rect rect;
public GUIStyle style;
private void OnGUI()
{
    GUI.Label(rect, "这是一个文本"); // 显示文本
    GUI.Label(rect, tex); // 显示图片
    GUI.Label(rect, new GUIContent("这是一个文本", tex)); // 同时显示文本和图片 
    //自定义样式
    GUI.Label(rect, "这是一个文本", style); // 显示自定义样式的文本
}
```

### 按钮控件

```csharp
public Rect rect;
public GUIStyle style;
public GUIContent content;
private void OnGUI()
{
    GUI.Button(rect, "这是一个按钮"); // 显示按钮
    //自定义样式
    GUI.Button(rect, content, style); // 显示自定义样式的按钮

    //使用
    if(GUI.Button(rect, content, style))
    {
        // 点击按钮后的操作
    }

    if(GUI.RepeatButton(rect, content, style))
    {
        // 长按按钮后的操作
    }
}
```

## 多选框和单选框

### 多选框

```csharp
private bool isSel;
public Rect rect;
public GUIStyle style;
private void OnGUI()
{
    isSel = GUI.Toggle(rect, isSel, "这是一个多选框");

    // 自定义样式
    //修改固定宽高 fixedWidth和fixedHeight
    //修改从GUIStyle边缘到内容起始处的空间 padding
    GUI.Toggle(rect, isSel, "这是一个多选框", style);
}
```

### 单选框

```csharp
private int index = 0;
private void OnGUI()
{
    //通过int变量来实现单选框
    if(index = GUI.Toggle(new Rect(0, 0, 100, 20), index == 1, "这是一个单选框"))
    {
        index = 1;
    }

    if(index = GUI.Toggle(new Rect(100, 0, 100, 20), index == 2, "这是一个单选框"))
    {
        index = 2;
    }

    if(index = GUI.Toggle(new Rect(200, 0, 100, 20), index == 3, "这是一个单选框"))
    {
        index = 3;
    }
}
```

## 输入框和拖动条

### 输入框

```csharp
private string input;
private string password;
private void OnGUI()
{
    //和文本框不同的是，输入框需要使用string变量来存储输入的内容
    input = GUI.TextField(new Rect(0, 0, 100, 20), input);

    //密码输入框
    password = GUI.PasswordField(new Rect(0, 0, 100, 20), password, '*');
}
```

### 拖动条

```csharp
private float valueHorizontal;
private float valueVertical;
private void OnGUI()
{
    //水平拖动条
    //当前的值
    //最小值
    //最大值
    valueHorizontal = GUI.HorizontalSlider(new Rect(0, 0, 100, 20), valueHorizontal, 0, 100);
    // 垂直拖动条
    valueVertical = GUI.VerticalSlider(new Rect(0, 0, 20, 100), valueVertical, 0, 100);
}
```

## 图片绘制和框

### 图片绘制

```csharp
private Texture tex;
public Rect rect;
public TextureMode mode;
public bool alpha = true;
public float wh = 1;
private void OnGUI()
{
    //这里修改rect后图片并不会是等比缩放的 这里和文本和按钮那的图片变化不同
    GUI.DrawTexture(rect, tex);
    //当然可以通过mode参数来设置图片的显示模式
    //默认是TextureMode.StretchToFill
    //ScaleAndCrop：等比例缩放并裁剪图片
    //ScaleToFit：等比例缩放图片并填充矩形
    GUI.DrawTexture(rect, tex, mode);
    GUI.DrawTexture(rect, tex, mode, alpha, wh); //alpha参数用于设置是否显示透明通道 wh参数用于设置缩放比例
}
```

### 框

```csharp
public Rect rect;
public GUIStyle style;
private void OnGUI()
{
    GUI.Box(rect, "这是一个框");
}
```

## 工具栏和选择网格

### 工具栏

```csharp
public Rect rect;
private int toolBarIndex = 0;
private string[] toolBarItems = {"选项1", "选项2", "选项3"};
private void OnGUI()
{
    toolBarIndex = GUI.Toolbar(rect, "这是一个工具栏", toolBarIndex, toolBarItems);
    //使用
    switch(toolBarIndex)
    {
        case 0:
            // 选项1的操作
            break;
        case 1:
            // 选项2的操作
            break;
        case 2:
            // 选项3的操作
            break;
        default:
            break;
    }
}
```

### 选择网格

```csharp
public Rect rect;
private int gridIndex = 0;
private string[] gridItems = {"选项1", "选项2", "选项3"};
private void OnGUI()
{
    //相比于toolBar多了xCount参数 代表水平方向最多的显示按钮的数量
    gridIndex = GUI.Grid(rect, gridIndex, gridItems, new GUIContent("这是一个选择网格"),3);
    //使用
    switch(gridIndex)
    {
        case 0:
            // 选项1的操作
            break;
        case 1:
            // 选项2的操作
            break;
        case 2:
            // 选项3的操作
            break;
        default:
            break;
    }
}
```

## 滚动视图和分组

### 分组

用于批量控制控件位置    
可以理解为 包裹着的控件加了一个父对象   
可以通过控制分组来控制包裹控件的位置

```csharp
public Rect groupPos;
private void OnGUI()
{
    GUI.BeginGroup(groupPos);
    GUI.Button(new Rect(0, 0, 100, 50), "这是一个按钮");
    GUI.Text(new Rect(0, 0, 100, 20), "这是一个文本框");
    GUI.EndGroup();
}
```

### 滚动列表

```csharp
public Rect scrollPos;
public Rect showPos;
public Vector2 nowPos;
private void OnGUI()
{
    nowPos = GUI.BeginScrollView(scrollPos, nowPos, showPos);
    //中间添加内容
    GUI.EndScrollView();
}
```

## 窗口

```csharp
private void OnGUI()
{
    //参数一为窗口id 是唯一的窗口id 不要与别的窗口重复
    //委托参数 用于绘制窗口用的函数 想要显示的内容直接在函数中绘制

    GUI.Window(1，new Rect(0, 0, 100, 100), DrawWindow, "测试窗口");
}

private void DrawWindow(int id)
{

}

```

### 模态窗口

可以让其它控件不再有用 可以理解为该窗口在最上层 其他按钮都点击不到了    
只能点击该窗口上的控件 可以理解为常见的警告窗口 必须先处理该窗口上的操作才能继续操作

```csharp
private void OnGUI()
{
    GUI.ModalWindow(1, new Rect(0, 0, 100, 100), DrawWindow, "这是一个模态窗口");
}

private void DrawWindow(int id)
{

}
```

### 拖动窗口

```csharp
private Rect windowRect;

private void OnGUI()
{
    //需要在DrawWindow中调用DragWindow方法 否则窗口不会被拖动
    windowRect = GUI.DragWindow(1, windowRect, DrawWindow, "这是一个拖动窗口");
}

private void DrawWindow(int id)
{
    switch(id)
    {
        case 1:
            //该API写在窗口函数中调用
            //传入Rect参数的重载
            //是决定窗口中哪一个部分可以被拖动
            //默认是全都可以拖动
            GUI.DragWindow();
            break;
        default:
            break;
    }
}
```

## 自定义样式

### 全局颜色

```csharp
private void OnGUI()
{
    //设置全局着色颜色 影响背景和文本颜色
    GUI.color = Color.red;

    //设置文本颜色 会和全局着色颜色混合起来
    GUI.contentColor = Color.white;
    //设置背景颜色 会和全局着色颜色混合起来
    GUI.backgroundColor = Color.blue;
}
```

### 整体皮肤样式

```csharp
public GUISkin skin;

private void OnGUI()
{
    //通过将skin赋值给GUI.skin 来改变全局控件的样式
    GUI.skin = skin;

    //若是在绘制控件时传入了自定义的GUIStyle 则会使用自定义的样式
}
```

## GUILayout

### GUILayout 自动布局

主要用于游戏编辑器开发

```csharp
private void ONGUI()
{
    GUILayout.BeginHorizontal();
    GUILayout.Label("这是一个标签1");
    GUILayout.Label("这是一个标签2");
    GUILayout.Label("这是一个标签3");
    GUILayout.EndHorizontal();

}
```

### GUILayoutOption 布局选项

```csharp
//控件的固定宽高
GUILayout.Width(100);
GUILayout.Height(50);
//允许控件的最小宽高
GUILayout.MinWidth(100);
GUILayout.MinHeight(50);
//允许控件的最大宽高
GUILayout.MaxWidth(100);
GUILayout.MaxHeight(50);
//允许或禁止水平拓展
GUILayout.ExpandWidth(true);
//允许或禁止垂直拓展
GUILayout.ExpandHeight(true);
```

## 分辨率自适应

分辨率自适应就是当分辨率改变时重新计算UI控件的位置  
控件坐标计算公式：相对屏幕位置 + 中心点偏移位置 + 偏移位置  