---
title: Json学习
published: 2026-07-26
pinned: false
description: Unity Json学习
tags: [Unity,学习,数据持久化]
author: boluobao
draft: true
---

# Json学习

## JsonUtility

JsonUtility 是Unity自带的用于解析Json的公共类   
它可以
1.将内存中的对象序列化为Json格式的字符串    
2.将Json字符串反序列化为对象    

### 在文件中存读字符串

```csharp
//1.存储字符串到指定路径文件夹中
//第一个参数填写的是存储的路径 第二个参数填写的是要存储的字符串
//第一个参数必须是存在的文件夹路径 如果文件夹不存在则会报错
File.WriteAllText(Application.persistentDataPath + "/JsonStudy.json", "存储的字符串");

//2.从指定路径文件夹中读取字符串
//第一个参数填写的是读取的路径
string json = File.ReadAllText(Application.persistentDataPath + "/JsonStudy.json");
```

### 使用JsonUtility进行序列化

序列化：把内存中的数据存储到硬盘上  

```csharp
//方法:
//JsonUtility.ToJson(对象)
[System.Serializable]
public class Student
{
    public string name;
    public int age;
}

public class Test
{
    public int age;
    public Student student;
    public List<Student> students;
    public Dictionary<string, int> dict;
    public List<int> list;
    [SerializeField]
    protected int protectedInt;
    [SerializeField]
    private int privateInt;

    public Test()
    {
        age = 18;
        student = new Student();
        students = new List<Student>() { new Student() { name = "张三", age = 18 }, new Student() { name = "李四", age = 18 } };
        dict = new Dictionary<string, int>() { {"name", 18} , {"age", 18} };
        list = new List<int>() { 1, 21, 2, 3 };
        protectedInt = 100;
        privateInt = 200;
    }
}

void Start()
{
    string json = JsonUtility.ToJson(new Test());

    File.WriteAllText(Application.persistentDataPath + "/JsonStudy.json", json);
}
```

最后打印出来的内容应该是

```Json
{"age":18,"student":{"name":"","age":0},"students":[{"name":"张三","age":18},{"name":"李四","age":18}],"list":[1,21,2,3],"protectedInt":100,"privateInt":200}
```

注意：  
1.float序列化时看起来会有一些误差   
2.自定义类需要加上序列化特性[System.Serializable]   
3.想要序列化私有变量 需要加上特性[SerializeField]  
4.JsonUtility不支持字典     
5.JsonUtility存储空对象不会是null 而是数据的默认值  

### 使用JsonUtility进行反序列化

反序列化：把硬盘上的数据读取到内存中  

```csharp
//方法:
//JsonUtility.FromJson(字符串,对象类型)
string json = File.ReadAllText(Application.persistentDataPath + "/JsonStudy.json");

Test test = JsonUtility.FromJson<Test>(json);

//若是数据没有完全读取 则未读取到数据的字段则会为空 且不会报错
```

注意：  
1.JsonUtility无法直接读取数据集合   
2.文本编码格式需要是utf-8 否则会报错    


## LitJson

LitJson 是一个第三方的Json的库 用于处理Json的序列化和反序列化 
LitJson是C#编写的，体积小、速度快、易于使用 
它可以很容易的嵌入到我们的代码中    
只需要将LitJson代码拷贝到工程中即可

### 使用LitJson进行序列化

```csharp
//方法:
//LitJson.ToJson(对象)
string json = JsonMapper.ToJson(new Test());
```

打印出的内容为
```Json
{"age":18,"student":{"name":null,"age":0},"students":[{"name":"\u5F20\u4E09","age":18},{"name":"\u674E\u56DB","age":18}],"dict":{"name":18,"age":18},"list":[1,21,2,3]}
```

注意：  
1.相对JsonUttility不需要添加特性    
2.不能序列化私有对象    
3.支持字典，字典的键 建议使用字符串类型 由于Json的特点 字典的键在Json中会加上双引号 
4.需要引用LitJson的命名空间 LitJson;  
5.LitJson可以准确的保存空对象

### 使用LitJson进行反序列化

```csharp
//方法:
//LitJson.FromJson(字符串,对象类型)
string json = File.ReadAllText(Application.persistentDataPath + "/JsonStudy.json");
//JsonData是LitJson提供的类对象 可以用键值对的形式去访问其中的内容
JsonData jsonData = JsonMapper.ToObject(json);
//访问JsonData中的内容
int age = (int)jsonData["age"];

//使用泛型
Test test = JsonMapper.ToObject<Test>(json);
```

注意：  
1.类结构需要无参构造函数 否则反序列化时会报错   
2.字典虽然支持 但是键在为数值时会有问题 需要使用字符串类型  
3.LitJson可以直接读取数据集合   
4.文本编码格式需要是utf-8 否则会报错    




