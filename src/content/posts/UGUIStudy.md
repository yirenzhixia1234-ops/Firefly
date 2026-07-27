---
title: UGUI学习
published: 2026-07-20
pinned: false
description: Unity UGUI学习 - Canvas、RectTransform、EventSystem等组件
tags: [Unity,学习,UI]
author: boluobao
draft: true
---

# UGUI学习

# 🖥️ UGUI学习

> [!NOTE] 什么是 UGUI？
> UGUI（Unity GUI）是 Unity 内置的**运行时 UI 系统**，自 Unity 4.6 起替代了旧的 IMGUI 系统。它基于 **Canvas（画布）** 渲染，提供了可视化的 UI 编辑工具，支持锚点布局、自适应分辨率和丰富的事件系统，是目前 Unity 项目中最主流的 UI 解决方案。

---

## 🎨 Canvas 组件

Canvas 是 UGUI 的**渲染容器**，所有 UI 元素都必须作为 Canvas 的子对象才能显示。它负责将 UI 元素合批渲染到屏幕上。

### 渲染模式（Render Mode）

Canvas 支持三种渲染模式，决定了 UI 与 3D 场景的显示关系：

| 模式 | 说明 | 适用场景 |
|------|------|----------|
| **Overlay（覆盖模式）** | UI 始终渲染在场景最前方，无需摄像机 | 常规 HUD、菜单界面（最常用） |
| **Screen Space - Camera** | UI 渲染在指定摄像机前方，3D 物体可插入 UI 之间 | 需要在 UI 前后放置 3D 特效 |
| **World Space** | UI 作为 3D 平面存在于场景中 | 游戏内血条、3D 交互面板 |

---

### Overlay（覆盖模式）

> UI 始终显示在场景内容前方。

| 参数 | 说明 |
|------|------|
| **Pixel Perfect** | 是否开启无锯齿精确渲染（性能换效果） |
| **Sort Order** | 排序层编号，用于控制多个 Canvas 时的渲染先后顺序 |
| **Target Display** | 目标显示设备（多显示器环境下选择在哪个设备上显示） |
| **Additional Shader Channels** | 其他着色器通道，决定着色器可以读取哪些数据（如法线、切线等） |

---

### Screen Space - Camera（屏幕空间-摄像机模式）

> 3D 物体可以显示在 UI 之前。

| 参数 | 说明 |
|------|------|
| **Render Camera** | 用于渲染 UI 的摄像机（如果不设置将类似于覆盖模式） |
| **Plane Distance** | UI 平面在摄像机前方的距离，类似整体 Z 轴偏移 |
| **Sorting Layer** | 所在排序层 |
| **Order in Layer** | 排序层内的序号 |

> [!TIP] 使用场景
> 当需要在 UI 和场景之间插入粒子特效或 3D 模型时，使用此模式并将 Plane Distance 设置在特效和场景之间。

---

### World Space（世界空间模式）

> UI 像 3D 物体一样存在于场景世界中，可以旋转、缩放，3D 物体可遮挡或穿插 UI。

常用于：游戏内角色头顶血条、3D 交互面板、VR/AR 界面。

---

## 📐 Canvas Scaler 组件

Canvas Scaler 控制 **UI 元素在不同分辨率下的缩放适配**，是自适应 UI 的核心组件。

### 缩放模式对比

| 模式 | 原理 | 适用场景 |
|------|------|----------|
| **Constant Pixel Size** | 像素大小固定不变 | 固定分辨率项目、像素风格游戏 |
| **Scale With Screen Size** | 按参考分辨率等比缩放 | 多分辨率适配（推荐，最常用） |
| **Constant Physical Size** | 按物理尺寸（DPI）保持一致 | 跨设备物理尺寸一致的场景 |

---

### Constant Pixel Size（恒定像素模式）

> 无论屏幕分辨率如何，UI 元素的**像素大小都保持不变**。

| 参数 | 说明 |
|------|------|
| **Scale Factor** | 缩放系数，按此系数缩放画布中的所有 UI 元素 |
| **Reference Pixels Per Unit** | 单位参考像素，多少像素对应 Unity 中的一个单位（默认 100 像素/单位） |

> 图片设置中的 **Pixels Per Unit** 设置会和该参数一起参与计算：

```
UI 原始尺寸 = 图片大小（像素） / (Pixels Per Unit / Reference Pixels Per Unit)

```

---

### Scale With Screen Size（随屏幕缩放模式）⭐

> 最常用的缩放模式！根据屏幕分辨率自动适配 UI 元素大小。

| 参数 | 说明 |
|------|------|
| **Reference Resolution** | 参考分辨率，UI 以此为基准进行缩放设计 |
| **Screen Match Mode** | 屏幕匹配模式，决定缩放系数的计算方式 |
| **Match** | 宽高匹配权重（0=按宽度，1=按高度），仅在 MatchWidthOrHeight 模式下有效 |

#### 三种 Screen Match Mode

| 模式 | 原理 | 效果 |
|------|------|------|
| **Match Width Or Height** | 根据 Match 参数在宽度和高度间插值 | ==最灵活，推荐使用== |
| **Expand** | 取宽高中较小的缩放比 | UI 完整显示不裁剪，可能出现黑边 |
| **Shrink** | 取宽高中较大的缩放比 | UI 填满屏幕不黑边，可能被裁剪 |

**计算公式：**

| 模式 | 缩放系数 |
|------|------|
| Expand | `Mathf.Min(屏幕宽 / 参考宽, 屏幕高 / 参考高)` |
| Shrink | `Mathf.Max(屏幕宽 / 参考宽, 屏幕高 / 参考高)` |
| 画布尺寸 | `屏幕尺寸 / 缩放系数` |

> [!IMPORTANT] Expand vs Shrink
> - **Expand**：画布 ≥ 屏幕 → UI 始终完整显示，但边缘可能有黑边（适合固定布局）
> - **Shrink**：画布 ≤ 屏幕 → UI 填满屏幕不留黑边，但内容可能被裁剪（适合全屏界面）

---

### Constant Physical Size（恒定物理模式）

> 无论屏幕分辨率如何，UI 元素的**物理大小**都保持不变。

| 参数 | 说明 |
|------|------|
| **Physical Unit** | 物理单位（厘米/毫米/英寸/点/派卡） |
| **Fallback Screen DPI** | 备用 DPI（当系统无法获取 DPI 时使用） |
| **Default Sprite DPI** | 默认精灵 DPI |

**计算步骤：**

```
① 新单位参考像素 = 单位参考像素 × Physical Unit / Default Sprite DPI
② UI 原始尺寸   = 图片大小（像素） / (Pixels Per Unit / 新单位参考像素)
```

---

## 🎯 Graphic Raycaster 组件

Graphic Raycaster 是 UGUI 的**射线检测器**，负责检测玩家的输入事件（点击、拖拽等）是否命中了 UI 元素。它挂载在 Canvas 上，由 EventSystem 驱动。

| 参数 | 说明 |
|------|------|
| **Ignore Reversed Graphics** | 是否忽略翻转的图形（背面朝上的 UI 不会被检测到） |
| **Blocking Objects** | 射线被哪些类型的 3D 碰撞器阻挡（在 Overlay 模式下无效） |
| **Blocking Mask** | 射线被哪些层级的碰撞器阻挡（在 Overlay 模式下无效） |

> [!NOTE] 工作原理
> Graphic Raycaster 对 Canvas 下的所有 `Graphic` 组件（Image、Text 等）进行射线检测。当检测到命中时，将事件传递给 EventSystem 进行分发处理。

---

## ⚡ EventSystem 组件

EventSystem 是 UGUI 的**事件管理器**，负责接收玩家输入并分发给各个 UI 控件进行处理。它是整个 UI 交互系统的"大脑"。

> [!IMPORTANT] 核心作用
> EventSystem 类似一个**中转站**，它持续轮询检测输入事件，并与 Input Module、Raycaster 等模块协同工作。**没有 EventSystem，所有点击、拖拽等 UI 交互都不会响应。**

---

### 🔄 EventSystem 工作流程

```
┌──────────────┐    ┌──────────────────┐    ┌─────────────────┐    ┌──────────────┐
│  玩家输入     │───▶│  Input Module    │───▶│  EventSystem    │───▶│  目标UI控件   │
│ (鼠标/键盘/触摸)│    │ (处理/标准化输入)  │    │ (事件分发/管理)   │    │ (响应事件)    │
└──────────────┘    └──────────────────┘    └─────────────────┘    └──────────────┘
                                                    │
                                                    ▼
                                           ┌─────────────────┐
                                           │  Raycaster      │
                                           │ (射线检测命中UI)  │
                                           └─────────────────┘
```

| 阶段 | 负责模块 | 做什么 |
|------|----------|--------|
| **① 原始输入** | 硬件 | 鼠标移动/点击、键盘按键、触摸屏 |
| **② 输入标准化** | Input Module | 将原始输入转为标准事件（PointerEventData、AxisEventData） |
| **③ 事件管理** | EventSystem | 驱动 Input Module 更新、管理选中状态、分发事件 |
| **④ 射线检测** | GraphicRaycaster | 检测输入命中了哪个 UI 物体 |
| **⑤ 事件响应** | 各 UI 控件 | Button、Slider 等组件或实现了事件接口的脚本 |

> 每帧的更新顺序：`EventSystem.Update()` → `InputModule.Process()` → 射线检测 → 事件分发到目标对象

---

### 📋 Inspector 参数详解

| 参数 | 类型 | 说明 |
|------|------|------|
| **First Selected** | GameObject | ==启动时默认选中的 UI 对象==。用于设置初始焦点（如主菜单默认高亮的"开始游戏"按钮）。设置后该对象会收到 `ISelectHandler.OnSelect` 事件 |
| **Send Navigation Events** | bool | 是否发送导航事件。开启后，玩家用键盘方向键/WASD/Tab 可以在 UI 控件间切换选中。==关闭则完全禁用键盘导航== |
| **Drag Threshold** | int | ==拖拽阈值（像素）==。鼠标按下后移动超过此距离才算"拖拽"，否则视为"点击"。默认 5px，调大可防止手抖误触发拖拽 |

---

### 🧠 EventSystem 核心 API

#### 1. 获取 EventSystem 实例

```csharp
using UnityEngine.EventSystems;

// 方式一：静态属性（推荐，最常用）
EventSystem currentES = EventSystem.current;

// 方式二：FindObjectOfType（较慢，不推荐频繁调用）
EventSystem currentES = FindObjectOfType<EventSystem>();
```

> `EventSystem.current` 是静态属性，任何时候都可以直接访问当前场景中激活的 EventSystem。==每个场景只有一个 EventSystem 是活跃的==。

#### 2. 判断是否点击在 UI 上

```csharp
// 移动端：判断触摸是否落在 UI 上
if (EventSystem.current.IsPointerOverGameObject())
{
    // 点击在 UI 上，阻止穿透到场景中的 3D 物体
    return;
}

// 通过触摸/鼠标 ID 判断（多点触控场景）
// fingerId: 鼠标左键=-1, 鼠标右键=-2, 触摸=0/1/2...
if (EventSystem.current.IsPointerOverGameObject(fingerId))
{
    // 该手指正在 UI 上
}

// 判断是否在特定类型的 UI 上
// 使用 PointerEventData 可以获取更详细的信息
```

> [!IMPORTANT] 典型用途
> 在 3D 场景中，如果点击 UI 按钮时不希望同时触发场景中物体的点击（如角色移动），就在 Update 中用 `IsPointerOverGameObject()` 判断并拦截。

#### 3. 选中管理

```csharp
// 获取当前选中的对象
GameObject selected = EventSystem.current.currentSelectedGameObject;

// 手动设置选中对象（常用于键盘导航、手柄操作）
EventSystem.current.SetSelectedGameObject(someButton.gameObject);

// 取消所有选中
EventSystem.current.SetSelectedGameObject(null);

// 判断是否正在切换选中的过程中（防止无限循环）
if (!EventSystem.current.alreadySelecting)
{
    EventSystem.current.SetSelectedGameObject(target);
}
```

| 属性/方法 | 说明 |
|-----------|------|
| `currentSelectedGameObject` | 当前选中的 GameObject（只读） |
| `SetSelectedGameObject(GameObject)` | ==手动设置选中对象==，会触发 Select/Deselect 事件 |
| `firstSelectedGameObject` | 对应 Inspector 的 First Selected |
| `alreadySelecting` | 是否正在执行选中操作（防止递归） |
| `sendNavigationEvents` | 对应 Inspector 的 Send Navigation Events |

#### 4. ExecuteEvents — 手动执行 UI 事件

> `ExecuteEvents` 是 EventSystem 提供的**静态工具类**，可以手动对任意 GameObject 模拟执行 UI 事件。

```csharp
using UnityEngine.EventSystems;

// 对目标执行事件的标准写法：
// ExecuteEvents.Execute<接口类型>(目标对象, eventData, 回调函数)

// 示例1：手动触发点击
PointerEventData pointerData = new PointerEventData(EventSystem.current);
ExecuteEvents.Execute<IPointerClickHandler>(
    targetGameObject,
    pointerData,
    (handler, data) => handler.OnPointerClick(data)
);

// 示例2：手动触发拖拽开始
ExecuteEvents.Execute<IBeginDragHandler>(
    targetGameObject,
    pointerData,
    (handler, data) => handler.OnBeginDrag(data)
);

// 示例3：手动触发选中
ExecuteEvents.Execute<ISelectHandler>(
    targetGameObject,
    new BaseEventData(EventSystem.current),
    (handler, data) => handler.OnSelect(data)
);
```

| ExecuteEvents 常用方法 | 说明 |
|------------------------|------|
| `Execute<T>(target, eventData, functor)` | 在目标对象及其父对象上==沿层级冒泡==执行事件 |
| `ExecuteHierarchy<T>(target, eventData, functor)` | 在目标及其父对象上执行（同 Execute） |
| `GetEventHandler<T>(target)` | 沿层级向上查找第一个能处理该类型事件的对象 |
| `CanHandleEvent<T>(target)` | 判断目标是否能处理该类型事件 |

> [!TIP] 什么时候手动调用 ExecuteEvents？
> - 实现**自定义 UI 交互逻辑**（如自己写摇杆、手势识别）
> - 编写**自动化测试**（模拟点击、拖拽操作）
> - 在**非 UI 系统**中转发事件到 UI

---

### 💻 代码使用场景

#### 场景1：点击 UI 时不触发场景点击

```csharp
void Update()
{
    // 鼠标左键按下
    if (Input.GetMouseButtonDown(0))
    {
        // 如果鼠标在 UI 上，不做场景交互
        if (EventSystem.current.IsPointerOverGameObject())
            return;

        // 否则执行场景点击逻辑（如角色移动）
        Ray ray = Camera.main.ScreenPointToRay(Input.mousePosition);
        if (Physics.Raycast(ray, out RaycastHit hit))
        {
            MovePlayerTo(hit.point);
        }
    }
}
```

#### 场景2：键盘导航菜单

```csharp
using UnityEngine.EventSystems;

public class MenuNavigation : MonoBehaviour
{
    public Button playButton;    // 默认选中的按钮
    public Button settingsButton;
    public Button quitButton;

    void Start()
    {
        // 游戏开始时让"开始游戏"按钮高亮
        EventSystem.current.SetSelectedGameObject(playButton.gameObject);
    }

    void Update()
    {
        // 手动实现 Tab 键切换（EventSystem 本身不处理 Tab）
        if (Input.GetKeyDown(KeyCode.Tab))
        {
            GameObject current = EventSystem.current.currentSelectedGameObject;

            if (current == playButton.gameObject)
                EventSystem.current.SetSelectedGameObject(settingsButton.gameObject);
            else if (current == settingsButton.gameObject)
                EventSystem.current.SetSelectedGameObject(quitButton.gameObject);
            else
                EventSystem.current.SetSelectedGameObject(playButton.gameObject);
        }
    }
}
```

#### 场景3：控制拖拽灵敏度

```csharp
// 运行时动态修改拖拽阈值
EventSystem.current.pixelDragThreshold = 10;  // 默认 5，调大更"迟钝"

// 应用场景：
// - 手机端：调大阈值防止手抖误拖拽
// - 平板/触控笔：调小阈值让拖拽更灵敏
```

#### 场景4：手动进行 UI 射线检测

```csharp
using UnityEngine.EventSystems;
using System.Collections.Generic;

public class ManualRaycast : MonoBehaviour
{
    void Update()
    {
        if (Input.GetMouseButtonDown(0))
        {
            // 创建 PointerEventData
            PointerEventData pointerData = new PointerEventData(EventSystem.current);
            pointerData.position = Input.mousePosition;

            // 存储射线检测结果
            List<RaycastResult> results = new List<RaycastResult>();

            // 使用 GraphicRaycaster 进行射线检测
            GraphicRaycaster raycaster = GetComponent<GraphicRaycaster>();
            raycaster.Raycast(pointerData, results);

            // 遍历所有被命中的 UI
            foreach (RaycastResult result in results)
            {
                Debug.Log($"命中 UI: {result.gameObject.name}");
            }
        }
    }
}
```

#### 场景5：监听选中变化

```csharp
// EventSystem 本身没有 OnSelectedChanged 事件
// 而是通过接口分发给被选中/取消选中的对象自身
// 如果要全局监听，可以定时轮询：

GameObject lastSelected;

void Update()
{
    GameObject current = EventSystem.current.currentSelectedGameObject;
    if (current != lastSelected)
    {
        Debug.Log($"选中变化: {(lastSelected ? lastSelected.name : "null")} → {(current ? current.name : "null")}");
        lastSelected = current;
    }
}
```

---

### 🚀 EventSystem 的自动创建

> 当场景中**没有任何 EventSystem** 时，首次创建 Canvas 或 UI 控件时 Unity 会**自动创建**一个名为 `EventSystem` 的 GameObject，上面挂载了：
> - `EventSystem` 组件
> - `StandaloneInputModule` 组件

```
EventSystem (GameObject)
├── EventSystem 组件          ← 事件管理
└── StandaloneInputModule     ← 输入处理（随平台自动切换）
```

> [!WARNING] 注意
> 如果手动删除了自动创建的 EventSystem，需要再次创建 UI 才能触发自动创建，或手动 `GameObject → UI → EventSystem` 创建。

---

## ⌨️ Standalone Input Module 组件

> StandaloneInputModule 是 EventSystem 的**输入来源**，负责捕获鼠标、键盘、触摸、手柄的原始输入，将其标准化为 EventData 后交给 EventSystem 分发。

---

### 📋 Inspector 参数详解

| 参数 | 类型 | 说明 |
|------|------|------|
| **Horizontal Axis** | string | 水平导航轴名称，默认 `"Horizontal"`（对应 InputManager 中的水平轴，即 A/D 或 方向键左右） |
| **Vertical Axis** | string | 垂直导航轴名称，默认 `"Vertical"`（对应 W/S 或 方向键上下） |
| **Submit Button** | string | ==提交按钮名称==，默认 `"Submit"`（回车/Space/手柄A键）。被选中控件收到 `ISubmitHandler.OnSubmit` |
| **Cancel Button** | string | ==取消按钮名称==，默认 `"Cancel"`（Esc/手柄B键）。收到 `ICancelHandler.OnCancel` |
| **Input Actions Per Second** | float | ==每秒最大输入次数==（用于导航/提交），默认 10。控制长按时事件触发的最大频率 |
| **Repeat Delay** | float | ==首次输入延迟（秒）==，默认 0.5。按下方向键后等待多久才开始重复触发 |
| **Force Module Active** | bool | 是否强制该模块保持激活状态。默认不勾选，EventSystem 会自动管理 |

---

### 🔄 输入处理流程

```
鼠标/触摸输入:
  鼠标移动 → 更新 PointerEventData.position
  鼠标按下 → PointerEventData.button + 调用 Press / 开始计时区分点击/拖拽
  鼠标释放 → 判断是 Click 还是 EndDrag

键盘导航:
  检测 Horizontal/Vertical 轴 → 超过阈值 → 在当前选中对象的可导航方向上查找下一个 → 触发 Select/Deselect

提交/取消:
  检测 Submit/Cancel 按钮 → 发送 OnSubmit/OnCancel 事件
```

---

### 📊 拖拽判定逻辑

```
鼠标按下 (eligibleForClick = true)
    │
    ▼
鼠标移动 < Drag Threshold? ─── 是 ──▶ 仍然是待点击状态
    │
    否（移动距离 ≥ Drag Threshold）
    │
    ▼
触发 IBeginDragHandler.OnBeginDrag  ← 进入拖拽模式
    │
    ▼
每帧触发 IDragHandler.OnDrag
    │
    ▼
鼠标释放 → IEndDragHandler.OnEndDrag（eligibleForClick = false，不再触发 Click）
```

---

### 🎮 Input Module 类型选择

| 平台 | Input Module | 说明 |
|------|-------------|------|
| **PC/Mac/Linux** | `StandaloneInputModule` | 同时支持鼠标+键盘+手柄 |
| **移动端 (iOS/Android)** | 自动使用 `StandaloneInputModule` | 自动检测触摸输入，无需额外配置 |
| **旧版 (已废弃)** | `TouchInputModule` | 仅触摸，不建议使用 |

---

### 🔧 自定义输入配置

```csharp
using UnityEngine.EventSystems;

public class CustomInputModule : StandaloneInputModule
{
    // 重写 Process 方法来自定义输入处理逻辑
    public override void Process()
    {
        // 保留原有的输入处理
        base.Process();

        // 添加自定义输入逻辑
        // 例如：支持长按、手势识别等
    }
}
```

> 继承 `StandaloneInputModule` 或 `BaseInputModule` 即可实现自定义输入模块，适用于==手势识别、语音控制、眼动追踪==等特殊输入设备。

---

### ⚠️ EventSystem 常见问题排查

| 问题 | 可能原因 | 解决方法 |
|------|----------|----------|
| **点击 UI 无反应** | ① 没有 EventSystem ② Canvas 上无 GraphicRaycaster ③ UI 对象 Raycast Target 未勾选 | 依次检查三个条件 |
| **按钮点击穿透到场景** | 没有判断 `IsPointerOverGameObject` | 在场景点击逻辑前加 UI 判断 |
| **键盘导航不生效** | ① `Send Navigation Events` 未勾选 ② Button 的 Navigation 未正确设置 | 检查 EventSystem 和各控件的 Navigation 设置 |
| **拖拽过于灵敏/迟钝** | Drag Threshold 设置不当 | 调整像素阈值（手机端建议 8~15） |
| **多个 EventSystem 冲突** | 场景中存在多个激活的 EventSystem | 删除多余的，==每个场景只保留一个== |
| **事件被父对象拦截** | 父对象实现了事件接口并消耗了事件 | 检查事件调用链，或使用 `Use()` 控制 |

---

## RectTransform组件

RectTransform 是 UGUI 中每个 UI 元素都必带的组件，继承自 Transform，专门用于 2D 平面上的 UI 布局。它是理解 UGUI 布局系统的核心，区别于普通 Transform 的位置/旋转/缩放，RectTransform 引入了**锚点、轴心、偏移**等概念来适应不同屏幕分辨率。

> [!IMPORTANT] 核心理解
> RectTransform 通过**锚点（Anchors）**将 UI 元素绑定到父容器的相对位置，使得 UI 能够在不同分辨率下保持正确的布局。

### 位置与尺寸参数

**Pos X / Pos Y / Pos Z**

UI 元素的 ==轴心（Pivot）== 相对于 ==锚点（Anchors）== 的偏移量（单位：像素）。这是实际在 Inspector 中最常调整的位置参数，与普通 Transform 的 `position` 不同，它表示的是相对于锚点的本地坐标。

**Width / Height**

当前 UI 元素的宽高（单位：像素）。当锚点分离时（如 Stretch 模式），这两个字段会被替换为 **Left / Right / Top / Bottom** 偏移量。

### 锚点（Anchors） ⭐

锚点是 RectTransform 最核心的概念。锚点用**相对于父 RectTransform 的归一化坐标**（0~1 范围）来表示：

> ==锚点的值表示该点位于父容器的百分比位置。== 例如：`Min(0, 0)` 表示父容器左下角，`Max(1, 1)` 表示父容器右上角。

| 参数 | 说明 |
|------|------|
| **Anchor Min** | 锚框的左下角坐标 (X, Y)，相对于父容器的归一化值 |
| **Anchor Max** | 锚框的右上角坐标 (X, Y)，相对于父容器的归一化值 |

**锚点集中时（Min = Max）**：UI 元素的大小固定，位置会随父容器等比缩放而保持相对位置。

**锚点分离时（Min ≠ Max，即 Stretch 模式）**：UI 元素会随父容器伸缩，Width/Height 会被替换为：

| 参数 | 说明 |
|------|------|
| **Left** | 元素左边缘到锚框左边缘的距离 |
| **Right** | 元素右边缘到锚框右边缘的距离 |
| **Top** | 元素上边缘到锚框上边缘的距离 |
| **Bottom** | 元素下边缘到锚框下边缘的距离 |

> [!TIP] 使用技巧
> 点击 Inspector 中 RectTransform 左上角的**锚点预设面板**（小方块图标），可以快速设置常见的锚点布局（居中、拉伸、靠边等），按住 `Alt` 键点击会同时设置位置，按住 `Shift` 键会同时设置轴心。

### 轴心（Pivot）

> 轴心是 UI 元素自身旋转、缩放和定位的**参考中心点**。

| 说明 | 详情 |
|------|------|
| **取值范围** | (0, 0) ~ (1, 1)，(0.5, 0.5) 为中心 |
| **影响** | 旋转的圆心、缩放的中心、Pos 的参考点 |
| **可视化** | 在 Scene 视图中显示为蓝色小圆圈 |

### Size Delta

> Size Delta = 元素的**实际尺寸** - 锚框的**定义尺寸**。

| 场景 | Size Delta 含义 |
|------|------|
| **锚点集中时** | 直接等于元素的 Width × Height |
| **锚点分散时** | 表示元素相对于锚框的偏移量 |

当锚点分离时，若 Left/Right/Top/Bottom 都为 0，则 Size Delta 为 (0, 0)，元素完全贴紧锚框。

### 锚点位置（Anchored Position）

> 锚点位置 = 元素的 Pivot 相对于 Anchor 参考点的位置（以 Anchor 为原点）。

在 Inspector 中看到的 Pos X / Pos Y 就是 Anchored Position，它是 UI 布局的核心位置参数。

### Blueprint Mode（蓝图模式）和 Raw Edit Mode（原始编辑模式）

| 模式 | 说明 |
|------|------|
| **Blueprint Mode** | 编辑矩形时忽略旋转和缩放，以原始未旋转的形态显示。方便在旋转后的 UI 上进行调整 |
| **Raw Edit Mode** | 直接编辑 Pivot 和 Anchor 的数值而不自动调整位置。适用于需要精确修改锚点/轴心而不想影响当前位置的场景 |

> 这两个模式在 Inspector 中 RectTransform 组件的右上角通过小按钮切换。

### 属性计算公式总结

```
Anchored Position = 元素 Pivot 位置 - 锚点位置
Size Delta = 元素实际大小 - (Anchor Max - Anchor Min) × 父元素大小
实际位置 = 父元素左下角 + Anchor相对位置 × 父元素大小 + Anchored Position - Pivot × 元素大小
```

### 常用锚点预设

| 预设 | Anchor Min | Anchor Max | 效果 |
|------|:--:|:--:|------|
| 左上角 | (0, 1) | (0, 1) | 固定在左上角，不随屏幕缩放移动 |
| 居中 | (0.5, 0.5) | (0.5, 0.5) | 固定在屏幕中央 |
| 水平拉伸 | (0, 0.5) | (1, 0.5) | 宽度随父容器拉伸，高度固定 |
| 垂直拉伸 | (0.5, 0) | (0.5, 1) | 高度随父容器拉伸，宽度固定 |
| 全屏拉伸 | (0, 0) | (1, 1) | 宽高均随父容器拉伸（背景图常用） |

---

## 🖼️ Image 组件

> Image 是 UGUI 中**最常用的UI显示组件**，用于在 UI 中显示图片/精灵（Sprite）。继承自 `MaskableGraphic` → `Graphic`，同时也是 `Mask`（遮罩）和 `RawImage` 的兄弟组件。

### 📋 Inspector 参数详解

#### Source Image（源图片）

| 说明 | 详情 |
|------|------|
| **类型** | `Sprite` |
| **作用** | 要显示的精灵图片资源 |
| **代码** | `image.sprite = Resources.Load<Sprite>("xxx");` |

> 只能使用 **Sprite（2D and UI）** 类型的纹理，普通 Texture 无法直接赋值。

#### Color（颜色）

| 说明 | 详情 |
|------|------|
| **类型** | `Color` |
| **作用** | 图片的叠加颜色。默认白色（255, 255, 255, 255）表示原图显示，修改颜色会与图片混合 |
| **用途** | 实现变色/半透明效果而不用换图 |
| **代码** | `image.color = new Color(1, 0, 0, 0.5f);` // 红色半透明 |

#### Material（材质）

| 说明 | 详情 |
|------|------|
| **类型** | `Material` |
| **作用** | 自定义材质，用于实现特殊效果（灰度、模糊、溶解等 Shader 特效） |
| **默认** | 为 null 时使用 UGUI 内置的默认材质 |

#### Raycast Target（射线检测目标）

| 说明 | 详情 |
|------|------|
| **类型** | `bool` |
| **作用** | 是否接收射线检测（点击事件） |
| **代码** | `image.raycastTarget = false;` |

> [!TIP] 性能优化
> 大量非交互的装饰性 Image（如背景、边框、纯装饰图标），==务必关闭 Raycast Target==，否则 GraphicRaycaster 会逐一检测它们，浪费性能。

#### Maskable（可遮罩）

| 说明 | 详情 |
|------|------|
| **类型** | `bool` |
| **作用** | 是否受父级 Mask 组件的裁剪影响 |
| **代码** | `image.maskable = false;` |

#### Image Type（图片类型）⭐

Image 支持四种显示模式：

| 模式 | 原理 | 适用场景 |
|------|------|----------|
| **Simple** | 整张图片直接拉伸填充 | 普通图标、整体图片 |
| **Sliced** | 九宫格切片，中心拉伸，边缘保持不变 | 圆角边框、对话框背景（保持圆角不拉伸） |
| **Tiled** | 图片像瓷砖一样平铺重复 | 重复纹理背景（地板、网格） |
| **Filled** | 按比例部分填充显示 | 血条、技能冷却、进度条、加载条 |

##### Simple（简单模式）

> 最简单的模式，将整张图片等比或拉伸填充到 Image 区域。

| 子参数 | 说明 |
|------|------|
| **Preserve Aspect** | ==是否保持原始宽高比==。勾选后图片不会变形，未覆盖区域透明 |

##### Sliced（切片/九宫格模式）

> 将图片分为 **9 个区域**（四角 + 四边 + 中心），四角保持不变，四边单向拉伸，中心双向拉伸，实现==边框不变形==的效果。

| 子参数 | 说明 |
|------|------|
| **Pixels Per Unit Multiplier** | 像素单位乘数，控制切片边框的粗细 |

> [!IMPORTANT] 使用前提
> 必须先在图片的 **Sprite Editor** 中设置好**切片边框（Border）**，否则 Sliced 模式效果和 Simple 没有区别。

##### Tiled（平铺模式）

> 图片从**中心开始**像瓷砖一样向外重复平铺，直到填满整个 Image 区域。

| 子参数 | 说明 |
|------|------|
| **Pixels Per Unit Multiplier** | 像素单位乘数，控制每张平铺图的大小。值越大 → 每块越小 → 平铺越密集 |

##### Filled（填充模式）⭐

> 最常用的**进度条/血条**模式！按指定比例和方向部分填充图片。

| 子参数 | 说明 |
|------|------|
| **Fill Method** | 填充方式：`Radial 360`（环形360°）、`Radial 180`（环形180°）、`Radial 90`（环形90°）、`Horizontal`（水平）、`Vertical`（垂直） |
| **Fill Origin** | 填充起点方向（如：Left、Right、Bottom、Top） |
| **Fill Amount** | 填充比例，范围 0~1。**0 = 完全透明，1 = 完全填充** |
| **Clockwise** | 环形模式下是否顺时针填充 |

| Fill Method | 适用场景 |
|:--|------|
| **Horizontal** | 水平血条、冷却进度 |
| **Vertical** | 垂直能量条、音量条 |
| **Radial 360** | 技能冷却圆圈（最常用） |
| **Radial 180** | 半圆仪表盘 |
| **Radial 90** | 扇形填充 |

---

### 💻 代码控制方式

#### 基本属性控制

```csharp
using UnityEngine;
using UnityEngine.UI;

public class ImageController : MonoBehaviour
{
    public Image image;

    void Start()
    {
        // 1. 设置/切换图片
        image.sprite = Resources.Load<Sprite>("Sprites/MyIcon");
        // 或通过图集加载
        // image.sprite = atlas.GetSprite("icon_name");

        // 2. 设置颜色
        image.color = Color.white;           // 恢复原色
        image.color = Color.red;             // 红色叠加
        image.color = new Color(1, 1, 1, 0.5f); // 半透明

        // 3. 设置图片类型
        image.type = Image.Type.Simple;   // 简单模式
        image.type = Image.Type.Sliced;   // 九宫格模式
        image.type = Image.Type.Tiled;    // 平铺模式
        image.type = Image.Type.Filled;   // 填充模式

        // 4. 保持宽高比（仅 Simple 模式）
        image.preserveAspect = true;

        // 5. 射线检测开关
        image.raycastTarget = false;  // 关闭点击响应，优化性能

        // 6. 遮罩裁剪
        image.maskable = false;  // 不受父级 Mask 影响

        // 7. 图片是否启用（控制显示/隐藏）
        image.enabled = true;
    }
}
```

#### 填充模式（Filled）控制

```csharp
using UnityEngine;
using UnityEngine.UI;

public class FillBarController : MonoBehaviour
{
    public Image fillImage;  // 已设为 Filled 模式的 Image

    void Start()
    {
        // 设置填充方式
        fillImage.fillMethod = Image.FillMethod.Horizontal;  // 水平填充
        fillImage.fillMethod = Image.FillMethod.Vertical;    // 垂直填充
        fillImage.fillMethod = Image.FillMethod.Radial360;   // 环形360°填充
        fillImage.fillMethod = Image.FillMethod.Radial180;   // 环形180°填充
        fillImage.fillMethod = Image.FillMethod.Radial90;    // 环形90°填充

        // 设置填充起点
        fillImage.fillOrigin = (int)Image.OriginHorizontal.Left;  // 从左往右填
        fillImage.fillOrigin = (int)Image.OriginVertical.Bottom;   // 从下往上填

        // 设置填充量（核心属性）
        fillImage.fillAmount = 0f;    // 完全为空
        fillImage.fillAmount = 0.5f;  // 填充一半
        fillImage.fillAmount = 1f;    // 完全填满

        // 是否顺时针填充（仅环形模式有效）
        fillImage.fillClockwise = true;
    }

    // 实战：血条更新方法
    public void UpdateHealthBar(float currentHealth, float maxHealth)
    {
        fillImage.fillAmount = currentHealth / maxHealth;
        // 血量低于 30% 变红色
        fillImage.color = fillImage.fillAmount > 0.3f ? Color.green : Color.red;
    }

    // 实战：技能冷却动画（平滑过渡）
    IEnumerator CooldownRoutine(float cooldownTime)
    {
        float timer = 0f;
        while (timer < cooldownTime)
        {
            timer += Time.deltaTime;
            fillImage.fillAmount = timer / cooldownTime;
            yield return null;
        }
        fillImage.fillAmount = 0f; // 冷却完毕
    }
}
```

#### 动态创建与设置

```csharp
using UnityEngine;
using UnityEngine.UI;

public class DynamicImageCreator : MonoBehaviour
{
    public Transform parent;

    void Start()
    {
        // 方法1：通过 new GameObject 创建
        GameObject go = new GameObject("DynamicImage");
        go.transform.SetParent(parent, false);
        Image img = go.AddComponent<Image>();
        img.sprite = Resources.Load<Sprite>("Sprites/Icon");
        img.color = Color.white;
        img.raycastTarget = false;

        // 方法2：通过 Instantiate 克隆预制体
        // GameObject prefab = Resources.Load<GameObject>("Prefabs/ImageItem");
        // GameObject clone = Instantiate(prefab, parent);
        // Image img = clone.GetComponent<Image>();
    }
}
```

| 属性 | 代码 | 说明 |
|------|------|------|
| Sprite | `image.sprite = xxx` | 切换显示的图片 |
| Color | `image.color = new Color(r,g,b,a)` | 修改颜色和透明度 |
| Type | `image.type = Image.Type.XXX` | 切换 Simple/Sliced/Tiled/Filled |
| Fill Amount | `image.fillAmount = 0~1` | 填充进度（仅 Filled 模式） |
| Fill Method | `image.fillMethod = Image.FillMethod.XXX` | 填充方向方式 |
| Preserve Aspect | `image.preserveAspect = true/false` | 是否保持宽高比 |
| Raycast Target | `image.raycastTarget = true/false` | 是否响应点击检测 |
| Material | `image.material = xxx` | 设置自定义材质 |
| Enabled | `image.enabled = true/false` | 显示/隐藏 |
| SetNativeSize | `image.SetNativeSize()` | 将 Image 尺寸还原为图片原始尺寸 |

> [!NOTE] Image 的属性命名空间
> `Image` 组件位于 `UnityEngine.UI` 命名空间，使用前需要 `using UnityEngine.UI;`。不要和 `UnityEngine.UIElements.Image` 混淆。

---

## 📝 Text 组件

> Text 是 UGUI 中**显示文本内容**的核心组件，继承自 `MaskableGraphic` → `Graphic`，用于在 UI 中渲染文字。支持富文本标记、字体设置、对齐方式、换行等丰富功能。

> [!NOTE] Text vs TextMeshPro
> 传统 `Text` 组件简单易用但在放大时会模糊（基于位图字体）。Unity 官方推荐使用 **TextMeshPro (TMP)** 作为文本渲染的替代方案（基于 SDF 字体，任意缩放保持清晰）。但 `Text` 依然是快速原型和小型项目的首选。

---

### 📋 Inspector 参数详解

#### Text（文本内容）

| 说明 | 详情 |
|------|------|
| **类型** | `string` |
| **作用** | 显示的文本字符串 |
| **代码** | `textComponent.text = "Hello World";` |

> 支持**富文本（Rich Text）**标记，如 `<b>加粗</b>`、`<i>斜体</i>`、`<color=red>红色</color>`、`<size=30>大小</size>` 等。

#### Character（字符属性）

| 参数 | 说明 |
|------|------|
| **Font** | 字体资源（TrueType `.ttf` 或 OpenType `.otf`） |
| **Font Style** | 字体样式：Normal（正常）、Bold（粗体）、Italic（斜体）、**Bold And Italic**（粗斜体） |
| **Font Size** | 字体大小（像素），默认 14 |
| **Line Spacing** | 行间距倍数。1 = 正常，>1 加宽，<1 缩窄 |
| **Rich Text** | 是否启用富文本标记。关闭后 `<b>` 等标签将作为普通文字显示 |

#### Paragraph（段落属性）

| 参数 | 说明 |
|------|------|
| **Alignment** | 对齐方式：水平（左/中/右）× 垂直（上/中/下）共 9 种组合 |
| **Align By Geometry** | 是否按字形几何对齐（而非字体的基线/升部/降部） |
| **Horizontal Overflow** | 水平溢出处理：**Wrap**（自动换行）/ **Overflow**（超出显示区域继续渲染） |
| **Vertical Overflow** | 垂直溢出处理：**Truncate**（截断超出内容）/ **Overflow**（超出显示区域继续渲染） |
| **Best Fit** | ==自适应字号== —— 文本无法完整显示时自动缩小字号以适应区域。可设置 Min Size 和 Max Size |

> [!TIP] Best Fit 使用建议
> 勾选 Best Fit 后设置合理的 **Min Size** 和 **Max Size** 范围（如 10~30），文本过长时会自动缩小以适应显示区域。适合多语言适配，但注意字号过小会影响可读性。

#### Color & Material

| 参数 | 说明 |
|------|------|
| **Color** | 文字颜色 |
| **Material** | 自定义材质（如描边、阴影等特效 Shader） |
| **Raycast Target** | 是否接收射线检测（与 Image 同理，非交互文字建议关闭） |

---

### 💻 代码控制方式

#### 基本属性控制

```csharp
using UnityEngine;
using UnityEngine.UI;

public class TextController : MonoBehaviour
{
    public Text text;

    void Start()
    {
        // 1. 设置文本内容
        text.text = "你好，Unity！";
        // 多行文本
        text.text = "第一行\n第二行\n第三行";

        // 2. 富文本标记（需勾选 Rich Text）
        text.text = "<b>粗体</b> 和 <i>斜体</i> <color=red>红色文字</color>";
        text.text = "<size=30>大号</size> 和 <size=12>小号</size>";
        // 颜色支持十六进制
        text.text = "颜色十六进制：<color=#FF8800>橙色</color>";

        // 3. 设置字体
        text.font = Resources.Load<Font>("Fonts/MyFont");

        // 4. 字体样式
        text.fontStyle = FontStyle.Normal;      // 正常
        text.fontStyle = FontStyle.Bold;         // 粗体
        text.fontStyle = FontStyle.Italic;       // 斜体
        text.fontStyle = FontStyle.BoldAndItalic; // 粗斜体

        // 5. 字体大小
        text.fontSize = 24;

        // 6. 行间距
        text.lineSpacing = 1.5f; // 1.5倍行间距

        // 7. 对齐方式
        text.alignment = TextAnchor.MiddleCenter;     // 居中
        text.alignment = TextAnchor.UpperLeft;        // 左上角
        text.alignment = TextAnchor.LowerRight;       // 右下角

        // 8. 颜色
        text.color = Color.white;
        text.color = new Color(1, 1, 1, 0.5f); // 白色半透明

        // 9. 溢出处理
        text.horizontalOverflow = HorizontalWrapMode.Wrap;    // 自动换行
        text.horizontalOverflow = HorizontalWrapMode.Overflow; // 超出不换行
        text.verticalOverflow = VerticalWrapMode.Truncate;     // 超出截断
        text.verticalOverflow = VerticalWrapMode.Overflow;     // 超出继续显示

        // 10. Best Fit（自适应字号）
        text.resizeTextForBestFit = true;
        text.resizeTextMinSize = 10;  // 最小字号
        text.resizeTextMaxSize = 30;  // 最大字号

        // 11. 是否支持富文本
        text.supportRichText = false; // 禁用富文本标记

        // 12. 按几何对齐
        text.alignByGeometry = false;

        // 13. 射线检测
        text.raycastTarget = false; // 非交互文本建议关闭
    }
}
```

#### 动态更新与格式化

```csharp
using UnityEngine;
using UnityEngine.UI;

public class TextDynamicController : MonoBehaviour
{
    public Text scoreText;
    public Text timerText;
    public Text hpText;
    public Text messageText;

    private int score = 0;
    private float timer = 0f;

    void Start()
    {
        // 格式化字符串 ✅
        scoreText.text = string.Format("分数：{0}", score);
        scoreText.text = $"分数：{score}"; // C# 6.0+ 字符串插值
        scoreText.text = "分数：" + score;  // 字符串拼接（会产生GC）

        // 数值格式化
        timerText.text = timer.ToString("F2");         // 保留两位小数 "0.00"
        timerText.text = string.Format("{0:F1}", timer); // 保留一位小数
        
        // 百分比显示
        float hpPercent = 0.75f;
        hpText.text = string.Format("{0:P0}", hpPercent); // "75%"

        // 富文本动态变色（血量颜色）
        int currentHP = 60;
        int maxHP = 100;
        float ratio = (float)currentHP / maxHP;
        string colorTag = ratio > 0.5f ? "green" : (ratio > 0.2f ? "yellow" : "red");
        hpText.text = string.Format("HP: <color={0}>{1}/{2}</color>", colorTag, currentHP, maxHP);

        // 添加千位分隔符
        int gold = 1234567;
        messageText.text = string.Format("金币：{0:N0}", gold); // "金币：1,234,567"
    }

    void Update()
    {
        // 实时更新计时器
        timer += Time.deltaTime;
        int minutes = (int)timer / 60;
        int seconds = (int)timer % 60;
        timerText.text = string.Format("{0:00}:{1:00}", minutes, seconds); // "02:35"
    }

    // 得分增加（避免字符串拼接产生GC）
    public void AddScore(int value)
    {
        score += value;
        scoreText.text = $"分数：{score}";
    }
}
```

#### 动态创建与常用技巧

```csharp
using UnityEngine;
using UnityEngine.UI;

public class DynamicTextCreator : MonoBehaviour
{
    public Transform parent;

    void Start()
    {
        // 方法1：通过 new GameObject 动态创建
        GameObject go = new GameObject("DynamicText");
        go.transform.SetParent(parent, false);
        Text txt = go.AddComponent<Text>();
        txt.text = "动态创建的文本";
        txt.font = Resources.GetBuiltinResource<Font>("LegacyRuntime.ttf");
        txt.fontSize = 24;
        txt.color = Color.white;
        txt.alignment = TextAnchor.MiddleCenter;
        txt.raycastTarget = false;

        // 必须设置 RectTransform 的尺寸，否则文本可能无法显示
        RectTransform rt = txt.GetComponent<RectTransform>();
        rt.sizeDelta = new Vector2(200, 50);
    }

    // 获取文本宽度（用于自适应布局）
    public float GetTextWidth(Text text)
    {
        TextGenerator generator = text.cachedTextGenerator;
        float width = generator.GetPreferredWidth(
            text.text,
            text.GetGenerationSettings(text.rectTransform.rect.size)
        );
        return width / text.pixelsPerUnit;
    }

    // 让 Image 背景自适应文本宽度
    public void AutoSizeBackground(Text text, RectTransform background)
    {
        float textWidth = text.preferredWidth;   // 文本理想宽度
        float textHeight = text.preferredHeight; // 文本理想高度
        background.sizeDelta = new Vector2(textWidth + 20, textHeight + 10); // 留边距
    }
}
```

---

### 📊 属性速查表

| 属性 | 代码 | 说明 |
|------|------|------|
| Text | `text.text = "xxx"` | 设置文字内容 |
| Font | `text.font = xxx` | 设置字体 |
| Font Style | `text.fontStyle = FontStyle.Bold` | 字体样式 |
| Font Size | `text.fontSize = 24` | 字体大小 |
| Line Spacing | `text.lineSpacing = 1.5f` | 行间距 |
| Alignment | `text.alignment = TextAnchor.MiddleCenter` | 对齐方式（9种） |
| Color | `text.color = Color.red` | 文字颜色 |
| Rich Text | `text.supportRichText = true/false` | 是否启用富文本 |
| Horizontal Overflow | `text.horizontalOverflow = HorizontalWrapMode.Wrap` | 水平溢出模式 |
| Vertical Overflow | `text.verticalOverflow = VerticalWrapMode.Truncate` | 垂直溢出模式 |
| Best Fit | `text.resizeTextForBestFit = true` | 自适应字号开关 |
| Min Size | `text.resizeTextMinSize = 10` | 最小字号 |
| Max Size | `text.resizeTextMaxSize = 40` | 最大字号 |
| Align By Geometry | `text.alignByGeometry = false` | 几何对齐 |
| Raycast Target | `text.raycastTarget = false` | 射线检测开关 |
| Preferred Width | `text.preferredWidth` | 文本理想宽度（只读） |
| Preferred Height | `text.preferredHeight` | 文本理想高度（只读） |
| Cached Text Generator | `text.cachedTextGenerator` | 文本生成器（用于高级测量） |

---

### 🏷️ 富文本标记速查

| 标记 | 用法 | 效果 |
|------|------|------|
| **粗体** | `<b>文字</b>` | **粗体文字** |
| *斜体* | `<i>文字</i>` | *斜体文字* |
| 字号 | `<size=30>文字</size>` | 指定大小文字 |
| 颜色 | `<color=red>文字</color>` | 红色文字 |
| 颜色(hex) | `<color=#FF8800>文字</color>` | 橙色文字 |
| 材质 | `<material=xxx>文字</material>` | 使用指定材质的文字 |

> [!WARNING] 富文本注意事项
> - 必须勾选 `Rich Text` 选项才能生效
> - 颜色名称支持 HTML 标准颜色名（red、green、blue、yellow、cyan 等）
> - 十六进制颜色需要 `#` 前缀，支持 `#RGB`、`#RRGGBB`、`#RRGGBBAA` 格式
> - 嵌套使用时注意闭合顺序：`<b><color=red>文字</color></b>`


---

## 🎞️ RawImage 组件

> RawImage 是 UGUI 中显示**任意 Texture（纹理）**的组件，不限于 Sprite。与 Image 不同，它可以直接渲染 `Texture`、`Texture2D`、`RenderTexture` 等，继承自 `MaskableGraphic` → `Graphic`。

> [!IMPORTANT] RawImage vs Image 核心区别
> | 对比 | Image | RawImage |
> |------|-------|----------|
> | **可显示的纹理** | 仅 `Sprite`（2D and UI 类型） | 任意 `Texture`（Texture2D、RenderTexture 等） |
> | **UV 控制** | ❌ 无 | ✅ 通过 `uvRect` 精确控制显示区域 |
> | **九宫格/填充** | ✅ Sliced / Tiled / Filled | ❌ 不支持 |
> | **图集合批** | ✅ 同一个图集的 Image 可合批 | ❌ 每个 RawImage 额外 1 个 DrawCall |
> | **典型用途** | UI 图标、按钮、血条 | 摄像机画面、视频播放、网络图片、小地图 |

---

### 📋 Inspector 参数详解

#### Texture（纹理）

| 说明 | 详情 |
|------|------|
| **类型** | `Texture`（可接受 Texture2D、RenderTexture 等） |
| **作用** | 要显示的纹理资源 |
| **代码** | `rawImage.texture = myTexture;` |

> ==可以接受任何 Texture 类型==，这是 RawImage 最核心的优势。常用于显示摄像机渲染结果、动态加载的图片等。

#### Color（颜色）

| 说明 | 详情 |
|------|------|
| **类型** | `Color` |
| **作用** | 纹理的叠加颜色，与 Image 的 Color 完全一致 |
| **代码** | `rawImage.color = Color.white;` |

#### Material（材质）

| 说明 | 详情 |
|------|------|
| **类型** | `Material` |
| **作用** | 自定义材质，用于变换纹理显示效果（灰度、模糊等 Shader 特效） |

#### Raycast Target（射线检测目标）

与 Image 同理，非交互的 RawImage 建议关闭以优化性能。

#### UV Rect（UV 矩形）⭐

> RawImage ==独有的核心特性==，控制纹理的**哪一部分**被显示以及如何**缩放/裁剪**。

| 参数 | 说明 | 取值范围 |
|------|------|:--:|
| **X** | 采样起点的 U 坐标（水平起点） | 0 ~ 1 |
| **Y** | 采样起点的 V 坐标（垂直起点） | 0 ~ 1 |
| **W** | 采样的 UV 宽度 | 0 ~ 1 |
| **H** | 采样的 UV 高度 | 0 ~ 1 |

| UV Rect 效果 | 设置值 | 场景 |
|:--|:--|------|
| 显示整张图 | `(0, 0, 1, 1)` | 默认 |
| 显示左上角 1/4 | `(0, 0, 0.5, 0.5)` | 截取部分纹理 |
| 水平镜像 | `(1, 0, -1, 1)` | 翻转效果 |
| UV 滚动（流动边框） | 代码动态改变 X/Y | 流体/滚动背景 |
| 纹理平铺放大 | `(0, 0, 2, 2)` | 2×2 平铺 |

> [!TIP] UV Rect 实战思路
> - **小地图**：截取 RenderTexture 的部分区域来显示局部地图
> - **技能冷却流光**：代码中每帧改变 UV 的 X/Y 偏移量，实现纹理滚动
> - **镜面翻转**：W 或 H 设为负值实现镜像效果

---

### 💻 代码控制方式

#### 基本属性控制

```csharp
using UnityEngine;
using UnityEngine.UI;

public class RawImageController : MonoBehaviour
{
    public RawImage rawImage;

    void Start()
    {
        // 1. 设置纹理（Texture2D）
        Texture2D tex = Resources.Load<Texture2D>("Textures/MyImage");
        rawImage.texture = tex;

        // 2. 设置颜色
        rawImage.color = Color.white;
        rawImage.color = new Color(1, 1, 1, 0.5f); // 半透明

        // 3. 设置 UV Rect（核心）
        rawImage.uvRect = new Rect(0, 0, 1, 1);       // 默认：显示整张图
        rawImage.uvRect = new Rect(0, 0, 0.5f, 0.5f); // 显示左下角 1/4
        rawImage.uvRect = new Rect(0.25f, 0.25f, 0.5f, 0.5f); // 显示中间部分
        rawImage.uvRect = new Rect(1, 0, -1, 1);      // 水平镜像翻转

        // 4. 射线检测
        rawImage.raycastTarget = false;

        // 5. 材质
        rawImage.material = Resources.Load<Material>("Materials/GrayScale");
    }
}
```

#### RenderTexture 实战 ⭐

```csharp
using UnityEngine;
using UnityEngine.UI;

public class RenderTextureRawImage : MonoBehaviour
{
    public RawImage rawImage;     // 显示渲染结果的 RawImage
    public Camera renderCamera;   // 用于渲染的独立摄像机
    private RenderTexture rt;

    void Start()
    {
        // 创建 RenderTexture
        rt = new RenderTexture(512, 512, 16);
        rt.name = "RawImageRT";

        // 设置给摄像机
        renderCamera.targetTexture = rt;

        // 设置给 RawImage
        rawImage.texture = rt;
    }

    void OnDestroy()
    {
        // 释放 RenderTexture 资源
        if (rt != null)
        {
            renderCamera.targetTexture = null;
            rt.Release();
            Destroy(rt);
        }
    }
}
```

> 常见用途：==3D 模型预览（角色展示界面）==、小地图、摄像机监控画面、后视镜。

#### UV 滚动动画（流动边框 / 背景滚动）

```csharp
using UnityEngine;
using UnityEngine.UI;

public class UVScrollController : MonoBehaviour
{
    public RawImage rawImage;
    public Vector2 scrollSpeed = new Vector2(0.1f, 0.2f);

    void Update()
    {
        // 方法1：直接修改 uvRect
        Rect uv = rawImage.uvRect;
        uv.x += scrollSpeed.x * Time.deltaTime;
        uv.y += scrollSpeed.y * Time.deltaTime;
        rawImage.uvRect = uv;

        // 方法2：通过 Material 的纹理偏移（需要 Shader 支持）
        // rawImage.material.mainTextureOffset += scrollSpeed * Time.deltaTime;
    }
}
```

#### 动态加载图片（Texture2D）

```csharp
using UnityEngine;
using UnityEngine.UI;
using UnityEngine.Networking;
using System.Collections;

public class WebImageLoader : MonoBehaviour
{
    public RawImage rawImage;

    // 从 Resources 加载
    public void LoadFromResources(string path)
    {
        Texture2D tex = Resources.Load<Texture2D>(path);
        if (tex != null)
        {
            rawImage.texture = tex;
            // 让 RawImage 自适应纹理尺寸
            rawImage.SetNativeSize();
        }
    }

    // 从网络加载图片
    IEnumerator LoadFromWeb(string url)
    {
        using (UnityWebRequest www = UnityWebRequestTexture.GetTexture(url))
        {
            yield return www.SendWebRequest();

            if (www.result == UnityWebRequest.Result.Success)
            {
                Texture2D tex = DownloadHandlerTexture.GetContent(www);
                rawImage.texture = tex;
                rawImage.SetNativeSize();
            }
        }
    }

    // 从本地文件加载
    IEnumerator LoadFromFile(string filePath)
    {
        string url = "file://" + filePath;
        using (UnityWebRequest www = UnityWebRequestTexture.GetTexture(url))
        {
            yield return www.SendWebRequest();

            if (www.result == UnityWebRequest.Result.Success)
            {
                rawImage.texture = DownloadHandlerTexture.GetContent(www);
                rawImage.SetNativeSize();
            }
        }
    }
}
```

#### 动态创建与常用技巧

```csharp
using UnityEngine;
using UnityEngine.UI;

public class DynamicRawImageCreator : MonoBehaviour
{
    public Transform parent;

    void Start()
    {
        // 动态创建 RawImage
        GameObject go = new GameObject("DynamicRawImage");
        go.transform.SetParent(parent, false);
        RawImage ri = go.AddComponent<RawImage>();
        ri.texture = Resources.Load<Texture2D>("Textures/BG");
        ri.color = Color.white;
        ri.raycastTarget = false;

        // 设置 RectTransform
        RectTransform rt = ri.GetComponent<RectTransform>();
        rt.sizeDelta = new Vector2(256, 256);

        // 调用 SetNativeSize 让尺寸匹配纹理原始大小
        ri.SetNativeSize();
    }

    // 截取 RawImage 指定区域的像素颜色
    public Color GetPixelColor(RawImage rawImage, Vector2 localPoint)
    {
        Texture2D tex = rawImage.texture as Texture2D;
        if (tex == null) return Color.clear;

        RectTransform rt = rawImage.rectTransform;
        Rect uv = rawImage.uvRect;

        // 将本地坐标映射到 UV 坐标
        float u = (localPoint.x / rt.rect.width) * uv.width + uv.x;
        float v = (localPoint.y / rt.rect.height) * uv.height + uv.y;

        int px = Mathf.FloorToInt(u * tex.width);
        int py = Mathf.FloorToInt(v * tex.height);

        return tex.GetPixel(px, py);
    }
}
```

---

### 📊 属性速查表

| 属性 | 代码 | 说明 |
|------|------|------|
| Texture | `rawImage.texture = xxx` | 设置纹理（Texture2D / RenderTexture） |
| Texture (Texture2D) | `rawImage.texture = tex2D` | 设置 2D 纹理 |
| Color | `rawImage.color = new Color(r,g,b,a)` | 修改颜色和透明度 |
| UV Rect | `rawImage.uvRect = new Rect(x,y,w,h)` | UV 采样区域（核心特性） |
| Material | `rawImage.material = xxx` | 设置自定义材质 |
| Raycast Target | `rawImage.raycastTarget = false` | 是否响应点击检测 |
| Enabled | `rawImage.enabled = true/false` | 显示/隐藏 |
| SetNativeSize | `rawImage.SetNativeSize()` | 将 RawImage 尺寸还原为纹理原始尺寸 |

> [!NOTE] 性能提醒
> 每个 RawImage 都会产生 ==1 个额外 DrawCall==（即使多个 RawImage 使用同一纹理也不会合批），大量使用时需注意性能。如果只是显示普通 UI 图标，优先使用 `Image` 组件以利用图集合批优势。

> [!NOTE] 命名空间
> `RawImage` 组件位于 `UnityEngine.UI` 命名空间，使用前需要 `using UnityEngine.UI;`。

---

## 🔘 Button 组件

> Button 是 UGUI 中**最常用的交互控件**，继承自 `Selectable` → `UIBehaviour`。它提供了完整的可交互状态系统（Normal / Highlighted / Pressed / Selected / Disabled）和**点击事件（onClick）**回调机制。

> [!IMPORTANT] Button 的继承链
> `Button` → `Selectable` → `UIBehaviour` → `MonoBehaviour`。Selectable 提供了按钮的核心交互能力：**状态切换、导航、过渡效果**。

---

### 📋 Inspector 参数详解

#### Interactable（可交互）

| 说明 | 详情 |
|------|------|
| **类型** | `bool` |
| **作用** | 按钮是否可以交互。设为 false 时按钮变灰并不可点击 |
| **代码** | `button.interactable = false;` |

#### Transition（过渡效果）

> 按钮在不同状态间的**视觉过渡方式**，有三种模式可选：

| 模式 | 原理 | 适用场景 |
|------|------|----------|
| **Color Tint** | 切换颜色（最多人用） | 普通按钮，简单高效 |
| **Sprite Swap** | 切换 Sprite 图片 | 需要不同外观的按钮状态 |
| **Animation** | 播放 Animator 动画 | 复杂动效（缩放、旋转、渐入等） |

##### Color Tint（颜色过渡）⭐

| 参数 | 说明 |
|------|------|
| **Target Graphic** | 受颜色变化影响的目标 Graphic（通常是按钮上的 Image 或 Text） |
| **Normal Color** | 正常状态颜色 |
| **Highlighted Color** | 鼠标悬停时颜色 |
| **Pressed Color** | 按下时颜色 |
| **Selected Color** | 选中状态颜色（需配合 Navigation） |
| **Disabled Color** | 禁用状态颜色（Interactable = false） |
| **Color Multiplier** | 颜色乘数，控制过渡强度（默认 1） |
| **Fade Duration** | 颜色过渡时间（秒），0 = 即时切换 |

> 代码获取/设置颜色块：
> ```csharp
> ColorBlock cb = button.colors;
> cb.normalColor = Color.white;
> cb.highlightedColor = Color.gray;
> cb.pressedColor = Color.green;
> button.colors = cb;
> ```

##### Sprite Swap（精灵切换）

| 参数 | 说明 |
|------|------|
| **Target Graphic** | 必须是 Image 组件（其他 Graphic 无法使用 Sprite Swap） |
| **Highlighted Sprite** | 悬停时显示的 Sprite |
| **Pressed Sprite** | 按下时显示的 Sprite |
| **Selected Sprite** | 选中时显示的 Sprite |
| **Disabled Sprite** | 禁用时显示的 Sprite |

##### Animation（动画过渡）

| 参数 | 说明 |
|------|------|
| **Target Graphic** | （自动关联） |
| **Normal Trigger** | 正常状态的触发器名 |
| **Highlighted Trigger** | 悬停时的触发器名 |
| **Pressed Trigger** | 按下时的触发器名 |
| **Selected Trigger** | 选中时的触发器名 |
| **Disabled Trigger** | 禁用时的触发器名 |

> [!TIP] 自动生成动画
> 选中 Animation 模式后点击 **"Auto Generate Animation"** 按钮，Unity 会自动创建 Animator Controller 和四个状态的动画片段，然后在 Animation 窗口中微调即可。

#### Navigation（导航）

> 控制按钮在**键盘/手柄导航**时的焦点切换方向。

| 参数 | 说明 |
|------|------|
| **None** | 不启用导航 |
| **Horizontal** | 仅水平方向导航 |
| **Vertical** | 仅垂直方向导航 |
| **Automatic** | 自动选择最近的可导航控件 |
| **Explicit** | 手动指定上下左右四个方向的导航目标 |

#### Visualize（可视化按钮）

> （可选）当鼠标悬停或选中时，按钮可以显示一个小箭头指示器。

#### OnClick（点击事件）⭐

> Button 最核心的功能 —— 绑定点击事件的回调列表。可以通过 Inspector 面板底部的 `+` `-` 号添加/删除监听。

| 参数 | 说明 |
|------|------|
| **Runtime Only** | 运行时对象引用 |
| **Object** | 挂载脚本的 GameObject |
| **Function** | 要调用的 public 方法（无参或接受单个简单类型参数） |

---

### 💻 代码控制方式

#### 基本属性控制

```csharp
using UnityEngine;
using UnityEngine.UI;

public class ButtonController : MonoBehaviour
{
    public Button button;
    public Text buttonText;
    public Image buttonImage;

    void Start()
    {
        // 1. 可交互开关
        button.interactable = true;  // 可点击
        button.interactable = false; // 灰态禁用

        // 2. 过渡模式
        button.transition = Selectable.Transition.ColorTint;   // 颜色过渡
        button.transition = Selectable.Transition.SpriteSwap;  // 精灵切换
        button.transition = Selectable.Transition.Animation;   // 动画过渡
        button.transition = Selectable.Transition.None;        // 无过渡

        // 3. 设置 Color Tint 颜色块
        ColorBlock cb = button.colors;
        cb.normalColor = Color.white;
        cb.highlightedColor = new Color(0.9f, 0.9f, 0.9f);
        cb.pressedColor = new Color(0.7f, 0.7f, 0.7f);
        cb.disabledColor = new Color(0.5f, 0.5f, 0.5f, 0.5f);
        cb.fadeDuration = 0.1f;
        button.colors = cb;

        // 4. 导航模式
        button.navigation = new Navigation()
        {
            mode = Navigation.Mode.None
        };

        // 5. 获取按钮子组件
        buttonText = button.GetComponentInChildren<Text>();
        buttonImage = button.GetComponent<Image>();
    }
}
```

#### 动态创建按钮

```csharp
using UnityEngine;
using UnityEngine.UI;

public class DynamicButtonCreator : MonoBehaviour
{
    public Transform parent;

    void Start()
    {
        // 动态创建完整按钮
        GameObject go = new GameObject("DynamicButton");
        go.transform.SetParent(parent, false);

        // 添加 RectTransform 并设置尺寸
        RectTransform rt = go.AddComponent<RectTransform>();
        rt.sizeDelta = new Vector2(160, 40);

        // 添加 Image（背景）
        Image img = go.AddComponent<Image>();
        img.color = Color.white;

        // 添加 Button
        Button btn = go.AddComponent<Button>();
        btn.targetGraphic = img; // 指定过渡效果的目标
        // 设置颜色块
        ColorBlock cb = btn.colors;
        cb.normalColor = Color.white;
        cb.highlightedColor = Color.gray;
        cb.pressedColor = new Color(0.6f, 0.6f, 0.6f);
        btn.colors = cb;

        // 添加子物体 Text（按钮文字）
        GameObject textGo = new GameObject("Text");
        textGo.transform.SetParent(go.transform, false);
        Text txt = textGo.AddComponent<Text>();
        txt.text = "动态按钮";
        txt.font = Resources.GetBuiltinResource<Font>("LegacyRuntime.ttf");
        txt.fontSize = 16;
        txt.color = Color.black;
        txt.alignment = TextAnchor.MiddleCenter;
        txt.raycastTarget = false; // Text 不开射线检测，由 Button 统一接收

        RectTransform textRt = textGo.GetComponent<RectTransform>();
        textRt.anchorMin = Vector2.zero;
        textRt.anchorMax = Vector2.one;
        textRt.sizeDelta = Vector2.zero; // 拉伸填满父级
    }
}
```

---

### 🔔 添加监听事件（核心技能）⭐

> Button 的点击事件通过 `UnityEvent` 实现，支持**代码绑定**和**Inspector 绑定**两种方式。以下是所有常用方式：

#### 方式1：Lambda 表达式（最简洁，最常用）

```csharp
using UnityEngine;
using UnityEngine.UI;

public class ButtonEventListener : MonoBehaviour
{
    public Button button;

    void Start()
    {
        // 无参方法
        button.onClick.AddListener(() =>
        {
            Debug.Log("按钮被点击了！");
        });

        // 带参数方法
        button.onClick.AddListener(() =>
        {
            OnButtonClick("参数1", 100);
        });

        // 拖动条等组件也可以用到 lambda
        // slider.onValueChanged.AddListener((float val) => { ... });
    }

    void OnButtonClick(string msg, int value)
    {
        Debug.Log($"{msg} : {value}");
    }
}
```

#### 方式2：直接绑定方法（最规范，方便管理）

```csharp
using UnityEngine;
using UnityEngine.UI;

public class ButtonMethodBinder : MonoBehaviour
{
    public Button playBtn;
    public Button pauseBtn;
    public Button quitBtn;

    void Start()
    {
        // 绑定无参方法
        playBtn.onClick.AddListener(OnPlayClick);
        pauseBtn.onClick.AddListener(OnPauseClick);
        quitBtn.onClick.AddListener(OnQuitClick);
    }

    void OnPlayClick()
    {
        Debug.Log("开始游戏");
    }

    void OnPauseClick()
    {
        Debug.Log("暂停游戏");
    }

    void OnQuitClick()
    {
        #if UNITY_EDITOR
        UnityEditor.EditorApplication.isPlaying = false;
        #else
        Application.Quit();
        #endif
    }

    void OnDestroy()
    {
        // 好的习惯：销毁时移除监听，防止内存泄漏
        playBtn.onClick.RemoveListener(OnPlayClick);
        pauseBtn.onClick.RemoveListener(OnPauseClick);
        quitBtn.onClick.RemoveListener(OnQuitClick);
    }
}
```

#### 方式3：Delegate 委托绑定

```csharp
using UnityEngine;
using UnityEngine.UI;
using UnityEngine.Events;

public class ButtonDelegateBinder : MonoBehaviour
{
    public Button button;

    void Start()
    {
        // 通过 UnityAction 委托
        UnityAction action = new UnityAction(MyClickHandler);
        button.onClick.AddListener(action);

        // 也可以直接 new UnityAction
        button.onClick.AddListener(new UnityAction(() =>
        {
            Debug.Log("通过 UnityAction 绑定");
        }));
    }

    void MyClickHandler()
    {
        Debug.Log("委托方式触发");
    }
}
```

#### 方式4：代码动态添加带参数的监听

```csharp
using UnityEngine;
using UnityEngine.UI;
using System;

public class ButtonParameterBinder : MonoBehaviour
{
    public Transform buttonParent;
    public GameObject buttonPrefab;

    void Start()
    {
        // 实战：动态生成多个按钮，每个按钮携带不同参数
        string[] items = { "道具1", "道具2", "道具3", "道具4", "道具5" };

        for (int i = 0; i < items.Length; i++)
        {
            int index = i; // ⚠️ 必须在循环内用局部变量捕获，否则闭包问题
            string itemName = items[i];

            GameObject go = Instantiate(buttonPrefab, buttonParent);
            Button btn = go.GetComponent<Button>();
            Text txt = go.GetComponentInChildren<Text>();
            txt.text = itemName;

            btn.onClick.AddListener(() =>
            {
                OnItemClick(index, itemName);
            });
        }
    }

    void OnItemClick(int index, string name)
    {
        Debug.Log($"点击了第 {index} 个道具：{name}");
    }
}
```

> [!WARNING] 闭包陷阱
> `for (int i = 0; ...)` 中，如果直接在 lambda 中使用 `i`，所有按钮的 `i` 都会指向循环结束后的最终值。==必须用 `int index = i;` 捕获当前值==，这是 C# 闭包最常见的坑。

#### 方式5：UnityEvent.Invoke（手动触发）

```csharp
using UnityEngine;
using UnityEngine.UI;

public class ButtonManualTrigger : MonoBehaviour
{
    public Button button;

    void Start()
    {
        button.onClick.AddListener(OnButtonClick);
    }

    void Update()
    {
        // 按键盘空格键模拟点击按钮
        if (Input.GetKeyDown(KeyCode.Space))
        {
            button.onClick.Invoke(); // 手动触发点击事件
        }
    }

    void OnButtonClick()
    {
        Debug.Log("按钮被触发");
    }
}
```

---

### 📊 所有监听方式对比

| 方式 | 代码量 | 灵活性 | 适用场景 |
|------|:--:|:--:|------|
| **Lambda 表达式** | 最少 | ⭐⭐⭐ | 快速原型、简单回调、带参数 |
| **直接绑定方法** | 适中 | ⭐⭐ | 正式项目、团队协作（方便 RemoveListener） |
| **Delegate (UnityAction)** | 较多 | ⭐⭐⭐ | 需要复杂的委托管理 |
| **动态生成 + 参数** | 较多 | ⭐⭐⭐ | 背包格子、商店列表等批量 UI |
| **onClick.Invoke()** | 最少 | ⭐ | 快捷键触发、自动化测试 |

---

### 🔄 事件监听管理

```csharp
using UnityEngine;
using UnityEngine.UI;

public class ButtonListenerManager : MonoBehaviour
{
    public Button button;

    void Start()
    {
        button.onClick.AddListener(OnClick1);
        button.onClick.AddListener(OnClick2);
    }

    void OnClick1() { Debug.Log("监听1"); }
    void OnClick2() { Debug.Log("监听2"); }

    // 移除指定监听
    public void RemoveOneListener()
    {
        button.onClick.RemoveListener(OnClick1);
    }

    // 移除所有监听
    public void RemoveAllListeners()
    {
        button.onClick.RemoveAllListeners();
    }

    // 查看当前有多少个监听
    public int GetListenerCount()
    {
        return button.onClick.GetPersistentEventCount(); // Inspector 绑定的数量
        // 注：代码绑定的数量无法直接获取 PersistentEventCount
        // UnityEvent 的运行时监听器数量需要通过反射获取，通常避免依赖此信息
    }

    // 仅在未绑定时添加（避免重复绑定）
    public void AddListenerOnce(UnityEngine.Events.UnityAction action)
    {
        button.onClick.RemoveListener(action); // 先移除
        button.onClick.AddListener(action);     // 再添加
    }

    void OnDestroy()
    {
        // 销毁时清理所有监听，防止内存泄漏
        button.onClick.RemoveAllListeners();
    }
}
```

| 操作 | 代码 |
|------|------|
| 添加监听 | `button.onClick.AddListener(方法名)` |
| 移除单个监听 | `button.onClick.RemoveListener(方法名)` |
| 移除所有监听 | `button.onClick.RemoveAllListeners()` |
| 手动触发 | `button.onClick.Invoke()` |
| 防止重复绑定 | 先 `RemoveListener` 再 `AddListener` |
| 获取 Inspector 绑定数量 | `button.onClick.GetPersistentEventCount()` |

> [!TIP] 最佳实践
> 1. ==Lambda 方式最简洁==，适合快速开发和大多数场景
> 2. ==正式方法绑定便于 RemoveListener==，适合需要动态切换监听的场景
> 3. ==循环生成按钮记得捕获变量==（闭包陷阱）
> 4. ==OnDestroy 中移除监听==，防止对象已销毁但事件仍被调用的 bug

---

### 📊 Button 属性速查表

| 属性 | 代码 | 说明 |
|------|------|------|
| Interactable | `button.interactable = false` | 禁用/启用按钮 |
| Transition | `button.transition = Selectable.Transition.ColorTint` | 过渡模式 |
| Colors | `button.colors = new ColorBlock(){...}` | 颜色块设置 |
| Target Graphic | `button.targetGraphic = myImage` | 过渡效果作用的目标 Graphic |
| Navigation | `button.navigation = new Navigation(){...}` | 导航设置 |
| onClick | `button.onClick.AddListener(() => {...})` | 点击事件监听 |
| IsPressed() | `button.IsPressed()` | 按钮当前是否被按下（只读） |
| Select() | `button.Select()` | 手动选中按钮 |
| OnDeselect() | `OnDeselect(BaseEventData)` | 取消选中回调（可重写） |

> [!NOTE] 命名空间
> `Button` 组件位于 `UnityEngine.UI` 命名空间，`UnityEvent` 位于 `UnityEngine.Events` 命名空间。

---

## ✅ Toggle 组件

> Toggle 是 UGUI 中的**开关/复选框**控件，继承自 `Selectable` → `UIBehaviour`。用于表示二元状态（开/关、启用/禁用），常见用途包括：设置开关、选项勾选、多选框等。

> [!IMPORTANT] Toggle 的核心组成
> Toggle 由 **Background**（背景图）、**Checkmark**（勾选标记 √ 图）和 **Label**（文字标签）三部分组成。三者都是 Toggle 的子物体，创建 Toggle 时自动生成。

---

### 📋 Inspector 参数详解

#### Interactable（可交互）

| 说明 | 详情 |
|------|------|
| **类型** | `bool` |
| **作用** | 是否可交互，设为 false 时灰色且无法点击 |
| **代码** | `toggle.interactable = false;` |

#### Transition（过渡效果）

> 与 Button 完全一致，三种模式可选：**Color Tint** / **Sprite Swap** / **Animation**。

| 模式 | 适用场景 |
|------|------|
| **Color Tint** | 最常用，切换 Background 和 Checkmark 的颜色 |
| **Sprite Swap** | 切换不同状态的 Sprite（如不同外观的勾选框） |
| **Animation** | 勾选时有弹出/缩放动画效果 |

#### Navigation（导航）

> 控制键盘/手柄导航时的焦点切换，与 Button 一致。

#### Is On（初始状态）

| 说明 | 详情 |
|------|------|
| **类型** | `bool` |
| **作用** | Toggle 的初始开/关状态 |
| **代码** | `toggle.isOn = true;` |

#### Group（单选框组）⭐

> 如果多个 Toggle 属于同一个 `ToggleGroup`，则它们之间**互斥**，同一时间只能有一个被选中（实现==单选按钮组==效果）。

| 说明 | 详情 |
|------|------|
| **类型** | `ToggleGroup` |
| **作用** | 将多个 Toggle 归入一个互斥组 |
| **代码** | `toggle.group = myToggleGroup;` |

> [!TIP] ToggleGroup 使用技巧
> 1. 创建一个空 GameObject，添加 `ToggleGroup` 组件
> 2. 将需要互斥的所有 Toggle 的 `Group` 属性拖入该 ToggleGroup
> 3. 同一组内==至少有一个 Toggle 的 `isOn` 初始为 true==，否则所有 Toggle 都可以被关闭（即组内无选中项）
> 4. ToggleGroup 的 `Allow Switch Off` 参数：勾选后允许用户关闭当前选中项（组内可全部为 off）

#### Graphic（目标图形）

| 说明 | 详情 |
|------|------|
| **类型** | `Graphic` |
| **作用** | 过渡效果影响的目标图形（通常是 Background Image） |

#### On Value Changed（值变化事件）⭐

| 说明 | 详情 |
|------|------|
| **类型** | `UnityEvent<bool>` |
| **参数** | `bool` — 当前 isOn 状态（true = 勾选，false = 取消勾选） |
| **触发时机** | 每次 `isOn` 值发生变化时 |
| **代码** | `toggle.onValueChanged.AddListener((bool isOn) => { ... });` |

---

### 💻 代码控制方式

#### 基本属性控制

```csharp
using UnityEngine;
using UnityEngine.UI;

public class ToggleController : MonoBehaviour
{
    public Toggle toggle;
    public Image background;
    public Image checkmark;
    public Text label;

    void Start()
    {
        // 1. 开启/关闭
        toggle.isOn = true;  // 勾选
        toggle.isOn = false; // 取消勾选

        // 2. 可交互开关
        toggle.interactable = true;

        // 3. 过渡模式
        toggle.transition = Selectable.Transition.ColorTint;
        toggle.transition = Selectable.Transition.SpriteSwap;

        // 4. 设置过渡目标图形
        toggle.targetGraphic = background;

        // 5. 设置颜色块
        ColorBlock cb = toggle.colors;
        cb.normalColor = Color.white;
        cb.highlightedColor = new Color(0.9f, 0.9f, 0.9f);
        cb.pressedColor = new Color(0.7f, 0.7f, 0.7f);
        cb.disabledColor = new Color(0.5f, 0.5f, 0.5f, 0.5f);
        toggle.colors = cb;

        // 6. 获取子组件引用
        background = toggle.transform.Find("Background").GetComponent<Image>();
        checkmark = toggle.transform.Find("Background/Checkmark").GetComponent<Image>();
        label = toggle.GetComponentInChildren<Text>();

        // 7. 切换勾选标记显示
        checkmark.enabled = toggle.isOn; // 同步勾选标记
        // 或通过代码控制
        checkmark.gameObject.SetActive(toggle.isOn);
    }
}
```

#### ToggleGroup 单选框组

```csharp
using UnityEngine;
using UnityEngine.UI;

public class ToggleGroupController : MonoBehaviour
{
    public ToggleGroup group;
    public Toggle toggle1, toggle2, toggle3;

    void Start()
    {
        // 创建 ToggleGroup
        group = group.GetComponent<ToggleGroup>();

        // 将多个 Toggle 归入同一组
        toggle1.group = group;
        toggle2.group = group;
        toggle3.group = group;

        // 设置 ToggleGroup 属性
        group.allowSwitchOff = false; // 不允许全部关闭（至少有一个选中）

        // 初始默认选中 toggle1
        toggle1.isOn = true;

        // 获取当前激活的 Toggle
        Toggle activeToggle = group.ActiveToggles().FirstOrDefault();
        // 注意：需要 using System.Linq;
    }
}
```

#### 动态创建 Toggle

```csharp
using UnityEngine;
using UnityEngine.UI;

public class DynamicToggleCreator : MonoBehaviour
{
    public Transform parent;

    void Start()
    {
        // 动态创建 Toggle
        GameObject go = new GameObject("DynamicToggle");
        go.transform.SetParent(parent, false);

        RectTransform rt = go.AddComponent<RectTransform>();
        rt.sizeDelta = new Vector2(200, 30);

        // 创建 Background 子物体
        GameObject bgGo = new GameObject("Background");
        bgGo.transform.SetParent(go.transform, false);
        Image bgImage = bgGo.AddComponent<Image>();
        bgImage.color = Color.white;
        RectTransform bgRt = bgGo.GetComponent<RectTransform>();
        bgRt.anchorMin = new Vector2(0, 0.5f);
        bgRt.anchorMax = new Vector2(0, 0.5f);
        bgRt.pivot = new Vector2(0, 0.5f);
        bgRt.sizeDelta = new Vector2(20, 20);
        bgRt.anchoredPosition = new Vector2(0, 0);

        // 创建 Checkmark 子物体（勾选标记）
        GameObject cmGo = new GameObject("Checkmark");
        cmGo.transform.SetParent(bgGo.transform, false);
        Image cmImage = cmGo.AddComponent<Image>();
        cmImage.color = Color.green;
        RectTransform cmRt = cmGo.GetComponent<RectTransform>();
        cmRt.anchorMin = Vector2.zero;
        cmRt.anchorMax = Vector2.one;
        cmRt.sizeDelta = Vector2.zero;

        // 创建 Label 子物体
        GameObject lblGo = new GameObject("Label");
        lblGo.transform.SetParent(go.transform, false);
        Text lblText = lblGo.AddComponent<Text>();
        lblText.text = "选项";
        lblText.font = Resources.GetBuiltinResource<Font>("LegacyRuntime.ttf");
        lblText.fontSize = 16;
        lblText.color = Color.black;
        lblText.alignment = TextAnchor.MiddleLeft;
        lblText.raycastTarget = false;
        RectTransform lblRt = lblGo.GetComponent<RectTransform>();
        lblRt.anchorMin = Vector2.zero;
        lblRt.anchorMax = Vector2.one;
        lblRt.offsetMin = new Vector2(25, 0); // 给勾选框留空间
        lblRt.offsetMax = Vector2.zero;

        // 添加 Toggle 组件并关联
        Toggle toggle = go.AddComponent<Toggle>();
        toggle.targetGraphic = bgImage;
        toggle.graphic = cmImage;    // 勾选标记
        toggle.isOn = false;
    }
}
```

---

### 🔔 事件监听

#### 方式1：Lambda 表达式（最简洁）

```csharp
using UnityEngine;
using UnityEngine.UI;

public class ToggleEventLambda : MonoBehaviour
{
    public Toggle musicToggle;
    public Toggle sfxToggle;
    public AudioSource audioSource;

    void Start()
    {
        // Lambda 直接监听
        musicToggle.onValueChanged.AddListener((bool isOn) =>
        {
            Debug.Log(isOn ? "音乐开启" : "音乐关闭");
        });

        // 调用带参数方法
        sfxToggle.onValueChanged.AddListener((bool isOn) =>
        {
            SetSFXEnabled(isOn);
        });

        // 闭包捕获外部变量
        musicToggle.onValueChanged.AddListener((bool isOn) =>
        {
            audioSource.mute = !isOn;
        });
    }

    void SetSFXEnabled(bool enabled)
    {
        Debug.Log("音效: " + (enabled ? "开" : "关"));
    }
}
```

#### 方式2：直接绑定方法

```csharp
using UnityEngine;
using UnityEngine.UI;

public class ToggleEventMethod : MonoBehaviour
{
    public Toggle fullscreenToggle;

    void Start()
    {
        fullscreenToggle.onValueChanged.AddListener(OnFullscreenChanged);
    }

    void OnFullscreenChanged(bool isOn)
    {
        Screen.fullScreen = isOn;
        Debug.Log("全屏: " + (isOn ? "是" : "否"));
    }

    void OnDestroy()
    {
        fullscreenToggle.onValueChanged.RemoveListener(OnFullscreenChanged);
    }
}
```

#### 方式3：多个 Toggle 共享同一回调（区分来源）

```csharp
using UnityEngine;
using UnityEngine.UI;

public class ToggleMultiController : MonoBehaviour
{
    public Toggle toggle1, toggle2, toggle3;

    void Start()
    {
        toggle1.onValueChanged.AddListener(OnAnyToggleChanged);
        toggle2.onValueChanged.AddListener(OnAnyToggleChanged);
        toggle3.onValueChanged.AddListener(OnAnyToggleChanged);
    }

    void OnAnyToggleChanged(bool isOn)
    {
        // 通过当前选中的 Toggle 来判断是哪个
        // 方法：遍历找出 isOn 为 true 的那个
        Toggle[] allToggles = { toggle1, toggle2, toggle3 };
        foreach (Toggle t in allToggles)
        {
            if (t.isOn)
            {
                Debug.Log("当前选中: " + t.name);
            }
        }
    }
}
```

#### 方式4：ToggleGroup 中监听选中变化

```csharp
using UnityEngine;
using UnityEngine.UI;
using System.Linq;

public class ToggleGroupListener : MonoBehaviour
{
    public ToggleGroup group;
    public Toggle[] toggles;

    void Start()
    {
        // 给每个 Toggle 绑定监听
        foreach (Toggle t in toggles)
        {
            t.onValueChanged.AddListener((bool isOn) =>
            {
                if (isOn)
                {
                    OnToggleSelected(t);
                }
            });
        }
    }

    void OnToggleSelected(Toggle selected)
    {
        Debug.Log("选中了: " + selected.name);
    }

    // 手动获取当前选中的 Toggle
    public Toggle GetActiveToggle()
    {
        return group.ActiveToggles().FirstOrDefault();
        // 需要 using System.Linq;
    }

    // 通过索引选中
    public void SelectToggleByIndex(int index)
    {
        if (index >= 0 && index < toggles.Length)
        {
            toggles[index].isOn = true;
        }
    }
}
```

#### 方式5：双向绑定（代码设置 isOn 不会触发事件）

```csharp
using UnityEngine;
using UnityEngine.UI;

public class ToggleTwoWayBind : MonoBehaviour
{
    public Toggle toggle;

    void Start()
    {
        toggle.onValueChanged.AddListener(OnToggleChanged);

        // 注意：以下代码设置 isOn 也会触发 onValueChanged 事件！
        toggle.isOn = true; // 会触发 OnToggleChanged(true)
    }

    void OnToggleChanged(bool isOn)
    {
        Debug.Log("Toggle 变化: " + isOn);
    }

    // 如果不希望触发事件，先移除监听再恢复
    public void SetIsOnWithoutNotify(bool value)
    {
        toggle.onValueChanged.RemoveListener(OnToggleChanged);
        toggle.isOn = value;
        toggle.onValueChanged.AddListener(OnToggleChanged);
    }
}
```

> [!WARNING] isOn 赋值会触发事件
> 代码中设置 `toggle.isOn = true/false` ==也会触发 `onValueChanged` 事件==。如果不想触发，需要先 `RemoveListener` 再赋值再 `AddListener`，或者用一个 bool 标记来跳过。

---

### 📊 Toggle 属性速查表

| 属性 | 代码 | 说明 |
|------|------|------|
| Is On | `toggle.isOn = true/false` | 勾选状态（赋值也会触发事件） |
| Interactable | `toggle.interactable = false` | 禁用/启用 |
| Transition | `toggle.transition = Selectable.Transition.ColorTint` | 过渡模式 |
| Colors | `toggle.colors = new ColorBlock(){...}` | 颜色块 |
| Target Graphic | `toggle.targetGraphic = bgImage` | 过渡目标图形 |
| Graphic (checkmark) | `toggle.graphic = checkmarkImage` | 勾选标记图形 |
| Group | `toggle.group = myToggleGroup` | 归属的单选组 |
| onValueChanged | `toggle.onValueChanged.AddListener((bool v) => {...})` | 值变化事件 |
| IsActive() | `toggle.IsActive()` | 当前是否处于可交互激活状态 |
| Select() | `toggle.Select()` | 手动选中 |

---

### 🎯 Toggle 实战：设置面板

```csharp
using UnityEngine;
using UnityEngine.UI;

public class SettingsPanel : MonoBehaviour
{
    public Toggle musicToggle;
    public Toggle sfxToggle;
    public Toggle fullscreenToggle;
    public Toggle vibrationToggle;

    void Start()
    {
        // 初始化状态（从 PlayerPrefs 读取或默认值）
        musicToggle.isOn = PlayerPrefs.GetInt("Music", 1) == 1;
        sfxToggle.isOn = PlayerPrefs.GetInt("SFX", 1) == 1;
        fullscreenToggle.isOn = Screen.fullScreen;
        vibrationToggle.isOn = PlayerPrefs.GetInt("Vibration", 1) == 1;

        // 绑定事件
        musicToggle.onValueChanged.AddListener(OnMusicToggled);
        sfxToggle.onValueChanged.AddListener(OnSFXToggled);
        fullscreenToggle.onValueChanged.AddListener(OnFullscreenToggled);
        vibrationToggle.onValueChanged.AddListener(OnVibrationToggled);
    }

    void OnMusicToggled(bool isOn)
    {
        PlayerPrefs.SetInt("Music", isOn ? 1 : 0);
        // AudioManager.Instance.SetMusicEnabled(isOn);
    }

    void OnSFXToggled(bool isOn)
    {
        PlayerPrefs.SetInt("SFX", isOn ? 1 : 0);
    }

    void OnFullscreenToggled(bool isOn)
    {
        Screen.fullScreen = isOn;
    }

    void OnVibrationToggled(bool isOn)
    {
        PlayerPrefs.SetInt("Vibration", isOn ? 1 : 0);
    }
}
```

> [!NOTE] 命名空间
> `Toggle` 组件位于 `UnityEngine.UI` 命名空间。`ToggleGroup` 同样在 `UnityEngine.UI`。

---

## 📝 InputField 组件

> InputField 是 UGUI 中的**文本输入框**控件，继承自 `Selectable` → `UIBehaviour`。用于接收玩家的键盘输入，常见用途包括：登录/注册表单、聊天输入、搜索框、设置参数输入等。

> [!IMPORTANT] InputField 的子物体结构
> InputField 由 **Placeholder**（占位提示文字）和 **Text**（实际显示的文本）两个子物体组成。两者都是 Text 组件，Placeholder 在输入框为空时显示（灰色提示），有内容时自动隐藏。

---

### 📋 Inspector 参数详解

#### Interactable（可交互）

| 说明 | 详情 |
|------|------|
| **类型** | `bool` |
| **作用** | 是否允许输入，false 时灰色且不可编辑 |
| **代码** | `inputField.interactable = false;` |

#### Transition / Navigation

> 过渡效果和导航，与 Button / Toggle 完全一致。

---

#### Text Component（文本组件）

| 说明 | 详情 |
|------|------|
| **类型** | `Text` |
| **作用** | 指向显示输入内容的 Text 组件（子物体 "Text"） |
| **代码** | `inputField.textComponent = myText;` |

#### Text（默认文本内容）

| 说明 | 详情 |
|------|------|
| **类型** | `string` |
| **作用** | 输入框的初始/当前文本内容 |
| **代码** | `inputField.text = "默认文字";` |

#### Character Limit（字符限制）

| 说明 | 详情 |
|------|------|
| **类型** | `int` |
| **作用** | 最大输入字符数。0 = 无限制 |
| **代码** | `inputField.characterLimit = 10;` |

---

#### Content Type（内容类型）⭐

> 决定输入框接受的字符类型和键盘弹出类型（移动端）。

| 类型 | 说明 | 典型用途 |
|------|------|----------|
| **Standard** | 标准文本，任意字符 | 通用输入、昵称 |
| **Autocorrected** | 自动纠正拼写 | 外文输入 |
| **Integer Number** | 仅整数 | 等级、数量输入 |
| **Decimal Number** | 仅数字和小数点 | 价格、百分比输入 |
| **Alphanumeric** | 仅字母和数字 | 用户名、验证码 |
| **Name** | 人名格式（首字母大写） | 角色名输入 |
| **Email Address** | 邮件地址格式 | 邮箱输入 |
| **Password** | ==密码模式（显示为 *）== | 密码输入 |
| **Pin** | PIN码模式（显示为 *，仅数字键盘） | PIN码输入 |
| **Custom** | 自定义模式，手动指定字符验证 | 特殊需求 |

| 代码设置 | 示例 |
|------|------|
| `inputField.contentType = InputField.ContentType.Standard;` |
| `inputField.contentType = InputField.ContentType.Password;` |
| `inputField.contentType = InputField.ContentType.IntegerNumber;` |
| `inputField.contentType = InputField.ContentType.DecimalNumber;` |
| `inputField.contentType = InputField.ContentType.Alphanumeric;` |
| `inputField.contentType = InputField.ContentType.EmailAddress;` |

---

#### Line Type（行类型）

| 类型 | 说明 |
|------|------|
| **Single Line** | 单行输入，按回车提交（默认） |
| **Multi Line Submit** | 多行输入，Ctrl+回车提交，回车换行 |
| **Multi Line Newline** | 多行输入，按回车即换行 |

| 代码 | 示例 |
|------|------|
| `inputField.lineType = InputField.LineType.SingleLine;` |
| `inputField.lineType = InputField.LineType.MultiLineSubmit;` |
| `inputField.lineType = InputField.LineType.MultiLineNewline;` |

---

#### Placeholder（占位符）

| 说明 | 详情 |
|------|------|
| **类型** | `Graphic`（通常是 Text 组件） |
| **作用** | 输入框为空时显示的提示文字（如"请输入用户名..."） |
| **代码** | `inputField.placeholder = myPlaceholderText;` |

#### Caret（光标）

| 参数 | 说明 |
|------|------|
| **Caret Blink Rate** | 光标闪烁频率（秒），默认 0.85。设为 0 则不闪烁 |
| **Caret Width** | 光标宽度（像素），默认 1 |
| **Caret Color** | 光标颜色 |
| **Custom Caret Color** | 是否使用自定义光标颜色（否则使用 Text 颜色） |

#### Selection（选中）

| 参数 | 说明 |
|------|------|
| **Selection Color** | 选中文本时的背景高亮颜色 |

#### Read Only（只读）

| 说明 | 详情 |
|------|------|
| **类型** | `bool` |
| **作用** | 只读模式，可选中/复制但不可编辑 |
| **代码** | `inputField.readOnly = true;` |

#### Rich Text / Hide Mobile Input 等

| 参数 | 说明 |
|------|------|
| **Rich Text** | 输入内容是否支持富文本标记 |
| **Hide Mobile Input** | 移动端是否隐藏原生输入框（使用自定义输入框） |
| **Should Activate On Select** | 选中时是否自动激活输入 |

#### On Value Changed（文本变化事件）

| 说明 | 详情 |
|------|------|
| **类型** | `UnityEvent<string>` |
| **参数** | `string` — 当前的完整文本内容 |
| **触发时机** | 每次文本内容发生变化时 |
| **代码** | `inputField.onValueChanged.AddListener((string text) => { ... });` |

#### On End Edit（编辑结束事件）

| 说明 | 详情 |
|------|------|
| **类型** | `UnityEvent<string>` |
| **参数** | `string` — 编辑结束时的文本内容 |
| **触发时机** | 按回车（单行模式）或失去焦点时 |
| **代码** | `inputField.onEndEdit.AddListener((string text) => { ... });` |

> [!IMPORTANT] OnValueChanged vs OnEndEdit
> - `onValueChanged`：==每次输入字符都触发==（每打一个字触发一次），适合实时搜索、字数统计
> - `onEndEdit`：==编辑完成时触发一次==，适合提交表单、验证输入

---

### 💻 代码控制方式

#### 基本属性控制

```csharp
using UnityEngine;
using UnityEngine.UI;

public class InputFieldController : MonoBehaviour
{
    public InputField inputField;
    public Text placeholderText;

    void Start()
    {
        // 1. 文本内容
        inputField.text = "默认文字";
        string currentText = inputField.text; // 获取当前文本

        // 2. 关联文本组件
        inputField.textComponent = inputField.transform.Find("Text").GetComponent<Text>();

        // 3. 字符限制
        inputField.characterLimit = 10; // 最多10个字符
        inputField.characterLimit = 0;  // 无限制

        // 4. 内容类型
        inputField.contentType = InputField.ContentType.Standard;
        inputField.contentType = InputField.ContentType.Password;
        inputField.contentType = InputField.ContentType.IntegerNumber;
        inputField.contentType = InputField.ContentType.EmailAddress;

        // 5. 行类型
        inputField.lineType = InputField.LineType.SingleLine;
        inputField.lineType = InputField.LineType.MultiLineSubmit;
        inputField.lineType = InputField.LineType.MultiLineNewline;

        // 6. 只读
        inputField.readOnly = false;

        // 7. 可交互
        inputField.interactable = true;

        // 8. 占位符
        inputField.placeholder = placeholderText;
        placeholderText.text = "请输入用户名...";

        // 9. 光标设置
        inputField.caretBlinkRate = 0.85f;  // 闪烁频率
        inputField.caretWidth = 2;           // 光标宽度
        inputField.customCaretColor = true;
        inputField.caretColor = Color.red;

        // 10. 选中文本高亮色
        inputField.selectionColor = new Color(0.2f, 0.5f, 1f, 0.5f);
    }
}
```

#### 文本操作方法

```csharp
using UnityEngine;
using UnityEngine.UI;

public class InputFieldOperations : MonoBehaviour
{
    public InputField inputField;

    void Start()
    {
        // 获取文本
        string txt = inputField.text;

        // 清空文本
        inputField.text = "";

        // 激活输入框（获得焦点，弹出键盘）
        inputField.ActivateInputField();

        // 取消焦点
        inputField.DeactivateInputField();

        // 是否获得焦点
        bool isFocused = inputField.isFocused;

        // 移动光标到末尾（输入后保持光标在最后）
        inputField.MoveTextEnd(false); // false = 不选中 shift
        inputField.MoveTextEnd(true);  // true = 同时选中（相当于 Shift+End）

        // 强制刷新文本显示
        inputField.ForceLabelUpdate();

        // 选中全部文本
        inputField.Select();
        // 或
        inputField.selectionAnchorPosition = 0;
        inputField.selectionFocusPosition = inputField.text.Length;

        // 光标位置
        int caretPos = inputField.caretPosition;
        inputField.caretPosition = 0; // 光标移到开头
        inputField.caretPosition = inputField.text.Length; // 光标移到末尾

        // 选中区域
        int selectionStart = inputField.selectionAnchorPosition;
        int selectionEnd = inputField.selectionFocusPosition;
        string selectedText = inputField.text.Substring(
            Mathf.Min(selectionStart, selectionEnd),
            Mathf.Abs(selectionEnd - selectionStart)
        );
    }
}
```

---

### 🔔 事件监听

#### 方式1：Lambda 表达式（最简洁）

```csharp
using UnityEngine;
using UnityEngine.UI;

public class InputFieldEventLambda : MonoBehaviour
{
    public InputField inputField;
    public Text charCountText; // 显示字数的 Text

    void Start()
    {
        // 文本变化监听（每打一个字都触发）
        inputField.onValueChanged.AddListener((string text) =>
        {
            Debug.Log("当前文本: " + text);
        });

        // 实时字数统计
        inputField.onValueChanged.AddListener((string text) =>
        {
            charCountText.text = text.Length + " / " + inputField.characterLimit;
        });

        // 编辑完成监听（回车或失焦时触发）
        inputField.onEndEdit.AddListener((string text) =>
        {
            Debug.Log("提交内容: " + text);
        });
    }
}
```

#### 方式2：直接绑定方法

```csharp
using UnityEngine;
using UnityEngine.UI;

public class InputFieldEventMethod : MonoBehaviour
{
    public InputField nameInput;
    public InputField passwordInput;

    void Start()
    {
        nameInput.onEndEdit.AddListener(OnNameSubmitted);
        passwordInput.onEndEdit.AddListener(OnPasswordSubmitted);
    }

    void OnNameSubmitted(string text)
    {
        Debug.Log("用户名: " + text);
    }

    void OnPasswordSubmitted(string text)
    {
        Debug.Log("密码长度: " + text.Length);
    }

    void OnDestroy()
    {
        nameInput.onEndEdit.RemoveListener(OnNameSubmitted);
        passwordInput.onEndEdit.RemoveListener(OnPasswordSubmitted);
    }
}
```

#### 方式3：InputField 校验与过滤

```csharp
using UnityEngine;
using UnityEngine.UI;

public class InputFieldValidator : MonoBehaviour
{
    public InputField inputField;
    public Text errorText;

    void Start()
    {
        // 实时监听做校验
        inputField.onValueChanged.AddListener((string text) =>
        {
            ValidateInput(text);
        });

        // 提交时做最终校验
        inputField.onEndEdit.AddListener((string text) =>
        {
            OnSubmit(text);
        });
    }

    void ValidateInput(string text)
    {
        // 过滤非法字符
        string filtered = System.Text.RegularExpressions.Regex.Replace(text, @"[^a-zA-Z0-9_]", "");
        if (filtered != text)
        {
            inputField.text = filtered;
            inputField.MoveTextEnd(false); // 保持光标在末尾
        }
    }

    void OnSubmit(string text)
    {
        if (string.IsNullOrEmpty(text))
        {
            errorText.text = "输入不能为空！";
            errorText.color = Color.red;
        }
        else if (text.Length < 3)
        {
            errorText.text = "至少需要3个字符！";
            errorText.color = Color.red;
        }
        else
        {
            errorText.text = "✓ 输入有效";
            errorText.color = Color.green;
        }
    }
}
```

#### 方式4：Tab 键切换输入框

```csharp
using UnityEngine;
using UnityEngine.UI;
using System.Collections.Generic;

public class InputFieldTabNavigation : MonoBehaviour
{
    public List<InputField> inputFields;

    void Update()
    {
        if (Input.GetKeyDown(KeyCode.Tab))
        {
            for (int i = 0; i < inputFields.Count; i++)
            {
                if (inputFields[i].isFocused)
                {
                    int nextIndex = (i + 1) % inputFields.Count;
                    inputFields[nextIndex].ActivateInputField();
                    break;
                }
            }
        }
    }
}
```

---

### 🎯 InputField 实战：登录面板

```csharp
using UnityEngine;
using UnityEngine.UI;

public class LoginPanel : MonoBehaviour
{
    public InputField usernameInput;
    public InputField passwordInput;
    public Button loginButton;
    public Text errorText;

    void Start()
    {
        // 密码输入框设为密码模式
        passwordInput.contentType = InputField.ContentType.Password;
        passwordInput.characterLimit = 16;

        // 用户名输入框
        usernameInput.characterLimit = 12;
        usernameInput.contentType = InputField.ContentType.Alphanumeric;

        // 绑定事件
        usernameInput.onValueChanged.AddListener(OnInputChanged);
        passwordInput.onValueChanged.AddListener(OnInputChanged);

        usernameInput.onEndEdit.AddListener((string text) =>
        {
            passwordInput.ActivateInputField(); // 回车自动跳到密码框
        });

        passwordInput.onEndEdit.AddListener((string text) =>
        {
            OnLoginClick(); // 密码框回车自动登录
        });

        loginButton.onClick.AddListener(OnLoginClick);

        // 初始化登录按钮状态
        UpdateLoginButton();
    }

    void OnInputChanged(string text)
    {
        errorText.text = "";
        UpdateLoginButton();
    }

    void UpdateLoginButton()
    {
        // 用户名和密码都不为空才能点击登录
        loginButton.interactable = !string.IsNullOrEmpty(usernameInput.text)
                                   && !string.IsNullOrEmpty(passwordInput.text);
    }

    void OnLoginClick()
    {
        string username = usernameInput.text.Trim();
        string password = passwordInput.text;

        if (username.Length < 3)
        {
            errorText.text = "用户名至少3个字符";
            return;
        }
        if (password.Length < 6)
        {
            errorText.text = "密码至少6个字符";
            return;
        }

        errorText.text = "登录中...";
        errorText.color = Color.yellow;
        // 发送登录请求...
    }
}
```

---

### 📊 InputField 属性速查表

| 属性 | 代码 | 说明 |
|------|------|------|
| Text | `inputField.text = "xxx"` | 获取/设置文本 |
| Character Limit | `inputField.characterLimit = 10` | 最大字符数 |
| Content Type | `inputField.contentType = InputField.ContentType.Password` | 内容类型 |
| Line Type | `inputField.lineType = InputField.LineType.MultiLineNewline` | 单行/多行 |
| Read Only | `inputField.readOnly = true` | 只读模式 |
| Placeholder | `inputField.placeholder = txt` | 占位提示文字 |
| Caret Blink Rate | `inputField.caretBlinkRate = 0.5f` | 光标闪烁频率 |
| Caret Width | `inputField.caretWidth = 2` | 光标宽度 |
| Caret Color | `inputField.caretColor = Color.red` | 光标颜色 |
| Custom Caret Color | `inputField.customCaretColor = true` | 是否自定义光标色 |
| Selection Color | `inputField.selectionColor = Color.blue` | 选中高亮色 |
| Interactable | `inputField.interactable = false` | 是否可交互 |
| onValueChanged | `inputField.onValueChanged.AddListener(...)` | 文本变化事件 |
| onEndEdit | `inputField.onEndEdit.AddListener(...)` | 编辑结束事件 |
| ActivateInputField | `inputField.ActivateInputField()` | 激活获得焦点 |
| DeactivateInputField | `inputField.DeactivateInputField()` | 取消焦点 |
| isFocused | `inputField.isFocused` | 是否获得焦点（只读） |
| MoveTextEnd | `inputField.MoveTextEnd(false)` | 移动光标到末尾 |
| ForceLabelUpdate | `inputField.ForceLabelUpdate()` | 强制刷新文本 |

> [!NOTE] 命名空间
> `InputField` 组件位于 `UnityEngine.UI` 命名空间。



---

## 🎚️ Slider 组件

> Slider 是 UGUI 中的**滑动条**控件，继承自 `Selectable` → `UIBehaviour`。用于通过拖动滑块在最小值和最大值之间选择一个数值，常见用途：音量调节、亮度设置、进度控制、灵敏度调节等。

> [!IMPORTANT] Slider 的子物体结构
> Slider 由 **Background**（背景轨道）、**Fill Area → Fill**（填充区域，显示当前进度）、**Handle Slide Area → Handle**（拖动手柄）三部分组成。

---

### 📋 Inspector 参数详解

#### Interactable / Transition / Navigation

> 可交互、过渡效果、导航设置，与 Button / Toggle 完全一致。

---

#### Fill Rect（填充矩形）

| 说明 | 详情 |
|------|------|
| **类型** | `RectTransform` |
| **作用** | 指向填充区域的 RectTransform（Fill Area 下的 Fill 物体） |
| **代码** | `slider.fillRect = fillRectTransform;` |

#### Handle Rect（滑块矩形）

| 说明 | 详情 |
|------|------|
| **类型** | `RectTransform` |
| **作用** | 指向拖动手柄的 RectTransform（Handle Slide Area 下的 Handle 物体） |
| **代码** | `slider.handleRect = handleRectTransform;` |

---

#### Direction（滑动方向）

| 选项 | 说明 | 适用场景 |
|------|------|------|
| **Left To Right** | 水平左→右（默认） | 最常见的水平滑动条 |
| **Right To Left** | 水平右→左 | 从右往左的进度条 |
| **Bottom To Top** | 垂直下→上 | 垂直音量条 |
| **Top To Bottom** | 垂直上→下 | 从上往下的滑动条 |

| 代码 | 示例 |
|------|------|
| `slider.direction = Slider.Direction.LeftToRight;` |
| `slider.direction = Slider.Direction.BottomToTop;` |

---

#### Min Value / Max Value（最小/最大值）

| 参数 | 说明 | 默认值 |
|------|------|:--:|
| **Min Value** | 滑动条的最小值 | 0 |
| **Max Value** | 滑动条的最大值 | 1 |

```csharp
slider.minValue = 0f;
slider.maxValue = 100f;
slider.minValue = -50f; // 支持负值
```

> [!TIP] 常见数值范围
> 音量：0~1 或 0~100 / 百分比：0~1 / 速度：0.5~2.0 / 灵敏度：-1~1

#### Whole Numbers（整数模式）

| 说明 | 详情 |
|------|------|
| **类型** | `bool` |
| **作用** | 勾选后滑块值自动取整 |
| **代码** | `slider.wholeNumbers = true;` |

#### Value（当前值）

| 说明 | 详情 |
|------|------|
| **类型** | `float` |
| **作用** | 滑动条的当前值（自动限制在 Min ~ Max 之间） |
| **代码** | `slider.value = 50f;` |

---

#### On Value Changed（值变化事件）⭐

| 说明 | 详情 |
|------|------|
| **类型** | `UnityEvent<float>` |
| **参数** | `float` — 当前的值 |
| **触发时机** | ==拖动过程中每帧持续触发==（与 Button.onClick 只在点击时触发完全不同） |
| **代码** | `slider.onValueChanged.AddListener((float val) => { ... });` |

> [!IMPORTANT] Slider 事件特点
> `onValueChanged` 在拖动滑块时 ==每帧都会触发==。如果在回调中做重操作（如保存文件），应使用 EventTrigger 监听 PointerUp（松手时才处理），或做节流。

---

### 💻 代码控制方式

#### 基本属性控制

```csharp
using UnityEngine;
using UnityEngine.UI;

public class SliderController : MonoBehaviour
{
    public Slider slider;
    public Image background;
    public Image fill;
    public Image handle;

    void Start()
    {
        // 1. 当前值
        slider.value = 50f;
        float current = slider.value;

        // 2. 取值范围
        slider.minValue = 0f;
        slider.maxValue = 100f;

        // 3. 整数模式
        slider.wholeNumbers = true;

        // 4. 方向
        slider.direction = Slider.Direction.LeftToRight;
        slider.direction = Slider.Direction.BottomToTop;

        // 5. 可交互
        slider.interactable = true;

        // 6. 获取子组件引用
        background = slider.transform.Find("Background").GetComponent<Image>();
        fill = slider.transform.Find("Fill Area/Fill").GetComponent<Image>();
        handle = slider.transform.Find("Handle Slide Area/Handle").GetComponent<Image>();

        // 7. 关联 Fill 和 Handle 的 RectTransform
        slider.fillRect = fill.GetComponent<RectTransform>();
        slider.handleRect = handle.GetComponent<RectTransform>();

        // 8. 过渡效果
        slider.transition = Selectable.Transition.ColorTint;
        ColorBlock cb = slider.colors;
        cb.normalColor = Color.white;
        cb.highlightedColor = new Color(0.9f, 0.9f, 0.9f);
        cb.pressedColor = new Color(0.7f, 0.7f, 0.7f);
        slider.colors = cb;

        // 9. 关闭 Handle（纯进度条模式）
        handle.gameObject.SetActive(false);
        slider.interactable = false; // 不可拖动，仅展示
    }
}
```

#### 归一化值（Normalized Value）

```csharp
// 获取归一化值（0~1，不受 minValue/maxValue 影响）
float normalized = slider.normalizedValue;

// 设置归一化值
slider.normalizedValue = 0.5f; // 设置为范围的 50%
slider.normalizedValue = 1f;   // 设置为最大值

// 等价计算
// normalizedValue = (value - minValue) / (maxValue - minValue)
// value = normalizedValue * (maxValue - minValue) + minValue
```

#### 动态创建 Slider

```csharp
using UnityEngine;
using UnityEngine.UI;

public class DynamicSliderCreator : MonoBehaviour
{
    public Transform parent;

    void Start()
    {
        GameObject go = new GameObject("DynamicSlider");
        go.transform.SetParent(parent, false);
        RectTransform rt = go.AddComponent<RectTransform>();
        rt.sizeDelta = new Vector2(300, 20);

        // Background
        GameObject bgGo = new GameObject("Background");
        bgGo.transform.SetParent(go.transform, false);
        Image bgImage = bgGo.AddComponent<Image>();
        bgImage.color = new Color(0.8f, 0.8f, 0.8f);
        RectTransform bgRt = bgGo.GetComponent<RectTransform>();
        bgRt.anchorMin = Vector2.zero;
        bgRt.anchorMax = Vector2.one;
        bgRt.sizeDelta = Vector2.zero;

        // Fill Area + Fill
        GameObject fillArea = new GameObject("Fill Area");
        fillArea.transform.SetParent(go.transform, false);
        RectTransform faRt = fillArea.AddComponent<RectTransform>();
        faRt.anchorMin = Vector2.zero;
        faRt.anchorMax = Vector2.one;
        faRt.sizeDelta = new Vector2(-10, 0);
        faRt.anchoredPosition = new Vector2(5, 0);

        GameObject fillGo = new GameObject("Fill");
        fillGo.transform.SetParent(fillArea.transform, false);
        Image fillImage = fillGo.AddComponent<Image>();
        fillImage.color = Color.green;
        RectTransform fillRt = fillGo.GetComponent<RectTransform>();
        fillRt.sizeDelta = Vector2.zero;

        // Handle Slide Area + Handle
        GameObject handleArea = new GameObject("Handle Slide Area");
        handleArea.transform.SetParent(go.transform, false);
        RectTransform haRt = handleArea.AddComponent<RectTransform>();
        haRt.anchorMin = Vector2.zero;
        haRt.anchorMax = Vector2.one;
        haRt.sizeDelta = new Vector2(-20, 0);
        haRt.anchoredPosition = new Vector2(10, 0);

        GameObject handleGo = new GameObject("Handle");
        handleGo.transform.SetParent(handleArea.transform, false);
        Image handleImage = handleGo.AddComponent<Image>();
        handleImage.color = Color.white;
        RectTransform handleRt = handleGo.GetComponent<RectTransform>();
        handleRt.sizeDelta = new Vector2(20, 20);

        // 添加 Slider 组件并关联
        Slider slider = go.AddComponent<Slider>();
        slider.targetGraphic = handleImage;
        slider.fillRect = fillRt;
        slider.handleRect = handleRt;
        slider.minValue = 0f;
        slider.maxValue = 100f;
        slider.value = 50f;
    }
}
```

---

### 🔔 事件监听

#### 方式1：Lambda 表达式

```csharp
using UnityEngine;
using UnityEngine.UI;

public class SliderEventLambda : MonoBehaviour
{
    public Slider volumeSlider;
    public Text volumeText;

    void Start()
    {
        // Lambda 监听值变化（每帧触发）
        volumeSlider.onValueChanged.AddListener((float value) =>
        {
            Debug.Log("当前值: " + value);
        });

        // 实时更新显示
        volumeSlider.onValueChanged.AddListener((float value) =>
        {
            volumeText.text = Mathf.RoundToInt(value).ToString();
        });
    }
}
```

#### 方式2：直接绑定方法

```csharp
using UnityEngine;
using UnityEngine.UI;
using UnityEngine.Audio;

public class SliderEventMethod : MonoBehaviour
{
    public Slider masterSlider;
    public Slider musicSlider;
    public Slider sfxSlider;
    public AudioMixer audioMixer;

    void Start()
    {
        masterSlider.onValueChanged.AddListener(OnMasterVolumeChanged);
        musicSlider.onValueChanged.AddListener(OnMusicVolumeChanged);
        sfxSlider.onValueChanged.AddListener(OnSFXVolumeChanged);

        // 初始化值
        masterSlider.value = 80f;
        musicSlider.value = 70f;
        sfxSlider.value = 100f;
    }

    void OnMasterVolumeChanged(float value)
    {
        // 将 0~100 转换为分贝 (-80dB ~ 0dB)
        float dB = value > 0 ? Mathf.Log10(value / 100f) * 20 : -80f;
        audioMixer.SetFloat("MasterVolume", dB);
    }

    void OnMusicVolumeChanged(float value)
    {
        float dB = value > 0 ? Mathf.Log10(value / 100f) * 20 : -80f;
        audioMixer.SetFloat("MusicVolume", dB);
    }

    void OnSFXVolumeChanged(float value)
    {
        float dB = value > 0 ? Mathf.Log10(value / 100f) * 20 : -80f;
        audioMixer.SetFloat("SFXVolume", dB);
    }

    void OnDestroy()
    {
        masterSlider.onValueChanged.RemoveListener(OnMasterVolumeChanged);
        musicSlider.onValueChanged.RemoveListener(OnMusicVolumeChanged);
        sfxSlider.onValueChanged.RemoveListener(OnSFXVolumeChanged);
    }
}
```

#### 方式3：节流处理（松手时才保存）

```csharp
using UnityEngine;
using UnityEngine.UI;
using UnityEngine.EventSystems;

public class SliderThrottle : MonoBehaviour
{
    public Slider slider;
    public Text valueText;
    private bool isDragging = false;

    void Start()
    {
        // 拖动时仅更新 UI，不保存
        slider.onValueChanged.AddListener((float value) =>
        {
            valueText.text = Mathf.RoundToInt(value).ToString();
        });

        // 添加 EventTrigger 监听松手事件
        EventTrigger trigger = slider.gameObject.AddComponent<EventTrigger>();

        EventTrigger.Entry pointerDown = new EventTrigger.Entry();
        pointerDown.eventID = EventTriggerType.PointerDown;
        pointerDown.callback.AddListener((data) => { isDragging = true; });
        trigger.triggers.Add(pointerDown);

        EventTrigger.Entry pointerUp = new EventTrigger.Entry();
        pointerUp.eventID = EventTriggerType.PointerUp;
        pointerUp.callback.AddListener((data) => {
            isDragging = false;
            PlayerPrefs.SetFloat("SettingValue", slider.value);
            PlayerPrefs.Save();
            Debug.Log("保存值: " + slider.value);
        });
        trigger.triggers.Add(pointerUp);
    }
}
```

#### 方式4：代码设置值不触发事件

```csharp
public class SliderSilentSet : MonoBehaviour
{
    public Slider slider;

    void Start()
    {
        slider.onValueChanged.AddListener(OnValueChanged);

        // 方法1：先移除再恢复
        slider.onValueChanged.RemoveListener(OnValueChanged);
        slider.value = 75f;
        slider.onValueChanged.AddListener(OnValueChanged);

        // 方法2：用标记位
        SetValueSilent(75f);
    }

    private bool ignoreEvents = false;
    void SetValueSilent(float value)
    {
        ignoreEvents = true;
        slider.value = value;
        ignoreEvents = false;
    }

    void OnValueChanged(float value)
    {
        if (ignoreEvents) return;
        Debug.Log("值变化: " + value);
    }
}
```

---

### 🎯 Slider 实战：音量设置面板

```csharp
using UnityEngine;
using UnityEngine.UI;
using UnityEngine.Audio;

public class VolumeSettings : MonoBehaviour
{
    public Slider masterSlider, bgmSlider, sfxSlider;
    public Text masterValueText, bgmValueText, sfxValueText;
    public AudioMixer audioMixer;

    void Start()
    {
        // 从 PlayerPrefs 加载（默认80）
        float masterVol = PlayerPrefs.GetFloat("MasterVolume", 80f);
        float bgmVol = PlayerPrefs.GetFloat("BGMVolume", 80f);
        float sfxVol = PlayerPrefs.GetFloat("SFXVolume", 100f);

        // 初始化（先移除监听再设置，避免触发事件）
        SetupSlider(masterSlider, masterValueText, "MasterVolume", masterVol);
        SetupSlider(bgmSlider, bgmValueText, "MusicVolume", bgmVol);
        SetupSlider(sfxSlider, sfxValueText, "SFXVolume", sfxVol);
    }

    void SetupSlider(Slider sld, Text label, string mixerParam, float initValue)
    {
        sld.onValueChanged.RemoveAllListeners();
        sld.value = initValue;
        label.text = Mathf.RoundToInt(initValue).ToString();

        sld.onValueChanged.AddListener((float v) => {
            label.text = Mathf.RoundToInt(v).ToString();
            float dB = v > 0 ? Mathf.Log10(v / 100f) * 20 : -80f;
            audioMixer.SetFloat(mixerParam, dB);
            PlayerPrefs.SetFloat(mixerParam, v);
        });
    }
}
```

---

### 📊 Slider 属性速查表

| 属性 | 代码 | 说明 |
|------|------|------|
| Value | `slider.value = 50f` | 设置/获取当前值 |
| Normalized Value | `slider.normalizedValue = 0.5f` | 归一化值 0~1 |
| Min Value | `slider.minValue = 0f` | 最小值 |
| Max Value | `slider.maxValue = 100f` | 最大值 |
| Whole Numbers | `slider.wholeNumbers = true` | 整数模式 |
| Direction | `slider.direction = Slider.Direction.LeftToRight` | 滑动方向 |
| Fill Rect | `slider.fillRect = fillTransform` | 填充区域的 RectTransform |
| Handle Rect | `slider.handleRect = handleTransform` | 滑块手柄的 RectTransform |
| Interactable | `slider.interactable = false` | 可交互开关 |
| Transition | `slider.transition = Selectable.Transition.ColorTint` | 过渡模式 |
| onValueChanged | `slider.onValueChanged.AddListener((float v) => {...})` | 值变化事件（每帧） |

> [!NOTE] 命名空间
> `Slider` 组件位于 `UnityEngine.UI` 命名空间。



---

## 📜 Scrollbar 组件

> Scrollbar 是 UGUI 中的**滚动条**控件，继承自 `Selectable` → `UIBehaviour`。与 Slider 类似，但功能更简洁——只负责显示可拖动的滚动条，通常配合 `ScrollRect` 一起使用。独立使用时可用于任何 0~1 范围的数值控制。

> [!IMPORTANT] Scrollbar vs Slider
> | 对比 | Scrollbar | Slider |
> |------|-----------|--------|
> | 取值范围 | 固定 0~1（归一化） | 可自定义 Min/Max |
> | 整数模式 | ❌ 无 | ✅ WholeNumbers |
> | 方向控制 | 仅有 Direction | Direction + Min/Max/WholeNumbers |
> | 典型用途 | ScrollRect 的滚动条 | 音量、进度、参数调节 |

---

### 📋 Inspector 参数详解

#### Interactable / Transition / Navigation

> 可交互、过渡效果、导航设置，与 Button / Toggle / Slider 完全一致。

---

#### Handle Rect（滑块矩形）

| 说明 | 详情 |
|------|------|
| **类型** | `RectTransform` |
| **作用** | 指向拖动手柄的 RectTransform |
| **代码** | `scrollbar.handleRect = handleTransform;` |

#### Direction（滚动方向）⭐

> 控制滚动条的方向，确定了拖动的正方向和 Handle 的锚点规则。

| 选项 | 说明 |
|------|------|
| **Left To Right** | 水平左→右 |
| **Right To Left** | 水平右→左 |
| **Bottom To Top** | 垂直下→上 |
| **Top To Bottom** | 垂直上→下 |

| 代码 | 示例 |
|------|------|
| `scrollbar.direction = Scrollbar.Direction.LeftToRight;` |
| `scrollbar.direction = Scrollbar.Direction.BottomToTop;` |

---

#### Value（当前值）

| 说明 | 详情 |
|------|------|
| **类型** | `float`（范围 0~1） |
| **作用** | 滚动条的当前归一化位置 |
| **代码** | `scrollbar.value = 0.5f;` |

#### Size（滑块大小）

| 说明 | 详情 |
|------|------|
| **类型** | `float`（范围 0~1） |
| **作用** | ==Handle 的尺寸比例==。0 = 最小（手柄为一小方块），1 = 手柄撑满整个轨道 |
| **代码** | `scrollbar.size = 0.2f;` |

> [!TIP] Size 在 ScrollRect 中的意义
> 配合 ScrollRect 时，`size` 由系统自动计算 = `视口高度 / 内容总高度`。内容越多 → size 越小 → Handle 越短 → 用户体验更直观地感知内容长度。

#### Number Of Steps（步数）

| 说明 | 详情 |
|------|------|
| **类型** | `int` |
| **作用** | 分段滚动步数。0 或 1 = 连续模式（默认）。>1 = 离散模式，Handle 只停靠在固定位置上 |
| **代码** | `scrollbar.numberOfSteps = 10;` |

---

#### On Value Changed（值变化事件）⭐

| 说明 | 详情 |
|------|------|
| **类型** | `UnityEvent<float>` |
| **参数** | `float` — 当前的值（0~1） |
| **触发时机** | 拖动过程中每帧持续触发 |
| **代码** | `scrollbar.onValueChanged.AddListener((float val) => { ... });` |

---

### 💻 代码控制方式

#### 基本属性控制

```csharp
using UnityEngine;
using UnityEngine.UI;

public class ScrollbarController : MonoBehaviour
{
    public Scrollbar scrollbar;
    public Image handle;

    void Start()
    {
        // 1. 当前值
        scrollbar.value = 0.5f;
        float current = scrollbar.value;

        // 2. 滑块大小
        scrollbar.size = 0.2f; // Handle 占轨道的 20%

        // 3. 步数（离散模式）
        scrollbar.numberOfSteps = 5; // 5 个停靠位置
        scrollbar.numberOfSteps = 0; // 连续模式

        // 4. 方向
        scrollbar.direction = Scrollbar.Direction.LeftToRight;
        scrollbar.direction = Scrollbar.Direction.BottomToTop;

        // 5. 可交互
        scrollbar.interactable = true;

        // 6. 获取 Handle 引用
        handle = scrollbar.transform.Find("Sliding Area/Handle").GetComponent<Image>();

        // 7. 关联 Handle RectTransform
        scrollbar.handleRect = handle.GetComponent<RectTransform>();
    }
}
```

#### 动态创建 Scrollbar

```csharp
using UnityEngine;
using UnityEngine.UI;

public class DynamicScrollbarCreator : MonoBehaviour
{
    public Transform parent;

    void Start()
    {
        // 创建垂直滚动条
        GameObject go = new GameObject("DynamicScrollbar");
        go.transform.SetParent(parent, false);
        RectTransform rt = go.AddComponent<RectTransform>();
        rt.sizeDelta = new Vector2(20, 300); // 宽度20，高度300

        // 创建 Sliding Area
        GameObject slidingArea = new GameObject("Sliding Area");
        slidingArea.transform.SetParent(go.transform, false);
        RectTransform saRt = slidingArea.AddComponent<RectTransform>();
        saRt.anchorMin = Vector2.zero;
        saRt.anchorMax = Vector2.one;
        saRt.sizeDelta = Vector2.zero;

        // 创建 Handle
        GameObject handleGo = new GameObject("Handle");
        handleGo.transform.SetParent(slidingArea.transform, false);
        Image handleImage = handleGo.AddComponent<Image>();
        handleImage.color = Color.white;
        RectTransform handleRt = handleGo.GetComponent<RectTransform>();
        handleRt.anchorMin = Vector2.zero;
        handleRt.anchorMax = Vector2.one;
        handleRt.sizeDelta = new Vector2(-5, -5);

        // 添加 Scrollbar 组件并关联
        Scrollbar scrollbar = go.AddComponent<Scrollbar>();
        scrollbar.targetGraphic = handleImage;
        scrollbar.handleRect = handleRt;
        scrollbar.direction = Scrollbar.Direction.BottomToTop;
        scrollbar.value = 1f;
        scrollbar.size = 0.2f;
    }
}
```

---

### 🔔 事件监听

#### 方式1：Lambda 表达式

```csharp
using UnityEngine;
using UnityEngine.UI;

public class ScrollbarEventLambda : MonoBehaviour
{
    public Scrollbar scrollbar;
    public Text valueText;

    void Start()
    {
        scrollbar.onValueChanged.AddListener((float value) =>
        {
            Debug.Log("滚动位置: " + value.ToString("F2"));
        });

        scrollbar.onValueChanged.AddListener((float value) =>
        {
            valueText.text = Mathf.RoundToInt(value * 100) + "%";
        });
    }
}
```

#### 方式2：直接绑定方法

```csharp
using UnityEngine;
using UnityEngine.UI;

public class ScrollbarEventMethod : MonoBehaviour
{
    public Scrollbar scrollbar;
    public Text pageText;
    private int currentPage = 0;
    private int totalPages = 5;

    void Start()
    {
        scrollbar.numberOfSteps = totalPages;
        scrollbar.onValueChanged.AddListener(OnScrollValueChanged);
    }

    void OnScrollValueChanged(float value)
    {
        // 将 0~1 值映射到页数
        int page = Mathf.RoundToInt(value * (totalPages - 1));
        if (page != currentPage)
        {
            currentPage = page;
            pageText.text = (currentPage + 1) + " / " + totalPages;
        }
    }

    void OnDestroy()
    {
        scrollbar.onValueChanged.RemoveListener(OnScrollValueChanged);
    }
}
```

#### 方式3：代码值变化与 ScrollRect 联动

```csharp
using UnityEngine;
using UnityEngine.UI;

public class ScrollbarScrollRectSync : MonoBehaviour
{
    public ScrollRect scrollRect;
    public Scrollbar scrollbar;

    void Start()
    {
        // Scrollbar 变化时更新 ScrollRect
        scrollbar.onValueChanged.AddListener((float value) =>
        {
            scrollRect.verticalNormalizedPosition = value;
        });

        // ScrollRect 滚动时更新 Scrollbar
        scrollRect.onValueChanged.AddListener((Vector2 pos) =>
        {
            scrollbar.value = pos.y;
        });
    }
}
```

---

### 📊 Scrollbar 属性速查表

| 属性 | 代码 | 说明 |
|------|------|------|
| Value | `scrollbar.value = 0.5f` | 当前值（0~1） |
| Size | `scrollbar.size = 0.2f` | 滑块占轨道的比例 |
| Number Of Steps | `scrollbar.numberOfSteps = 10` | 离散步数（0=连续） |
| Direction | `scrollbar.direction = Scrollbar.Direction.BottomToTop` | 滚动方向 |
| Handle Rect | `scrollbar.handleRect = handleTransform` | 滑块 RectTransform |
| Interactable | `scrollbar.interactable = false` | 可交互开关 |
| Transition | `scrollbar.transition = Selectable.Transition.ColorTint` | 过渡模式 |
| onValueChanged | `scrollbar.onValueChanged.AddListener((float v) => {...})` | 值变化事件 |

> [!NOTE] 命名空间
> `Scrollbar` 组件位于 `UnityEngine.UI` 命名空间。



---

## 📜 ScrollRect（ScrollView）组件

> ScrollRect 是 UGUI 中的**滚动视图**控件，是==制作可滚动列表的核心组件==。它可以让内容区域（Content）在视口（Viewport）内滚动，配合 `Scrollbar` 和 `Mask` 实现完整的滚动列表效果。

> [!IMPORTANT] ScrollRect 的子物体结构
> ScrollRect 需要三个子物体：**Viewport**（视口，带 Mask 裁剪）+ **Content**（内容，所有列表项放在这里）+ 可选的 **Scrollbar Horizontal/Vertical**（滚动条）。

```
ScrollRect (GameObject)
├── Viewport (带 Image + Mask)
│   └── Content (内容容器，所有子项放在这里)
├── Scrollbar Horizontal (可选)
└── Scrollbar Vertical (可选)
```

---

### 📋 Inspector 参数详解

#### Content（内容容器）

| 说明 | 详情 |
|------|------|
| **类型** | `RectTransform` |
| **作用** | ==指向包含所有滚动内容的 RectTransform==。Content 的大小决定了可滚动的范围 |
| **代码** | `scrollRect.content = contentRectTransform;` |

> Content 的尺寸必须大于 Viewport 的尺寸才会出现滚动效果。

#### Horizontal / Vertical（水平/垂直滚动开关）

| 说明 | 详情 |
|------|------|
| **类型** | `bool` |
| **作用** | 是否允许水平/垂直方向滚动 |
| **代码** | `scrollRect.horizontal = true;` `scrollRect.vertical = false;` |

#### Movement Type（移动类型）

> 决定滚动时的==物理行为和边界处理==方式。

| 类型 | 说明 |
|------|------|
| **Unrestricted**（无限制） | 可以滚动到任意位置，不限制边界。适合无限画布 |
| **Elastic**（弹性）⭐ | ==默认模式==。滚动到边界时会回弹，有弹性效果 |
| **Clamped**（夹紧） | 严格限制在边界内，不超出。适合精确的分页滚动 |

| 子参数 | 说明 |
|------|------|
| **Elasticity** | 弹性系数（仅 Elastic 模式）。值越小弹性越大，0 = 无弹性 |

```csharp
scrollRect.movementType = ScrollRect.MovementType.Elastic;
scrollRect.movementType = ScrollRect.MovementType.Clamped;
scrollRect.movementType = ScrollRect.MovementType.Unrestricted;
scrollRect.elasticity = 0.1f; // 仅 Elastic 模式有效
```

#### Inertia（惯性）

| 说明 | 详情 |
|------|------|
| **类型** | `bool` |
| **作用** | 松手后是否继续滑行（惯性效果） |
| **代码** | `scrollRect.inertia = true;` |

| 子参数 | 说明 |
|------|------|
| **Deceleration Rate** | 减速率（0~1）。越小减速越快，0 = 立刻停止。默认 0.135 |

#### Scroll Sensitivity（滚动灵敏度）

| 说明 | 详情 |
|------|------|
| **类型** | `float` |
| **作用** | ==鼠标滚轮每格的滚动距离==。值越大滚动越快 |
| **代码** | `scrollRect.scrollSensitivity = 30f;` |

---

#### Viewport（视口）

| 说明 | 详情 |
|------|------|
| **类型** | `RectTransform` |
| **作用** | 指向显示区域的 RectTransform，通常带 Mask 组件 |
| **代码** | `scrollRect.viewport = viewportRectTransform;` |

#### Horizontal Scrollbar / Vertical Scrollbar

| 说明 | 详情 |
|------|------|
| **类型** | `Scrollbar` |
| **作用** | 关联水平/垂直滚动条组件，自动同步滚动位置 |
| **代码** | `scrollRect.horizontalScrollbar = myScrollbar;` |

| 子参数 | 说明 |
|------|------|
| **Visibility** | 滚动条显示模式：Permanent（始终显示）、Auto Hide（自动隐藏）、Auto Hide And Expand Viewport（自动隐藏并扩展视口） |
| **Spacing** | 滚动条与视口之间的间距 |

---

#### On Value Changed（值变化事件）⭐

| 说明 | 详情 |
|------|------|
| **类型** | `UnityEvent<Vector2>` |
| **参数** | `Vector2` — 归一化滚动位置 (horizontal, vertical)，0~1 |
| **触发时机** | 滚动过程中每帧触发 |
| **代码** | `scrollRect.onValueChanged.AddListener((Vector2 pos) => { ... });` |

---

### 💻 代码控制方式

#### 基本属性控制

```csharp
using UnityEngine;
using UnityEngine.UI;

public class ScrollRectController : MonoBehaviour
{
    public ScrollRect scrollRect;
    public RectTransform content;

    void Start()
    {
        // 1. 关联 Content
        scrollRect.content = content;

        // 2. 滚动方向开关
        scrollRect.horizontal = true;   // 允许水平滚动
        scrollRect.vertical = false;    // 禁止垂直滚动

        // 3. 移动类型
        scrollRect.movementType = ScrollRect.MovementType.Elastic;
        scrollRect.movementType = ScrollRect.MovementType.Clamped;
        scrollRect.movementType = ScrollRect.MovementType.Unrestricted;
        scrollRect.elasticity = 0.1f;

        // 4. 惯性
        scrollRect.inertia = true;
        scrollRect.decelerationRate = 0.135f;

        // 5. 灵敏度
        scrollRect.scrollSensitivity = 30f;

        // 6. 滚动位置
        scrollRect.horizontalNormalizedPosition = 0f; // 滚动到最左
        scrollRect.horizontalNormalizedPosition = 1f; // 滚动到最右
        scrollRect.verticalNormalizedPosition = 1f;   // 滚动到顶部
        scrollRect.verticalNormalizedPosition = 0f;   // 滚动到底部
    }
}
```

#### 滚动位置控制

```csharp
using UnityEngine;
using UnityEngine.UI;

public class ScrollRectNavigation : MonoBehaviour
{
    public ScrollRect scrollRect;

    // 滚动到顶部
    public void ScrollToTop()
    {
        scrollRect.verticalNormalizedPosition = 1f;
    }

    // 滚动到底部
    public void ScrollToBottom()
    {
        scrollRect.verticalNormalizedPosition = 0f;
    }

    // 滚动到指定位置
    public void ScrollTo(float normalizedPosition)
    {
        scrollRect.verticalNormalizedPosition = Mathf.Clamp01(normalizedPosition);
    }

    // 平滑滚动到目标位置
    IEnumerator SmoothScrollTo(float target, float duration)
    {
        float start = scrollRect.verticalNormalizedPosition;
        float elapsed = 0f;
        while (elapsed < duration)
        {
            elapsed += Time.deltaTime;
            float t = elapsed / duration;
            // 使用缓动曲线
            t = Mathf.SmoothStep(0f, 1f, t);
            scrollRect.verticalNormalizedPosition = Mathf.Lerp(start, target, t);
            yield return null;
        }
        scrollRect.verticalNormalizedPosition = target;
    }
}
```

#### 动态创建 ScrollRect

```csharp
using UnityEngine;
using UnityEngine.UI;

public class DynamicScrollRectCreator : MonoBehaviour
{
    public Transform parent;

    void Start()
    {
        // ScrollRect
        GameObject go = new GameObject("ScrollView");
        go.transform.SetParent(parent, false);
        RectTransform rt = go.AddComponent<RectTransform>();
        rt.sizeDelta = new Vector2(400, 600);

        // Viewport
        GameObject viewport = new GameObject("Viewport");
        viewport.transform.SetParent(go.transform, false);
        RectTransform vpRt = viewport.AddComponent<RectTransform>();
        vpRt.anchorMin = Vector2.zero;
        vpRt.anchorMax = Vector2.one;
        vpRt.sizeDelta = Vector2.zero;
        Image vpImage = viewport.AddComponent<Image>();
        vpImage.color = Color.clear;
        Mask mask = viewport.AddComponent<Mask>();
        mask.showMaskGraphic = false; // 不显示 Mask 图形

        // Content
        GameObject content = new GameObject("Content");
        content.transform.SetParent(viewport.transform, false);
        RectTransform ctRt = content.AddComponent<RectTransform>();
        ctRt.anchorMin = new Vector2(0, 1);
        ctRt.anchorMax = new Vector2(0, 1);
        ctRt.pivot = new Vector2(0, 1);
        ctRt.sizeDelta = new Vector2(400, 2000); // 内容比视口高 → 可滚动
        ContentSizeFitter csf = content.AddComponent<ContentSizeFitter>();
        csf.verticalFit = ContentSizeFitter.FitMode.PreferredSize;

        // 添加 ScrollRect
        ScrollRect scrollRect = go.AddComponent<ScrollRect>();
        scrollRect.content = ctRt;
        scrollRect.viewport = vpRt;
        scrollRect.horizontal = false;
        scrollRect.vertical = true;
        scrollRect.movementType = ScrollRect.MovementType.Clamped;
    }
}
```

---

### 🔔 事件监听

#### 方式1：Lambda 表达式

```csharp
using UnityEngine;
using UnityEngine.UI;

public class ScrollRectEventLambda : MonoBehaviour
{
    public ScrollRect scrollRect;
    public Text positionText;

    void Start()
    {
        scrollRect.onValueChanged.AddListener((Vector2 pos) =>
        {
            positionText.text = string.Format("H:{0:F2} V:{1:F2}", pos.x, pos.y);
        });
    }
}
```

#### 方式2：检测滚动到顶部/底部

```csharp
using UnityEngine;
using UnityEngine.UI;

public class ScrollEndDetector : MonoBehaviour
{
    public ScrollRect scrollRect;
    public GameObject scrollToTopBtn; // 回顶部按钮

    void Start()
    {
        scrollRect.onValueChanged.AddListener((Vector2 pos) =>
        {
            // 接近底部（0.05 以内）
            if (pos.y <= 0.05f)
            {
                OnReachBottom();
            }

            // 不在顶部时显示回顶部按钮
            scrollToTopBtn.SetActive(pos.y < 0.95f);
        });
    }

    void OnReachBottom()
    {
        Debug.Log("已滚动到底部，加载更多...");
        // LoadMoreData();
    }
}
```

#### 方式3：代码停止滚动

```csharp
using UnityEngine;
using UnityEngine.UI;

public class ScrollRectStop : MonoBehaviour
{
    public ScrollRect scrollRect;

    // 立即停止滚动
    public void StopScrolling()
    {
        scrollRect.StopMovement(); // 停止惯性滚动
        scrollRect.velocity = Vector2.zero;
    }

    // 精确设置滚动位置
    public void SnapToPosition(float normalizedY)
    {
        scrollRect.velocity = Vector2.zero;
        scrollRect.verticalNormalizedPosition = normalizedY;
    }
}
```

---

### 🎯 ScrollRect 实战：动态列表

```csharp
using UnityEngine;
using UnityEngine.UI;

public class DynamicScrollList : MonoBehaviour
{
    public ScrollRect scrollRect;
    public GameObject itemPrefab;    // 列表项预制体
    public RectTransform content;    // Content 的 RectTransform
    private float itemHeight = 100f; // 每个 item 的高度

    void Start()
    {
        // 生成 50 个列表项
        string[] items = new string[50];
        for (int i = 0; i < 50; i++)
        {
            items[i] = "第 " + (i + 1) + " 项";
        }
        CreateList(items);
    }

    void CreateList(string[] items)
    {
        // 设置 Content 高度
        content.sizeDelta = new Vector2(content.sizeDelta.x, items.Length * itemHeight);

        for (int i = 0; i < items.Length; i++)
        {
            GameObject go = Instantiate(itemPrefab, content);
            go.name = "Item_" + i;

            // 设置位置（从上往下排列）
            RectTransform rt = go.GetComponent<RectTransform>();
            rt.anchorMin = new Vector2(0, 1);
            rt.anchorMax = new Vector2(1, 1);
            rt.pivot = new Vector2(0.5f, 1);
            rt.sizeDelta = new Vector2(0, itemHeight);
            rt.anchoredPosition = new Vector2(0, -i * itemHeight);

            // 设置文本
            Text txt = go.GetComponentInChildren<Text>();
            txt.text = items[i];
        }
    }

    // 清空列表
    public void ClearList()
    {
        foreach (Transform child in content)
        {
            Destroy(child.gameObject);
        }
    }
}
```

---

### 📊 ScrollRect 属性速查表

| 属性 | 代码 | 说明 |
|------|------|------|
| Content | `scrollRect.content = ct` | 内容容器的 RectTransform |
| Viewport | `scrollRect.viewport = vp` | 视口的 RectTransform |
| Horizontal | `scrollRect.horizontal = true` | 水平滚动开关 |
| Vertical | `scrollRect.vertical = false` | 垂直滚动开关 |
| Movement Type | `scrollRect.movementType = ScrollRect.MovementType.Elastic` | 移动类型 |
| Elasticity | `scrollRect.elasticity = 0.1f` | 弹性系数 |
| Inertia | `scrollRect.inertia = true` | 惯性开关 |
| Deceleration Rate | `scrollRect.decelerationRate = 0.135f` | 减速率 |
| Scroll Sensitivity | `scrollRect.scrollSensitivity = 30f` | 滚轮灵敏度 |
| Horizontal Scrollbar | `scrollRect.horizontalScrollbar = bar` | 水平滚动条 |
| Vertical Scrollbar | `scrollRect.verticalScrollbar = bar` | 垂直滚动条 |
| horizontalNormalizedPosition | `scrollRect.horizontalNormalizedPosition = 0.5f` | 水平归一化位置 |
| verticalNormalizedPosition | `scrollRect.verticalNormalizedPosition = 1f` | 垂直归一化位置 |
| Velocity | `scrollRect.velocity = Vector2.zero` | 滚动速度 |
| StopMovement() | `scrollRect.StopMovement()` | 立即停止滚动 |
| onValueChanged | `scrollRect.onValueChanged.AddListener((Vector2 p) => {...})` | 滚动事件 |

> [!NOTE] 命名空间
> `ScrollRect` 组件位于 `UnityEngine.UI` 命名空间。创建时通过 `GameObject > UI > Scroll View` 菜单自动生成完整结构。



---

## 📋 Dropdown 组件

> Dropdown 是 UGUI 中的**下拉菜单**控件，继承自 `Selectable` → `UIBehaviour`。点击后展开一个选项列表供用户选择，选中后自动收起。常见用途：难度选择、分辨率设置、语言切换、服务器选择等。

> [!IMPORTANT] Dropdown 的子物体结构
> Dropdown 由 **Label**（当前选中项的文字）、**Arrow**（下拉箭头图标）、**Template**（下拉列表模板，展开时显示）三部分组成。Template 内部包含一个带 ScrollRect 的列表。

---

### 📋 Inspector 参数详解

#### Interactable / Transition / Navigation

> 可交互、过渡效果、导航设置，与其他 Selectable 组件一致。

---

#### Template（下拉模板）

| 说明 | 详情 |
|------|------|
| **类型** | `RectTransform` |
| **作用** | 指向下拉列表模板的 RectTransform（默认隐藏，点击时显示） |
| **代码** | `dropdown.template = templateRectTransform;` |

#### Caption Text（标题文本）

| 说明 | 详情 |
|------|------|
| **类型** | `Text` |
| **作用** | 显示当前选中项的文字（子物体 Label） |
| **代码** | `dropdown.captionText = labelText;` |

#### Caption Image（标题图片）

| 说明 | 详情 |
|------|------|
| **类型** | `Image` |
| **作用** | 显示当前选中项的图片（可选，通常用于带图标的选项） |
| **代码** | `dropdown.captionImage = labelImage;` |

#### Item Text（选项文本）

| 说明 | 详情 |
|------|------|
| **类型** | `Text` |
| **作用** | Template 中每个选项的默认文本组件，用于模板克隆时设置文字 |
| **代码** | `dropdown.itemText = itemTextPrefab;` |

#### Item Image（选项图片）

| 说明 | 详情 |
|------|------|
| **类型** | `Image` |
| **作用** | Template 中每个选项的默认图片组件（可选） |
| **代码** | `dropdown.itemImage = itemImagePrefab;` |

---

#### Options（选项列表）⭐

> ==Dropdown 的核心数据==。每个选项包含一个文本和可选图片。

| 参数 | 说明 |
|------|------|
| **Text** | 选项文字 |
| **Image** | 选项图片（可选） |

```csharp
// 添加选项
dropdown.options.Add(new Dropdown.OptionData("选项A"));
dropdown.options.Add(new Dropdown.OptionData("选项B", iconSprite));
// 清空选项
dropdown.options.Clear();
```

#### Value（当前选中索引）

| 说明 | 详情 |
|------|------|
| **类型** | `int` |
| **作用** | 当前选中选项的索引（从0开始）。-1 表示无选中 |
| **代码** | `dropdown.value = 2;` |

#### Alpha Fade Speed（透明过渡速度）

| 说明 | 详情 |
|------|------|
| **类型** | `float` |
| **作用** | 下拉列表展开/收起的透明度动画速度。默认 0.15 |
| **代码** | `dropdown.alphaFadeSpeed = 0.2f;` |

---

#### On Value Changed（值变化事件）⭐

| 说明 | 详情 |
|------|------|
| **类型** | `UnityEvent<int>` |
| **参数** | `int` — 当前选中项的索引 |
| **触发时机** | 用户从下拉列表中选择时（代码设置 `value` 也会触发） |
| **代码** | `dropdown.onValueChanged.AddListener((int index) => { ... });` |

---

### 💻 代码控制方式

#### 基本属性控制

```csharp
using UnityEngine;
using UnityEngine.UI;

public class DropdownController : MonoBehaviour
{
    public Dropdown dropdown;
    public Text label;

    void Start()
    {
        // 1. 获取/设置选中索引
        int currentIndex = dropdown.value;
        dropdown.value = 2;

        // 2. 获取选中项的文本
        string selectedText = dropdown.options[dropdown.value].text;
        // 或通过 captionText
        string captionText = dropdown.captionText.text;

        // 3. 关联组件
        dropdown.captionText = label;
        dropdown.captionImage = label.GetComponent<Image>();

        // 4. 透明过渡速度
        dropdown.alphaFadeSpeed = 0.15f;

        // 5. 可交互
        dropdown.interactable = true;
    }
}
```

#### 选项列表操作（增删改查）

```csharp
using UnityEngine;
using UnityEngine.UI;
using System.Collections.Generic;

public class DropdownOptionsManager : MonoBehaviour
{
    public Dropdown dropdown;

    void Start()
    {
        // 清空所有选项
        dropdown.options.Clear();

        // 添加单个选项
        dropdown.options.Add(new Dropdown.OptionData("简单"));
        dropdown.options.Add(new Dropdown.OptionData("普通"));
        dropdown.options.Add(new Dropdown.OptionData("困难"));
        dropdown.options.Add(new Dropdown.OptionData("地狱"));

        // 批量添加
        List<Dropdown.OptionData> options = new List<Dropdown.OptionData>
        {
            new Dropdown.OptionData("选项1"),
            new Dropdown.OptionData("选项2"),
            new Dropdown.OptionData("选项3")
        };
        dropdown.AddOptions(options);

        // 移除指定选项
        dropdown.options.RemoveAt(0); // 移除第一个

        // 修改指定选项文字
        dropdown.options[1].text = "修改后的文字";

        // 获取选项数量
        int count = dropdown.options.Count;

        // 刷新显示（修改 options 后通常自动更新，但有时需要手动）
        dropdown.RefreshShownValue();
    }
}
```

#### 动态创建 Dropdown

```csharp
using UnityEngine;
using UnityEngine.UI;

public class DynamicDropdownCreator : MonoBehaviour
{
    public Transform parent;

    void Start()
    {
        GameObject go = new GameObject("DynamicDropdown");
        go.transform.SetParent(parent, false);
        RectTransform rt = go.AddComponent<RectTransform>();
        rt.sizeDelta = new Vector2(200, 30);

        // Label
        GameObject labelGo = new GameObject("Label");
        labelGo.transform.SetParent(go.transform, false);
        Text labelText = labelGo.AddComponent<Text>();
        labelText.font = Resources.GetBuiltinResource<Font>("LegacyRuntime.ttf");
        labelText.fontSize = 16;
        labelText.alignment = TextAnchor.MiddleLeft;
        labelText.raycastTarget = false;
        RectTransform labelRt = labelGo.GetComponent<RectTransform>();
        labelRt.anchorMin = Vector2.zero;
        labelRt.anchorMax = Vector2.one;
        labelRt.offsetMin = new Vector2(10, 0);
        labelRt.offsetMax = new Vector2(-30, 0);

        // Arrow
        GameObject arrowGo = new GameObject("Arrow");
        arrowGo.transform.SetParent(go.transform, false);
        Text arrowText = arrowGo.AddComponent<Text>();
        arrowText.text = "▼";
        arrowText.font = Resources.GetBuiltinResource<Font>("LegacyRuntime.ttf");
        arrowText.fontSize = 12;
        arrowText.alignment = TextAnchor.MiddleCenter;
        arrowText.raycastTarget = false;
        RectTransform arrowRt = arrowGo.GetComponent<RectTransform>();
        arrowRt.anchorMin = new Vector2(1, 0.5f);
        arrowRt.anchorMax = new Vector2(1, 0.5f);
        arrowRt.pivot = new Vector2(1, 0.5f);
        arrowRt.sizeDelta = new Vector2(20, 20);
        arrowRt.anchoredPosition = new Vector2(-5, 0);

        // Template（简化版，通常用预制体创建较为合适）
        GameObject templateGo = new GameObject("Template");
        templateGo.transform.SetParent(go.transform, false);
        // ... Template 内部结构较复杂，建议使用预制体创建

        // Image（背景）
        Image bgImage = go.AddComponent<Image>();
        bgImage.color = Color.white;

        // Dropdown
        Dropdown dropdown = go.AddComponent<Dropdown>();
        dropdown.targetGraphic = bgImage;
        dropdown.captionText = labelText;
        dropdown.options.Add(new Dropdown.OptionData("选项1"));
        dropdown.options.Add(new Dropdown.OptionData("选项2"));
        dropdown.options.Add(new Dropdown.OptionData("选项3"));
        dropdown.RefreshShownValue();
    }
}
```

---

### 🔔 事件监听

#### 方式1：Lambda 表达式

```csharp
using UnityEngine;
using UnityEngine.UI;

public class DropdownEventLambda : MonoBehaviour
{
    public Dropdown dropdown;

    void Start()
    {
        dropdown.onValueChanged.AddListener((int index) =>
        {
            string selected = dropdown.options[index].text;
            Debug.Log("选中: " + selected + " (索引: " + index + ")");
        });
    }
}
```

#### 方式2：直接绑定方法

```csharp
using UnityEngine;
using UnityEngine.UI;

public class DropdownEventMethod : MonoBehaviour
{
    public Dropdown qualityDropdown;
    public Dropdown resolutionDropdown;

    void Start()
    {
        qualityDropdown.onValueChanged.AddListener(OnQualityChanged);
        resolutionDropdown.onValueChanged.AddListener(OnResolutionChanged);

        // 加载已保存的设置
        qualityDropdown.value = PlayerPrefs.GetInt("Quality", 2);
    }

    void OnQualityChanged(int index)
    {
        QualitySettings.SetQualityLevel(index);
        PlayerPrefs.SetInt("Quality", index);
    }

    void OnResolutionChanged(int index)
    {
        Debug.Log("切换到分辨率索引: " + index);
        // Screen.SetResolution(widths[index], heights[index], Screen.fullScreen);
    }

    void OnDestroy()
    {
        qualityDropdown.onValueChanged.RemoveListener(OnQualityChanged);
        resolutionDropdown.onValueChanged.RemoveListener(OnResolutionChanged);
    }
}
```

#### 方式3：代码设置值不触发事件

```csharp
using UnityEngine;
using UnityEngine.UI;

public class DropdownSilentSet : MonoBehaviour
{
    public Dropdown dropdown;

    void Start()
    {
        dropdown.onValueChanged.AddListener(OnValueChanged);

        // 方法1：先移除再恢复
        dropdown.onValueChanged.RemoveListener(OnValueChanged);
        dropdown.value = 2;
        dropdown.onValueChanged.AddListener(OnValueChanged);

        // 方法2：用标记位
        SetValueSilent(2);
    }

    private bool ignoreEvents = false;
    void SetValueSilent(int value)
    {
        ignoreEvents = true;
        dropdown.value = value;
        ignoreEvents = false;
    }

    void OnValueChanged(int index)
    {
        if (ignoreEvents) return;
        Debug.Log("选中索引: " + index);
    }
}
```

---

### 🎯 Dropdown 实战：设置面板

```csharp
using UnityEngine;
using UnityEngine.UI;
using System.Collections.Generic;

public class SettingsDropdown : MonoBehaviour
{
    public Dropdown qualityDropdown;
    public Dropdown resolutionDropdown;
    public Dropdown languageDropdown;

    void Start()
    {
        SetupQualityDropdown();
        SetupResolutionDropdown();
        SetupLanguageDropdown();
    }

    void SetupQualityDropdown()
    {
        qualityDropdown.options.Clear();
        string[] qualities = QualitySettings.names; // 获取系统质量等级名称
        foreach (string q in qualities)
        {
            qualityDropdown.options.Add(new Dropdown.OptionData(q));
        }
        // 还原保存的设置
        qualityDropdown.value = PlayerPrefs.GetInt("Quality", qualities.Length - 1);
        qualityDropdown.RefreshShownValue();

        qualityDropdown.onValueChanged.AddListener((int index) =>
        {
            QualitySettings.SetQualityLevel(index);
            PlayerPrefs.SetInt("Quality", index);
        });
    }

    void SetupResolutionDropdown()
    {
        resolutionDropdown.options.Clear();
        Resolution[] resolutions = Screen.resolutions;
        // 去重
        HashSet<string> uniqueRes = new HashSet<string>();
        foreach (Resolution res in resolutions)
        {
            string resStr = res.width + "×" + res.height;
            if (uniqueRes.Add(resStr))
            {
                resolutionDropdown.options.Add(new Dropdown.OptionData(resStr));
            }
        }
        resolutionDropdown.RefreshShownValue();
    }

    void SetupLanguageDropdown()
    {
        languageDropdown.options.Clear();
        languageDropdown.options.Add(new Dropdown.OptionData("简体中文"));
        languageDropdown.options.Add(new Dropdown.OptionData("English"));
        languageDropdown.options.Add(new Dropdown.OptionData("日本語"));

        languageDropdown.onValueChanged.AddListener((int index) =>
        {
            string lang = languageDropdown.options[index].text;
            Debug.Log("切换语言: " + lang);
            // LocalizationManager.Instance.SetLanguage(lang);
        });
    }
}
```

---

### 📊 Dropdown 属性速查表

| 属性 | 代码 | 说明 |
|------|------|------|
| Value | `dropdown.value = 2` | 当前选中索引 |
| Options | `dropdown.options.Add(new OptionData("x"))` | 添加选项 |
| Options Clear | `dropdown.options.Clear()` | 清空所有选项 |
| Caption Text | `dropdown.captionText = txt` | 标题文本组件 |
| Caption Image | `dropdown.captionImage = img` | 标题图片组件 |
| Item Text | `dropdown.itemText = txt` | 选项文本模板 |
| Item Image | `dropdown.itemImage = img` | 选项图片模板 |
| Template | `dropdown.template = tpl` | 下拉模板 RectTransform |
| Alpha Fade Speed | `dropdown.alphaFadeSpeed = 0.2f` | 展开/收起透明度速度 |
| Interactable | `dropdown.interactable = false` | 可交互开关 |
| AddOptions | `dropdown.AddOptions(list)` | 批量添加选项 |
| RefreshShownValue | `dropdown.RefreshShownValue()` | 刷新显示 |
| onValueChanged | `dropdown.onValueChanged.AddListener((int i) => {...})` | 值变化事件 |

> [!NOTE] 命名空间
> `Dropdown` 组件位于 `UnityEngine.UI` 命名空间。`Dropdown.OptionData` 用于构造每个选项的数据。


---

## 🗂️ Sprite Atlas（精灵图集）

> Sprite Atlas 是 Unity 提供的**精灵打包工具**，将多个小图合并到一张大纹理中，==通过减少 DrawCall 来大幅提升 UI 渲染性能==。是 UGUI 项目优化的核心技术之一。

> [!IMPORTANT] 为什么要打图集？
> UGUI 的合批（Batching）规则：==使用同一张纹理的 Image 可以合并为一个 DrawCall==。如果不打图集，每个散图都是一个独立纹理 → 每个 Image 各占一个 DrawCall → 性能灾难。打完图集后，图集内的所有 Sprite 共享一张纹理 → 可合并为一个 DrawCall → 性能大幅提升。

---

### 🎯 图集的核心作用

| 作用 | 说明 |
|------|------|
| **减少 DrawCall** | 同一图集的 UI 元素合并批处理，从 N 个 DrawCall 降到 1 个 |
| **减少内存碎片** | 散图各自占用内存，图集统一管理一张大纹理 |
| **提升加载效率** | 一次加载图集 = 加载所有子图，减少 IO 次数 |
| **便于版本管理** | 图集统一打包，支持变体（Variants），多分辨率适配 |

---

### 📐 图集合批机制（重点理解）

```
未打图集：
┌─────────┐ ┌─────────┐ ┌─────────┐ ┌─────────┐
│ 按钮.png │ │ 图标.png │ │ 边框.png │ │ 背景.png │  → 4 个 DrawCall
└─────────┘ └─────────┘ └─────────┘ └─────────┘

打完图集（UI Atlas）：
┌──────────────────────────────────┐
│  按钮   图标                     │
│                                  │
│  边框   背景                     │  → 1 个 DrawCall 🚀
└──────────────────────────────────┘
```

> [!NOTE] 合批条件
> 要成功合批，不仅需要同一纹理，还必须满足：同一材质、渲染顺序连续（Hierarchy 中相邻）、未被其他图集/纹理的 Image 打断。

---

### 🔧 创建与配置图集

#### 步骤1：安装 Sprite Atlas 包（Unity 2020+）

`Window → Package Manager → 搜索 "2D Sprite" → 安装`

#### 步骤2：创建 Sprite Atlas 资源

`Project 窗口右键 → Create → 2D → Sprite Atlas`

#### 步骤3：添加打包对象

> 在 Sprite Atlas 的 Inspector 中配置。

| 参数 | 说明 |
|------|------|
| **Objects for Packing** | ==要打包的 Sprite 列表==。可以直接拖入 Sprite 资源，也可以拖入包含 Sprite 的文件夹 |
| **Include in Build** | 是否包含在构建中。勾选后打包出的图集会随项目发布 |
| **Tight Packing** | 是否紧密打包。勾选后使用 Sprite 的实际像素边界打包（而不是整张图），更省空间 |
| **Allow Rotation** | 是否允许旋转 Sprite 来优化打包空间（打包时自动旋转，运行时自动还原） |
| **Padding** | 每个 Sprite 之间的间距（像素）。默认 4，防止采样时边缘溢出 |
| **Alpha Dilation** | 建议勾选，防止半透明边缘出现黑边 |

---

#### 关键参数详解

| 参数 | 推荐值 | 说明 |
|------|:--:|------|
| **Tight Packing** | ✅ 勾选 | 节省图集空间，按实际像素打包 |
| **Allow Rotation** | ✅ 勾选 | 让打包算法有更多排列可能 |
| **Padding** | 4~8 | 太小会出现边缘溢出，太大浪费空间 |
| **Alpha Dilation** | ✅ 勾选 | 防止透明边缘黑线 |

#### 步骤4：设置图集纹理格式

> 在图集 Inspector 中展开 "Platform Settings" 进行设置。也可以批量替换项目中的纹理设置。

| 参数 | 推荐值 | 说明 |
|------|:--:|------|
| **Max Texture Size** | 2048 或 4096 | 图集最大尺寸，超出的 Sprite 会分到另一张图集 |
| **Format** | RGBA Compressed DXT5 / ASTC | 根据平台选择合适的压缩格式 |
| **Compression Quality** | 正常 | 压缩质量 |
| **Generate Mip Maps** | ❌ 不勾选 | ==UI 图集不要生成 MipMap==，浪费内存 |
| **Filter Mode** | Bilinear | 双线性过滤（UI 不需要 Point） |
| **sRGB** | ✅ 勾选 | UI 纹理必须勾选 sRGB |

> [!WARNING] UI 图集的 MipMap
> ==UI 图集绝对不要勾选 Generate Mip Maps！== MipMap 会额外增加约 33% 内存占用，而 UI 始终显示在屏幕上，MipMap 等级永远不会被使用。

---

### 📦 图集打包模式

#### 模式1：Include in Build（构建时打包）

> 最常用的模式。编辑器中通过 `SpriteAtlasManager` 实时加载，构建时自动打包。

```csharp
// 确保 SpriteAtlasManager 在图集未准备好时不返回 null
// Edit → Project Settings → Editor → Sprite Atlas → 
//   Mode: "Always Enabled" (推荐)
//   或 "Enabled For Builds"
```

#### 模式2：Late Binding（延迟绑定）

> Sprite 不直接关联图集，运行时通过代码加载。（现在较少使用）

#### 模式3：变体图集（Variant）

> 用于多分辨率适配。创建一个 Master Atlas + 多个 Variant Atlas。

```
Master Atlas（原始分辨率，1024×1024）
├── SD Variant（0.5 倍分辨率，512×512）
├── HD Variant（1.5 倍，1536×1536）
└── 4K Variant（2 倍，2048×2048）
```

---

### 💻 代码控制方式

#### 加载与注册图集

```csharp
using UnityEngine;
using UnityEngine.U2D; // SpriteAtlas 命名空间
using UnityEngine.UI;

public class AtlasLoader : MonoBehaviour
{
    public Image image;

    // 方式1：拖拽 SpriteAtlas 引用到 Inspector
    public SpriteAtlas atlas;

    void Start()
    {
        // 从图集中获取 Sprite
        Sprite sprite = atlas.GetSprite("icon_001");
        // 注意：这里的名字是打包前的 Sprite 文件名，不是图集名
        if (sprite != null)
        {
            image.sprite = sprite;
        }
    }
}
```

#### 运行时加载图集

```csharp
using UnityEngine;
using UnityEngine.U2D;
using UnityEngine.UI;

public class RuntimeAtlasLoader : MonoBehaviour
{
    public Image[] images;

    void Start()
    {
        // 从 Resources 加载图集
        SpriteAtlas atlas = Resources.Load<SpriteAtlas>("Atlases/UIAtlas");

        // 从图集中获取所有子 Sprite
        Sprite[] sprites = new Sprite[atlas.spriteCount];
        atlas.GetSprites(sprites);

        // 按名称查找指定 Sprite
        Sprite btnSprite = atlas.GetSprite("btn_start");
        Sprite iconSprite = atlas.GetSprite("icon_gold");
        Sprite frameSprite = atlas.GetSprite("frame_dialog");
    }
}
```

#### 资产管理（AssetDatabase 编辑器脚本）

```csharp
#if UNITY_EDITOR
using UnityEngine;
using UnityEditor;
using UnityEngine.U2D;

public class AtlasEditorHelper : EditorWindow
{
    [MenuItem("Tools/Atlas/Print Atlas Info")]
    static void PrintAtlasInfo()
    {
        // 查找项目中所有 SpriteAtlas
        string[] guids = AssetDatabase.FindAssets("t:SpriteAtlas");
        
        foreach (string guid in guids)
        {
            string path = AssetDatabase.GUIDToAssetPath(guid);
            SpriteAtlas atlas = AssetDatabase.LoadAssetAtPath<SpriteAtlas>(path);
            
            Debug.Log(string.Format("图集: {0} | 子图数量: {1} | 格式: {2}",
                atlas.name,
                atlas.spriteCount,
                atlas.GetTexture().format));
        }
    }
}
#endif
```

---

### 🔍 验证合批效果

> 运行时打开 `Window → Analysis → Frame Debugger`，点击 Enable，在 Game 视图中观察 DrawCall。

```
Frame Debugger 中：
✅ BATCHED（绿色高亮）= 合批成功
❌ 不连续的 Image = 合批被打断
```

| 合批被打断的常见原因 | 解决方案 |
|------|------|
| 不同图集的 Image 交替排列 | 调整 Hierarchy 顺序，同图集的放一起 |
| Image → Text → Image | Text 打断了合批（材质不同），中间调整排序 |
| RawImage 插入 | RawImage 始终独立 DC，无法合批 |
| 修改了 Material 属性 | 确保使用相同材质或默认材质 |

---

### 📊 图集与传统散图对比

| 对比维度 | 散图模式 | 图集模式 |
|------|:--:|:--:|
| DrawCall（10个Image） | 10 个 | 1 个 |
| 纹理数量 | N 张 | 1 张（打包后） |
| 内存占用 | 各图独立占用 | 一张大图（可能含空白区域） |
| 加载次数 | N 次 IO | 1 次 IO |
| 维护成本 | 散图各自管理 | 统一图集管理 |
| 动态增删 | 任意增删无影响 | 需重新打包 |

---

### 🎯 最佳实践

> [!TIP] 图集设计原则
> - **按功能模块分图集**：登录图集、背包图集、商店图集……同屏显示多的放一起
> - **按 UI 界面对图集分组**：同一界面中的 UI 元素尽量放一个图集
> - **公共资源集中打包**：常驻内存的通用 UI 放一个图集
> - **图集不要太大**：控制在 2048×2048 以内，避免单张纹理过大
> - **不活跃的图集卸载**：切换场景后 `Resources.UnloadUnusedAssets()` 释放不用的图集

> [!NOTE] 命名空间
> `SpriteAtlas` 位于 `UnityEngine.U2D` 命名空间，代码中使用前需要 `using UnityEngine.U2D;`。`SpriteAtlasManager` 在同一命名空间下，管理图集的全局注册。



---

## 📡 UI 事件监听接口

> Unity 的 UI 事件系统（EventSystem）提供了一套**接口机制**来处理 UI 交互事件。任何实现了这些接口的脚本挂载到带 `Graphic`（Image/Text 等）组件的 GameObject 上，即可自动接收对应的 UI 事件。

> [!IMPORTANT] 核心原理
> EventSystem 检测到输入 → GraphicRaycaster 射线检测命中 UI → 在目标对象上查找实现了事件接口的组件 → 调用对应接口方法。==不需要手动添加监听器，实现接口即可自动接收事件。==

---

### 📋 事件接口分类与速查

#### 指针类（Pointer）

> 最常用的一组，处理鼠标/触屏的点击和悬停。

| 接口 | 方法 | 触发时机 |
|------|------|------|
| `IPointerEnterHandler` | `OnPointerEnter(PointerEventData)` | 指针进入 UI 区域时 |
| `IPointerExitHandler` | `OnPointerExit(PointerEventData)` | 指针离开 UI 区域时 |
| `IPointerDownHandler` | `OnPointerDown(PointerEventData)` | 指针按下时（鼠标左键按下瞬间） |
| `IPointerUpHandler` | `OnPointerUp(PointerEventData)` | 指针抬起时（鼠标左键松开瞬间） |
| `IPointerClickHandler` | `OnPointerClick(PointerEventData)` | 完整点击时（Down + Up 在同一对象上） |

#### 拖拽类（Drag & Drop）

> 处理拖拽操作，通常配合使用。

| 接口 | 方法 | 触发时机 |
|------|------|------|
| `IBeginDragHandler` | `OnBeginDrag(PointerEventData)` | 开始拖拽时（按下后移动超过 DragThreshold） |
| `IDragHandler` | `OnDrag(PointerEventData)` | ==拖拽过程中每帧触发== |
| `IEndDragHandler` | `OnEndDrag(PointerEventData)` | 拖拽结束时（松手） |
| `IDropHandler` | `OnDrop(PointerEventData)` | 其他拖拽对象在自身松开时 |
| `IInitializePotentialDragHandler` | `OnInitializePotentialDrag(PointerEventData)` | 准备开始拖拽前（按下时） |

#### 滚动 / 移动 / 提交类

| 接口 | 方法 | 触发时机 |
|------|------|------|
| `IScrollHandler` | `OnScroll(PointerEventData)` | 鼠标滚轮滚动时 |
| `IMoveHandler` | `OnMove(AxisEventData)` | 键盘方向键导航时 |
| `ISubmitHandler` | `OnSubmit(BaseEventData)` | 按回车/提交键时（当前选中的对象） |
| `ICancelHandler` | `OnCancel(BaseEventData)` | 按 Esc/取消键时（当前选中的对象） |

#### 选中类（Select）

| 接口 | 方法 | 触发时机 |
|------|------|------|
| `ISelectHandler` | `OnSelect(BaseEventData)` | 被选中时（导航切换或代码 Select()） |
| `IDeselectHandler` | `OnDeselect(BaseEventData)` | 取消选中时 |
| `IUpdateSelectedHandler` | `OnUpdateSelected(BaseEventData)` | 选中状态下每帧触发 |

---

### 💻 使用方式

#### 方式1：单个脚本实现多个接口

```csharp
using UnityEngine;
using UnityEngine.EventSystems; // 事件接口的命名空间

public class UIEventHandler : MonoBehaviour,
    IPointerEnterHandler,
    IPointerExitHandler,
    IPointerDownHandler,
    IPointerUpHandler,
    IPointerClickHandler
{
    // 指针进入
    public void OnPointerEnter(PointerEventData eventData)
    {
        Debug.Log("鼠标进入 UI");
        GetComponent<Image>().color = Color.yellow;
    }

    // 指针离开
    public void OnPointerExit(PointerEventData eventData)
    {
        Debug.Log("鼠标离开 UI");
        GetComponent<Image>().color = Color.white;
    }

    // 按下
    public void OnPointerDown(PointerEventData eventData)
    {
        Debug.Log("按下: " + eventData.position);
        transform.localScale = Vector3.one * 0.9f;
    }

    // 抬起
    public void OnPointerUp(PointerEventData eventData)
    {
        Debug.Log("抬起");
        transform.localScale = Vector3.one;
    }

    // 点击（Down + Up 在同一对象上才算）
    public void OnPointerClick(PointerEventData eventData)
    {
        Debug.Log("点击完成! 按键: " + eventData.button); // 0=左键 1=右键 2=中键
    }
}
```

> [!IMPORTANT] PointerClick vs PointerDown
> `OnPointerClick` 只有 Down 和 Up 都在**同一个对象**上才会触发。如果按下去后拖走了再松手，不会触发 Click。

#### 方式2：拖拽系统（完整示例）

```csharp
using UnityEngine;
using UnityEngine.EventSystems;

public class DragHandler : MonoBehaviour,
    IBeginDragHandler,
    IDragHandler,
    IEndDragHandler
{
    private RectTransform rectTransform;
    private Canvas canvas;
    private Vector2 originalPosition;

    void Awake()
    {
        rectTransform = GetComponent<RectTransform>();
        canvas = GetComponentInParent<Canvas>();
    }

    public void OnBeginDrag(PointerEventData eventData)
    {
        originalPosition = rectTransform.anchoredPosition;
        Debug.Log("开始拖拽: " + originalPosition);
    }

    public void OnDrag(PointerEventData eventData)
    {
        Vector2 delta = eventData.delta / canvas.scaleFactor;
        rectTransform.anchoredPosition += delta;
    }

    public void OnEndDrag(PointerEventData eventData)
    {
        Debug.Log("拖拽结束: " + originalPosition + " → " + rectTransform.anchoredPosition);
    }
}
```

#### 方式3：放置系统（Drop 接收）⭐

```csharp
using UnityEngine;
using UnityEngine.EventSystems;
using UnityEngine.UI;

// 拖拽的物体
public class Draggable : MonoBehaviour, IBeginDragHandler, IDragHandler, IEndDragHandler
{
    private CanvasGroup canvasGroup;

    void Awake()
    {
        canvasGroup = GetComponent<CanvasGroup>();
    }

    public void OnBeginDrag(PointerEventData eventData)
    {
        canvasGroup.alpha = 0.6f;
        canvasGroup.blocksRaycasts = false; // ⚠️ 关键：关闭射线让鼠标穿过
    }

    public void OnDrag(PointerEventData eventData)
    {
        transform.position = eventData.position;
    }

    public void OnEndDrag(PointerEventData eventData)
    {
        canvasGroup.alpha = 1f;
        canvasGroup.blocksRaycasts = true;
    }
}

// 接收放置的目标（背包格子等）
public class DropZone : MonoBehaviour, IDropHandler, IPointerEnterHandler, IPointerExitHandler
{
    public Image highlightImage;

    public void OnPointerEnter(PointerEventData eventData)
    {
        if (eventData.pointerDrag != null)
            highlightImage.enabled = true;
    }

    public void OnPointerExit(PointerEventData eventData)
    {
        highlightImage.enabled = false;
    }

    public void OnDrop(PointerEventData eventData)
    {
        GameObject dropped = eventData.pointerDrag;
        if (dropped != null)
        {
            dropped.transform.SetParent(transform);
            Debug.Log("接收到: " + dropped.name);
        }
        highlightImage.enabled = false;
    }
}
```

#### 方式4：滚轮缩放

```csharp
using UnityEngine;
using UnityEngine.EventSystems;

public class ScrollZoomHandler : MonoBehaviour, IScrollHandler
{
    public RectTransform target;
    public float zoomSpeed = 0.1f;
    public float minScale = 0.5f;
    public float maxScale = 2f;

    public void OnScroll(PointerEventData eventData)
    {
        float delta = -eventData.scrollDelta.y * zoomSpeed;
        Vector3 newScale = target.localScale + Vector3.one * delta;
        newScale = Vector3.Max(newScale, Vector3.one * minScale);
        newScale = Vector3.Min(newScale, Vector3.one * maxScale);
        target.localScale = newScale;
    }
}
```

#### 方式5：选中和提交（键盘/手柄操作）

```csharp
using UnityEngine;
using UnityEngine.EventSystems;
using UnityEngine.UI;

public class SelectableItem : MonoBehaviour,
    ISelectHandler, IDeselectHandler, ISubmitHandler, ICancelHandler
{
    public Image frameImage;

    public void OnSelect(BaseEventData eventData)
    {
        frameImage.enabled = true;
    }

    public void OnDeselect(BaseEventData eventData)
    {
        frameImage.enabled = false;
    }

    public void OnSubmit(BaseEventData eventData)
    {
        Debug.Log(gameObject.name + " 确认（回车/A键）");
    }

    public void OnCancel(BaseEventData eventData)
    {
        Debug.Log(gameObject.name + " 取消（Esc/B键）");
    }
}
```

---

### 🔄 两种实现方式对比

| 对比维度 | 接口实现（推荐） | EventTrigger 组件 |
|------|:--:|:--:|
| **灵活性** | ⭐⭐⭐ 代码完全控制 | ⭐⭐ Inspector 绑定有限制 |
| **代码量** | 需要写完整类 | 可不写代码 |
| **复用性** | ✅ 高（可继承复用） | ❌ 低（每个对象单独配置） |
| **性能** | ⭐⭐⭐ 好 | ⭐⭐ 略差（额外组件开销） |
| **复杂逻辑** | ✅ 适合拖拽等多步骤交互 | ❌ 不适合 |

#### EventTrigger 方式（备选）⭐

```
步骤：
1. 给 GameObject 添加 EventTrigger 组件
2. 点击 "Add New Event Type"
3. 选择事件类型（PointerEnter / PointerClick 等）
4. 拖入目标对象 + 选择 public 方法（带 BaseEventData 参数）
```

> [!TIP] 选择建议
> - **复杂交互**（拖拽、多层嵌套）→ **接口实现**
> - **简单回调**（原型、一次性点击）→ **EventTrigger** 或 Button.onClick
> - **正式项目** → 优先使用**接口实现**

---

### 📊 EventData 参数详解

> UI 事件回调的参数类型有三种：`BaseEventData`（基类）、`PointerEventData`（指针/鼠标/触屏事件）、`AxisEventData`（键盘导航事件）。不同接口接收不同的事件数据类型。

---

#### 类型继承关系

```
BaseEventData (基类)
├── PointerEventData (指针事件)
│   └── 用于所有 IPointer* / IDrag* / IDrop / IScroll 接口
└── AxisEventData (轴事件)
    └── 用于 IMoveHandler 接口
```

---

#### 一、PointerEventData（指针事件数据）⭐

> 所有 Pointer / Drag / Drop / Scroll 接口的回调参数，携带鼠标/触屏的所有信息。

##### 位置相关

| 属性 | 类型 | 说明 |
|------|------|------|
| `position` | `Vector2` | 指针当前屏幕像素坐标（左下角为原点） |
| `delta` | `Vector2` | ==指针自上一帧到当前位置的位移量==。拖拽的核心计算数据：`rectTransform.position += eventData.delta` |
| `pressPosition` | `Vector2` | 按下时的初始位置。用于判断拖拽距离：`Vector2.Distance(pressPosition, position) > threshold` |
| `worldPosition` | `Vector3` | 指针在世界空间中的位置（3D 场景用） |
| `worldNormal` | `Vector3` | 指针在世界空间中的法线方向（3D 场景用） |

##### 输入状态

| 属性 | 类型 | 说明 |
|------|------|------|
| `button` | `PointerEventData.InputButton` | ==按下的鼠标按键==：`Left=0` / `Right=1` / `Middle=2` |
| `clickCount` | `int` | 连击次数。双击时 clickCount=2，三连击=3 |
| `clickTime` | `float` | 上一次点击到此点击的时间间隔（秒） |
| `eligibleForClick` | `bool` | 当前是否满足点击条件（Down+Up 在同对象上） |
| `scrollDelta` | `Vector2` | 鼠标滚轮滚动量。`y > 0` 向上滚(放大)，`y < 0` 向下滚(缩小)。==通常在 IScrollHandler 中使用== |
| `pressure` | `float` | 触屏的按压力度（0~1），鼠标始终为 1 或 0 |
| `twist` | `float` | 触屏双指旋转角度（弧度） |
| `tilt` | `Vector2` | 触屏笔的倾斜角度 |
| `radius` | `Vector2` | 触屏触摸半径（用于判断手指覆盖面积） |

##### 拖拽相关

| 属性 | 类型 | 说明 |
|------|------|------|
| `pointerDrag` | `GameObject` | ==当前正在被拖拽的对象==。在 IDropHandler 中用 `eventData.pointerDrag` 获取拖拽物 |
| `dragging` | `bool` | 是否正在拖拽中（点击并移动超过 DragThreshold 后为 true） |
| `pointerId` | `int` | 指针的唯一 ID。触屏多点触摸时区分手指：-1=左键，-2=右键，-3=中键，≥0=触屏手指 |

##### 射线检测与穿透

| 属性 | 类型 | 说明 |
|------|------|------|
| `pointerEnter` | `GameObject` | 指针当前悬停/进入的对象。配合 IPointerEnter/Exit 使用 |
| `pointerPress` | `GameObject` | ==按下时命中的对象==（按下瞬间射线检测到的对象） |
| `pointerCurrentRaycast` | `RaycastResult` | 当前帧的射线检测结果，包含命中的 GameObject、距离、排序层等信息 |
| `pointerPressRaycast` | `RaycastResult` | 按下瞬间的射线检测结果 |
| `hovered` | `List<GameObject>` | ==当前指针悬停的所有对象列表==（可穿透多层 UI） |
| `rawPointerPress` | `GameObject` | 原始按下对象（不考虑穿透等处理） |
| `useDragThreshold` | `bool` | 是否使用拖拽阈值。设为 false 可让拖拽立即生效（无 Dead Zone） |
| `fullyExited` | `bool` | 指针是否完全退出（与 pointerEnter 配对使用，处理嵌套 UI） |
| `reentered` | `bool` | 指针是否重新进入（同上，处理从子对象返回到父对象的场景） |

##### 事件控制

| 属性 | 类型 | 说明 |
|------|------|------|
| `used` | `bool` | ==事件是否已被消费==。设为 true 后，后续对象不会再收到此事件。常用于阻止事件穿透 |
| **Use()** | `void` | 快捷方法：`eventData.Use()` = `eventData.used = true` |

```csharp
// 阻止点击穿透示例
public void OnPointerClick(PointerEventData eventData)
{
    // 做一些操作...
    eventData.Use(); // 标记事件已使用，背后的 UI 不会收到点击
}
```

##### RaycastResult 子结构

```csharp
// eventData.pointerCurrentRaycast 的内容
public struct RaycastResult
{
    public GameObject gameObject;     // 命中的对象
    public BaseRaycaster module;      // 命中的 Raycaster（通常是 GraphicRaycaster）
    public float distance;            // 射线检测距离
    public int depth;                 // 层级深度
    public int sortingLayer;          // 排序层
    public int sortingOrder;          // 排序层内序号
    public Vector2 screenPosition;    // 屏幕位置
    public Vector3 worldPosition;     // 世界位置
    public Vector3 worldNormal;       // 世界法线
}
```

---

#### 二、BaseEventData（基础事件数据）

> 所有 UI 事件的基类，Select / Submit / Cancel 接口使用。PointerEventData 和 AxisEventData 都继承自它。

| 属性 | 类型 | 说明 |
|------|------|------|
| `selectedObject` | `GameObject` | ==当前 EventSystem 中选中的对象==。ISelectHandler / ISelectHandler 中用此获取被选中的对象 |
| `currentInputModule` | `BaseInputModule` | 当前使用的输入模块（StandaloneInputModule 等） |
| `used` | `bool` | 事件是否已被消费 |
| **Use()** | `void` | 标记事件已使用 |
| **Reset()** | `void` | 重置事件数据状态 |

```csharp
// Select/Submit 中使用
public void OnSelect(BaseEventData eventData)
{
    Debug.Log("选中: " + eventData.selectedObject.name);
}

public void OnSubmit(BaseEventData eventData)
{
    Debug.Log("提交: " + eventData.selectedObject.name);
    eventData.Use(); // 标记已处理
}
```

---

#### 三、AxisEventData（轴事件数据）

> `IMoveHandler.OnMove()` 专用，携带键盘方向键/手柄摇杆的导航输入。

| 属性 | 类型 | 说明 |
|------|------|------|
| `moveVector` | `Vector2` | ==移动方向向量==。`(1,0)`=右、`(-1,0)`=左、`(0,1)`=上、`(0,-1)`=下 |
| `moveDir` | `MoveDirection` | 移动方向枚举：Left / Right / Up / Down / None |
| `used` | `bool` | 事件是否已被消费 |

```csharp
using UnityEngine;
using UnityEngine.EventSystems;

public class MoveHandler : MonoBehaviour, IMoveHandler
{
    public void OnMove(AxisEventData eventData)
    {
        Debug.Log("导航方向: " + eventData.moveVector);
        // 自定义导航逻辑
        switch (eventData.moveDir)
        {
            case MoveDirection.Left:  Debug.Log("←"); break;
            case MoveDirection.Right: Debug.Log("→"); break;
            case MoveDirection.Up:    Debug.Log("↑"); break;
            case MoveDirection.Down:  Debug.Log("↓"); break;
        }
    }
}
```

---

#### 📋 EventData 接口与类型对照

| 接口 | 回调参数类型 | 获取关键信息 |
|------|------|------|
| `IPointerEnter/Exit` | `PointerEventData` | `pointerEnter`、`position` |
| `IPointerDown/Up/Click` | `PointerEventData` | `button`、`clickCount`、`position`、`pressPosition` |
| `IBeginDrag` | `PointerEventData` | `position`、`pressPosition` |
| `IDrag` | `PointerEventData` | `delta`（核心）、`position`、`dragging` |
| `IEndDrag` | `PointerEventData` | `pointerDrag`、`position` |
| `IDrop` | `PointerEventData` | `pointerDrag`（获取拖拽物） |
| `IScroll` | `PointerEventData` | `scrollDelta`（核心） |
| `ISelect/Deselect` | `BaseEventData` | `selectedObject` |
| `ISubmit/Cancel` | `BaseEventData` | `selectedObject` |
| `IMove` | `AxisEventData` | `moveVector`、`moveDir` |

---

### ⚠️ 常见问题排查

| 问题 | 原因 | 解决方案 |
|------|------|------|
| 接口方法不触发 | 没有 EventSystem / GraphicRaycaster / Graphic 组件 / Raycast Target 未勾选 | 依次检查四个条件 |
| 拖拽无效 | DragThreshold 太大 / 事件被其他 UI 阻挡 | 调小 DragThreshold / 检查遮挡 |
| 拖拽中 Drop 接收不到 | 被拖拽物体阻挡了射线 | ==拖拽时设置 `canvasGroup.blocksRaycasts = false`== |
| 多对象穿透 | 前方 UI 消耗了射线检测 | `eventData.Use()` 标记事件已使用 |

> [!NOTE] 命名空间
> 所有 UI 事件接口位于 `UnityEngine.EventSystems` 命名空间。`PointerEventData`、`AxisEventData`、`BaseEventData` 同样位于此命名空间。

## 🎯 EventTrigger 组件

> EventTrigger 是 Unity 提供的**可视化事件绑定组件**。不需要写任何接口实现代码，直接在 Inspector 面板中拖拽绑定即可响应 UI 事件。适合==快速原型、简单交互、策划自行配置==等场景。

> [!NOTE] EventTrigger vs 接口实现
> EventTrigger 底层仍然是靠 `IEventSystemHandler` 接口机制实现的，只是 Unity 帮你封装了一层。==需要代码灵活性的场景优先用接口实现，需要可视化配置的场景用 EventTrigger==。

---

### 📋 Inspector 使用方式

```
操作步骤：
1. 选中 UI 对象（如 Image、Panel）
2. Add Component → Event → Event Trigger
3. 点击 "Add New Event Type" 按钮
4. 从下拉菜单中选择事件类型（如 PointerClick）
5. 点击 "+" 添加回调
6. 拖入目标 GameObject + 选择该对象上挂载脚本的 public 方法
```

> [!TIP]
> 被调用的方法**必须是 public**，且参数类型要与事件类型匹配（`BaseEventData` / `PointerEventData` / `AxisEventData`），也可以选择无参方法。

---

### 🔢 EventTriggerType 事件类型全览

> ==EventTrigger 支持的事件类型==通过 `EventTriggerType` 枚举定义，共 **17 种**，与 UI 事件接口一一对应。

| 枚举值 | 对应接口 | 触发时机 | 参数类型 |
|--------|----------|----------|----------|
| `PointerEnter` | `IPointerEnterHandler` | 指针进入 UI 区域 | `PointerEventData` |
| `PointerExit` | `IPointerExitHandler` | 指针离开 UI 区域 | `PointerEventData` |
| `PointerDown` | `IPointerDownHandler` | 指针按下（鼠标按下/手指触摸） | `PointerEventData` |
| `PointerUp` | `IPointerUpHandler` | 指针抬起（鼠标松开/手指离开） | `PointerEventData` |
| `PointerClick` | `IPointerClickHandler` | 按下并松开（完整一次点击） | `PointerEventData` |
| `BeginDrag` | `IBeginDragHandler` | 开始拖拽（按下+移动超过阈值） | `PointerEventData` |
| `Drag` | `IDragHandler` | 拖拽中（每帧触发） | `PointerEventData` |
| `EndDrag` | `IEndDragHandler` | 拖拽结束（松开） | `PointerEventData` |
| `Drop` | `IDropHandler` | 被拖拽物体在自身释放 | `PointerEventData` |
| `Scroll` | `IScrollHandler` | 鼠标滚轮滚动 | `PointerEventData` |
| `Select` | `ISelectHandler` | 被选中（导航/代码设置） | `BaseEventData` |
| `Deselect` | `IDeselectHandler` | 取消选中 | `BaseEventData` |
| `Submit` | `ISubmitHandler` | 确认提交（回车/Space/手柄A） | `BaseEventData` |
| `Cancel` | `ICancelHandler` | 取消操作（Esc） | `BaseEventData` |
| `Move` | `IMoveHandler` | 键盘导航移动 | `AxisEventData` |
| `UpdateSelected` | `IUpdateSelectedHandler` | 选中后==每帧触发== | `BaseEventData` |
| `InitializePotentialDrag` | `IInitializePotentialDragHandler` | 检测到潜在拖拽可能（按下瞬间） | `PointerEventData` |

---

### 💻 代码控制 EventTrigger

#### 方式一：完全由代码创建并添加事件

```csharp
using UnityEngine;
using UnityEngine.EventSystems;

public class EventTriggerCreator : MonoBehaviour
{
    void Start()
    {
        // 1. 确保有 EventTrigger 组件
        EventTrigger trigger = gameObject.AddComponent<EventTrigger>();

        // 2. 创建事件条目
        EventTrigger.Entry entry = new EventTrigger.Entry();
        entry.eventID = EventTriggerType.PointerClick;  // 选择事件类型

        // 3. 添加回调（带参数的方法）
        entry.callback.AddListener((data) => OnPointerClick((PointerEventData)data));

        // 4. 添加到 EventTrigger
        trigger.triggers.Add(entry);
    }

    public void OnPointerClick(PointerEventData eventData)
    {
        Debug.Log("点击位置: " + eventData.position);
    }
}
```

#### 方式二：Lambda 直接绑定

```csharp
EventTrigger trigger = gameObject.AddComponent<EventTrigger>();

// 点击事件
EventTrigger.Entry clickEntry = new EventTrigger.Entry();
clickEntry.eventID = EventTriggerType.PointerClick;
clickEntry.callback.AddListener((data) => 
{
    Debug.Log("被点击了！");
});
trigger.triggers.Add(clickEntry);

// 移入事件
EventTrigger.Entry enterEntry = new EventTrigger.Entry();
enterEntry.eventID = EventTriggerType.PointerEnter;
enterEntry.callback.AddListener((data) => 
{
    GetComponent<Image>().color = Color.yellow; // 高亮
});
trigger.triggers.Add(enterEntry);

// 移出事件
EventTrigger.Entry exitEntry = new EventTrigger.Entry();
exitEntry.eventID = EventTriggerType.PointerExit;
exitEntry.callback.AddListener((data) => 
{
    GetComponent<Image>().color = Color.white; // 恢复
});
trigger.triggers.Add(exitEntry);
```

#### 方式三：遍历动态添加多个事件

```csharp
public class DynamicEventTrigger : MonoBehaviour
{
    void Start()
    {
        // 要注册的事件类型列表
        EventTriggerType[] eventTypes = 
        {
            EventTriggerType.PointerEnter,
            EventTriggerType.PointerExit,
            EventTriggerType.PointerDown,
            EventTriggerType.PointerUp,
            EventTriggerType.PointerClick
        };

        EventTrigger trigger = gameObject.AddComponent<EventTrigger>();

        foreach (EventTriggerType type in eventTypes)
        {
            EventTrigger.Entry entry = new EventTrigger.Entry();
            entry.eventID = type;
            entry.callback.AddListener((data) => OnUIEvent(type, (PointerEventData)data));
            trigger.triggers.Add(entry);
        }
    }

    void OnUIEvent(EventTriggerType type, PointerEventData data)
    {
        Debug.Log($"[{gameObject.name}] 事件: {type}, 位置: {data.position}");
    }
}
```

---

### 🧩 常用场景

#### 场景1：UI 悬停高亮效果

```csharp
// 不用写接口，一行 AddListener 搞定
EventTrigger trigger = gameObject.AddComponent<EventTrigger>();

EventTrigger.Entry enter = new EventTrigger.Entry();
enter.eventID = EventTriggerType.PointerEnter;
enter.callback.AddListener((data) => GetComponent<Image>().color = new Color(1, 1, 1, 0.8f));
trigger.triggers.Add(enter);

EventTrigger.Entry exit = new EventTrigger.Entry();
exit.eventID = EventTriggerType.PointerExit;
exit.callback.AddListener((data) => GetComponent<Image>().color = Color.white);
trigger.triggers.Add(exit);
```

#### 场景2：Slider 松手时保存（PointerUp）

```csharp
// 配合 Slider.onValueChanged（拖动中实时预览）+ PointerUp（松手保存）
Slider slider = GetComponent<Slider>();
EventTrigger trigger = slider.gameObject.AddComponent<EventTrigger>();

EventTrigger.Entry entry = new EventTrigger.Entry();
entry.eventID = EventTriggerType.PointerUp;
entry.callback.AddListener((data) =>
{
    // 松手时才保存，避免拖动中频繁写文件
    PlayerPrefs.SetFloat("Volume", slider.value);
    PlayerPrefs.Save();
    Debug.Log($"已保存音量: {slider.value}");
});
trigger.triggers.Add(entry);
```

#### 场景3：长按功能实现

```csharp
using System.Collections;
using UnityEngine;
using UnityEngine.EventSystems;
using UnityEngine.Events;

public class LongPressHandler : MonoBehaviour
{
    public float longPressDuration = 0.5f;
    public UnityEvent onLongPress;  // 长按触发的事件

    private Coroutine longPressCoroutine;
    private bool isPressing = false;

    void Start()
    {
        EventTrigger trigger = gameObject.AddComponent<EventTrigger>();

        // 按下：开始计时
        EventTrigger.Entry downEntry = new EventTrigger.Entry();
        downEntry.eventID = EventTriggerType.PointerDown;
        downEntry.callback.AddListener((data) =>
        {
            isPressing = true;
            longPressCoroutine = StartCoroutine(LongPressRoutine());
        });
        trigger.triggers.Add(downEntry);

        // 抬起：取消计时
        EventTrigger.Entry upEntry = new EventTrigger.Entry();
        upEntry.eventID = EventTriggerType.PointerUp;
        upEntry.callback.AddListener((data) =>
        {
            isPressing = false;
            if (longPressCoroutine != null)
                StopCoroutine(longPressCoroutine);
        });
        trigger.triggers.Add(upEntry);
    }

    IEnumerator LongPressRoutine()
    {
        yield return new WaitForSeconds(longPressDuration);
        if (isPressing)
        {
            onLongPress?.Invoke();
            Debug.Log("长按触发！");
        }
    }
}
```

#### 场景4：移除与动态管理事件

```csharp
EventTrigger trigger = GetComponent<EventTrigger>();

// 移除所有事件
trigger.triggers.Clear();

// 移除特定类型的所有事件
trigger.triggers.RemoveAll(entry => entry.eventID == EventTriggerType.PointerClick);

// 临时禁用某个事件（直接清空回调但保留条目）
var clickEntry = trigger.triggers.Find(e => e.eventID == EventTriggerType.PointerClick);
if (clickEntry != null)
    clickEntry.callback.RemoveAllListeners();

// 恢复
clickEntry.callback.AddListener((data) => Debug.Log("恢复点击"));
```

---

### 📊 EventTrigger vs 接口实现 完整对比

| 对比维度 | EventTrigger | 接口实现 |
|----------|:---:|:---:|
| **学习成本** | 🟢 低（Inspector 拖拽即可） | 🟡 中（需要了解接口机制） |
| **代码量** | 可不写代码 | 需要实现接口方法 |
| **灵活性** | 🟡 中（受限于 Inspector 绑定） | 🟢 高（完全代码控制） |
| **性能** | 🟡 略差（额外组件 + 反射调用） | 🟢 好（直接方法调用） |
| **复用性** | ❌ 低（每个对象单独配置） | ✅ 高（脚本可复用到多个对象） |
| **多人协作** | ✅ 策划/美术可自行配置 | ❌ 需要程序员修改代码 |
| **复杂交互** | ❌ 不适合（如多步拖拽） | ✅ 完全胜任 |
| **运行时动态** | ✅ 支持代码动态添加 | ✅ 同样支持 |
| **调试** | 🟡 Inspector 直观但难批量改 | 🟡 代码调试 |
| **适用场景** | 原型、简单悬停/点击、策划配置 | 正式项目、复杂交互、可复用组件 |

> [!IMPORTANT] 选择建议
> - **原型阶段 / GameJam** → EventTrigger（快速出效果）
> - **简单悬停变色、音效反馈** → EventTrigger（减少代码文件）
> - **复杂拖拽系统、嵌套交互** → 接口实现（过程控制更强）
> - **可复用的 UI 组件库** → 接口实现（封装为 Prefab 脚本）
> - **需要策划频繁调整回调** → EventTrigger（Inspector 直接改）

---

### 📦 EventTrigger.Entry 结构

```csharp
// EventTrigger.Entry 是每个事件条目的数据结构
[System.Serializable]
public class Entry
{
    public EventTriggerType eventID;      // 事件类型
    public TriggerEvent callback;         // 回调（UnityEvent<BaseEventData>）
}

// TriggerEvent 本质是一个 UnityEvent<BaseEventData>
// 所以 AddListener 需要接受 BaseEventData 参数的委托
```

| 成员 | 类型 | 说明 |
|------|------|------|
| `eventID` | `EventTriggerType` | 事件类型枚举（17种之一） |
| `callback` | `TriggerEvent` | 回调事件，继承自 `UnityEvent<BaseEventData>` |

---

### ⚠️ 注意事项

| 注意点 | 说明 |
|--------|------|
| **方法参数匹配** | Inspector 绑定方法时，参数可选 `BaseEventData` / `PointerEventData` / `AxisEventData` 或无参。代码绑定建议用 `(data) => { }` Lambda 处理 |
| **类型转换** | `TriggerEvent` 传入的是 `BaseEventData`，需要转为具体类型：`PointerEventData pData = (PointerEventData)data;` |
| **性能开销** | EventTrigger 比接口实现多一层组件 + UnityEvent 序列化开销。==列表中有大量 UI 元素时不建议每个都挂 EventTrigger== |
| **互相干扰** | EventTrigger 和手动实现的接口在同一个对象上会共存，不会冲突。但如果一个处理了 `eventData.Use()`，另一个就不会收到 |
| **预制体覆盖** | Prefab 实例上的 EventTrigger Inspector 绑定不会被 Prefab 覆盖（UnityEvent 序列化行为） |


## 屏幕坐标转UI相对坐标

RectTransformUtility公共类是一个RectTransform的辅助类   
主要用于进行一些坐标转换等操作 最常用的是将屏幕空间上的点转换为UI坐标下的点

```csharp

RectTransformUtility.ScreenPointToLocalPointInRectangle(
    rectTransform,
    eventData.position,
    canvas,
    out Vector2 localPoint
)
//参数一：rectTransform 要转换的RectTransform组件 父对象坐标转为RectTransform坐标
//参数二：eventData.position 要转换的屏幕坐标
//参数三：canvas 当前场景中的Canvas组件
//参数四：localPoint 转换后的相对坐标
```

## 遮罩

在不改变图片的情况下 让图片在游戏中只显示其中的一部分   

### 使用

通过在父对象上添加Mask组件即可遮罩其子对象  

==注意==：  
1.想要被遮罩的Image需要勾选Maskable
2.只要父对象添加了Mask组件，那么所有的UI对象都会被遮罩  
3.遮罩父对象图片的制作，不透明的地方显示，透明的地方被遮罩
