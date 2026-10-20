---
title: C#学习笔记3
published: 2026-08-31
pinned: false
description: C#学习笔记3 - .NET跨语言与跨平台、Mono与IL2CPP执行机制与优缺点、IL2CPP代码剥离的解决方案
tags: [C#,学习]
author: boluobao
draft: false
---

# C#学习笔记3

## .Net相关

### 1. .NET 的跨语言（Cross-Language）

.NET 的核心是 **CLR（Common Language Runtime，公共语言运行时）** 和 **IL（Intermediate Language，中间语言，也称 CIL）**。

编译执行流程：

```
C# 源码 ──┐
VB.NET 源码 ──┼─→ 各自语言的编译器 ──→ IL（托管程序集 .dll/.exe）──→ CLR ──→ 机器码
F# 源码 ──┘
```

- 各种 .NET 语言的源码，最终都会被编译成**同一种中间语言 IL**。
- CLR 负责加载 IL 并把它编译成机器码执行（JIT）。
- 因为大家都"翻译"成同一种 IL，所以 **C# 写的类可以被 VB.NET / F# 直接调用**，这就是跨语言互操作。

保证互操作的两个基础规范：

- **CTS（Common Type System，公共类型系统）**：统一了所有 .NET 语言的类型定义，`int`、`string`、`class` 在不同语言里是同一套类型。
- **CLS（Common Language Specification，公共语言规范）**：定义了跨语言互操作必须遵守的公共子集，遵守 CLS 的代码才能被所有 .NET 语言使用。

### 2. .NET 的跨平台（Cross-Platform）

- IL 本身与 CPU、操作系统**无关**，它只运行在 CLR 之上。
- 只要目标平台有对应的运行时（早期 .NET Framework 仅支持 Windows；.NET Core / .NET 5+ 提供了 Windows / Linux / macOS / 移动端运行时），同一份 IL 就能直接运行——**编译一次，到处运行**。
- **Unity 中的跨平台**：C# 源码编译成 IL 后，由 Unity 的"脚本后端"把 IL 变成各平台可执行的代码：
  - **Mono**：运行时 JIT 编译；
  - **IL2CPP**：构建时转成 C++ 再编译成各平台原生机器码。
  所以同一份 C# 代码可以运行在 Windows、Android、iOS、主机、WebGL 上。

### 3. Mono 的执行机制

Mono 是 .NET Framework 的**开源跨平台实现**，也是 Unity 早期的默认脚本后端。

- **JIT（Just-In-Time，即时编译）**：运行时按需编译——某个方法**第一次被调用时**才把它的 IL 编译成机器码，编译结果缓存复用。
- **AOT（Ahead-Of-Time，预编译）**：构建时就把 IL 预编译成机器码，用于 iOS 等**禁止 JIT** 的平台（iOS 上 Mono 只能以 Full AOT 方式运行，限制较多）。

> [!NOTE] iOS 为什么禁止 JIT
> 苹果审核禁止应用在运行时动态执行代码（iOS 采用 W^X 内存保护），所以 iOS 上不允许 JIT，这也是 Unity 在 iOS 上默认使用 IL2CPP 的原因。

### 4. IL2CPP 的执行机制

IL2CPP = **IL To C++**，是 AOT（预编译）方案，目前 Unity 的主流脚本后端（iOS / Android / 主机 / WebGL 默认）。

执行流程（全部发生在**构建期**）：

```
C# 源码 → C# 编译器 → IL → il2cpp 工具 → C++ 源码 → 平台 C++ 编译器 → 原生机器码
```

1. C# 源码编译成 IL；
2. Unity 的 `il2cpp` 工具把 IL **逐方法转换成 C++ 源码**；
3. 调用各平台自带的 C++ 编译器，把 C++ 编译成**原生机器码**；
4. 运行时直接执行原生代码，没有 JIT；托管对象的生命周期由 Unity 提供的 GC（垃圾回收）管理。

因为转换发生在构建期，最终包里**没有 IL、没有 Mono 运行时**，只有原生代码和转换后的元数据。

### 5. Mono 与 IL2CPP 优缺点对比

| 维度 | Mono | IL2CPP |
|---|---|---|
| 编译方式 | JIT（运行时编译），iOS 需 Full AOT | AOT（构建时 IL→C++→原生码） |
| 构建速度 | 快，开发迭代方便 | 慢，构建时间长 |
| 运行性能 | 有 JIT 开销，冷启动慢 | 好，无 JIT 开销 |
| 包体大小 | 较大（自带 Mono 运行时） | 较小（可裁剪、去元数据） |
| 反射 / 动态代码 | 支持完整（含 Reflection.Emit） | 受限，不支持动态生成 IL，反射需 link.xml 保留 |
| 调试体验 | 好（IL 层断点、看 IL） | 较差（原生层调试） |
| 平台兼容性 | 差，iOS / 主机受限 | 好，几乎所有平台 |
| 反编译难度 | 低（IL 可反编译） | 高（原生机器码） |

> [!TIP] 怎么选
> - **开发期 / 编辑器**：默认 Mono，迭代快、好调试。
> - **发布版**：默认 IL2CPP，包体小、性能好、平台覆盖广（尤其 iOS / 主机）。
> - 如果项目大量依赖反射 / 动态生成代码（如热更框架），需要评估 IL2CPP 的限制并做好保留配置。

### 6. IL2CPP 下的代码剥离（Managed Stripping）

发布版使用 IL2CPP 时，Unity 构建会执行 **Managed Stripping（托管代码剥离）**：基于 IL Linker 从程序入口分析"引用图"，把**没有被静态引用的类型、方法、字段**从程序集中删除，从而减小包体。

剥离等级（Player Settings → Other Settings → **Managed Stripping Level**）：

- **Disabled**：完全不剥离
- **Low**：剥离未使用的内部类型
- **Medium**：剥离未使用的类型和成员（较激进）
- **High**：最激进，尽量剥离

> [!WARNING] 剥离引发的问题
> 裁剪器只能看到**静态引用**，看不到运行时通过**字符串 / 反射**的调用。以下场景的类型/成员会被误删：
> - `Type.GetType("类名")`、`Activator.CreateInstance`
> - `JsonUtility` / `XmlSerializer` 等序列化（按字段名字符串读写）
> - `SendMessage` / `BroadcastMessage` / `Invoke`（按方法名字符串调用）
> - 依赖自定义 Attribute 的反射扫描
>
> 表现：运行时报 `TypeLoadException`、`MissingMethodException`、`MissingFieldException`，或序列化结果为空、`null` 等。

### 7. 解决代码被剥离：link.xml / [Preserve] / 调整等级

**① 调整剥离等级（排查用）**

先降到 `Low` 或 `Disabled` 复测，确认问题确实是剥离导致；确认后再针对性地保留代码，而不是一直关掉剥离。

**② link.xml（最重要、最常用）**

在 Assets 目录下新建 `link.xml`，构建时 Unity 会自动收集项目里所有 link.xml 并应用：

```xml
<linker>
  <!-- 整个程序集全部保留 -->
  <assembly fullname="Assembly-CSharp" preserve="all" />

  <!-- 只保留指定类型（支持通配符） -->
  <assembly fullname="UnityEngine.CoreModule">
    <type fullname="UnityEngine.GameObject" preserve="all" />
    <type fullname="MyGame.Config.*" preserve="all" />
    <namespace fullname="MyGame.Data" preserve="all" />
  </assembly>
</linker>
```

- `preserve` 可选值：`all`（类型及其全部成员）、`fields`（只保留字段）、`nothing`。
- 命名空间可用 `<namespace>` 批量保留。

**③ `[Preserve]` 特性（源码级标记）**

在代码中直接标记，告诉 Linker 保留：

```csharp
using UnityEngine.Scripting;

[Preserve] // 保留该类（序列化 / 反射用到）
public class ConfigData
{
    [Preserve] // 保留字段
    public string Name;

    [Preserve]
    private void OnNotify() { } // 被 SendMessage 调用的方法
}
```

- 类型上加 `[Preserve(AllMembers = true)]` 可一次性保留所有成员。
- **④ 真实引用兜底**：静态代码里真实"用到"该类型（`new`、`typeof`、强转），裁剪器就会保留。

> [!TIP] 反射场景的最佳实践
> 优先用**强类型** API 代替字符串反射，例如用泛型 `JsonUtility.FromJson<T>()` 而不是手动 `Type.GetType`；实在要用反射，就把相关类型写进 link.xml。

### 8. 裁剪未引用代码的注意点与排查

**本质**：裁剪未引用代码是**特性**而不是 bug——问题只出在"看似没被引用、运行时要用的代码"被删掉。所以正确思路不是关闭裁剪，而是**把需要保留的代码标记出来**。

注意事项：

- **IL2CPP 不支持动态生成代码**：`System.Reflection.Emit`（ILGenerator）、`Expression.Compile` 在 IL2CPP 下会抛异常（iOS / Android 发布版），这类需求要改用预生成代码或解释执行方案。
- **Unity 的隐式保留**：`[SerializeField]` 字段、UnityEvent 绑定、AnimationEvent 等 Unity 会隐式保留，一般不用自己处理；但自定义序列化器 / 反射框架仍需显式保留。
- **第三方库**：JSON 库（如 Newtonsoft.Json）、依赖注入等框架，大多需要在 link.xml 里保留相关程序集，或使用库官方提供的裁剪配置文件。
- **构建后验证**：发布前开 `Development Build` + `Script Debugging` 在真机跑一遍完整流程，确认无 `MissingMethod` / `TypeLoad` 类报错。

排查步骤：

1. 看运行日志的报错类型（`MissingMethod` / `TypeLoad` / 序列化为空）；
2. 把 Managed Stripping Level 降到 `Disabled` 复测，确认是否由裁剪引起；
3. 定位被裁剪的类型 / 成员，用 **link.xml 或 [Preserve]** 保留；
4. 恢复原剥离等级复测，直到所有功能正常。
