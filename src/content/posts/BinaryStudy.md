---
title: 二进制学习笔记
published: 2026-09-02
pinned: false
description: 二进制学习笔记
tags: [二进制,学习,数据持久化]
author: boluobao
draft: false
---

# 二进制学习笔记

## 各类型转字节数组

### 2进制文件读写的本质

通过将各类型变量转换为字节数组 将字节数组直接存储到文件中  
一般人是看不懂存储的数据的 不仅可以节约存储空间、提升效率 还可以提升安全性  
而且在网络通信中我们直接传输的数据也是字节数据

### 各类型数据和字节数据相互转换
C#提供了一个公共类帮助我们进行转化
类名：BitConverter

- 1. 将各类型转字节
```csharp
//byte数组中每一个元素代表8位二进制数
byte[] bytes = BitConverter.GetBytes(123);
```

- 2. 将字节转各类型
```csharp
int num = BitConverter.ToInt32(bytes, 0);
```
 
- 3. 将字符串转字节数组
```csharp
byte[] bytes = Encoding.UTF8.GetBytes("hello");
```

- 4. 将字节数组转字符串
```csharp
string str = Encoding.UTF8.GetString(bytes);
```

## 文件操作相关

- 1. 判断文件是否存在
```csharp
File.Exists("文件路径");
```

- 2. 创建文件
```csharp
File.Create("文件路径");
```

- 3. 写入文件
```csharp
//将指定字节数组写入到指定路径的文件中
File.WriteAllBytes("文件路径", bytes);

//将指定的string数组内容 一行行写入到指定路径中
string[] lines = {"hello", "world"};
File.WriteAllLines("文件路径", lines);

//将指定字符串写入指定路径
File.WriteAllText("文件路径", "hello"); //支持转义字符
```

- 4. 读取文件
```csharp
//读取字节数据
byte[] bytes = File.ReadAllBytes("文件路径");

//读取所有行信息
string[] lines = File.ReadAllLines("文件路径");

//读取所有文本信息
string str = File.ReadAllText("文件路径");
```

- 5. 删除文件
```csharp
//如果删除正在打开的文件 会报错
File.Delete("文件路径");
``` 

- 6. 复制文件
```csharp
//将源文件复制到目标文件
File.Copy("源文件路径", "目标文件路径");

//将源文件复制到目标文件 如果目标文件存在 则覆盖
File.Copy("源文件路径", "目标文件路径", true);
```

- 7. 文件替换
```csharp
//将源文件替换目标文件
File.Replace("源文件路径", "目标文件路径", "备份文件路径");
```

- 8. 以流的形式 打开文件写入或读取
```csharp
//以流的形式打开文件
FileStream fs = File.Open("文件路径", FileMode.OpenOrCreate, FileAccess.ReadWrite);

```

## 文件流相关

### 什么是文件流
文件里存储的数据就像是一条数据流 我们可以通过FileStream一部分一部分的读写数据流 
比如我可以先存一个int(4个字节) 再存一个bool(1个字节) 再存一个string(n个字节)    
利用FileStream可以以流式逐个读写

### FileStream文件流常用方法

- 1. 打开或创建指定路径

```csharp
//方法一: new FileStream
//参数二 打开模式
// CreateNew：如果文件不存在则创建文件 如果文件存在则抛出异常
// Create：如果文件不存在则创建文件 如果文件存在则覆盖
// Open：如果文件不存在则抛出异常 如果文件存在则打开
// OpenOrCreate：如果文件不存在则创建文件 如果文件存在则打开
// Append：如果文件不存在则创建文件 如果文件存在则打开文件流并将文件指针移动到文件末尾
// Truncate：如果文件不存在则抛出异常 如果文件存在则打开文件流并将文件大小截断为0

//参数三 访问模式
//参数四 共享权限
// None：不共享
// Read：只读
// Write：只写
// ReadWrite：读写
FileStream fs = new FileStream("文件路径", FileMode.OpenOrCreate, FileAccess.ReadWrite);

//方法二：File.Create
//参数二 缓存大小
//参数三 文件选项（描述如何创建或覆盖该文件）
// None：不使用任何特殊选项
// Asynchronous：可用于异步读写
// Encrypted：加密文件
// RandomAccess：随机访问文件
// SequentialScan：顺序扫描文件
// DeleteOnClose：关闭文件时删除文件
FileStream fs = File.Create("文件路径",2048,FileOptions.None);

//方法三：File.Open
//参数二 打开模式
FileStream fs = File.Open("文件路径", FileMode.OpenOrCreate);
```

- 2. 重要属性和方法

```csharp
// 文本字节长度
int length = (int)fs.Length;

//是否可写
bool canWrite = fs.CanWrite;

//是否可读
bool canRead = fs.CanRead;

// 将字节写入文件 当写入后 一定执行一次的方法
fs.Flush();

//关闭流 当文件读写完毕后 一定执行
fs.Close();

//释放资源
fs.Dispose();
```

- 3. 写入字节
  
```csharp
//将字节数组写入文件
// 参数一：字节数组
// 参数二：数组中的开始索引
// 参数三：写入的字节数
fs.Write(bytes, 0, bytes.Length);

//写入字符串时 先写入长度 再写入字符串具体内容
byte[] bytes = Encoding.UTF8.GetBytes(str);
fs.Write(BitConverter.GetBytes(bytes.Length), 0, 4);
fs.Write(bytes, 0, bytes.Length);

//避免数据丢失
fs.Flush();
//销毁
fs.Dispose();
```

- 4. 读取字节

```csharp
//读取字节数组
// 参数一：字节数组
// 参数二：数组中的开始索引
// 参数三：读取的字节数  
// 返回值：当前流索引前进了几个位置

//读取一个整形
byte[] bytes = new byte[4];
fs.Read(bytes, 0, 4);
int num = BitConverter.ToInt32(bytes, 0);

//读取一个字符串
fs.Read(bytes, 0, 4);
int length = BitConverter.ToInt32(bytes, 0);
byte[] bytes2 = new byte[length];
fs.Read(bytes2, 0, length);
string str = Encoding.UTF8.GetString(bytes2, 8, length);

//避免数据丢失
fs.Flush();
//销毁
fs.Dispose();
```

## 文件夹相关

- 1. 判断文件夹是否存在
```csharp
Directory.Exists("文件夹路径");
```

- 2. 创建文件夹
```csharp
Directory.CreateDirectory("文件夹路径");
```

- 3. 删除文件夹
```csharp
//参数二：如果为true则删除整个目录 如果为false仅当目录为空时才可删除
Directory.Delete("文件夹路径");
```

- 4. 查找文件夹和文件
```csharp
//得到指定路径下所有文件夹名
string[] dirs = Directory.GetDirectories("文件夹路径");

//得到指定路径下所有文件名
string[] files = Directory.GetFiles("文件夹路径");
```

- 5. 移动文件夹
```csharp
//如果目标路径已经存在同名文件夹 则会抛出异常
Directory.Move("文件夹路径", "目标文件夹路径");
```

### DirectoryInfo和FileInfo

- 1. DirectoryInfo类
  我们可以通过它获取文件夹的更多信息
  创建文件夹方法的返回值
```csharp
DirectoryInfo dInfo = Directory.CreateDirectory("文件夹路径");

//获得文件夹全路径
print(dInfo.FullName);

//获得文件夹名称
print(dInfo.Name);
```

  查找上级文件夹信息
```csharp
DirectoryInfo dInfo = Directory.GetParent("文件夹路径");

//查找所有子文件夹的目录信息
DirectoryInfo[] dInfos = dInfo.GetDirectories();
```

- 2. FileInfo类
  我们可以通过DirectoryInfo类获取文件的信息
```csharp
FileInfo[] fInfos = dInfo.GetFiles();
for(int i = 0; i < fInfos.Length; i++)
{
    //获得文件全路径
    print(fInfos[i].FullName);

    //获得文件名称
    print(fInfos[i].Name);

    //获得文件大小
    print(fInfos[i].Length);
}
```
 
## 序列化与反序列化

- 1. 序列化
  如果要使用C#自带的序列化2进制方法 
  申明类时需要添加[System.Serializable]特性
```csharp
[System.Serializable]
public class Player
{
    public int id = 1;
    public string name = "张三";
    public int[] scores = {1, 2, 3};
    public Dictionary<string, int> items = new Dictionary<string, int>() {{"item1", 1}, {"item2", 2}};
    public List<int> list = new List<int>() {1, 2, 3};
    public StructTest structTest = new StructTest() {id = 1, name = "structTest"};
    public ClassTest classTest = new ClassTest()
}
[System.Serializable]
public struct StructTest
{
    public int id;
    public string name;
}
[System.Serializable]
public class ClassTest
{
    public int id = 1;
    public string name = "大黄蜂;
}
```
 
 使用MemoryStream进行序列化 主要用于得到字节数组

```csharp
player = new Player();
using(MemoryStream ms = new MemoryStream())
{
    //序列化
    BinaryFormatter bf = new BinaryFormatter();
    bf.Serialize(ms, player);
    byte[] bytes = ms.GetBuffer();
    //将字节数组写入文件
    File.WriteAllBytes("文件路径", bytes);
    ms.Close();
}
```

  使用文件流进行序列化 
```csharp
player = new Player();
using(FileStream fs = new FileStream("文件路径", FileMode.OpenOrCreate, FileAccess.ReadWrite))
{
    //序列化
    BinaryFormatter bf = new BinaryFormatter();
    bf.Serialize(fs, player);
    fs.Flush();
    fs.Close();
}
```

- 2. 反序列化 
  通过FileStream进行反序列化
```csharp
using(FileStream fs = File.Open("文件路径", FileMode.Open, FileAccess.Read))
{
    //反序列化
    BinaryFormatter bf = new BinaryFormatter();
    Player player = (Player)bf.Deserialize(fs);
    fs.Close();
}
```
  
  通过MemoryStream进行反序列化
```csharp
byte[] bytes = File.ReadAllBytes("文件路径");
using(MemoryStream ms = new MemoryStream(bytes))
{
    //反序列化
    BinaryFormatter bf = new BinaryFormatter();
    Player player = (Player)bf.Deserialize(ms);
    ms.Close();
}
```

## 加密

- 1. MD5加密
- 2. SHA1加密
- 3. HMAC加密
