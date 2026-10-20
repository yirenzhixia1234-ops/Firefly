---
title: Unity学习笔记3
published: 2026-08-04
pinned: false
description: Unity学习笔记3
tags: [Unity,学习]
author: boluobao
draft: false
---

# Unity学习笔记3

## 图片导入相关设置

Unity 官方文档对纹理导入设置有详细说明，以下是官方参考链接：

### 总览

- [纹理导入设置窗口参考](https://docs.unity3d.com/cn/current/Manual/class-TextureImporter.html)（中文）— Texture Import Settings 窗口的整体说明
- [Texture type and shape settings](https://docs.unity3d.com/cn/current/Manual/class-TextureImporter-type-and-shape.html)（中文）— 纹理类型和形状设置参考

### 各纹理类型

| 纹理类型 | 官方文档 |
|------|------|
| **Default / Normal map / Lightmap / Single Channel** | 包含在总览页面中 |
| **Editor GUI and Legacy GUI** | [Editor GUI and Legacy GUI 参考](https://docs.unity3d.com/cn/2022.3/Manual/texture-type-editor-gui-and-legacy-gui.html)（中文） |
| **Sprite (2D and UI)** | [Sprite Import Settings 参考](https://docs.unity3d.com/cn/2022.3/Manual/texture-type-sprite.html)（中文） |
| **Cursor** | [Cursor Import Settings 参考](https://docs.unity3d.com/Manual/texture-type-cursor.html) |
| **Cookie** | [Cookie Import Settings 参考](https://docs.unity3d.com/Manual/texture-type-cookie.html) |

### Sprite 相关组件

- [Sprite Editor 官方文档](https://docs.unity3d.com/Manual/SpriteEditor.html) — Custom Outline / Custom Physics Shape / Secondary Textures 等
- [Sprite Renderer 组件参考](https://docs.unity3d.com/Manual/class-SpriteRenderer.html) — Draw Mode / Mask Interaction / Sorting Layer 等
- [Sprite Mask 组件参考](https://docs.unity3d.com/Manual/class-SpriteMask.html)
- [Sprite Atlas 官方文档](https://docs.unity3d.com/Manual/class-SpriteAtlas.html) — Master / Variant / Objects for Packing 等

### 平台设置（压缩格式）

- [纹理压缩格式概述](https://docs.unity3d.com/Manual/class-TextureImporterOverride.html) — Max Size / Format / Compression / Crunch 等

---

# 模型导入相关设置

Unity 官方文档对各页签参数有详细说明，以下是官方参考链接：

## 总览

- [Model Import Settings window 总览](https://docs.unity.cn/2022.3/Documentation/Manual/class-FBXImporter.html) — 包含 Model / Rig / Animation / Materials 四个页签的入口说明

## Model 页签

- [Model tab Import Settings reference](https://docs.unity3d.com/6000.2/Documentation/Manual/FBXImporter-Model.html) — 包含 Scene / Meshes / Geometry 三个区域的完整参数参考
- [Model 选项卡（中文）](https://docs.unity3d.com/cn/current/Manual/FBXImporter-Model.html) — 中文版

## Rig 页签

- [Rig tab](https://docs.unity.cn/2022.3/Documentation/Manual/FBXImporter-Rig.html) — Animation Type / Avatar 配置 / Optimize Game Objects 等
- [Avatar 配置（Humanoid）](https://docs-alpha.unity3d.com/2022.3/Documentation/Manual/ConfiguringtheAvatar.html)

## Animation 页签

- [Animation tab](https://docs.unity.cn/2022.3/Documentation/Manual/FBXImporter-Animation.html) — 动画剪辑导入、Loop、Root Motion、Bake Into Pose 等

## Materials 页签

- [Materials tab](https://docs-alpha.unity3d.com/2022.3/Documentation/Manual/FBXImporter-Materials.html) — Material Creation Mode / Location / Naming / Search 等

## 补充

- [Importing a model（模型导入流程）](https://docs-alpha.unity3d.com/2022.3/Documentation/Manual/ImportingModelFiles.html)

---

# 3D动画相关

Unity 官方文档对 Mecanim 动画系统有完整的说明，以下是按 XMind 思维导图结构整理的官方参考链接：

## Animator 组件

- [Animator 组件参考](https://docs.unity3d.com/Manual/class-Animator.html) — Controller / Avatar / Apply Root Motion / Update Mode / Culling Mode

## Animator Controller（动画控制器）

- [Animator Controller 总览](https://docs.unity3d.com/Manual/class-AnimatorController.html) — 动画控制器的创建、结构和基本使用
- [Animator Controller（中文）](https://docs.unity3d.com/cn/current/Manual/class-AnimatorController.html)

## 状态机

- [动画状态机总览](https://docs.unity3d.com/Manual/AnimationStateMachines.html) — 状态机的概念、状态和子状态机
- [动画状态](https://docs.unity3d.com/Manual/AnimationStates.html) — Motion / Speed / Multiplier / Motion Time / Mirror / Cycle Offset / Foot IK / Write Defaults 等参数说明
- [动画过渡](https://docs.unity3d.com/Manual/class-Transition.html) — Has Exit Time / Exit Time / Fixed Duration / Transition Duration / Transition Offset / Interruption Source / Solo / Mute / Conditions
- [过渡（中文）](https://docs.unity3d.com/cn/current/Manual/class-Transition.html)
- [Any State](https://docs.unity3d.com/Manual/class-State.html) — AnyState 连线及 Can Transition To Self

## 动画参数

- [动画参数](https://docs.unity3d.com/Manual/AnimationParameters.html) — Float / Int / Bool / Trigger 四种参数类型
- [参数（中文）](https://docs.unity3d.com/cn/current/Manual/AnimationParameters.html)

## 混合树（Blend Tree）

- [混合树总览](https://docs.unity3d.com/Manual/class-BlendTree.html) — 1D / 2D Simple Directional / 2D Freeform Directional / 2D Freeform Cartesian / Direct
- [1D 混合树](https://docs.unity3d.com/Manual/BlendTree-1DBlending.html) — 单参数混合（如速度驱动 静止→行走→跑步）
- [2D 混合树](https://docs.unity3d.com/Manual/BlendTree-2DBlending.html) — 双参数混合（如前进/后退/左转/右转）
- [混合树附加选项](https://docs.unity3d.com/Manual/BlendTree-AdditionalOptions.html) — Time Scale / Homogeneous Speed / Mirroring
- [混合树（中文）](https://docs.unity3d.com/cn/current/Manual/class-BlendTree.html)

## 动画分层和遮罩

- [动画层](https://docs.unity3d.com/Manual/AnimationLayers.html) — Weight / Blending Mode (Override / Additive) / Sync / Timing / IK Pass
- [Avatar Mask](https://docs.unity3d.com/Manual/class-AvatarMask.html) — 人体部位遮罩 / Transform 层级遮罩
- [Avatar Mask（中文）](https://docs.unity3d.com/cn/current/Manual/class-AvatarMask.html)

## 子状态机

- [子状态机（Nested State Machine）](https://docs.unity3d.com/Manual/NestedStateMachines.html)

## 状态机行为脚本（State Machine Behaviour）

状态机行为脚本是一类特殊的脚本，继承指定的基类  
它主要用于关联到状态机中的状态矩形上 我们可以按照一定规则编写脚本  
当进入、退出、爆出在某一个特定状态时我们可以进行一些逻辑处理    
简单解释就是为Animator Controller状态机窗口中的某一个状态添加一个脚本 利用这个脚本我们可以做一些特殊功能    
比如：  
1.进入或退出某一状态时播放声音  
2.仅在某些状态下检测一些逻辑，比如是否接触地面等    
3.激活和控制某些状态相关的特效  

### 如何使用状态机脚本

1.新建一个脚本继承自StateMachineBehaviour的基类 
2.实现其中的特定方法进行状态行为监听    
```csharp
// OnStateEnter 进入状态时，第一个Update中调用
// OnStateExit 退出状态时，最后一个Update中调用
// OnStateIK OnAnimatorIK后调用
// OnStateMove OnAnimatorMove后调用
// OnStateUpdate 除第一帧和最后一帧，每一帧调用
// OnStateMachineEnter 子状态机进入时调用，第一个Update中调用
// OnStateMachineExit 子状态机退出时调用，最后一个Update中调用
```
- [State Machine Behaviour](https://docs.unity3d.com/Manual/StateMachineBehaviours.html) — OnStateEnter / OnStateExit / OnStateUpdate / OnStateMove / OnStateIK

## 状态机复用

- [Animator Override Controller](https://docs.unity3d.com/2022.3/Documentation/Manual/AnimatorOverrideController.html) — 复用状态机结构与逻辑、只替换动画剪辑

## 根运动（Root Motion）

- [Root Motion 总览](https://docs.unity3d.com/Manual/RootMotion.html) — 什么是根运动、如何使用
- [Animator.applyRootMotion](https://docs.unity3d.com/ScriptReference/Animator-applyRootMotion.html) — 脚本控制根运动开关

## 动作目标匹配

动画目标匹配主要指的是 当游戏中角色要以某种动作移动，该动作播放完毕后，人物的手或者脚必须落在某一个地方 

### 目标匹配的实现

Unity中的Animator组件提供了MatchTarget函数，可以方便的设置角色位置，使身体部位到达目标点    
```csharp
//参数一：目标位置
//参数二：目标角度
//参数三：匹配的骨骼位置
//参数四：位置角度权重
//参数五：开始位移动作的百分比
//参数六：结束位移动作的百分比
animator.MatchTarget(targetPos,targetRot,AvatarTarget.LeftHand,new MatchTargetWeight(Vector3.one,1),0.5f,0.5f);
```
==注意== 调用匹配动画的时机有一些限制   
1.必须保证动画已经切换到了目标动画上    
2.必须保证调用时动画并不是出于过度阶段  
3.需要开启Apply Root Motion
- [目标匹配（Target Matching）](https://docs.unity3d.com/Manual/TargetMatching.html) — 自动调整角色位置，使身体部位到达目标点
- [Animator.MatchTarget（API）](https://docs.unity3d.com/2022.3/Documentation/ScriptReference/Animator.MatchTarget.html)

## 动画IK控制

1.在状态机的层级设置中开启IK通道    
2.继承MonoBehavior的类中 Unity定义了—个Ik回调函数：OnAnimatorIK     
我们可以在该函数中调用Unity提供的IK相关API来控制IK  
3.Animator中的Ik相关API 
SetLookAtweight 设置头部IK权重      
SetLookAtPosition 设置头部IK看向位置    
SetIkPositionWeight 设置Ik位置权重  
SetIKRotationWeight 设置Ik旋转权重  
SetIKPosition 设置IK对应的位置  
SetIKRotation 设置IK对应的角度  
AvatarIKGoal枚举 四肢未端IK枚举 

- [Animation Rigging 包（中文）](https://docs.unity3d.com/cn/2022.3/Manual/com.unity.animation.rigging.html) — IK 约束系统（Two Bone IK / Chain IK / Multi-Aim 等）
- [OnAnimatorIK（API）](https://docs.unity3d.com/ScriptReference/MonoBehaviour.OnAnimatorIK.html) — 通过脚本设置 IK 

## 角色控制器（Character Controller）

- [Character Controller 组件](https://docs.unity3d.com/2022.3/Documentation/Manual/class-CharacterController.html) — Slope Limit / Step Offset / Skin Width / Min Move Distance / Center / Radius / Height
