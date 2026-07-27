---
title: GUI学习
published: 2026-07-15
pinned: false
description: Unity GUI学习 - IMGUI系统知识
tags: [Unity,学习,UI]
author: boluobao
draft: false
---

# 🎛️ GUI学习

> [!NOTE] 什么是 IMGUI？
> IMGUI（Immediate Mode GUI，即时模式图形用户界面）是 Unity 内置的**代码驱动** UI 系统。它通过 `OnGUI` 函数中直接调用 GUI 静态方法来绘制界面，无需在场景中创建 UI 对象。

---

## 📖 GUI 概念

> 全称 **即时模式游戏用户交互界面（IMGUI）**，是一个由代码驱动的 UI 系统。

---

## 🎯 GUI 作用

| 用途 | 说明 |
|------|------|
| 🔧 调试工具 | 创建游戏内调试工具，快速显示运行时数据 |
| 🎨 自定义检视面板 | 为脚本组件创建自定义 Inspector 面板 |
| 🏗️ 编辑器扩展 | 创建新的编辑器窗口和工具以拓展 Unity 本身 |

> [!WARNING] 重要提醒
> **不要用它为玩家制作 UI 功能！** IMGUI 主要用于开发和编辑器阶段，实际游戏的玩家 UI 请使用 UGUI。

---

## ⚙️ GUI 的工作原理

在继承 `MonoBehaviour` 的脚本中，使用 GUI 函数绘制 IMGUI：

```csharp
private void OnGUI()
{
    // 绘制IMGUI
}
```

> [!IMPORTANT] OnGUI 注意事项
> 1. 它**每帧执行**，是专门用于绘制 GUI 界面的函数
> 2. 一般只在其中执行 GUI 相关界面绘制和操作逻辑
> 3. 该函数在 `OnDisable` 之前、`LateUpdate` 之后执行
> 4. 只要是继承 `MonoBehaviour` 的脚本，都可以使用 `OnGUI` 函数绘制 IMGUI

---

## 🔘 文本和按钮控件

### GUI 绘制的共同点

| 特点 | 说明 |
|------|------|
| **来源** | 都是 `GUI` 公共类中提供的静态函数，直接调用即可 |
| **参数设计** | 各控件参数大同小异 |
| **重载丰富** | 每种控件都有多种重载，是各个参数的排列组合 |
| **必备参数** | 位置信息和显示信息 |

### 通用参数一览

| 参数类型 | 参数名 | 说明 |
|------|------|------|
| 位置参数 | `Rect` | xy 位置 + wh 尺寸 |
| 显示文本 | `string` | 文本内容 |
| 图片信息 | `Texture` | 贴图资源 |
| 综合信息 | `GUIContent` | 同时包含文本、图片、提示信息 |
| 自定义样式 | `GUIStyle` | 外观样式 |

---

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

---

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

| 按钮类型 | 触发方式 | 适用场景 |
|------|------|------|
| **Button** | 点击一次触发一次 | 普通按钮、确认/取消 |
| **RepeatButton** | 按住时每帧持续触发 | 滚动条箭头、加减按钮 |

---

## ✅ 多选框和单选框

### 多选框（Toggle）

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

> [!TIP] Toggle 样式调节
> 如果要让 Toggle 的方框和文字对齐更好看，调整 `GUIStyle` 的 `fixedWidth`、`fixedHeight` 和 `padding` 属性。

---

### 单选框（Radio Button）

> IMGUI 没有内置单选框控件，通过 **Toggle + 手动控制 int 索引** 来实现互斥选择。

```csharp
private int index = 0;
private void OnGUI()
{
    //通过int变量来实现单选框
    if(GUI.Toggle(new Rect(0, 0, 100, 20), index == 1, "选项一"))
    {
        index = 1;
    }

    if(GUI.Toggle(new Rect(100, 0, 100, 20), index == 2, "选项二"))
    {
        index = 2;
    }

    if(GUI.Toggle(new Rect(200, 0, 100, 20), index == 3, "选项三"))
    {
        index = 3;
    }
}
```

> 核心思路：用一个 int 变量记录当前选中项，每个 Toggle 的值 = `index == 自己的编号`，点击时修改 index。

---

## 📝 输入框和拖动条

### 输入框（TextField）

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

| 控件 | 说明 | 返回值 |
|------|------|------|
| **TextField** | 单行文本输入 | 更新后的字符串 |
| **PasswordField** | 密码输入框（字符被替换） | 更新后的字符串 |

---

### 拖动条（Slider）

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

| 参数 | 说明 |
|------|------|
| `position` | 滑动条位置和尺寸 |
| `value` | 当前值 |
| `leftValue` / `topValue` | 最小值 |
| `rightValue` / `bottomValue` | 最大值 |

---

## 🖼️ 图片绘制和框

### 图片绘制

```csharp
private Texture tex;
public Rect rect;
public ScaleMode mode;
public bool alpha = true;
public float wh = 1;
private void OnGUI()
{
    //这里修改rect后图片并不会是等比缩放的 这里和文本和按钮那的图片变化不同
    GUI.DrawTexture(rect, tex);
    //当然可以通过mode参数来设置图片的显示模式
    //默认是ScaleMode.StretchToFill
    //ScaleAndCrop：等比例缩放并裁剪图片
    //ScaleToFit：等比例缩放图片并填充矩形
    GUI.DrawTexture(rect, tex, mode);
    GUI.DrawTexture(rect, tex, mode, alpha, wh); //alpha参数用于设置是否显示透明通道 wh参数用于设置缩放比例
}
```

| ScaleMode | 效果 |
|------|------|
| **StretchToFill**（默认） | 拉伸填充，不保持宽高比 |
| **ScaleAndCrop** | 等比例缩放并裁剪多余部分 |
| **ScaleToFit** | 等比例缩放，完整显示图片 |

---

### 框（Box）

```csharp
public Rect rect;
public GUIStyle style;
private void OnGUI()
{
    GUI.Box(rect, "这是一个框");
}
```

> `Box` 是一个带边框和背景的矩形区域，常用于面板背景或分组装饰。

---

## 🧭 工具栏和选择网格

### 工具栏（Toolbar）

```csharp
public Rect rect;
private int toolBarIndex = 0;
private string[] toolBarItems = {"选项1", "选项2", "选项3"};
private void OnGUI()
{
    toolBarIndex = GUI.Toolbar(rect, toolBarIndex, toolBarItems);
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

> 工具栏是一组水平排列的按钮，同一时间只有一个被选中，类似编辑器顶部的工具切换条。

---

### 选择网格（SelectionGrid）

```csharp
public Rect rect;
private int gridIndex = 0;
private string[] gridItems = {"选项1", "选项2", "选项3"};
private void OnGUI()
{
    //相比于toolBar多了xCount参数 代表水平方向最多显示的按钮数量
    gridIndex = GUI.SelectionGrid(rect, gridIndex, gridItems, 3);
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

| 控件 | 布局 | 特殊参数 |
|------|------|------|
| **Toolbar** | 水平单行 | 无 |
| **SelectionGrid** | 网格多行 | `xCount`：每行的按钮数量 |

---

## 📜 滚动视图和分组

### 分组（BeginGroup / EndGroup）

> 用于批量控制控件位置。可以理解为**包裹着的控件加了一个父对象**，通过控制分组来控制包裹控件的位置。

```csharp
public Rect groupPos;
private void OnGUI()
{
    GUI.BeginGroup(groupPos);
    GUI.Button(new Rect(0, 0, 100, 50), "这是一个按钮");
    GUI.Label(new Rect(0, 50, 100, 20), "这是一个标签");
    GUI.EndGroup();
}
```

> [!TIP] 分组的好处
> Group 内部的控件坐标为**相对于 Group 左上角的坐标**，移动 Group 即可整体移动内部所有控件。

---

### 滚动视图（ScrollView）

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

| 参数 | 说明 |
|------|------|
| `scrollPos` | 滚动视图的显示区域 Rect |
| `nowPos` | 当前滚动位置（Vector2） |
| `showPos` | 内容的总区域 Rect（大于 scrollPos 时出现滚动条） |

---

## 🪟 窗口

### 普通窗口

```csharp
private void OnGUI()
{
    //参数一为窗口id 是唯一的窗口id 不要与别的窗口重复
    //委托参数 用于绘制窗口用的函数 想要显示的内容直接在函数中绘制

    GUI.Window(1, new Rect(0, 0, 100, 100), DrawWindow, "测试窗口");
}

private void DrawWindow(int id)
{
    // 在此绘制窗口内的控件
}
```

### 模态窗口

> 可以让其它控件不再有用，该窗口在最上层，其他按钮都点击不到了。只能点击该窗口上的控件 —— 类似常见的**警告弹窗**，必须先处理该窗口上的操作才能继续操作。

```csharp
private void OnGUI()
{
    GUI.ModalWindow(1, new Rect(0, 0, 100, 100), DrawWindow, "这是一个模态窗口");
}

private void DrawWindow(int id)
{
    // 在此绘制窗口内的控件
}
```

### 拖动窗口

```csharp
private Rect windowRect;

private void OnGUI()
{
    //需要在DrawWindow中调用DragWindow方法 否则窗口不会被拖动
    windowRect = GUI.Window(1, windowRect, DrawWindow, "这是一个拖动窗口");
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

> [!IMPORTANT] 拖动窗口的关键
> `GUI.DragWindow()` 必须写在窗口的回调函数中才能生效。可以传入一个 Rect 参数来指定只有该区域可拖动（如只让标题栏可拖动）。

| 窗口类型 | 特点 |
|------|------|
| **Window** | 普通可交互窗口 |
| **ModalWindow** | 模态窗口，阻塞其他控件交互 |
| **DragWindow (Window + DragWindow)** | 可拖动的窗口 |

---

## 🎨 自定义样式

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

| 颜色属性 | 影响范围 |
|------|------|
| **GUI.color** | ==全局着色==，同时影响背景和文本颜色 |
| **GUI.contentColor** | 仅文本/内容颜色（会与 GUI.color 混合） |
| **GUI.backgroundColor** | 仅背景颜色（会与 GUI.color 混合） |

> 最终颜色 = GUI.color × 对应颜色（contentColor 或 backgroundColor）

---

### 整体皮肤样式

```csharp
public GUISkin skin;

private void OnGUI()
{
    //通过将skin赋值给GUI.skin 来改变全局控件的样式
    GUI.skin = skin;

    //若是在绘制控件时传入了自定义的GUIStyle 则会使用自定义的样式（覆盖皮肤）
}
```

> [!NOTE] GUISkin vs GUIStyle
> - **GUISkin**：一套完整的皮肤，包含所有控件的默认样式（批量设置）
> - **GUIStyle**：单个控件的自定义样式（单独设置，优先级更高）

---

## 📐 GUILayout 自动布局

> 主要用于**游戏编辑器开发**。GUILayout 自动计算控件位置和大小，无需手动指定 Rect。

```csharp
private void OnGUI()
{
    GUILayout.BeginHorizontal();
    GUILayout.Label("这是一个标签1");
    GUILayout.Label("这是一个标签2");
    GUILayout.Label("这是一个标签3");
    GUILayout.EndHorizontal();

    GUILayout.BeginVertical();
    GUILayout.Button("按钮1");
    GUILayout.Button("按钮2");
    GUILayout.EndVertical();
}
```

| 布局模式 | 说明 |
|------|------|
| **BeginHorizontal / EndHorizontal** | 水平排列子控件 |
| **BeginVertical / EndVertical** | 垂直排列子控件 |

---

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

| 选项 | 说明 |
|------|------|
| **Width / Height** | 固定宽高 |
| **MinWidth / MinHeight** | 最小宽高限制 |
| **MaxWidth / MaxHeight** | 最大宽高限制 |
| **ExpandWidth / ExpandHeight** | 是否自动扩展填充剩余空间 |

---

## 🔄 分辨率自适应

> 分辨率自适应就是当分辨率改变时**重新计算 UI 控件的位置**。

```
控件坐标计算公式：
相对屏幕位置 + 中心点偏移位置 + 偏移位置
```

> 通常结合 `Screen.width` 和 `Screen.height` 来动态计算 Rect，使 UI 在不同分辨率下保持相对位置一致。
