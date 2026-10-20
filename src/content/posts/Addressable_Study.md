---
title: Addressable学习笔记
published: 2026-08-28
pinned: false
description: Addressable学习笔记
tags: [热更新,学习,Addressable]
author: boluobao
draft: false
---

# Addressable学习笔记

## 什么是Addressable

Addressable是Unity官方提供的一个热更新解决方案，它可以帮助我们在游戏运行时动态地加载和卸载资源，从而实现游戏的快速更新和优化。

## Addressable的优势

1. 热更新：Addressable支持在游戏运行时动态地加载和卸载资源，从而实现游戏的快速更新和优化。
2. 资源管理：Addressable提供了一套完整的资源管理机制，可以方便地管理和组织游戏中的资源。

### 加载资源
```csharp
//加载单个资源
public void LoadAsset()
{
    //创建一个资源地址
    AssetAddressable = "Assets/AddressableAssets/Textures/Texture1.png";
    //创建一个资源加载器
    AssetBundleLoader = Addressables.LoadAssetAsync<Texture2D>(AssetAddressable);
    //等待资源加载完成
    AssetBundleLoader.Completed += OnAssetBundleLoaded;
}

//资源加载完成回调
private void OnAssetBundleLoaded(AsyncOperationHandle<Texture2D> obj)
{
    //获取加载的资源
    Texture2D assetBundle = obj.Result;
    //将资源赋值给UI
    Image.texture = assetBundle;
}
```

### 卸载资源
```csharp
//卸载单个资源
public void UnloadAsset()
{
    //卸载资源
    Addressables.Release(AssetBundleLoader);
}
```

### 加载场景
```csharp
//加载场景
public void LoadScene()
{
    //创建一个场景地址
    SceneAddressable = "Assets/AddressableAssets/Scenes/Scene1.unity";
    //创建一个场景加载器
    SceneLoader = Addressables.LoadSceneAsync(SceneAddressable);
    //等待场景加载完成
    SceneLoader.Completed += OnSceneLoaded;
}

//场景加载完成回调
private void OnSceneLoaded(AsyncOperationHandle<SceneInstance> obj)
{
    //获取加载的场景
    SceneInstance scene = obj.Result;
    //激活场景
    scene.Activate();
}
```

### 卸载场景
```csharp
//卸载场景
public void UnloadScene()
{
    //卸载场景
    Addressables.UnloadSceneAsync(SceneLoader);
}
```
