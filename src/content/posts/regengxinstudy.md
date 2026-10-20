---
title: 热更新学习笔记
published: 2026-08-28
pinned: false
description: 热更新学习笔记
tags: [热更新,学习]
author: boluobao
draft: false
---

# 热更新学习笔记

## AssestBundle

### 1.1 AssetBundle是什么

AssetBundle是Unity提供的一种资源打包方式，可以将游戏中的资源（如模型、纹理、音频等）打包成一个或多个文件，以便在运行时动态加载和卸载。

### 1.2 AssetBundle的作用

AssetBundle可以用于以下场景：

- **减少初始加载时间**：将资源打包成AssetBundle可以减少初始加载时间，因为玩家只需要下载和加载所需的AssetBundle，而不是整个游戏资源。
- **优化内存使用**：AssetBundle可以按需加载资源，从而优化内存使用，避免一次性加载大量资源导致的内存溢出问题。
- **实现热更新**：AssetBundle可以实现热更新，即在不重新安装游戏的情况下更新游戏资源，从而提高游戏的更新效率和用户体验。

### 1.3 生成AssetBundle资源文件

在Unity中，可以使用以下步骤生成AssetBundle资源文件：（前提是unity中要有AssetBundle Browser）

1. 在Unity编辑器中，选择需要打包的资源。
2. 在Inspector面板中，点击“AssetBundle”选项卡，设置资源的AssetBundle名称和变体。
3. 在菜单栏中选择“Window”->“AssetBundle Browser”，打开AssetBundle浏览器。
4. 在AssetBundle浏览器中，点击“Build AssetBundles”按钮，选择保存AssetBundle文件的路径和格式。
5. Unity会自动将选中的资源打包成AssetBundle文件，并保存到指定的路径中。

### 1.4 加载AssetBundle资源文件


在Unity中，可以使用以下步骤加载AssetBundle资源文件：

1. 在代码中，使用`AssetBundle.LoadFromFile`方法加载AssetBundle文件，例如：

```csharp
AssetBundle myLoadedAssetBundle = AssetBundle.LoadFromFile(Path.Combine(Application.streamingAssetsPath, "myassetbundle"));
```

2. 使用`myLoadedAssetBundle.LoadAsset`方法加载AssetBundle中的资源，例如：

```csharp
GameObject myObject = myLoadedAssetBundle.LoadAsset<GameObject>("myobject");
```

3.异步加载AssetBundle资源文件

```csharp
IEnumerator LoadAssetBundleAsync(string path,string assetName)
{
    AssetBundleCreateRequest bundleRequest = AssetBundle.LoadFromFileAsync(path);
    yield return bundleRequest;
    AssetBundle bundle = bundleRequest.assetBundle;
    AssetBundleRequest assetRequest = bundle.LoadAssetAsync<GameObject>(assetName);
    yield return assetRequest;
    GameObject prefab = assetRequest.asset as GameObject;
    Instantiate(prefab);
}
```

### 1.5 卸载AssetBundle资源文件

在Unity中，可以使用以下步骤卸载AssetBundle资源文件：

1. 使用`AssetBundle.Unload`方法卸载AssetBundle文件，例如：

```csharp
myLoadedAssetBundle.Unload(false); // 参数为false表示只卸载AssetBundle文件，不卸载其中的资源
```

2. 如果需要卸载AssetBundle中的资源，可以使用`AssetBundle.UnloadAsset`方法，例如：

```csharp
myLoadedAssetBundle.UnloadAsset(myObject);
```

3. 如果需要完全卸载AssetBundle文件，可以使用`AssetBundle.Unload`方法，并将参数设置为`true`，例如：

```csharp
myLoadedAssetBundle.Unload(true);
```

4. 卸载全部AssetBundle

```csharp
AssetBundle.UnloadAllAssetBundles(true);
```

### 1.6 AssetBundle不同包之间的依赖

在Unity中，AssetBundle之间可以存在依赖关系，即一个AssetBundle可能依赖于另一个AssetBundle中的资源。在这种情况下，需要确保在加载AssetBundle之前，先加载其依赖的AssetBundle。

例如，假设有两个AssetBundle文件：`bundle1`和`bundle2`，其中`bundle2`依赖于`bundle1`中的资源。在这种情况下，需要先加载`bundle1`，然后再加载`bundle2`，例如：

```csharp
AssetBundle bundle1 = AssetBundle.LoadFromFile(Path.Combine(Application.streamingAssetsPath, "bundle1"));
AssetBundle bundle2 = AssetBundle.LoadFromFile(Path.Combine(Application.streamingAssetsPath, "bundle2"));
```

当一个包依赖于多个包时，就要通过主包来加载依赖包，主包中记录了依赖包的包名，通过路径来加载依赖包。

```csharp
string manifestPath = Path.Combine(Application.streamingAssetsPath, "AssetBundles");
AssetBundle manifestBundle = AssetBundle.LoadFromFile(manifestPath); // 加载主清单 bundle
AssetBundleManifest manifest = manifestBundle.LoadAsset<AssetBundleManifest>("AssetBundleManifest");
string[] dependencies = manifest.GetAllDependencies("bundle1");
foreach (string dependency in dependencies)
{
    AssetBundle.LoadFromFile(Path.Combine(Application.streamingAssetsPath, dependency));
}
//待依赖包加载完毕后再加载目标包
AssetBundle bundle1 = AssetBundle.LoadFromFile(Path.Combine(Application.streamingAssetsPath, "bundle1"));
```

### 1.7 AssetBundle资源加载管理器

前面介绍的都是零散的加载/卸载方法，实际项目中通常会把它们封装成一个统一的资源加载管理器，集中管理包的加载、依赖解析、缓存和卸载。下面是一个完整的 `AssetBundleManager`，包含以下功能：

- **同步加载**：`LoadBundle` / `LoadAsset<T>` / `LoadGameObject`
- **异步加载**：`LoadBundleAsync` / `LoadAssetAsync<T>`（协程实现，回调返回结果）
- **依赖加载**：通过主清单 `AssetBundleManifest` 自动递归加载依赖包
- **缓存与引用计数**：已加载的包不会重复加载；引用计数减到 0 时才真正卸载
- **卸载**：`UnloadBundle`（单个包，自动卸载依赖）/ `UnloadAsset`（单个资源）/ `UnloadAll`（全部）

将以下代码保存为一个 C# 脚本（如 `AssetBundleManager.cs`）挂到任意对象或直接使用单例：

```csharp
using System.Collections;
using System.Collections.Generic;
using System.IO;
using UnityEngine;

/// <summary>
/// AssetBundle资源加载管理器
/// 功能：同步加载、异步加载、依赖加载、缓存、引用计数、卸载
/// </summary>
public class AssetBundleManager : MonoBehaviour
{
    private static AssetBundleManager _instance;
    public static AssetBundleManager Instance
    {
        get
        {
            if (_instance == null)
            {
                GameObject go = new GameObject("AssetBundleManager");
                _instance = go.AddComponent<AssetBundleManager>();
                DontDestroyOnLoad(go); // 跨场景保持
            }
            return _instance;
        }
    }

    /// <summary>AssetBundle存放路径（本地）</summary>
    private string bundlePath = Application.streamingAssetsPath;

    /// <summary>已加载的AssetBundle缓存，key为包名</summary>
    private Dictionary<string, AssetBundle> bundleCache = new Dictionary<string, AssetBundle>();

    /// <summary>包的引用计数，为0时才可以真正卸载</summary>
    private Dictionary<string, int> bundleRefCount = new Dictionary<string, int>();

    /// <summary>主清单bundle及其中的AssetBundleManifest，用于解析依赖</summary>
    private AssetBundle manifestBundle;
    private AssetBundleManifest manifest;

    private string ManifestBundleName 
    {
        get
        {
#if UNITY_ANDROID
            return "manifest_android";
#elif UNITY_IOS
            return "manifest_ios";
#else
            return "manifest_windows";
#endif
        }
    }

    /// <summary>
    /// 初始化：加载主清单（游戏启动时调用一次）
    /// </summary>
    public void Init()
    {
        if (manifestBundle != null)
        {
            manifestBundle.Unload(false);
        }
    }
    /// <summary>
    /// 初始化：加载主清单（游戏启动时调用一次）
    /// </summary>
    public void Init()
    {
        if (manifestBundle != null)
        {
            manifestBundle.Unload(false);
            manifestBundle = null;
            manifest = null;
        }

        manifestBundle = AssetBundle.LoadFromFile(Path.Combine(bundlePath, manifestBundleName));
        if (manifestBundle == null)
        {
            Debug.LogError("加载主清单失败：" + manifestBundleName);
            return;
        }
        manifest = manifestBundle.LoadAsset<AssetBundleManifest>("AssetBundleManifest");
    }

    // ==================== 同步加载 ====================

    /// <summary>
    /// 同步加载AssetBundle（会自动先加载其依赖包）
    /// </summary>
    public AssetBundle LoadBundle(string bundleName)
    {
        // 已在缓存中，直接返回并增加引用计数
        if (bundleCache.TryGetValue(bundleName, out AssetBundle cached) && cached != null)
        {
            bundleRefCount[bundleName]++;
            return cached;
        }

        // 通过主清单找到该包的全部依赖，先加载依赖
        if (manifest != null)
        {
            string[] dependencies = manifest.GetAllDependencies(bundleName);
            foreach (string dep in dependencies)
            {
                LoadBundle(dep);
            }
        }

        AssetBundle bundle = AssetBundle.LoadFromFile(Path.Combine(bundlePath, bundleName));
        if (bundle == null)
        {
            Debug.LogError("加载AssetBundle失败：" + bundleName);
            return null;
        }

        bundleCache[bundleName] = bundle;
        bundleRefCount[bundleName] = 1;
        return bundle;
    }

    /// <summary>
    /// 同步加载包中的某个资源
    /// </summary>
    public T LoadAsset<T>(string bundleName, string assetName) where T : Object
    {
        AssetBundle bundle = LoadBundle(bundleName);
        return bundle == null ? null : bundle.LoadAsset<T>(assetName);
    }

    /// <summary>
    /// 同步加载包中的Prefab并实例化
    /// </summary>
    public GameObject LoadGameObject(string bundleName, string assetName)
    {
        GameObject prefab = LoadAsset<GameObject>(bundleName, assetName);
        return prefab == null ? null : Instantiate(prefab);
    }

    // ==================== 异步加载 ====================

    /// <summary>
    /// 异步加载AssetBundle（协程，自动先加载依赖）
    /// </summary>
    public IEnumerator LoadBundleAsync(string bundleName)
    {
        if (bundleCache.TryGetValue(bundleName, out AssetBundle cached) && cached != null)
        {
            bundleRefCount[bundleName]++;
            yield break;
        }

        if (manifest != null)
        {
            string[] dependencies = manifest.GetAllDependencies(bundleName);
            foreach (string dep in dependencies)
            {
                yield return LoadBundleAsync(dep); // 嵌套协程，先等依赖加载完
            }
        }

        AssetBundleCreateRequest request = AssetBundle.LoadFromFileAsync(Path.Combine(bundlePath, bundleName));
        yield return request;

        AssetBundle bundle = request.assetBundle;
        if (bundle == null)
        {
            Debug.LogError("异步加载AssetBundle失败：" + bundleName);
            yield break;
        }

        bundleCache[bundleName] = bundle;
        bundleRefCount[bundleName] = 1;
    }

    /// <summary>
    /// 异步加载包中的资源，加载完成后通过回调返回
    /// </summary>
    public IEnumerator LoadAssetAsync<T>(string bundleName, string assetName, System.Action<T> onCompleted) where T : Object
    {
        yield return LoadBundleAsync(bundleName);

        if (!bundleCache.TryGetValue(bundleName, out AssetBundle bundle) || bundle == null)
        {
            onCompleted?.Invoke(null);
            yield break;
        }

        AssetBundleRequest request = bundle.LoadAssetAsync<T>(assetName);
        yield return request;

        onCompleted?.Invoke(request.asset as T);
    }

    // ==================== 卸载 ====================

    /// <summary>
    /// 卸载指定包（引用计数减一；减到0时才真正卸载，并递归卸载其依赖包）
    /// </summary>
    public void UnloadBundle(string bundleName, bool unloadAllLoadedObjects = false)
    {
        if (!bundleCache.ContainsKey(bundleName)) return;

        bundleRefCount[bundleName]--;
        if (bundleRefCount[bundleName] > 0) return; // 还有其他地方在用，暂不卸载

        // 先递归卸载它的依赖包
        if (manifest != null)
        {
            string[] dependencies = manifest.GetAllDependencies(bundleName);
            foreach (string dep in dependencies)
            {
                UnloadBundle(dep, unloadAllLoadedObjects);
            }
        }

        bundleCache[bundleName].Unload(unloadAllLoadedObjects);
        bundleCache.Remove(bundleName);
        bundleRefCount.Remove(bundleName);
    }

    /// <summary>
    /// 卸载包中的单个资源对象（注意：不能用于GameObject/Component，它们要用Destroy销毁）
    /// </summary>
    public void UnloadAsset(string bundleName, Object asset)
    {
        if (bundleCache.TryGetValue(bundleName, out AssetBundle bundle) && asset != null)
        {
            bundle.UnloadAsset(asset);
        }
    }

    /// <summary>
    /// 卸载所有AssetBundle（切换场景或整个关卡结束时调用）
    /// </summary>
    public void UnloadAll(bool unloadAllLoadedObjects = true)
    {
        foreach (var kv in bundleCache)
        {
            if (kv.Value != null) kv.Value.Unload(unloadAllLoadedObjects);
        }
        bundleCache.Clear();
        bundleRefCount.Clear();

        if (manifestBundle != null)
        {
            manifestBundle.Unload(false);
            manifestBundle = null;
            manifest = null;
        }
    }

    private void OnDestroy()
    {
        UnloadAll(true);
    }
}
```

## Lua 

### 简单数据结构

```lua
print("************变量*************");
--lua中简单的数据类型
--nil number boolean string

--当使用未被申明的变量时 lua会自动将其赋值为nil
print(b);

--lua中在申明变量时不需要指定类型 ，变量类型由值决定 类似与C#中的var
print("************nil*************");
a=nil;
print(a);
--打印nil

--lua中同一个变量可以装不同类型的数据 
print("************number*************");
a=1;
print(a);
--打印1

print("************boolean*************");
a=true;
print(a);
--打印true

print("************string*************");
--单引号和双引号都可以代表字符串 
a="hello world";
print(a);
a='hello world';
print(a);
--[[打印hello world
hello world]]--

print("************type*************");
--可以通过type函数获取变量的类型
print(type(a));
print(type(1));
print(type(true));
print(type("hello world"));
print(type(nil));
--[[打印string
number
boolean
string
nil]]--

```

### 字符串

```lua
print("**************字符串**************");
str1 = "双引号字符串";
str2 = '单引号字符串';

--获取字符串的长度
print("**************字符串长度**************");
s = "adcdefg你好"
print(#s);
--打印的值为13
--在lua中英文字符占1个长度 一个汉字占3个长度

print("**************字符串多行打印**************");
--通过转义符\n实现多行打印
print("第一行\n第二行\n第三行");
--通过[[]]实现多行打印
s = [[你好
世界]];
print(s);
--[[打印的结果为
你好
世界]]--

--也可以直接通过print打印
print([[
你好
世界]]);

print("**************字符串拼接**************");
--通过..实现字符串拼接
s1 = "你好";
s2 = "世界";
print(s1..s2);
--打印的结果为
--你好世界

--通过table.concat实现字符串拼接
s1 = "你好";
s2 = "世界";
print(table.concat({s1,s2}));
--打印的结果为
--你好世界

--通过string.format实现字符串拼接
s1 = "你好";
s2 = "世界";
print(string.format("%s%s",s1,s2));
--打印的结果为
--你好世界
--%s表示字符串
--%d表示整数
--%f表示浮点数
--%x表示十六进制数
--%o表示八进制数
--%b表示二进制数
--%p表示指针
--%c表示字符
--%u表示无符号整数
--%e表示科学计数法
--%g表示科学计数法或者小数点
--%f表示小数点

print("**************别的类型转字符串**************");
a=true;
print(tostring(a));
--打印的结果为
--true

print("**************字符串提供的公共方法**************");
str = "abCdEf";
--小写转大写方法
print(string.upper(str));
--打印的结果为
--ABCDEF

--大写转小写方法
print(string.lower(str));
--打印的结果为
--abcdef

--字符串长度方法
print(string.len(str));
--打印的结果为
--6

--字符串截取方法
print(string.sub(str,3,5));
--打印的结果为
--CdE 这里结果为CdE的原因是lua中数组的索引是从1开始的

--字符串查找方法
print(string.find(str,"Cd"));
--打印的结果为
--3	4

--字符串替换方法
print(string.gsub(str,"Cd","CD"));
--打印的结果为
--abCDEf 1 这里结果为abCDEf 1的原因是替换了1个

--字符串重复
print(string.rep(str,3));
--打印的结果为
--abCdEfabCdEfabCdEf

--字符串转ASCII码
print(string.byte(str,3));
--打印的结果为
--67

--字符串转字符
print(string.char(67));
--打印的结果为
--C

```

### 运算符

```lua
print("*************运算符**************");

print("*************算术运算符**************");
-- + - * / % ^
--没有自增自减 ++ --
--没有复合运算符 += -= *= /= %=
print("加法运算" .. 1 + 1);
--打印结果为 加法运算2
print("123.4" + 1);
--打印结果为 124.4  lua会将字符串自动的转为number去进行运算 前提是字符串里面的内容可以转为number

print("减法运算" .. 1 - 1);
--打印结果为 减法运算0
print("123.4" - 1);
--打印结果为 122.4

print("乘法运算" .. 1 * 1);
--打印结果为 乘法运算1
print("123.4" * 1);
--打印结果为 123.4

print("除法运算" .. 1 / 2);
--打印结果为 除法运算0.5
print("123.4" / 2);
--打印结果为 61.7

print("取余运算" .. 1 % 1);
--打印结果为 取余运算0
print("123.4" % 1);
--打印结果为 0.4

print("幂运算" .. 2 ^ 3);
--打印结果为 幂运算8

print("*************条件运算符**************");
-- > < >= <= == ~=
print(3>1);
--打印结果为 true
print(3<1);
--打印结果为 false
print(3>=1);
--打印结果为 true
print(3<=1);
--打印结果为 false
print(3==1);
--打印结果为 false
--lua中不等于符号为~=
print(3~=1);
--打印结果为 true

print("*************逻辑运算符**************");
-- and or not lua中也遵循逻辑运算的短路原则
print(3>1 and 3<5);
--打印结果为 true
print(3>1 or 3<5);
--打印结果为 true
print(not (3>1));

print("*************位运算符**************");
-- lua中不支持位运算符

print("*************三目运算符**************")
-- lua中不支持三目运算符

```

### 条件分支语句

```lua
print("***************条件分支语句*****************");
a=9;

--单分支语句
--if 条件 then...end
if a>5 then
    print("true");
end

--双分支语句
--if 条件 then...else...end
if a<5 then
    print("true");
else
    print("false");
end

--多分支语句
--if 条件 then...elseif 条件 then...else...end
if a == 5 then
    print("a==5");
elseif a==10 then
    print("a==10");
else
    print("a就是a")
end

--lua中没有switch语法
```

### 循环

```lua
print("**************循环***************")

print("**************while循环***************")
num = 0
--while 条件 do ... end
while num < 5 do
    print(num)
    num = num + 1
end
print("**************do while循环***************")
num = 0;
--repeat ... until 条件 
repeat
    print(num)
    num = num + 1
until num > 5 --这是退出条件 当num>5时退出循环

print("**************for循环***************")
for i = 1, 5 do --lua的for循环会默认加1
    print(i)
end

for i = 1, 5, 2 do --第三个参数是步长 表示每次循环加2
    print(i)
end

for i = 5, 1, -1 do --第三个参数是步长 表示每次循环减1
    print(i)
end

```

### 函数
```lua
print("*************函数***************")
--function 函数名(参数列表)
--end

--a = function(参数列表)
--end   

print("*************无参数无返回值***************")
--要注意 lua中的代码执行顺序是从上往下执行的 ，所以函数的声明一定要在调用之前
function F1()
    print("hello world")
end
F1()

F2 = function ()
    print("你好世界")
end
F2()

print("*************有参数***************")
function F3(a)
    print(a)
end
F3("hello world")
F3(1)
F3(true)
F3()
--[[打印
hello world
1
true
nil]]--
--这说明lua中的函数的参数类型是可选的，如果没有传入参数，那么函数会默认传入nil 
--若传入的参数类型不能实现函数中的逻辑，那么函数会报错

print("*************有返回值***************")
function F4()
    return "hello world",1,true
end
a,b,c=F4()
print(a)
print(b)
print(c)
--[[打印
hello world
1
true]]--
--这说明lua中的函数可以有多个返回值，返回值之间用逗号隔开，调用函数时用多个变量接收返回值，变量之间用逗号隔开
--如果变量的个数少于返回值的个数，那么多余的返回值会被忽略

print("*************函数的类型***************")
--函数的类型就是function
F5 = function ()
    print("hello world")
end
print(type(F5))
--[[打印
function
]]--

print("*************函数的重载***************")
--lua中函数不支持重载 默认会调用最后一个申明的函数

print("*************变长参数***************")
function F6(...)
    arg = {...}
    for i = 1,#arg do
        print(arg[i])
    end
end
F6(1,2,3,4,5)
--[[打印
1
2
3
4
5]]--
--在函数的参数列表中，使用...表示该函数可以接收任意数量的参数，
--这些参数会被存储在一个table中，可以通过arg[i]来访问这些参数

print("*************函数的嵌套***************")
--相当于将函数作为返回值传出去 这样F9接受到的就是一个函数
function F8()
    return function()
        print("hello world")
    end
end
F9 = F8()
F9()
--[[打印
hello world]]--

```

- **面试考点**
  - 闭包 简单来说就是改变了外部函数变量的生命周期
  - 闭包的优缺点
    - 优点：可以访问函数外部的局部变量
    - 缺点：闭包会常驻内存，会增大内存消耗，使用不当容易造成内存泄露
  - 闭包的典型应用：防抖节流、单例模式、柯里化、函数式编程

```lua
print("*************闭包***************")
function F10()
    local a = 9
    return function()
        a = a + 1
        return a
    end
end
F11 = F10()
print(F11()) --打印10
```

### 表

```lua
print("*************复杂数据类型 table***************")
--所有的复杂数据类型都是table
print("*************数组***************")
a = {1,2,3,"123",true,nil}
print(a[1])
print(a[4])
print(a[5])
print(a[6])
--[[打印结果为
1
123
true
nil]]

--数组长度的获取
print(#a)
--打印的结果为5 这是由于表会忽略nil
--当nil在表中间位置时
--[[Lua 5.1 的 luaH_getn 大致逻辑：表创建时（{[1]=1,...} 这种带显式下标的写法）
初始数组区大小为 0，键先进哈希区；取长度时先用二分/倍增在哈希区里找边界（unbound_search），
找到第一个"非 nil → nil"的转折点就停。所以结果经常是"碰巧经过的那个边界"，而不是最大边界，
更不是元素个数]]--

print("*************数组的遍历***************")
for i=1,#a do
    print(a[i])
end

print("*************二维数组***************")
a = {{1,2,3},{4,5,6},{7,8,9}}
print(a[1][1])
print(a[1][2])
print(a[1][3])
print(a[2][1])
print(a[2][2])
print(a[2][3])
print(a[3][1])
print(a[3][2])
print(a[3][3])
--打印结果为
--1
--2
--3
--4
--5
--6
--7
--8
--9

print("*************二维数组的遍历***************")
for i=1,#a do
    for j=1,#a[i] do
        print(a[i][j])
    end
end

print("*************自定义索引***************")
a = {[1] = 1, [2] = 2, [4] = 4, [6] = 100}
print(a[6])

```

### 迭代器
```lua
print("**************迭代器遍历**************")
--迭代器遍历主要是用来遍历表的
--因为用#去遍历得到的结果不准确

a= {[0] = 0,[1] = 1,[2] = 2, [3] = 3, [-1] = -1,[4] = 4,[100] =100}
--ipairs遍历是从1开始的，无法去遍历索引小于等于0的元素,并且当它遇到nil时就会停止遍历
--即只能遍历连续索引
print("**************ipairs迭代器遍历**************")
for i,k in ipairs(a) do
    print(i,k)
end
--[[打印结果为
1       1
2       2
3       3
4       4
]]
print("**************pairs迭代器遍历**************")
--能够把所有键都找到 i代表键，k代表值 面试就喜欢问pairs和ipairs的区别
for i,k in pairs(a) do
    print(i,k)
end


```

### 表的复杂使用

```lua
print("*************复杂数据类型 table***************")

print("*************字典***************")
print("*************字典的申明***************")
a = {["studentNumber"] = 1, ["studentName"] = "张三",["age"] = 14}

--可以通过如下方式访问
print(a["studentNumber"])
print(a["studentName"])
print(a["age"])

--也可以通过如下方式访问
print(a.studentNumber)
print(a.studentName)
print(a.age)

print("*************字典的遍历***************")
for key, value in pairs(a) do
    print(key, value)
end

print("*************字典的添加***************")
a["studentSex"] = "男"
print(a["studentSex"])

print("*************字典的删除***************")
a["studentSex"] = nil
print(a["studentSex"])


print("*************类和结构体***************")
Student = 
{
    age = 14,
    sex = "男",
    GetStudentSex = function(t)
        print(t.sex)
    end,
    GetStudentSex1 = function()
        print(Student.sex) --若是想要在类的内部访问类的内部成员 需要指定类
    end
}

--可以在外部申明类的成员变量
Student.name = "张三"
--可以在外部申明类的成员函数
Student.GexStudentAge = function(t)
    print(t.age)
end
--也可以用冒号申明
function Student:GetStudentName()
    print(self.name) --self代表冒号申明时作为第一个参数的代表 不是this!!!!
end

--调用
Student.GetStudentSex(Student)
--用冒号也可以调用 ，但是冒号调用时，相当于将调用者作为第一个参数传入
Student:GetStudentSex()
Student:GexStudentAge()
Student:GetStudentName()

print("*************表的公共操作***************")

print("*************插入***************")
table1 = {{"123","231","321"},{"111","222"}}
table2 = {"333"}
table.insert(table1,1,table2) --在table1的索引1处插入table2
print(table1[1][1])

print("*************删除***************")
table.remove(table1,1) --删除table1的索引1处的元素
print(table1[1][1])

table.remove(table1) --默认删除table1的最后一个元素

print("*************排序***************")
table3 = {1,5,3,8,3,6,1,3}
table.sort(table3) --默认从小到大排序
for i,v in pairs(table3) do
    print(v)
end

print("*************自定义排序***************")
table.sort(table3, function(a,b) return a>b end) --从大到小排序 后面的函数参数代表排序规则
for i,v in pairs(table3) do
    print(v)
end


print("*************拼接***************")
table4 = {"123","231","321"}
str = table.concat(table4,"-") --将table4中的元素拼接成字符串，中间用"-"隔开 返回的值是string类型
print(str)
--打印 123-231-321
```

### 多脚本执行

```lua
print("*************多脚本执行***************")

print("*************全局变量和本地变量***************")
--关键字 local
--一般来说 在一个lua脚本中定义的变量都是全局变量
--只有在前面加了local关键字后才定义的是局部变量

--在函数内部定义的变量也同理
function F1() 
    a = 1
end

F1() --调用后个a赋值了 使a成为全局变量
print(a)
--打印1

function F2() 
    local b = 1 --加了local关键字后 b成为局部变量
end

F2()
print(b)
--打印nil

print("*************多脚本执行***************")
--关键字 require("脚本名")
--require在执行一个脚本时 可以在脚本最后返回一个外部希望获取的内容

print("*************脚本卸载***************")
--package.loaded["脚本名"] 返回的是boolean类型 代表脚本是否被加载出来
--package.loaded["脚本名"] = nil 脚本卸载

print("*************大G表***************")
-- _G表是一个总表(table) 他将我们申明的所有全局的变量都存储在其中

```

### 特殊用法

```lua
print("**************特殊用法****************")

print("**************多变量赋值****************")
a, b, c = 1, 2, 3
print(a)
print(b)
print(c)

--当后面的值个数少于前面的变量个数时，后面的变量值为nil
a, b, c = 1, 2
print(a)
print(b)
print(c)
--[[打印
1
2
nil]]

--当后面的值个数多于前面的变量个数时，会忽略多余的值
a, b, c = 1, 2, 3, 4, 5
print(a)
print(b)
print(c)
--[[打印
1
2
3]]

print("**************多返回值****************")
function F1()
    return 1, 2, 3
end

--返回值个数相关和上面同理
a, b, c = F1()
print(a)
print(b)
print(c)
--[[打印
1
2
3]]

print("**************and or****************")
--逻辑与和逻辑或
--and和or在lua中可以连接任意两个类型的变量
--只有nil和false才会被认为是false
--遵循短路规则 当第一个变量的值满足条件时 则后续就不会再执行
print(1 and 2)
--打印 2 因为1为true不满足and的条件 所以要去看后面一个数是否满足 所以打印的是2
print(nil and 3)
--打印nil 因为nil为false 满足and的条件 所以不用看后面一个数是否满足 所以打印的是nil

print(1 or 2)
--打印1 因为1为true 满足or的条件 所以不用看后面一个数是否满足 所以打印的是1
print(nil or 3)
--打印3 因为nil为false 不满足or的条件 所以要去看后面一个数是否满足 所以打印的是3

--自定义三目运算符
x = 1
y = 2
local res = (x>y) and x or y --如果x>y 则res=x 否则res=y 运用了短路的原则 先执行 ((x>y) and x) 再执行 or y
print(res)

```

### 协程

```lua
print("**************协同程序****************")

print("**************协程的创建****************")
function F1()
    print("F1 start")
end
--用create创建
Co1 = coroutine.create(F1) --返回值是thread
print(type(Co1)) --thread

--用wrap创建
Co2 = coroutine.wrap(F1) --返回值是function
print(type(Co2)) --function

print("**************协程的运行****************")
--类型为thread的通过corountine.resume()来运行
coroutine.resume(Co1)
--类型为function的通过()来运行
Co2()
print("**************协程的挂起****************")

function F2()
    i = 1
    while true do
        print(i)
        print(coroutine.status(Co3))
        coroutine.yield(i) --挂起 这里可以有返回值
        i = i + 1
    end
end

Co3 = coroutine.create(F2)
coroutine.resume(Co3) 
coroutine.resume(Co3) --协程挂起后，再次resume，会从上次挂起的位置继续执行
--[[打印的结果为
1
2]]--

Co4 = coroutine.wrap(F2)
Co4()   
Co4() --协程挂起后，再次resume，会从上次挂起的位置继续执行
--[[打印的结果为
1
2]]--

--获取协程挂起的返回值
--当协程类型为therad时
print(coroutine.resume(Co3)) --打印结果为true 3 第一个返回值为协程是否成功执行 第二个返回值为协程挂起的返回值
--当协程类型为function时
print(Co4()) --打印结果为4 直接返回协程挂起的返回值

print("**************协程的状态****************")
--协程状态有 dead 结束 running 运行中 suspended 暂停
--通过coroutine.status()来获取协程状态
print(coroutine.status(Co3)) --打印结果为suspended

--可以得到正在运行的协程的线程号coroutine.running() 要在函数内部调用才行
```

### 元表

```lua
print("**************元表****************")

print("**************元表的概念****************")
--任何表变量都可以作为另一个表变量的元表
--任何表变量都可以有自己的元表
--当子表进行一些特定操作时 会执行元表的一些内容

print("**************设置元表****************")
meta = {}
Mytable = {}
setmetatable(Mytable,meta) --将meta设置为Mytable的元表

print("**************特定操作****************")

print("**************特定操作-__tostring****************")
Meta1 = {
    __tostring = function(self)
        return "this is a table"
    end
}
Mytable2 = {}
setmetatable(Mytable2,Meta1)
--当子表被当作字符串输出时 会执行元表中的__tostring函数
print((Mytable2)) --打印this is a table

print("**************特定操作-__call****************")
Meta2 = {
    __call = function(self,a,b)
        return a+b
    end
}

Mytable3 = {}
setmetatable(Mytable3,Meta2)
--当子表被当作函数调用时 会执行元表中的__call函数
print((Mytable3)(1,2)) --打印3


print("**************特定操作-运算符重载****************")
Meta3 = {
    __add = function(self,a)
        return self.num+a
    end,
    --减法
    __sub = function(self,a)
        return self.num-a
    end,
    --乘法
    __mul = function(self,a)
        return self.num*a
    end,
    --除法
    __div = function(self,a)
        return self.num/a
    end,
    --幂运算
    __pow = function(self,a)
        return self.num^a
    end,
}

Mytable4 = {num = 1}
setmetatable(Mytable4,Meta3)
--当子表进行加法运算时 会执行元表中的__add函数
print(Mytable4+1) --打印3

print("**************特定操作-__index和__newIndex****************")
--__index
Meta4 = {
    age = 1;
    --不要在表内使用__index = Meta4，因为此时Meta4还未被定义
}
Meta4.__index = Meta4 --设置元表的__index为元表自身
Mytable5 = {}
setmetatable(Mytable5,Meta4)
--当子表访问不存在的属性时 会执行元表中的__index函数
print(Mytable5.age) --打印1
--如果只想在子表中查找属性 而不查找元表中的属性 需要使用rawget函数
print(rawget(Mytable5,"age")) --打印nil

--__newindex
Meta5 = {}
Meta5.__newindex = Meta5
Mytable6 = {}
setmetatable(Mytable6,Meta5)
--当子表设置不存在的属性时 会执行元表中的__newindex函数
Mytable6.name = "Tom" --打印set name = Tom
print(Mytable6.name) --打印Tom
print(Meta5.name) --打印Tom

--如果想要在子表中添加属性 而不是在元表中添加属性 需要使用rawset函数
Mytable6.rawset(Mytable6,"name","Jerry")
```

### 面向对象

```lua
print("***************面向对象****************")

print("***************封装****************")
--面向对象 类 其实都是基于table来实现的
Object = {}
Object.id = 1

function Object:new()
    local obj = {}
    self.__index = self
    setmetatable(obj, self)
    return obj
end

function Object:Test()
    print("test")
end

myObject = Object:new() --调用这个方法后相当于实例化了一个类对象
print(myObject.id) --通过__index去找Object中的id属性
myObject.id = 2 --相当于在myObject中添加了一个id属性
print(myObject.id)
print(Object.id) --但是不会改变元表中的属性

myObject:Test() --调用元表中的方法

print("***************继承****************")
--继承
--继承其实就是将一个table作为另一个table的元表
--继承的table会自动继承元表中的方法

function Object:SubClass(ClassName)
    _G[ClassName] = {} --_G也是一个表，通过这种方法在全局中创建了一个新的类
    local subClass = _G[ClassName]
    --实现基类
    subClass.base = self
    self.__index = self
    setmetatable(subClass, self)
end 

Object:SubClass("Sub") --subObj继承了Object
Sub.name = "subObj"
mySubObj = Sub:new() --实例化subObj 

print(mySubObj.name)

print("***************多态****************")
Object:SubClass("GameObject")
GameObject.posX = 0
GameObject.posY = 0
function GameObject:Move(x, y)
    self.posX = self.posX + x
    self.posY = self.posY + y
    print(self.posX..","..self.posY)
end
GameObject:SubClass("Player")
function Player:Move(x, y)
    print("player move")
    self.base.Move(self, x, y) --调用基类中的方法 一定要用.去调用防止冒号调用将GameObject基类传入方法中导致子类在使用父类里的变量
end

GameObject:SubClass("Enemy")
function Enemy:Move(x, y)
    print("enemy move")
    self.base.Move(self, x, y)
end

player = Player:new()
player:Move(1, 2)

enemy = Enemy:new()
enemy:Move(3, 4)

--多态
--多态就是同一个方法，不同的对象有不同的实现
--多态的实现就是通过元表中的方法来实现的
--在调用方法时，会先在当前对象中查找方法，如果没有找到，就会在元表中查找方法
```

### 自带库

```lua
print("**************自带库****************")

print("**************时间****************")
--系统时间
print(os.time())

print(os.time({year=2021,month=10,day=1,hour=12,minute=30,second=45}))

--获取当前时间
local nowTime = os.date("*t") --返回的是table类型
for key, value in pairs(nowTime) do
    print(key, value)
end

print("**************常用数学工具****************")
--math
--绝对值
print(math.abs(-10))

--向上取整
print(math.ceil(10.1))

--向下取整
print(math.floor(10.9))

--取最大值
print(math.max(10, 20))

--取最小值
print(math.min(10, 20))

--取随机数种子
print(math.randomseed(os.time()))

--取随机数 要先设置随机数种子
print(math.random())

--取随机数范围
print(math.random(1, 10))

--取平方根
print(math.sqrt(9))

--弧度转角度
print(math.deg(1.5708))

--角度转弧度
print(math.rad(90))

--三角函数 传弧度
print(math.sin(math.pi))

--小数分离
print(math.modf(10.5))

print("**************路径****************")
--lua脚本加载路径
print(package.path)
--路径改变
package.path = package.path..";C:\\"
```

### 垃圾回收

```lua
print("****************垃圾回收*****************")
--垃圾回收关键字 collectgarbage

print(collectgarbage("count")) --返回当前Lua解释器中所有对象占用的内存的字节数 k字节 用返回值*1024可以得到具体占用字节数

test = {1,2,3,4,5}

print(collectgarbage("count")) 

test = nil --将变量置为nil，释放内存

collectgarbage("collect") --手动触发垃圾回收

print(collectgarbage("count")) 

--lua中有自动定时进行垃圾回收的方法
--但在unity热更新开发中尽量不要使用该方法

```

## xLua

### C#CallLua

#### lua解析器

```Csharp
using System.Collections;
using System.Collections.Generic;
using UnityEngine;
using XLua;

public class Lesson1_LuaEnv : MonoBehaviour
{
    // Start is called before the first frame update
    void Start()
    {
        //Lua解析器 能够让我们在unity中执行lua语言
        //一般来说要保持解析器的唯一性
        LuaEnv luaEnv = new LuaEnv();

        //执行lua语言
        luaEnv.DoString("print(\" 你好世界\")");

        //通过lua中的require去执行一个lua脚本
        //默认寻找到路径是在Resources文件夹下 lua脚本后要加txt后缀
        //luaEnv.DoString("require(\"\")");

        //清楚Lua中我们没有手动去释放的对象 垃圾回收
        //帧更新中定时执行或 切场景中执行
        luaEnv.Tick();

        //销毁lua解析器
        luaEnv.Dispose();
    }

    // Update is called once per frame
    void Update()
    {
        
    }
}
```

#### lua文件加载重定向

```csharp
using System;
using System.Collections;
using System.Collections.Generic;
using UnityEngine;
using XLua;

public class Lesson2_Loader : MonoBehaviour
{
    // Start is called before the first frame update
    void Start()
    {
        LuaEnv luaEnv = new LuaEnv();

        //传入一个自定义寻找路径的函数用于去寻找lua文件
        luaEnv.AddLoader(CustomLoader);

        luaEnv.DoString("require('TestLoader')");
    }

    // Update is called once per frame
    void Update()
    {
        
    }

    private byte[] CustomLoader(ref string filepath)
    {
        //自定义寻找路径
        string path = Application.dataPath + "/LuaScripts/" + filepath + ".lua";
        if (System.IO.File.Exists(path))
        {
            return System.IO.File.ReadAllBytes(path);
        }
        else
        {
            Debug.Log("Can not find file: " + path);
        }
        return null;
    }
}

```

#### lua管理器

```csharp
using System.Collections;
using System.Collections.Generic;
using UnityEngine;
using XLua;

public class LuaMgr : BaseManager<LuaMgr>
{
    private LuaEnv luaEnv;

    /// <summary>
    /// 得到lua中的_G
    /// </summary>
    public LuaTable GlobleTable
    {
        get
        {
            return luaEnv.Global;
        }
    }

    public void Init()
    {
        if(luaEnv != null)
        {
            return;
        }

        luaEnv = new LuaEnv();
        luaEnv.AddLoader(CustomLoader);
        luaEnv.AddLoader(CustomABLoader);
    }

    /// <summary>
    /// 执行lua文件 
    /// </summary>
    /// <param name="fileName">lua文件名</param>
    public void DoLuaFile(string fileName)
    {
        string _fileName = string.Format("require('{0}')",fileName);
        DoString(_fileName);
    }

    public void DoString(string str)
    {
        if(luaEnv == null)
        {
            Debug.Log("lua解析器未初始化");
        }
        luaEnv.DoString(str);
    }
    
    /// <summary>
    /// lua垃圾清除
    /// </summary>
    public void Tick()
    {
        if(luaEnv == null)
        {
            Debug.Log("lua解析器未初始化");
        }
        luaEnv.Tick();
    }

    /// <summary>
    /// lua解析器销毁
    /// </summary>
    public void Dispose()
    {
        if(luaEnv == null)
        {
            Debug.Log("lua解析器未初始化");
        }
        luaEnv.Dispose();
    }

    /// <summary>
    /// 自定义重定向方法
    /// </summary>
    /// <param name="fileName">文件名</param>
    /// <returns></returns>
    public byte[] CustomLoader(ref string fileName)
    {
        //自定义寻找路径
        string path = Application.dataPath + "/LuaScripts/" + fileName + ".lua";
        if (System.IO.File.Exists(path))
        {
            return System.IO.File.ReadAllBytes(path);
        }
        else
        {
            Debug.Log("Can not find file: " + path);
        }
        return null;
    }

    /// <summary>
    /// 自定义AB包重定向方法
    /// </summary>
    /// <param name="fileName">文件名</param>
    /// <returns></returns>
    public byte[] CustomABLoader(ref string fileName)
    {
        TextAsset luaFile = ABMgr.GetInstance().LoadRes<TextAsset>("lua",fileName + ".lua");
        if(luaFile == null)
        {
            Debug.Log("AB包lua脚本重定向失败 检查文件名是否拼写正确" + fileName);
            return null;
        }

        return luaFile.bytes;
    }
}
```

#### C#访问lua全局变量

```csharp
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

public class Lesson4_CallValue : MonoBehaviour
{
    // Start is called before the first frame update
    void Start()
    {
        LuaMgr.GetInstance().Init();

        LuaMgr.GetInstance().DoLuaFile("Main");

        //访问Lua中的值
        int i  = LuaMgr.GetInstance().GlobleTable.Get<int>("TestValue");
        //设置lua中的值
        LuaMgr.GetInstance().GlobleTable.Set("TestValue", 100);

        //指定类型List访问 值拷贝
        List<int> list = LuaMgr.GetInstance().GlobleTable.Get<List<int>>("TestList");
        //不指定类型List访问
        List<object> list2 = LuaMgr.GetInstance().GlobleTable.Get<List<object>>("TestList");
 
        //指定类型Dictionary访问 值拷贝
        Dictionary<string, int> dic = LuaMgr.GetInstance().GlobleTable.Get<Dictionary<string, int>>("TestDic");
        //不指定类型Dictionary访问
        Dictionary<object, object> dic2 = LuaMgr.GetInstance().GlobleTable.Get<Dictionary<object, object>>("TestDic");
    }

    // Update is called once per frame
    void Update()
    {
        
    }
}

```

#### C#调用lua函数

```csharp
using System;
using System.Collections;
using System.Collections.Generic;
using Unity.VisualScripting;
using UnityEngine;
using UnityEngine.Events;
using XLua;

public class Lesson5_CallFunction : MonoBehaviour
{
    // Start is called before the first frame update
    private delegate void CallFunc();
    [CSharpCallLua]
    private delegate int CallFunc2(int a);
    [CSharpCallLua]
    private delegate void CallFunc3(int a,out int b,out bool c,out string d);
    [CSharpCallLua]
    private delegate void CallFunc4(int a,params string[] s);
    void Start()
    {
        LuaMgr.GetInstance().Init();
        LuaMgr.GetInstance().DoLuaFile("Main");

        //无参无返回
        //委托
        CallFunc func = LuaMgr.GetInstance().GlobleTable.Get<CallFunc>("TestFunc");
        func();
        //unity自带委托
        UnityAction unityAction = LuaMgr.GetInstance().GlobleTable.Get<UnityAction>("TestFunc");
        unityAction.Invoke();
        //C#自带委托
        Action action = LuaMgr.GetInstance().GlobleTable.Get<Action>("TestFunc");
        action.Invoke();
        //xLua提供的一种获取函数的方式 少用 因为会造成性能消耗 
        LuaFunction lf = LuaMgr.GetInstance().GlobleTable.Get<LuaFunction>("TestFunc");
        lf.Call();

        //有参有返回
        //委托
        CallFunc2 func2 = LuaMgr.GetInstance().GlobleTable.Get<CallFunc2>("TestFunc2");
        int i = func2(1);
        //C#自带委托
        Func<int, int> action2 = LuaMgr.GetInstance().GlobleTable.Get<Func<int, int>>("TestFunc2");
        int k = action2(2);
        //xLua
        LuaFunction lf2 = LuaMgr.GetInstance().GlobleTable.Get<LuaFunction>("TestFunc2");
        int j = lf2.Call(3)[0].ConvertTo<int>();

        //多返回
        //委托
        CallFunc3 func3 = LuaMgr.GetInstance().GlobleTable.Get<CallFunc3>("TestFunc3");
        int a = 0;
        bool b = false;
        string c = "";
        func3(1, out a, out b, out c);

        //xLua
        LuaFunction lf3 = LuaMgr.GetInstance().GlobleTable.Get<LuaFunction>("TestFunc3");
        System.Object[] obj = lf3.Call(1);
        for (int n = 0; n < obj.Length; n++)
        {
            var v = obj[n];
        }
        
        //变长参数
        //委托
        CallFunc4 func4 = LuaMgr.GetInstance().GlobleTable.Get<CallFunc4>("TestFunc4");
        func4(1, "a", "b", "c");
        //xLua
        LuaFunction lf4 = LuaMgr.GetInstance().GlobleTable.Get<LuaFunction>("TestFunc4");
        lf4.Call(1, "a", "b", "c"); 

    }

    // Update is called once per frame
    void Update()
    {
        
    }
}

```

#### C#调用lua类

```csharp
using System;
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

//类中成员变量名必须与lua中变量名一致 且修饰符必须为public
//如果类中变量比lua那边的少 则会忽略掉少的部分
//如果类中变量比lua那边的多 则不会去初始化多出来的部分
public class CallLuaClass
{
    public int testInt;
    public float testFloat;
    public string testString;
    public bool testBool;
    public Action testFunc;
}
public class Lesson6_CallClass : MonoBehaviour
{
    // Start is called before the first frame update
    void Start()
    {
        LuaMgr.GetInstance().Init();
        LuaMgr.GetInstance().DoLuaFile("Main");

        //访问Lua中的类 值拷贝
        CallLuaClass callLuaClass = LuaMgr.GetInstance().GlobleTable.Get<CallLuaClass>("callLuaClass");

    }

    // Update is called once per frame
    void Update()
    {
        
    }
}

```

#### C#调用lua接口

```csharp
using System;
using System.Collections;
using System.Collections.Generic;
using UnityEngine;
using XLua;
//接口不能用成员变量去接受
//要使用属性 其余规则和类相同
[CSharpCallLua] //要注意标特性
public interface ICallLuaInterface
{
    int testInt
    {
        get;
        set;
    }

    float testFloat
    {
        get;
        set;
    }

    bool testBool
    {
        get;
        set;
    }

    string testString
    {
        get;
        set;
    }

    Action testFunc
    {
        get;
        set;
    }
}
public class Lesspn7_CallInterface : MonoBehaviour
{
    // Start is called before the first frame update
    void Start()
    {
        LuaMgr.GetInstance().Init();
        LuaMgr.GetInstance().DoLuaFile("Main");

        //访问lua中的接口 引用拷贝 改变里面的值后 lua表中的值也会跟着改变 ！！！！！！！！！
        ICallLuaInterface iCallLuaInterface = LuaMgr.GetInstance().GlobleTable.Get<ICallLuaInterface>("ICallLuaInterface");
    }

    // Update is called once per frame
    void Update()
    {
        
    }
}

```

### LuaCallC#

#### Lua调用C#类

在C#中有自定义类
```csharp
public class Test1
{
    public Test1()
    {
        
    }
    public void TestMethod()
    {
        Debug.Log("This is a test method");
    }
}

namespace Test
{
    public class Test2
    {
        public Test2()
        {

        }
        public void TestMethod()
        {
            Debug.Log("This is a test method");
        }
    }
}
```

```lua
print("***************lua调用Csharp类相关知识点***************")

--lua中调用Csharp中的类
--CS.命名空间.类名
--Unity的类 比如 GameObject Transform等等 —— CS.UnityEngine.GameObject

--实例化对象
local obj = CS.UnityEngine.GameObject("cube");

--可以用一个把CS.UnityEngine.GameObject赋值给一个全局变量，方便使用

GameObject = CS.UnityEngine.GameObject

--类中的静态方法可以直接通过.调用
local objTemp = GameObject.Find("cube")

--得到找到的对象的成员变量
print(objTemp.transform.position);

--如果调用对象中的成员方法一定要用：!!!!!!!!!!!
objTemp:GetComponent("CubeController")

--自定义类使用方法相同 只是要注意命名空间

--对于在Csharp中没有规定命名空间的
local test = CS.Test1()
test:TestMethod()

--在Csharp中规定了命名空间的
local test2 = CS.Test.Test2()
test2:TestMethod()

--关于在lua中给场景上的物体添加脚本
--用xLua提供的typeof()传type给AddComponent
local objTest = GameObject("test")
objTest:AddComponent(typeof(CS.LuaCallCsharp))

```

#### Lua调用C#枚举

在C#中有自定义枚举
```csharp
public enum MyEnum
{
    Idle,
    Move,
    Attack
}
```

```lua
print("***************lua调用Csharp枚举相关知识点***************")

--枚举调用
--调用unity中的枚举
--枚举的调用规则和类的调用规则是一样的

GameObject = CS.UnityEngine.GameObject
PrimitiveType = CS.UnityEngine.PrimitiveType

local cube = GameObject.CreatePrimitive(PrimitiveType.Cube)

--自定义枚举也差不多 也只是要注意枚举的命名空间
local enumType = CS.MyEnum
--数字转枚举
local a = enumType.__CastFrom(1);
--字符串转枚举
local b = enumType.__CastFrom("Move");
```

#### Lua调用C#数组、字典、List

C#中自定义的数组、字典、List

```csharp
public class Lesson3
{
    public int[] array = new int[5] { 1, 2, 3, 4, 5 };
    public List<int> list = new List<int>();
    public Dictionary<int, string> dict = new Dictionary<int, string>();
}
```

```Lua
print("***************lua调用Csharp数组、List、字典相关知识点***************")

print("***************数组***************")
Class = CS.Lesson3()

local array = Class.array

--遍历
for i = 0, array.Length - 1 do
    print(array[i])
end

--修改
array[0] = 3
print(array[0])

--Lua中创建一个C#的数组
local array2 = CS.System.Array.CreateInstance(typeof(CS.System.String), 13)
array2[0] = "Lua创建的数组"
print(array2[0])

print("***************List***************")
local list = Class.list

--添加
list:Add(1)
list:Add(2)
list:Add(3)

--获取长度
print(list.Count)

--遍历
for i =0 , list.Count - 1 do
    print(list[i])
end

--Lua中创建一个C#的List
local List_Int32 = CS.System.Collections.Generic.List(CS.System.String)
local list2 = List_Int32()

list2:Add("22222")
print(list2[0])

print("***************字典***************")
local dict = Class.dict

--添加
dict:Add(1, "1")
dict:Add(2, "2")
dict:Add(3, "3")

--获取长度
print(dict.Count)

--遍历
for i ,v in pairs(dict) do
    print(i, v)
end

--Lua中创建一个C#的字典
local Dic_Int32_String = CS.System.Collections.Generic.Dictionary(CS.System.String, CS.System.String)
local dic2  = Dic_Int32_String()
dic2:Add("22", "111112341321311")
print(dic2:TryGetValue("22"))
--只有当字典的键值为string类型时才能用这两个方法 
dic2:set_Item("22","23232")
print(dic2:get_Item("22"))



```

#### Lua调用C#拓展方法

```csharp
//lua要使用C#中的拓展方法必须要加此特性且要生成代码
//建议以后lua要访问的C#类都加此特性 可以提升性能
//因为不加特性的化lua是通过反射去获取类的 效率较低
[LuaCallCSharp]
public static class Tool
{
    public static void Print(this Lesson4 lesson4)
    {
        Debug.Log("打印了一些东西");
    }
}

public class Lesson4
{
    public void TestMethod()
    {
        Debug.Log("This is a test method");
    }
}
```

```lua
print("***************lua调用Csharp拓展方法**************")

local Class = CS.Lesson4()

Class:TestMethod()
Class:Print()

```

#### Lua调用C#ref和out

```csharp
public class Lesson5
{
    public int RefFunc(int a,ref int b, ref int c)
    {
        c = a + b;
        return a + b;
    }

    public int OutFunc( int a, out int b,out int c)
    {
        b = a;
        c = a;
        return a;
    }

    public int AllFunc(int a, ref int b, out int c)
    {
        c = a + b;
        return a + b;
    }
}
```

```lua
print("***************lua调用CsharpRefAndOut**************")

local Class = CS.Lesson5()

--ref这边需要占位符
--a代表函数默认返回值(如果有的话)
--b代表第一个ref参数
--c代表第二个ref参数
--注意 如果传入的ref位置为变量 由于lua中不会传引用 你实际上传的那个变量不会变 只有把返回值跟他赋值一次才行
local a,b,c = Class:RefFunc(3,1,1)

print("a=",a)
print("b=",b)
print("c=",c)

--out不需要占位符直接忽略即可
--d代表函数的默认返回值(如果有的话)
--e代表第一个out参数
--f代表第二个out参数
local d,e,f = Class:OutFunc(1)

print("d=",d)
print("e=",e)
print("f=",f)

--综合两者规则
local g,h,i = Class:AllFunc(1,1)
print("g=",g)
print("h=",h)
print("i=",i)


```

#### Lua调用C#重载函数

```csharp
public class Lesson6
{
    public int Calc()
    {
        return 1;
    }

    public int Calc(int a,int b)
    {
        return a + b;
    }

    public int Calc(int a)
    {
        return a;
    }

    public float Calc(float a)
    {
        return a;
    }

    public string Calc(string a)
    {
        return a;
    }
}
```

```lua
print("***************lua调用Csharp重载**************")

local Class = CS.Lesson6()

print(Class:Calc())
print(Class:Calc(1, 2))

--lua中在调用这种float精度类型的重载时会出现误差（虽然不会报错）
--所以在lua中调用C#重载时，尽量避免使用多精度类型
print(Class:Calc(1))
print(Class:Calc(1.1))

print(Class:Calc("23123"))

--解决重载函数模糊的问题
--xlua提供了解决方案 反射机制
--Type是反射的关键类
--得到指定函数的相关信息

local m1 = typeof(CS.Lesson6):GetMethod("Calc",{typeof(CS.System.Int32)})
local m2 = typeof(CS.Lesson6):GetMethod("Calc",{typeof(CS.System.Single)})

local f1 = xlua.tofunction(m1)
local f2 = xlua.tofunction(m2)
--第一个参数传调用对象 第二个参数传值
print(f1(Class,1))
print(f2(Class,1.1))
```

#### Lua调用C#委托和事件

```csharp
public class Lesson7
{
    public Action action;
    public event Action eventAction;

    public void DoEvent()
    {
        eventAction?.Invoke();
    }

    public void ClearEvent()
    {
        eventAction = null;
    }
}
```

```lua
print("***************lua调用Csharp委托和事件**************")

local Class = CS.Lesson7()


local func= function ()
    print("lua调用Csharp委托")
end
print("***************委托**************")
--注意委托刚从C#拿出来时不能直接+=要先赋一个值后才能+=
Class.action = func

Class.action = Class.action + func

Class.action()

--删除
Class.action = Class.action - func

--清除
Class.action = nil

local func2= function ()
    print("lua调用Csharp事件")
end
print("***************事件**************")
--事件
--事件的订阅和取消订阅与委托很不一样
Class:eventAction("+",func2)
Class:eventAction("+",func2)

Class:DoEvent()

--取消订阅
Class:eventAction("-",func2)

Class:DoEvent()

--事件的清空只能写在C#中
Class:ClearEvent()
```

#### Lua调用C#二维数组

```csharp
public class Lesson8
{
    public int[,] array = new int[2,2] { { 1, 2 }, { 3, 4 } };
}
```

```lua
print("***************lua调用Csharp二维数组**************")

local Class = CS.Lesson8()

--得到行
print("行数"..Class.array:GetLength(0))

--得到列
print("列数"..Class.array:GetLength(1))

--访问二维数组的值不能简单地通过[]获得
--要使用GetValue方法
for i=0,Class.array:GetLength(0)-1 do
    for j=0,Class.array:GetLength(1)-1 do
        print("第"..i.."行第"..j.."列的值为"..Class.array:GetValue(i,j))
    end
end
```

#### nil和null的比较

由于lua中nil和C#中的null是不同的类型 所以不能直接比较
可以通过全局函数的方法 申明一个比较函数 当要比较时调用

```lua

function IsNull(obj)
    return obj == nil or obj:Equals(nil)
end

```

或者在C#中定义一个拓展方法

```csharp
[LuaCallCSharp]
public static class Tool
{
    public static bool IsNull(this object obj)
    {
        return obj == null;
    }
}
```

#### 使lua能够调用系统类

写一个静态类
```csharp
public static class GenConfig
{
    [CsharpCallLua]
    public static List<Type> list1 = new List<Type>() {
        typeof(UnityAction<float>),
    };
    [LuaCallCSharp]
    public static List<Type> list2 = new List<Type>() {
        typeof(GameObject),
    };
}
```

#### Lua调用C#协程

```lua
print("**********Lua调用C# 协程相关知识点**********")
--xlua提供的一个工具表
--一定要通过require调用之后 才能用
util = require("xlua.util")
--C#中协程启动都是通过继承了Mono的类 通过里面的启动函数StartCoroutine
GameObject = CS.UnityEngine.GameObject
WaitForSeconds = CS.UnityEngine.WaitForSeconds
--在场景中新建一个空物体 然后挂一个脚本上去 脚本继承mono使用它来开启协程
local obj = GameObject("Coroutine")
local mono = obj:AddComponent(typeof(CS.LuaCallCSharp))
--希望用来被开启的协程函数
fun = function()
    local a = 1
    while true do
        --lua中 不能直接使用 C#中的 yield return
        --就使用lua中的协程返回
        coroutine.yield(WaitForSeconds(1))
        print(a)
        a = a + 1
        if a > 10 then
            --停止协程和C#当中一样
            mono:StopCoroutine(b)
        end
    end
end
--我们不能直接将 lua函数传入到开启协程中！！！！！
--如果要把lua函数当做协程函数传入
--必须 先调用 xlua.util中的cs_generator(lua函数)
b = mono:StartCoroutine(util.cs_generator(fun))

```
