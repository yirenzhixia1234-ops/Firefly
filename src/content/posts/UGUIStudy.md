---
title: UGUI学习
published: 2026-07-16
pinned: false
description: Unity UGUI学习 - Canvas、RectTransform、EventSystem等组件
tags: [Unity,学习,UI]
author: boluobao
draft: true
---

# UGUI学习

## Canvas组件

### 渲染模式

Overlay:覆盖模式 UI始终显示在场景内容前方
参数：
PixelPerfect：是否开启无锯齿精确渲染（性能换效果）  
SortOrder：排序层编号（用于控制多个Canvas时的渲染先后顺序） 
TargetDisplay：目标设备（在哪个显示设备上显示） 
Addit:ona!ShaderChannels：其他着色器通道，决定着色器可以读取哪些数据    


ScreenSpace:屏幕空间模式 3D物体可以显示在UI之前
参数：
RenderCamera：用于渲染Ul的摄像机（如果不设置将类似于覆盖模式）  
PlaneDistance：Ul平面在摄像机前方的距离，类似整体Z轴的感觉  
SortingLayer:所在排序层 
Order in Layer:排序层的序号     

WorldSpace:世界空间模式 3D物体可以显示在UI之前

## CanvasScaler组件

### ConstantPixelSize(恒定像素模式)
无论屏幕分辨率如何，UI元素的像素大小都保持不变  
ScaleFactor：缩放系数，按此系数缩放画布中的所有UI元素   
Reference Pixels Per Unit：位参考像素，多少像素对应Unity中的一个单位（默认一个单位为100像素）  
图片设置中的PixelsPerUnit设置，会和该参数一起参与计算   

UI原始尺寸 = 图片大小（像素） / (Pixels Per Unit / Reference Pixels Per Unit)   

### ScaleWithScreenSize(缩放模式)
是否根据屏幕大小缩放UI元素
ScreenMatchMode：屏幕匹配模式，根据屏幕分辨率和画布大小，选择合适的缩放模式 
    MatchWidthOrHeight：是否匹配宽度或高度，根据该参数，选择合适的缩放模式  参数为0时，匹配宽度，参数为1时，匹配高度
    Expanded：让UI元素始终能完整的显示在屏幕上 可能会出现黑边 
        计算公式：缩放系数 = Mathf.min(屏幕宽 / 参考分辨率宽, 屏幕高 / 参考分辨率高)
                 画布尺寸 = 屏幕尺寸 / 缩放系数
    Shrink：让UI元素根据屏幕分辨率自动缩放，可能会出现内容被裁剪的情况
        计算公式：缩放系数 = Mathf.max(屏幕宽 / 参考分辨率宽, 屏幕高 / 参考分辨率高)
                 画布尺寸 = 屏幕尺寸 / 缩放系数 

### ConstantPhysicalSize(恒定物理模式)
无论屏幕分辨率如何，UI元素的物理大小都保持不变
    先计算新单位参考像素 = 单位参考像素 * PhysicalUnit / Default Sprite DPI
    UI原始尺寸 = 图片大小（像素） / (Pixels Per Unit / 新单位参考像素)

## Graphic Raycaster组件(图形射线投射器组件)

用于检测UI输入事件的射线发射器 主要负责通过射线检测玩家和UI元素之间的交互 判断是否点击到了UI元素

### 参数
Ignore Reversed Graphics:是否忽略反转图形   
Blocking Objects:射线被哪些类型的碰撞器阻挡（在覆盖渲染模式下无效） 
Blocking Mask:射线被哪些层级的碰撞器阻挡（在覆盖渲染模式下无效）    

## EventSystem组件

EventSystem是事件系统 它是用于管理玩家的输入事件并分发给各UI控件 是事件逻辑处理模块 
所有的UI事件都通过EventSystem组件中轮询检测并做相应的执行   
它类似一个中转站 和许多模块一起共同协作 如果没有它 所有点击、拖拽等等行为都不会响应

### 参数
FirstSelected：首先选择的游戏对象，可以设置游戏一开始的默认选择 
SendNavigationEvents：是否允许导航事件（移动/按下/取消） 相当于用键盘的wasd来控制UI的选择       
DragThreshold：拖拽操作的阀值（移动多少像素算拖拽）     


## Standalone Input Module组件(独立输入系统组件)

主要针对于处理鼠标/键盘/控制器/触屏的输入，输入的事件通过EventSystem组件分发给各UI控件

## RectTransform组件





