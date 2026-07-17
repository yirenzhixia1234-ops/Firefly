---
title: UGUI学习
published: 2026-07-16
pinned: false
description: 
tags: [Unity,学习,UI]
author: boluobao
draft: true
---

# NGUI学习

## 三大基础组件

### Root组件

Root是用于分辨率自适应的根对象  
可以设置基本分辨率 相当于设置UI显示区域 
并且管理所有UI控件的分辨率自适应    

可以简单理解成 它管理一个UI画布 所有的UI都是显示在这个画布上的
它会管理 UI画布 和不同屏幕分辨率的适应关系  

#### RootStyle 

Flexible 灵活模式 在该模式下，UI都是以像素为基础，100像素的物体无论在多少分辨率上都是100像素    
这就意味着，100像素在分辨率低的屏幕上可能显示正常，但是在高分辨率上就会显得很小 
Minimum Height 最小高度 屏幕小于该高度时，UI会自动缩放保持原有比例
Maximum Height 最大高度 屏幕大于该高度时，UI会自动缩放保持原有比例

Constrained 约束模式 该模式下，屏幕按尺寸比例来适配，不管实际屏幕有多大NGUI都会通过合适的缩放来适配屏幕。   
这样在高分辨率上显示的UI就会被放大保持原有大小，但有可能会模糊，好处是各设备看到的UI和屏幕比例是一样的  
