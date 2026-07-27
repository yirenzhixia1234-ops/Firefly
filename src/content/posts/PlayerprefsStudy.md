---
title: Playerprefs学习
published: 2026-07-24
pinned: false
description: Unity Playerprefs学习
tags: [Unity,学习,数据持久化]
author: boluobao
draft: true
---

# Playerprefs学习

Playerprefs是Unity提供的可以存储读取玩家数据的公开类

## Playerprefs的存储

Playerprefs的数据存储类似于键值对存储 一个键对应一个值  
它提供了存储三种数据的方法 int float string 
键：string
值：int float string

```csharp
//直接调用Set方法数据只会存在内存中 不会存在硬盘中
PlayerPrefs.SetInt("Score", 100);
PlayerPrefs.SetFloat("Volume", 0.5f);
PlayerPrefs.SetString("Name", "Player1");
//调用Save方法将数据保存到硬盘中
PlayerPrefs.Save();

//Playerprefs的存储是存在局限性的 它只能存储简单的数据类型 int float string
//如果需要存储复杂的对象类型 则只能降低精度或上升精度来进行存储
```

## Playerprefs的读取
```csharp
//读取整数数据
int score = PlayerPrefs.GetInt("Score", 0);
//读取浮点数数据
float volume = PlayerPrefs.GetFloat("Volume", 0.5f);
//读取字符串数据
string name = PlayerPrefs.GetString("Name", "Player1");

//Get方法里第二个参数是默认值 如果键不存在则返回默认值

```

## Playerprefs的存储位置

> PlayerPrefs 在不同平台上的存储位置**完全不同**，但都是键值对形式的持久化存储。存储路径由 Unity 的 **Company Name** 和 **Product Name** 决定（在 `Edit → Project Settings → Player` 中设置）。

---

### 🪟 Windows

> Windows 平台上，PlayerPrefs 存储在**注册表**中。

```
注册表路径：
HKEY_CURRENT_USER\Software\[Company Name]\[Product Name]
```

| 示例配置 | 注册表路径 |
|----------|-----------|
| Company: `MyCompany`, Product: `MyGame` | `HKCU\Software\MyCompany\MyGame` |
| Company: `DefaultCompany`, Product: `MyProject`（Unity默认） | `HKCU\Software\DefaultCompany\MyProject` |

**查看方式：**

1. `Win + R` → 输入 `regedit` → 回车打开注册表编辑器
2. 导航到 `HKEY_CURRENT_USER\Software\[Company Name]\[Product Name]`
3. 右侧面板可以看到所有键值对

```
注册表编辑器中的显示：
┌─────────────────────────────────────────────────────┐
│  名称              类型          数据                │
│  Score_h205074379  REG_DWORD     0x00000064 (100)   │
│  Volume_h205074379 REG_BINARY    3F 00 00 00        │
│  Name_h205074379   REG_SZ        "Player1"          │
└─────────────────────────────────────────────────────┘
```

> [!NOTE] 键名后的哈希后缀
> 键名末尾的 `_h205074379` 是 Unity 根据 key 字符串生成的**哈希值**，用于防止键名冲突。原始 key 在注册表中不可见（被哈希保护）。

---

### 🤖 Android

> Android 平台上，PlayerPrefs 存储在应用的**内部存储**中，实质是一个 XML 文件。

```
文件路径：
/data/data/[Package Name]/shared_prefs/[Package Name].v2.playerprefs.xml
```

| 示例配置 | 文件路径 |
|----------|----------|
| Package: `com.mycompany.mygame` | `/data/data/com.mycompany.mygame/shared_prefs/com.mycompany.mygame.v2.playerprefs.xml` |

**XML 文件内容示例：**

```xml
<?xml version='1.0' encoding='utf-8' standalone='yes' ?>
<map>
    <int name="Score" value="100" />
    <float name="Volume" value="0.5" />
    <string name="Name">Player1</string>
</map>
```

> 底层使用的是 Android 原生的 `SharedPreferences` 机制。

**限制：**

| 限制项 | 说明 |
|--------|------|
| **单条数据上限** | 约 1.4 MB（SharedPreferences 限制） |
| **总存储上限** | 受设备存储空间限制 |
| **安全性** | ==明文 XML 存储==，root 设备可直接读取修改 |

**查看方式：**

```
# 通过 adb 查看（需要 root 或 debuggable 应用）
adb shell
run-as com.mycompany.mygame
cat /data/data/com.mycompany.mygame/shared_prefs/com.mycompany.mygame.v2.playerprefs.xml

# 或使用 adb pull 导出
adb pull /data/data/com.mycompany.mygame/shared_prefs/com.mycompany.mygame.v2.playerprefs.xml
```

---

### 🍎 iOS

> iOS 平台上，PlayerPrefs 使用系统的 `NSUserDefaults` 存储，数据保存在 plist 文件中。

```
文件路径：
[App Sandbox]/Library/Preferences/[Bundle Identifier].plist

实际沙盒路径：
/var/mobile/Containers/Data/Application/[Application UUID]/Library/Preferences/[Bundle ID].plist
```

| 示例配置 | 文件路径 |
|----------|----------|
| Bundle: `com.mycompany.mygame` | `.../Library/Preferences/com.mycompany.mygame.plist` |

**plist 文件内容示例：**

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
    <key>Score</key>
    <integer>100</integer>
    <key>Volume</key>
    <real>0.5</real>
    <key>Name</key>
    <string>Player1</string>
</dict>
</plist>
```

> [!NOTE] 与 Android 的差异
> iOS 的 NSUserDefaults 是**内存缓存 + 异步写入磁盘**的。Unity 调用 `PlayerPrefs.Save()` 时会强制同步（`[NSUserDefaults synchronize]`），确保数据写入磁盘。

---

### 📊 三平台对比总览

| 对比维度 | 🪟 Windows | 🤖 Android | 🍎 iOS |
|----------|:---:|:---:|:---:|
| **存储方式** | 注册表 | XML 文件 (SharedPreferences) | plist 文件 (NSUserDefaults) |
| **存储路径** | `HKCU\Software\[Company]\[Product]` | `/data/data/[Package]/shared_prefs/` | `[Sandbox]/Library/Preferences/` |
| **文件格式** | 注册表键值 | XML 明文 | plist (XML/Binary) |
| **数据安全** | ❌ 注册表可被任意读取 | ❌ root 设备可读 | ❌ 越狱设备可读 |
| **数据上限** | 无硬性限制 | 单条约 1.4 MB | 无硬性限制 |
| **决定因素** | Company Name + Product Name | Package Name | Bundle Identifier |
| **同步时机** | `Save()` 写入注册表 | `Save()` 写入磁盘 | `Save()` 调用 synchronize |

---

### 🔐 安全性提醒

> [!WARNING] PlayerPrefs 不是安全存储
> 所有平台的 PlayerPrefs 都是**明文存储**，不具备任何加密保护。以下数据 **不要** 存入 PlayerPrefs：
> - ❌ 密码、Token、API Key
> - ❌ 付费状态、会员到期时间
> - ❌ 游戏核心数据（防止玩家篡改）
>
> 敏感数据应使用：
> - `PlayerPrefs` → 仅用于==用户偏好设置==（音量、画质、语言等）
> - 加密存储 / `Application.persistentDataPath` + 自定义序列化 → 游戏进度
> - 服务器验证 → 付费状态、防作弊数据

---

### 🛠️ 实用 API

```csharp
// 查看持久化数据目录（Android/iOS 的文件根目录）
Debug.Log(Application.persistentDataPath);
// Android: /storage/emulated/0/Android/data/[Package]/files
// iOS:     /var/mobile/Containers/Data/Application/[UUID]/Documents
// Windows: C:\Users\[用户名]\AppData\LocalLow\[Company]\[Product]

// 删除所有 PlayerPrefs 数据
PlayerPrefs.DeleteAll();
PlayerPrefs.Save();

// 删除指定键
PlayerPrefs.DeleteKey("Score");
PlayerPrefs.Save();

// 检查键是否存在
bool hasScore = PlayerPrefs.HasKey("Score");
```

| API | 说明 |
|-----|------|
| `Application.persistentDataPath` | 持久化数据根目录（可写、不随更新删除） |
| `PlayerPrefs.DeleteAll()` | 删除所有键值（==不可逆，慎用==） |
| `PlayerPrefs.DeleteKey(key)` | 删除指定键 |
| `PlayerPrefs.HasKey(key)` | 判断键是否存在（推荐在 Get 之前调用，避免返回默认值导致逻辑错误） |

## 数据唯一性
PlayerPrefs中不同数据的唯一性   
是由key决定的，不同的key决定了不同的数据    
同一项目中如果不同数据key相同会造成数据丢失     
要保证数据不丢失就要建立一个保证key唯一的规则   

## PlayerPrefs数据管理器实例

```csharp
using System;
using System.Collections;
using System.Collections.Generic;
using System.Reflection;
using UnityEngine;

/// <summary>
/// PlayerPrefs数据管理类 统一管理数据的存储和读取
/// </summary>
public class PlayerPrefsDataMgr
{
    private static PlayerPrefsDataMgr instance = new PlayerPrefsDataMgr();

    public static PlayerPrefsDataMgr Instance
    {
        get
        {
            return instance;
        }
    }

    private PlayerPrefsDataMgr()
    {

    }

    /// <summary>
    /// 存储数据
    /// </summary>
    /// <param name="data">数据对象</param>
    /// <param name="keyName">数据对象的唯一key 自己控制</param>
    public void SaveData( object data, string keyName )
    {
        //就是要通过 Type 得到传入数据对象的所有的 字段
        //然后结合 PlayerPrefs来进行存储

        #region 第一步 获取传入数据对象的所有字段
        Type dataType = data.GetType();
        //得到所有的字段
        FieldInfo[] infos = dataType.GetFields();
        #endregion

        #region 第二步 自己定义一个key的规则 进行数据存储
        //我们存储都是通过PlayerPrefs来进行存储的
        //保证key的唯一性 我们就需要自己定一个key的规则

        //我们自己定一个规则
        // keyName_数据类型_字段类型_字段名
        #endregion

        #region 第三步 遍历这些字段 进行数据存储
        string saveKeyName = "";
        FieldInfo info;
        for (int i = 0; i < infos.Length; i++)
        {
            //对每一个字段 进行数据存储
            //得到具体的字段信息
            info = infos[i];
            //通过FieldInfo可以直接获取到 字段的类型 和字段的名字
            //字段的类型 info.FieldType.Name
            //字段的名字 info.Name;

            //要根据我们定的key的拼接规则 来进行key的生成
            //Player1_PlayerInfo_Int32_age
            saveKeyName = keyName + "_" + dataType.Name + 
                "_" + info.FieldType.Name + "_" + info.Name;

            //现在得到了Key 按照我们的规则
            //接下来就要来通过PlayerPrefs来进行存储
            //如何获取值
            //info.GetValue(data)
            //封装了一个方法 专门来存储值 
            SaveValue(info.GetValue(data), saveKeyName);
        }

        PlayerPrefs.Save();
        #endregion
    }

    private void SaveValue(object value, string keyName)
    {
        //直接通过PlayerPrefs来进行存储了
        //就是根据数据类型的不同 来决定使用哪一个API来进行存储
        //PlayerPrefs只支持3种类型存储 
        //判断 数据类型 是什么类型 然后调用具体的方法来存储
        Type fieldType = value.GetType();

        //类型判断
        //是不是int
        if( fieldType == typeof(int) )
        {
            //为int数据加密
            int rValue = (int)value;
            rValue += 10;
            PlayerPrefs.SetInt(keyName, rValue);
        }
        else if (fieldType == typeof(float))
        {
            PlayerPrefs.SetFloat(keyName, (float)value);
        }
        else if (fieldType == typeof(string))
        {
            PlayerPrefs.SetString(keyName, value.ToString());
        }
        else if (fieldType == typeof(bool))
        {
            //自己顶一个存储bool的规则
            PlayerPrefs.SetInt(keyName, (bool)value ? 1 : 0);
        }
        //如何判断 泛型类的类型呢
        //通过反射 判断 父子关系
        //这相当于是判断 字段是不是IList的子类
        else if( typeof(IList).IsAssignableFrom(fieldType) )
        {
            //父类装子类
            IList list = value as IList;
            //先存储 数量 
            PlayerPrefs.SetInt(keyName, list.Count);
            int index = 0;
            foreach (object obj in list)
            {
                //存储具体的值
                SaveValue(obj, keyName + index);
                ++index;
            }
        }
        //判断是不是Dictionary类型 通过Dictionary的父类来判断
        else if( typeof(IDictionary).IsAssignableFrom(fieldType) )
        {
            //父类装自来
            IDictionary dic = value as IDictionary;
            //先存字典长度
            PlayerPrefs.SetInt(keyName, dic.Count);
            //遍历存储Dic里面的具体值
            //用于区分 表示的 区分 key
            int index = 0;
            foreach (object key in dic.Keys)
            {
                SaveValue(key, keyName + "_key_" + index);
                SaveValue(dic[key], keyName + "_value_" + index);
                ++index;
            }
        }
        //基础数据类型都不是 那么可能就是自定义类型
        else
        {
            SaveData(value, keyName);
        }
    }

    /// <summary>
    /// 读取数据
    /// </summary>
    /// <param name="type">想要读取数据的 数据类型Type</param>
    /// <param name="keyName">数据对象的唯一key 自己控制</param>
    /// <returns></returns>
    public object LoadData( Type type, string keyName )
    {
        //不用object对象传入 而使用 Type传入
        //主要目的是节约一行代码（在外部）
        //假设现在你要 读取一个Player类型的数据 如果是object 你就必须在外部new一个对象传入
        //现在有Type的 你只用传入 一个Type typeof(Player) 然后我在内部动态创建一个对象给你返回出来
        //达到了 让你在外部 少写一行代码的作用

        //根据你传入的类型 和 keyName
        //依据你存储数据时  key的拼接规则 来进行数据的获取赋值 返回出去

        //根据传入的Type 创建一个对象 用于存储数据
        object data = Activator.CreateInstance(type);
        //要往这个new出来的对象中存储数据 填充数据
        //得到所有字段
        FieldInfo[] infos = type.GetFields();
        //用于拼接key的字符串
        string loadKeyName = "";
        //用于存储 单个字段信息的 对象
        FieldInfo info;
        for (int i = 0; i < infos.Length; i++)
        {
            info = infos[i];
            //key的拼接规则 一定是和存储时一模一样 这样才能找到对应数据
            loadKeyName = keyName + "_" + type.Name +
                "_" + info.FieldType.Name + "_" + info.Name;

            //有key 就可以结合 PlayerPrefs来读取数据
            //填充数据到data中 
            info.SetValue(data, LoadValue(info.FieldType, loadKeyName));
        }
        return data;
    }

    /// <summary>
    /// 得到单个数据的方法
    /// </summary>
    /// <param name="fieldType">字段类型 用于判断 用哪个api来读取</param>
    /// <param name="keyName">用于获取具体数据</param>
    /// <returns></returns>
    private object LoadValue(Type fieldType, string keyName)
    {
        //根据 字段类型 来判断 用哪个API来读取
        if( fieldType == typeof(int) )
        {
            //解密 减10
            return PlayerPrefs.GetInt(keyName, 0) - 10;
        }
        else if (fieldType == typeof(float))
        {
            return PlayerPrefs.GetFloat(keyName, 0);
        }
        else if (fieldType == typeof(string))
        {
            return PlayerPrefs.GetString(keyName, "");
        }
        else if (fieldType == typeof(bool))
        {
            //根据自定义存储bool的规则 来进行值的获取
            return PlayerPrefs.GetInt(keyName, 0) == 1 ? true : false;
        }
        else if( typeof(IList).IsAssignableFrom(fieldType) )
        {
            //得到长度
            int count = PlayerPrefs.GetInt(keyName, 0);
            //实例化一个List对象 来进行赋值
            //用了反射中双A中 Activator进行快速实例化List对象
            IList list = Activator.CreateInstance(fieldType) as IList;
            for (int i = 0; i < count; i++)
            {
                //目的是要得到 List中泛型的类型 
                list.Add(LoadValue(fieldType.GetGenericArguments()[0], keyName + i));
            }
            return list;
        }
        else if( typeof(IDictionary).IsAssignableFrom(fieldType) )
        {
            //得到字典的长度
            int count = PlayerPrefs.GetInt(keyName, 0);
            //实例化一个字典对象 用父类装子类
            IDictionary dic = Activator.CreateInstance(fieldType) as IDictionary;
            Type[] kvType = fieldType.GetGenericArguments();
            for (int i = 0; i < count; i++)
            {
                dic.Add(LoadValue(kvType[0], keyName + "_key_" + i),
                         LoadValue(kvType[1], keyName + "_value_" + i));
            }
            return dic;
        }
        else
        {
            return LoadData(fieldType, keyName);
        }

    }
}

```