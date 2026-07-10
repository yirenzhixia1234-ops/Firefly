---
title: C#学习笔记2
published: 2026-07-06
pinned: false
description: C#学习笔记2 - 集合、泛型、委托事件、反射、多线程、排序算法等进阶知识
tags: [C#,学习]
author: boluobao
draft: false
---

# 🚀 C#学习笔记2

本笔记涵盖 C# 进阶知识，从集合类型（ArrayList、Stack、Queue、HashTable）到泛型、委托事件、反射、多线程以及经典排序算法，帮助你深入掌握 C# 的核心能力。

---

## 📦 ArrayList

> ArrayList 是 C# 为我们封装好的类，它的本质是一个 `object` 类型的数组。ArrayList 类帮助我们实现了很多方法，比如数组的增删查改。

### 声明

```csharp
ArrayList arrayList = new ArrayList(); //主要要引用System.Collections命名空间
```

### 增

```csharp
ArrayList arrayList = new ArrayList();
//ArrayList可以添加任意类型的对象 加到数组的末尾
arrayList.Add(1);
arrayList.Add("hello");
arrayList.Add(true);
arrayList.Add(null);

//批量添加
arrayList.AddRange(new ArrayList[] { 2, "world", false, null });

//插入指定位置的元素 在索引为0的位置插入"hello world"
arrayList.Insert(0, "hello world");
```

### 删

```csharp
//移除指定元素 从数组头开始遍历 删除第一个匹配的元素
arrayList.Remove(1);
//移除指定索引的元素
arrayList.RemoveAt(0);
//清空数组
arrayList.Clear();
```

### 查

```csharp
//查找指定位置的元素
object element = arrayList[0];

//判断元素是否存在
bool isExist = arrayList.Contains(1);

//正向查找元素的索引 如果不存在返回-1 从数组头开始遍历
int index = arrayList.IndexOf(1);
//反向查找元素的索引 如果不存在返回-1
int reverseIndex = arrayList.LastIndexOf(1);

```

### 改

```csharp
//修改指定位置的元素
arrayList[0] = "hello world";
```

### 遍历

```csharp
//遍历数组
//for循环遍历
for (int i = 0; i < arrayList.Count; i++)   //注意这里的Count是数组的长度 从0开始计数 而不是用Length
{
    Console.WriteLine(arrayList[i]);
}


//foreach循环遍历
foreach (object element in arrayList)
{
    Console.WriteLine(element);
}
```

### 装箱拆箱

> [!WARNING] 性能陷阱
> ArrayList 本质上是一个可以自动扩容的 `object` 数组，由于用万物之父来存储数据，自然存在装箱拆箱。
> 当往其中进行值类型存储时就是在装箱，当将值类型对象取出来转换使用时，就存在拆箱。
> ==所以 ArrayList 尽量少用。==

```csharp
int i =1;
arrayList.Add(i); //装箱

int j = (int)arrayList[0]; //拆箱
```

### 数组和 ArrayList 的区别

在C#中，`Array`（数组）和`ArrayList`都用于存储数据集合，但它们在设计哲学和特性上存在根本差异。简单来说，`Array`是**类型安全、大小固定**的高性能基础数据结构；而`ArrayList`是**非类型安全、可动态增长**的旧式集合类。

> [!TIP] 现代推荐
> 目前，`ArrayList` 在实际开发中已基本被泛型的 `List<T>` 所取代，但在理解C#集合类型演进时，了解其与 `Array` 的区别仍然很有价值。

#### 核心区别概览

| 特性 | Array (数组) | ArrayList |
| :--- | :--- | :--- |
| **类型安全** | ✅ **是**，存储类型在声明时固定，编译时检查 | ❌ **否**，以 `object` 类型存储所有元素，可混存不同类型 |
| **类型** | 强类型 | 非泛型集合 |
| **大小** | **固定**，创建后长度不可变 | **动态**，可自动扩容 |
| **性能** | **更高**，无额外类型转换开销 | **较低**，涉及装箱/拆箱和类型转换 |
| **内存分配** | 通常在**栈**上 | 在**堆**上 |
| **维度** | 支持**多维**（如二维数组） | 始终是**一维**的 |
| **命名空间** | `System.Array` | `System.Collections` |

---

#### 详细对比分析

**1. 类型安全与存储内容**

- `Array` 是类型安全的：它在声明时就确定了元素类型（如 `int[]`, `string[]`），任何类型不匹配的操作都会在编译时报错，这能有效避免运行时的类型转换异常。
- `ArrayList` 是非类型安全的：它将所有元素都当作 `object` 类型处理，因此可以在同一个 `ArrayList` 中存储字符串、整数、自定义对象等不同类型的数据。但这种灵活性带来了风险，在取出元素时**必须进行显式类型转换**，如果类型不匹配，会抛出运行时异常。

**2. 性能差异：装箱与拆箱**

这是两者性能差异的核心原因。

- `Array` 性能优越：由于存储的是特定类型（如 `int`），它直接操作值，**没有装箱和拆箱的开销**。
- `ArrayList` 性能开销大：当向 `ArrayList` 添加一个值类型（如 `int`）时，该值会被**装箱**（Boxing）为 `object` 类型；当从 `ArrayList` 读取该值并转换为 `int` 时，又会发生**拆箱**（Unboxing）。这个过程涉及内存分配和类型检查，会显著影响性能。

**3. 容量与灵活性**

- `Array` 大小固定：一旦创建，其长度（`Length`）就不可改变。如果需要更大或更小的数组，必须创建一个新数组并复制元素。
- `ArrayList` 可动态调整：它就像一个会自动增长的容器，你可以随时添加（`Add`）、插入（`Insert`）或删除（`Remove`）元素，其容量（`Capacity`）会根据需要自动扩充。

**4. 功能与使用简便性**

- `Array` 功能基础：提供基本的索引访问，本身没有提供添加、插入、删除元素的方法。
- `ArrayList` 功能丰富：提供了 `Add`、`Insert`、`Remove`、`Sort` 等众多便捷方法来操作集合。

#### 总结与建议

> [!IMPORTANT] 最佳实践
> 1. 当集合大小固定且元素类型单一时，应优先使用 `Array`。例如，存储一周七天的名称。
> 2. 在现代C#开发中，应避免使用 `ArrayList`。它存在的类型安全问题和性能缺陷，已由泛型集合 `List<T>` 完美解决。
> 3. 对于需要动态增删、且元素类型单一的场景，应使用 `List<T>`。它兼具了 `ArrayList` 的灵活性和 `Array` 的类型安全与高性能。

---

## 📚 Stack

### Stack 的本质

> Stack（栈）是一个 C# 为我们封装好的类。
> 它的本质也是 `object[]` 数组，只是封装了特殊的存储规则。
> Stack 是栈存储容器，栈是一种==先进后出==的数据结构。
> 先存入的数据后获取，后存入的数据先获取。

### 声明

```csharp
Stack stack = new Stack<int>();  //也要引用System.Collections命名空间
```

### 增

```csharp
//只能一个一个的存入
stack.Push(1);
//和ArrayList一样可以存储不同类型的数据 本质都是object[]数组
stack.Push("hello");
stack.Push(3.14);
```

### 取

```csharp
//只能一个一个的取
int i = stack.Pop();
```

### 查

```csharp
//栈无法查看指定位置的元素只能查看栈顶的元素
int top = stack.Peek();

//查找指定元素是否存在
bool isExist = stack.Contains(1);
```

### 改

```csharp
stack.Clear();
```

### 遍历

```csharp
//用foreach遍历栈 顺序由栈顶到栈底
foreach (object item in stack)
{
    Console.WriteLine(item);
}

//将Stack转换为object[]数组
object[] array = stack.ToArray();
for (int i = 0; i < array.Length; i++)
{
    Console.WriteLine(array[i]);
}

//while循环遍历栈
while (stack.Count > 0)
{
    Console.WriteLine(stack.Pop());
}
```

> [!NOTE] 装箱拆箱
> 由于用万物之父来存储数据，自然存在装箱拆箱。
> 当往其中进行值类型存储时就是在装箱，当将值类型对象取出来转换使用时，就存在拆箱。

---

## 🚶 Queue

### Queue 本质

> Queue 是一个 C# 为我们封装好的类。
> 它的本质也是 `object[]` 数组，只是封装了特殊的存储规则。
> Queue 是队列存储容器，队列是一种==先进先出==的数据结构。
> 先存入的数据先获取，后存入的数据后获取。

### 声明

```csharp
Queue queue = new Queue<int>();  //也要引用System.Collections命名空间
```

### 增

```csharp
//只能一个一个的存入
queue.Enqueue(1);
//和ArrayList一样可以存储不同类型的数据 本质都是object[]数组
queue.Enqueue("hello");
queue.Enqueue(3.14);
```

### 取

```csharp
//只能一个一个的取
int i = queue.Dequeue();
```

### 查

```csharp
//队列无法查看指定位置的元素只能查看队头的元素
int front = queue.Peek();

//查找指定元素是否存在
bool isExist = queue.Contains(1);
```

### 改

```csharp
queue.Clear();
```

### 遍历

```csharp
//用foreach遍历队列 顺序由队头到队尾
foreach (object item in queue)
{
    Console.WriteLine(item);
}

//将Queue转换为object[]数组
object[] array = queue.ToArray();
for (int i = 0; i < array.Length; i++)
{
    Console.WriteLine(array[i]);
}

//while循环遍历队列
while (queue.Count > 0)
{
    Console.WriteLine(queue.Dequeue());
}
```

> [!NOTE] 装箱拆箱
> 由于用万物之父来存储数据，自然存在装箱拆箱。
> 当往其中进行值类型存储时就是在装箱，当将值类型对象取出来转换使用时，就存在拆箱。

---

## 🗂️ HashTable

### HashTable 本质

> Hashtable（又称散列表）是基于键的哈希代码组织起来的 ==键/值对==。
> 它的主要作用是提高数据查询的效率。
> 使用键来访问集合中的元素。

### 声明

```csharp
Hashtable hashtableTable = new Hashtable();  //也要引用System.Collections命名空间
```

### 增

```csharp
//添加键值对 键值对可以是任意类型
hashtableTable.Add("name", "shinjayo");
//添加键值对
hashtableTable.Add(1, "hello");
```

> [!CAUTION] 注意
> 不能在同一个 Hashtable 中添加相同的键！
>
> ```csharp
> hashtableTable.Add("name", "shinjayo2"); // 报错
> ```

### 删

```csharp
//只能通过键来删除
hashtableTable.Remove("name");

//删除不存在的键 也不会报错
hashtableTable.Remove(2); //无事发生

//清空
hashtableTable.Clear();
```

### 查

```csharp
//根据键来获取值 找不到会返回null
string name = (string)hashtableTable["name"];

//查看指定键是否存在
bool isExist = hashtableTable.ContainsKey("name");

//查看指定值是否存在
bool isExistValue = hashtableTable.ContainsValue("shinjayo");
```

### 改

```csharp
//只能改 键对应的值 不能改键
hashtableTable["name"] = "shinjayo3";
```

### 遍历

```csharp
//得到所有键值对
Console.WriteLine(hashtableTable.Count);

//遍历所有键
foreach (object key in hashtableTable.Keys)
{
    Console.WriteLine(key);
}

//遍历所有值
foreach (object value in hashtableTable.Values)
{
    Console.WriteLine(value);
}

//遍历所有键值对
foreach (DictionaryEntry entry in hashtableTable)
{
    Console.WriteLine(entry.Key + " -> " + entry.Value);
}

//迭代器遍历所有键值对
IEnumerator enumerator = hashtableTable.GetEnumerator();
while (enumerator.MoveNext())
{
    DictionaryEntry entry = (DictionaryEntry)enumerator.Current;
    Console.WriteLine(entry.Key + " -> " + entry.Value);
}
```

> [!NOTE] 装箱拆箱
> 由于用万物之父来存储数据，自然存在装箱拆箱。
> 当往其中进行值类型存储时就是在装箱，当将值类型对象取出来转换使用时，就存在拆箱。

---

## 🧬 泛型

### 泛型是什么

> 泛型实现了类型参数化，达到代码重用目的。
> 通过类型参数化来实现同一份代码上操作多种类型。
> ==泛型相当于类型占位符==：定义类或方法时使用替代符代表变量类型，当真正使用类或者方法时再具体指定类型。

### 泛型分类

#### 泛型类和泛型接口

**基本语法：**

```csharp
// class 类名<泛型占位字母>
class GenericClass<T>
{
    public T value;
}
// interface 接口名<泛型占位字母>
interface IGenericInterface<T>
{
    //泛型接口的成员
    T value
    {
        get;
        set;
    }
}

//泛型占位字母可以有多个
void GenericMethod<T, U>(T param1, U param2)
{
    //泛型函数的实现
    public T result = param1;
    public U result2 = param2;
}
```

**使用：**

```csharp
//实例化泛型类
GenericClass<int> genericClass = new GenericClass<int>();
genericClass.value = 100;
Console.WriteLine(genericClass.value); //打印出100

GenericClass<string> genericClass2 = new GenericClass<string>();
genericClass2.value = "hello";
Console.WriteLine(genericClass2.value); //打印出hello

GenericMethod<int, string> genericMethod = new GenericMethod<int, string>();
genericMethod(100, "hello");
Console.WriteLine(genericMethod.result); //打印出100
Console.WriteLine(genericMethod.result2); //打印出hello

//泛型接口继承
class GenericClass2 : IGenericInterface<int>
{
    public int value
    {
        get;
        set;
    }
}
```

#### 泛型函数

```csharp
//泛型函数
//普通类中泛型函数的实现
class Test
{
    public T GenericMethod<T>(T param)
    {
        return param;
    }

    public void TestMethod<T>()
    {
        //用泛型在函数内部进行逻辑处理
        T param = default(T);
    }

    public void TestMethod2<T,U>(T param ,U param2) //泛型用作多个参数
    {
        return;
    }
}

//泛型函数的调用
Test test = new Test();
int result = test.GenericMethod<int>(100);
Console.WriteLine(result); //打印出100

//泛型类中的泛型函数
class GenericClass<T>
{
    //泛型函数所用的占位符不能与泛型类的占位符相同 不过可以将泛型类的占位符作为参数的类型占位符
    public K GenericMethod<K>(K param ,T param2)
    {
        return param;
    }
}

//泛型类中的泛型函数的调用
GenericClass<int> genericClass = new GenericClass<int>();
int result2 = genericClass.GenericMethod<int>(100, 200);
Console.WriteLine(result2); //打印出100

//泛型函数的重载
class Test
{
    public void GenericMethod<T>(T param)
    {
        return;
    }

    public void GenericMethod()
    {
        return;
    }
}

//调用
test.GenericMethod<int>(100);
test.GenericMethod();
```

### 泛型的作用

> [!IMPORTANT] 核心价值
> 1. 不同类型对象的相同逻辑处理就可以选择泛型
> 2. 使用泛型可以一定程度**避免装箱拆箱**，提高性能

---

## 🔒 泛型约束

### 什么是泛型约束？

> 让泛型的类型有一定的限制。
> 关键字：`where`

泛型约束一共有 **6 种**：

| 序号 | 约束类型 | 语法 |
|:--:|------|------|
| 1 | 值类型 | `where T : struct` |
| 2 | 引用类型 | `where T : class` |
| 3 | 存在无参公共构造函数 | `where T : new()` |
| 4 | 某个类本身或者其派生类 | `where T : 类名` |
| 5 | 某个接口的派生类型 | `where T : 接口名` |
| 6 | 另一个泛型类型本身或者派生类型 | `where T : U` |

#### 值类型约束

```csharp
//值类型约束 只能使用值类型作为参数的类型
class GenericClass<T> where T : struct
{
    public T value;

    public void Function<K>(K param) where K : struct
    {
        value = param;
    }
}
```

#### 引用类型约束

```csharp
//引用类型约束 只能使用引用类型作为参数的类型
class GenericClass<T> where T : class
{
    public T value;

    public void Function<K>(K param) where K : class
    {
        value = param;
    }
}
```

#### 存在无参公共构造函数约束

```csharp
//存在无参公共构造函数约束 只能使用存在无参公共构造函数的类型作为参数的类型
class GenericClass<T> where T : new()
{
    public T value;

    public void Function<K>(K param) where K : new()
    {
        value = param;
    }
}

class Test1
{

}

class Test2
{
    public Test2(int i)
    {

    }
}

GenericClass<Test1> genericClass = new GenericClass<Test1>(); //可以使用Test1作为参数的类型
GenericClass<Test2> genericClass2 = new GenericClass<Test2>(); //不能使用Test2作为参数的类型 因为Test2没有无参公共构造函数
```

#### 某个类本身或者其派生类约束

```csharp
//某个类本身或者其派生类约束 只能使用某个类本身或者其派生类作为参数的类型
class GenericClass<T> where T : Test1
{
    public T value;

    public void Function<K>(K param) where K : Test1
    {
        value = param;
    }
}

class Test3 : Test1
{

}

GenericClass<Test3> genericClass3 = new GenericClass<Test3>(); //可以使用Test3作为参数的类型
```

#### 某个接口的派生类型约束

```csharp
//某个接口的派生类型约束 只能使用某个接口的或者其派生类作为参数的类型
class GenericClass<T> where T : IGenericInterface<int>
{
    public T value;

    public void Function<K>(K param) where K : IGenericInterface<int>
    {
        value = param;
    }
}

class Test4 : IGenericInterface<int>
{
    public int value
    {
        get;
        set;
    }
}

GenericClass<Test4> genericClass4 = new GenericClass<Test4>(); //可以使用Test4作为参数的类型
```

#### 另一个泛型类型本身或者派生类型约束

```csharp
class GenericClass<T,U> where T : U //T必须是U的派生类或U本身
{
    public void Function<K,V,L>(K param,V param2,L param3) where K : V where L : U
    {

    }
}

class Test5 : Test1
{

}

GenericClass<Test5,Test1> genericClass5 = new GenericClass<Test5,Test1>(); //可以使用Test5作为参数的类型 因为Test5是Test1的派生类
GenericClass<Test1,Test1> genericClass52 = new GenericClass<Test1,Test1>();
```

### 泛型约束的组合使用

```csharp
class Test6<T> where T : class,new()
{

}
```

### 多个泛型有约束

```csharp
class Test7<T,K> where T : class where K : struct
{

}
```

---

## 📋 List

### List 的本质

> List 是一个 C# 为我们封装好的类，它的本质是一个==可变类型的泛型数组==，List 类帮助我们实现了很多方法，比如泛型数组的增删查改等操作。

### 声明

```csharp
List<int> list = new List<int>(); //引用命名空间：System.Collections.Generic
List<string> list2 = new List<string>();  //申明一个字符串类型的List
```

### 增

```csharp
list.Add(100);

list.AddRange(new int[] {1,2,3});

list.Insert(0,1000); //在索引为0的位置插入1000
```

### 删

```csharp
//移除指定元素
list.Remove(100);

//移除指定索引的元素
list.RemoveAt(0);

//清空
list.Clear();
```

### 查

```csharp
//获取指定索引的元素
int value = list[0];

//查看元素是否存在
bool isExist = list.Contains(100);

//正向查找元素位置
int index = list.IndexOf(100);

//反向查找元素位置
int index2 = list.LastIndexOf(100);
```

### 改

```csharp
//修改指定索引的元素
list[0] = 1000;
```

### 遍历

```csharp
//for循环遍历
for(int i = 0;i < list.Count;i++)
{
    int value = list[i];
    Console.WriteLine(value);
}

//foreach循环遍历
foreach(int value in list)
{
    Console.WriteLine(value);
}
```

### 排序

```csharp
//list提供的排序方法
list.Sort(); //默认是升序排序
//ArrayList也有这种方法
```

**自定义排序：**

```csharp
class Item : IComparer<Item>
{
    public int money;

    public Item(int money)
    {
        this.money = money;
    }

    //只有在自定义类中实现CompareTo方法，才能使用list.Sort()方法 要实现IComparer<Item>接口
    public int CompareTo([AllowNull] Item other)
    {
        //返回值的含义 等于0 表示相等 小于0 表示当前对象小于其他对象 大于0 表示当前对象大于其他对象

        //升序排序
        if(this.money > other.money)
        {
            return 1
        }
        else if(this.money < other.money)
        {
            return -1
        }
        else
        {
            return 0
        }
    }
}
```

**用委托进行排序：**

```csharp
class ShopItem
{
    public int id;

    public ShopItem(int id)
    {
        this.id = id;
    }
}

class Program
{
    static void Main()
    {
        List<ShopItem> shopItems = new List<ShopItem>();
        shopItems.Add(new ShopItem(1));
        shopItems.Add(new ShopItem(2));
        shopItems.Add(new ShopItem(3));

        shopItems.Sort(SortShopItems); //根据id升序排序

        foreach(ShopItem item in shopItems)
        {
            Console.WriteLine(item.id);
        }
    }

    static int SortShopItems(ShopItem x,ShopItem y)
    {
        //传入的两个参数x和y是ShopItem类型的对象
        //进行两两比较 用左边的id和右边的id进行比较
        //返回值的含义 等于0 表示相等 小于0 表示当前对象小于其他对象 大于0 表示当前对象大于其他对象
        //升序排序
        if(x.id > y.id)
        {
            return 1
        }
        else if(x.id < y.id)
        {
            return -1
        }
        else
        {
            return 0
        }
    }
}
```

### 实战示例：怪物管理系统

> 一个 Monster 基类，Boss 和 Goblin 类继承它。
> 在怪物类的构造函数中，将其存储到一个怪物 List 中。
> 遍历列表可以让 Boss 和 Goblin 对象产生不同攻击。

```csharp
using System;
using System.Collections.Generic;

// 1. 定义怪物基类（抽象类，强制派生类实现 Attack）
public abstract class Monster
{
    // 静态列表，用于存储所有已创建的怪物实例
    private static List<Monster> _allMonsters = new List<Monster>();

    // 公共属性，方便外部访问所有怪物
    public static IReadOnlyList<Monster> AllMonsters => _allMonsters;

    // 基类构造函数：每创建一个 Monster（或其派生类）实例，就自动添加到列表中
    protected Monster()
    {
        _allMonsters.Add(this);
    }

    // 抽象攻击方法，派生类必须提供具体实现
    public abstract void Attack();
}

// 2. 派生类 Boss
public class Boss : Monster
{
    public Boss()
    {
        // 基类构造函数会自动将 this 添加到列表中
    }

    public override void Attack()
    {
        Console.WriteLine("Boss 发动毁灭性打击！造成 999 点伤害。");
    }
}

// 3. 派生类 Goblin
public class Goblin : Monster
{
    public Goblin()
    {
        // 同样自动注册
    }

    public override void Attack()
    {
        Console.WriteLine("Goblin 挥动小刀，造成 5 点伤害。");
    }
}

// 4. 测试
class Program
{
    static void Main()
    {
        // 创建怪物实例 —— 每次 new 都会自动加入 Monster.AllMonsters 列表
        Boss boss1 = new Boss();
        Boss boss2 = new Boss();
        Goblin goblin1 = new Goblin();
        Goblin goblin2 = new Goblin();

        // 遍历所有怪物，调用 Attack 方法 —— 多态发挥作用
        Console.WriteLine("所有怪物开始攻击：");
        foreach (Monster monster in Monster.AllMonsters)
        {
            monster.Attack();
        }

        // 输出结果：
        // Boss 发动毁灭性打击！造成 999 点伤害。
        // Boss 发动毁灭性打击！造成 999 点伤害。
        // Goblin 挥动小刀，造成 5 点伤害。
        // Goblin 挥动小刀，造成 5 点伤害。
    }
}
```

---

## 📖 Dictionary

### Dictionary 的本质

> 可以将 Dictionary 理解为拥有泛型的 Hashtable。它也是基于键的哈希代码组织起来的键/值对，==键值对类型从 Hashtable 的 `object` 变为了可以自己指定的泛型==。

### 声明

```csharp
Dictionary<int,string> dict = new Dictionary<int,string>(); //引用命名空间：System.Collections.Generic
```

### 增

```csharp
//不能出现相同键
dict.Add(100,"100");
```

### 删

```csharp
//只能通过键去删除 删除不存在的键无事发生
dict.Remove(100);

//清空
dict.Clear();
```

### 查

```csharp
//获取指定键的值 找不到会报错（这里和Hashtable不同）
string value = dict[100];

//查看键是否存在
bool isExist = dict.ContainsKey(100);

//查看值是否存在
bool isExist2 = dict.ContainsValue("100");
```

### 改

```csharp
//修改指定键的值
dict[100] = "1000";
```

### 遍历

```csharp
//遍历所有键
foreach(int key in dict.Keys)
{
    Console.WriteLine(key);
}

//遍历所有值
foreach(string value in dict.Values)
{
    Console.WriteLine(value);
}

//遍历所有键值对
foreach(KeyValuePair<int,string> pair in dict)
{
    Console.WriteLine($"键：{pair.Key}，值：{pair.Value}");
}
```

---

## 🔗 顺序存储和链式存储

### 数据结构

> 数据结构是计算机存储、组织数据的方式（规则）。
> 数据结构是指相互之间存在一种或多种特定关系的数据元素的集合。
> 比如自定义的一个类也可以称为一种数据结构——自己定义的数据组合规则。

> [!TIP] 理解要点
> 不要把数据结构想的太复杂。简单点理解，就是人定义的存储数据和表示数据之间关系的规则而已。

常用的数据结构（前辈总结和制定的一些经典规则）：
==数组、栈、队列、链表、树、图、堆、散列表==

### 线性表

> 线性表是一种数据结构，是由 n 个具有相同特性的数据元素的有限序列。
> 比如数组、ArrayList、Stack、Queue、链表等等。

### 顺序存储

> 用一组地址连续的存储单元依次存储线性表的各个数据元素。

### 链式存储

> 用一组任意的存储单元依次存储线性表的各个数据元素。

#### 简单单链表的实现

```csharp
class LinkedList<T>
{
    public LinkedNode<T> head = null;
    public LinkedNode<T> tail = null;

    public void Add(T value)
    {
        LinkedNode<T> node = new LinkedNode<T>(value);

        if (head == null)
        {
            head = node;
            tail = node;
        }
        else
        {
            tail.nextNode = node;
            tail = node;
        }
    }

    public void Remove(T value)
    {
        if(head == null)
        {
            return;
        }

        if(head.value.Equals(value))
        {
            head = head.nextNode;
            if(head == null)
            {
                tail = null;
            }
        }

        LinkedNode<T> node = head;
        while (node.nextNode != null)
        {
            if (node.nextNode.value.Equals(value))
            {
                node.nextNode = node.nextNode.nextNode;
                break;
            }
            node = node.nextNode;
        }
    }
}
```

#### 简单双链表的实现

```csharp
using System;

/// <summary>
/// 双向链表节点
/// </summary>
public class DoublyNode<T>
{
    public T Data { get; set; }
    public DoublyNode<T>? Prev { get; set; } // 前驱
    public DoublyNode<T>? Next { get; set; } // 后继

    public DoublyNode(T data)
    {
        Data = data;
        Prev = null;
        Next = null;
    }
}

/// <summary>
/// 自定义双向链表
/// </summary>
public class DoublyLinkedList<T>
{
    // ===== 公有属性 =====
    public DoublyNode<T>? Head { get; private set; }
    public DoublyNode<T>? Tail { get; private set; }
    public int Count { get; private set; }

    public DoublyLinkedList()
    {
        Head = null;
        Tail = null;
        Count = 0;
    }

    // ===== 1. 添加数据到链表最后 =====
    public void AddLast(T data)
    {
        var newNode = new DoublyNode<T>(data);

        if (Head == null) // 空链表
        {
            Head = newNode;
            Tail = newNode;
        }
        else
        {
            // 链接新节点到尾部
            Tail!.Next = newNode; // Tail 不可能为 null，用 ! 消除警告
            newNode.Prev = Tail;
            Tail = newNode;
        }
        Count++;
    }

    // ===== 2. 删除指定位置节点（索引从 0 开始） =====
    public bool RemoveAt(int index)
    {
        // 检查索引有效性
        if (index < 0 || index >= Count)
            return false;

        // 情况1：删除唯一节点（链表只有一个节点）
        if (Count == 1)
        {
            Head = null;
            Tail = null;
            Count = 0;
            return true;
        }

        // 定位到要删除的节点
        DoublyNode<T> current;
        // 优化：如果索引在前半段，从头向后找；否则从尾向前找
        if (index < Count / 2)
        {
            current = Head!;
            for (int i = 0; i < index; i++)
                current = current.Next!;
        }
        else
        {
            current = Tail!;
            for (int i = Count - 1; i > index; i--)
                current = current.Prev!;
        }

        // 删除逻辑
        if (current.Prev == null) // 删除头节点
        {
            Head = current.Next;
            Head!.Prev = null;
        }
        else if (current.Next == null) // 删除尾节点
        {
            Tail = current.Prev;
            Tail!.Next = null;
        }
        else // 删除中间节点
        {
            current.Prev.Next = current.Next;
            current.Next.Prev = current.Prev;
        }

        Count--;
        return true;
    }

    // ===== 辅助方法：正向打印（用于测试） =====
    public void PrintForward()
    {
        if (Head == null)
        {
            Console.WriteLine("链表为空");
            return;
        }
        var current = Head;
        Console.Write("Head -> ");
        while (current != null)
        {
            Console.Write($"{current.Data} -> ");
            current = current.Next;
        }
        Console.WriteLine("null");
    }

    // ===== 辅助方法：反向打印（验证双向链接） =====
    public void PrintBackward()
    {
        if (Tail == null)
        {
            Console.WriteLine("链表为空");
            return;
        }
        var current = Tail;
        Console.Write("Tail -> ");
        while (current != null)
        {
            Console.Write($"{current.Data} -> ");
            current = current.Prev;
        }
        Console.WriteLine("null");
    }
}
```

### 顺序存储和链式存储的优缺点

> 从增删查改的角度去思考：

| 操作 | 顺序存储 | 链式存储 | 原因 |
|:--:|:--:|:--:|------|
| **增** | ⭐⭐ | ⭐⭐⭐ | 中间插入时链式不用移动位置 |
| **删** | ⭐⭐ | ⭐⭐⭐ | 中间删除时链式不用移动位置 |
| **查** | ⭐⭐⭐ | ⭐⭐ | 数组可直接通过下标得到元素，链式需要遍历 |
| **改** | ⭐⭐⭐ | ⭐⭐ | 数组可直接通过下标得到元素，链式需要遍历 |

---

## 🔗 LinkedList

> LinkedList 是 C# 中提供的一个双向链表类，用于存储元素。
> 本质是一个==可变类型的泛型双向链表==。

### 声明

```csharp
using System.Collections.Generic;

LinkedList<int> linkedList = new LinkedList<int>();
```

### 增

```csharp
//在链表尾部添加元素
linkedList.AddLast(1);

//在链表头部添加元素
linkedList.AddFirst(0);

//在指定节点后添加元素
linkedList.AddAfter(linkedList.First, 2);

//在指定节点前添加元素
linkedList.AddBefore(linkedList.First, -1);
```

### 删

```csharp
//移除头节点
linkedList.RemoveFirst();
//移除尾节点
linkedList.RemoveLast();

//移除指定节点 无法通过索引删除，只能通过值删除
linkedList.Remove(1);

//清空
linkedList.Clear();
```

### 查

```csharp
//头节点
LinkedListNode<int> head = linkedList.First;
//尾节点
LinkedListNode<int> tail = linkedList.Last;
//链表长度
int count = linkedList.Count;

//找到指定值的节点
LinkedListNode<int> node = linkedList.Find(1); //找不到返回null

//判断元素是否存在
bool isExist = linkedList.Contains(1);
```

### 改

```csharp
//需要先得到节点
LinkedListNode<int> node = linkedList.Find(1);
//修改节点值
node.Value = 100;
```

### 遍历

```csharp
//foreach遍历链表
foreach (int item in linkedList)
{
    Console.WriteLine(item);
}

//通过节点遍历
//从头遍历
LinkedListNode<int> current = linkedList.First;
while (current != null)
{
    Console.WriteLine(current.Value);
    current = current.Next;
}

//从尾遍历
current = linkedList.Last;
while (current != null)
{
    Console.WriteLine(current.Value);
    current = current.Prev;
}
```

---

## 📚 泛型栈和队列

### 声明

```csharp
using System.Collections.Generic;

Stack<int> stack = new Stack<int>();
Queue<int> queue = new Queue<int>();
```

> [!TIP]
> 使用和之前的栈和队列是一致的，但泛型版本**避免了装箱拆箱**，性能更好。

---

## 🎯 委托

### 委托的定义

> 委托是函数（方法）的容器，可以理解为表示函数（方法）的变量类型，用来存储、传递函数（方法）。
> 委托的本质是一个类，用来定义函数（方法）的类型（返回值和参数的类型）。
> ==不同的函数（方法）必须对应各自"格式"一致的委托。==

### 基本语法

```csharp
// 关键字 delegate
// 访问修饰符 delegate 返回值 委托名(参数列表);
//写在哪里 在namespace中 或 class语句块中

//申明一个无返回值无参数的委托 委托是没有重载的 默认public
public delegate void MyDelegate();
```

### 使用自定义委托

```csharp
//申明一个自定义委托
public delegate void MyDelegate(int a, int b);

//申明符合委托格式的函数
public void MyMethod(int a, int b)
{
    Console.WriteLine($"a={a}, b={b}");
}

//将函数存放在委托实例中
MyDelegate myDelegate = new MyDelegate(MyMethod);
// 或者
MyDelegate myDelegate = MyMethod;

//调用委托实例
myDelegate.Invoke(1, 2);

//委托用于类的成员变量
public class MyClass
{
    public MyDelegate myDelegate { get; set; }
}

//委托用于函数的参数
public void MyMethod(MyDelegate del)
{
    del(1, 2);
}
```

### 多播委托

> 委托可以存储多个函数（方法）的实例，每个实例都可以被调用。

```csharp
//申明一个多播委托
public delegate void MyMulticastDelegate(int a, int b);

//申明符合委托格式的函数
public void MyMethod1(int a, int b)
{
    Console.WriteLine($"a={a}, b={b}");
}

public void MyMethod2(int a, int b)
{
    Console.WriteLine($"a={a}, b={b}");
}

//将函数存放在多播委托实例中
MyMulticastDelegate myMulticastDelegate = MyMethod1;
myMulticastDelegate += MyMethod2;

//调用多播委托实例
myMulticastDelegate.Invoke(1, 2);

```

**增：**

```csharp
//在多播委托实例中添加函数
myMulticastDelegate += MyMethod3;
```

**删：**

```csharp
//从多播委托实例中移除函数 如果多减也不会报错
myMulticastDelegate -= MyMethod3;

//清空多播委托实例
myMulticastDelegate = null;
```

### 系统提供的委托

| 委托类型 | 说明 | 示例 |
|------|------|------|
| `Action` | 无返回值，0~16个参数 | `Action<int,string>` |
| `Func<T>` | 有返回值，0~16个参数，最后一个泛型是返回值 | `Func<int,int,int>`（两个int参数，返回int） |

```csharp
//使用系统自带委托需要引入using System;
public void MyFunc()
{
    Console.WriteLine("MyFunc");
}
//无返回值无参数的委托
Action action = MyFunc;

public int MyFunc2()
{
    return 100;
}
//任意返回值无参数的委托 public delegate TResult Func<out TResult>();
Func<int> intFunc = MyFunc2;

public void MyFunc3(int a, string b, bool c)
{
    Console.WriteLine($"a={a}, b={b}, c={c}");
}

//可以传n个参数的 系统提供的1~16个参数的委托
Action<int,string,bool> action3 = MyFunc3;

public int MyFunc4(int a, int b)
{
    return a + b;
}
//可以传n个参数 并且有返回值的委托
Func<int,int,int> intFunc4 = MyFunc4;
```

---

## ⚡ 事件

> 事件是基于委托的存在。事件是委托的安全包裹，让委托的使用更具有安全性。是一种特殊的变量类型。

### 事件的使用

```csharp
//申明语法： 访问修饰符 event 委托类型 事件名;
//事件的使用：
//1.事件是作为成员变量存在于类中
//2.委托怎么用事件就怎么用
//事件相对于委托的区别：
//1.不能在类外部 赋值
//2.不能再类外部 调用
//注意：它只能作为成员存在于类和接口以及结构体中
using System;

namespace Lesson
{
    class Test
    {
        public Action MyDel;
        public event Action MyEvent;

        public Test()
        {

        }

        public void Func()
        {
            Console.WriteLine("123");
        }

        //事件只能通过类中的函数进行安全调用
        public void RaiseEvent()
        {
            MyEvent?.Invoke();
        }
    }
    class Program
    {
        static void Main(string[] args)
        {
            Test t = new Test();

            //委托可以在类的外部赋值和调用
            t.MyDel = Func1;
            t.MyDel += Func1;
            t.MyDel.Invoke();

            //事件不能在类的外部赋值和调用
            //t.MyEvent = Func1;
            //t.MyEvent.Invoke();
            //但事件可以在类的外部进行订阅和取消订阅
            t.MyEvent += Func1;
            t.MyEvent -= Func1;
        }

        public static void Func1()
        {
            Console.WriteLine("131");
        }
    }
}
```

### 事件的作用

> [!IMPORTANT] 三大安全保护
> 1. 防止外部随意置空委托
> 2. 防止外部随意调用委托
> 3. 事件相当于对委托进行了一次封装，让其更加安全

---

## 🔒 匿名函数

> 没有名称的函数，主要配合委托和事件来使用。脱离委托和事件是不会使用匿名函数的。

### 基本语法

```csharp
/*delegate (参数列表)
{
    //函数体
}
何时使用？
1.函数中传递委托参数时
2.委托或事件赋值时
*/

//使用：
//无参无返回
Action action = delegate()
{
    Console.WriteLine("123");
};

//有参数
Action<int> action2 = delegate(int a)
{
    Console.WriteLine($"a={a}");
};

//有返回值
Func<int> intFunc = delegate()
{
    return 100;
};

//一般情况作为函数参数传递 或者 作为函数的返回值
//作为参数传递
class Test
{
    public Action action;

    public void DoSomething(Action action)
    {
        action?.Invoke();
    }
//作为返回值
    public Action GetAction()
    {
        return delegate()
        {
            Console.WriteLine("123");
        };
    }
}

class Program
{
    static void Main(string[] args)
    {
        Test t = new Test();
        t.DoSomething(delegate()
        {
            Console.WriteLine("123");
        });
        Action action = t.GetAction();
        action?.Invoke();
    }
}
```

> [!WARNING] 匿名函数的缺点
> 添加到委托或事件容器后，不记录，无法单独移除。只能清空整个容器。

---

## ⚡ Lambda 表达式

> 可以将 Lambda 表达式理解为匿名函数的简写。它除了写法不同外，使用上和匿名函数一模一样，都是配合委托和事件来使用的。

### Lambda 表达式的基本语法

```csharp
/*(参数列表) =>
{
    //函数体
}
*/

//使用：
//无参无返回
Action action = () =>
{
    Console.WriteLine("123");
};

//有参数
Action<int> action2 = (int a) =>
{
    Console.WriteLine($"a={a}");
};

Action<int> action3 = (value) =>
{
    Console.WriteLine($"value={value}"); //省略参数类型
}

//有返回值
Func<int> intFunc = () =>
{
    return 100;
};
```

> [!WARNING]
> 缺点也与匿名函数一致。

---

### 闭包

> 内层的函数可以引用包含在它外层的函数的变量，即使外层函数的执行已经终止。
> ==注意==：该变量提供的值并非临时创建的值，而是在父函数范围内的**最终值**。

```csharp
//闭包的使用
class Test
{
    public Action action；

    public Test()
    {
        int value = 100;

        action = () =>
        {
            //这里形成了闭包 value的生命周期被改变了
            Console.WriteLine($"value={value}");
        };

        value = 1000;
    }

    public void DoSomething()
    {
        action?.Invoke(); //打印的value是1000 因为是父函数的最终值
    }
}
```

---

## 🔄 协变逆变

### 什么是协变逆变？

| 概念 | 说明 | 关键字 |
|------|------|:--:|
| **协变** | 和谐的变化，自然的变化。因为里氏替换原则，父类可以作为子类的容器 | `out` |
| **逆变** | 逆常规的变化，不正常的变化。因为里氏替换原则，子类不能用作父类的容器 | `in` |

> [!NOTE]
> 协变和逆变是用来修饰泛型的。只有在==泛型接口和泛型委托==中才能使用。

### 作用

```csharp
//1.返回值和参数
//用out修饰的泛型 只能用作返回值 不能用作参数
delegate T Get<out T>();

//用in修饰的泛型 只能用作参数 不能用作返回值
delegate void Set<in T>(T t);

//2.里氏替换原则

delegate T TestOut<out T>();
delegate void TestIn<in T>(T t);

class Father
{

}

class Son: Father
{

}

class Program
{
    static void Main(string[] args)
    {
        TestOut<Son> oS = () =>
        {
            return new Son();
        };

        TestOut<Father> oF = oS;  //协变 在delegate T TestOut<out T>()添加了out后 可以将子类赋值给父类的委托 这里是子类的委托装进父类中

        TestIn<Father> iF = (t) =>
        {

        };

        TestIn<Son> iS = iF;  //逆变 在delegate void TestIn<in T>()添加了in后 可以将父类赋值给子类的委托
        iS?.Invoke(new Son()); //实际上仍然是父类装子类 in代表的是参数 所以这里表示子类参数装进父类中
    }
}
```

---

## 🧵 多线程

### 进程

> 进程（Process）是计算机中的程序关于某数据集合上的一次运行活动，是系统进行资源分配和调度的基本单位，是操作系统结构的基础。
> 进程之间可以相互运行，互不干扰。进程之间也可以互相访问、操作。

### 线程

> 线程是操作系统进行运算调度的最小单位。它被包含在进程当中，是进程中的实际运作单位。
> 一条线程指的是进程中一个单一顺序的控制流，一个进程可以并发多个线程。

### 什么是多线程？

> 我们可以通过代码开启新的线程，可以同时运行代码的多条"管道"，就叫多线程。

### 语法相关

```csharp
//线程类 Thread
//需要引用System.Threading命名空间
//1.申明一个新的线程 注意线程执行的代码需要封装到一个函数中
Thread thread = new Thread(() =>
{
    //新线程 将要执行的代码逻辑 被封装进这个语句块中 和主函数就没有关系了
    Console.WriteLine("123");
}); //将委托作为参数传入

//2.启动线程
thread.Start();

//3.设置为后台线程（即主线程结束时 后台线程也会被结束）如果不设置为后台线程 可能会导致进程无法正常结束
thread.IsBackground = true;

//4.关闭释放一个线程  如果开启的线程不是死循环 那么不需要刻意去关闭它
//1.用bool变量控制线程是否执行
bool isRunning = true;
thread.Start(() =>
{
    while (isRunning)
    {
        Console.WriteLine("123");
    }
});

bool isRunning = false;

//2.通过线程提供的Abort方法关闭线程 （注意在.Net core版本中无法终止 会报错）
thread.Abort();
thread = null;

//5.线程休眠  单位是毫秒  在哪个线程中调用 就会在哪个线程中休眠
Thread.Sleep(1000);
```

### 线程之间共享数据

> [!WARNING] 线程安全
> 多个线程使用的内存是共享的，所以在多线程中需要注意数据的同步问题。
> 可以通过加锁来解决线程之间的数据问题，即在访问共享数据时，加上锁，确保只有一个线程可以访问共享数据。

```csharp
//1.用lock关键字加锁
//在访问共享数据时 加上锁 确保只有一个线程可以访问共享数据
//lock(引用类型对象)
int value = 0;
void Increment()
{
    lock (this)
    {
        value++;
    }
}
```

### 多线程的意义

> 可以专门处理一些复杂耗时的逻辑，比如寻路、网络通信等。

---

## 🔧 预处理器指令

> 预处理器指令指导编译器在实际编译开始之前对信息进行预处理。
> 预处理器指令都是以 `#` 开始。
> 预处理器指令不是语句，所以它们不以分号 `;` 结束。

### 常见的预处理器指令

```csharp
//1. #define 定义宏
#define MACRO_NAME 123
#undef MACRO_NAME //取消宏定义 这两个通常配合#if和#endif使用

//2.#if #elif #else #endif条件编译
#if MACRO_NAME == 123
    //如果MACRO_NAME等于123 则编译这里的内容
#endif //如果MACRO_NAME不等于123 则不编译这里的内容

//3.#warning 警告 #error 错误
#warning 这是一个警告
#error 这是一个错误
```

---

## 🔍 反射

### 程序集

> 程序集是由编译器编译得到的，供进一步编译执行的中间产物。
> 在 Windows 系统中，它一般表现为后缀为 `.dll` 的文件或 `.exe` 文件。

### 元数据

> 元数据就是用来描述数据的数据。这个概念不仅应用于程序上，在别的领域也有元数据。
> 简单来说，程序中的类、类中的函数、变量等等信息就是程序的元数据。

### 反射的概念

> 程序正在运行时，可以查看其他程序集或者自身的元数据。
> 一个运行的程序查看本身或者其他程序的元数据的行为就叫做==反射==。

### 反射的作用

> [!IMPORTANT] 核心价值
> 因为反射可以在程序编译后获得信息，所以它提高了程序的拓展性和灵活性。
> 1. 程序运行时得到所有元数据，包括元数据的特性
> 2. 程序运行时，实例化对象，操作对象
> 3. 程序运行时创建新对象，用这些对象执行任务

### 语法相关

#### Type

> Type（类的信息类）是反射功能的基础！
> 它是访问元数据的主要方式。
> 使用 Type 的成员获取有关类型声明的信息。
> 有关类型的成员（如构造函数、方法、字段、属性和类的事件）。

**获取 Type：**

```csharp
//1.通过类型名获取Type
Type type = typeof(int);

//2.通过对象实例获取Type
Type type = obj.GetType();

//3.通过类的命名获取Type（需要引用命名空间）
Type type = Type.GetType("System.Int32");

//得到类的程序集信息
Assembly assembly = type.Assembly;
```

**获取类中的成员：**

```csharp
class Test
{
    public int a = 1;
    public string b = "hello";
    private int c = 2;

    public Test()
    {

    }

    public Test(int a)
    {
        this.a = a;
    }

    public Test(int a, string b):this(a)
    {
        this.b = b;
    }

    public void Func()
    {

    }
}

Type testType = typeof(Test);

//得到公共成员需要引用System.Reflection命名空间
MemberInfo[] members = testType.GetMembers();

//遍历成员
foreach (MemberInfo member in members)
{
    Console.WriteLine(member.Name);
}

//获取类的构造函数
ConstructorInfo[] constructors = testType.GetConstructors();

//遍历构造函数
foreach (ConstructorInfo constructor in constructors)
{
    Console.WriteLine(constructor.Name);
}

//获取其中一个构造函数 并执行
//获取构造函数传入 Type数组 数组中按内容按顺序是参数类型
//执行构造函数传入 object数组 表示按顺序传入的参数
//得到无参构造函数
ConstructorInfo constructor = testType.GetConstructor(new Type[] { });

//执行构造函数
Test test = constructor.Invoke(null) as Test;

//得到有参构造函数
ConstructorInfo constructor1 = testType.GetConstructor(new Type[] { typeof(int), typeof(string) });
//执行构造函数
test = constructor1.Invoke(new object[] { 123, "hello" }) as Test;

//获取类的公共成员变量
FieldInfo[] fields = testType.GetFields();
//遍历成员变量
foreach (FieldInfo field in fields)
{
    Console.WriteLine(field.Name);
}

//获取指定一个公共成员变量
//获取成员变量传入字符串 表示成员变量的名称
FieldInfo fieldA = testType.GetField("a");
//执行成员变量
Test test1 = new Test;
int value = (int)fieldA.GetValue(test1);

//设置成员变量的值
fieldA.SetValue(test1, 100); //设置成员变量a的值为100

//获取类中的成员方法
MethodInfo[] methods = testType.GetMethods();
//遍历成员方法
foreach (MethodInfo method in methods)
{
    Console.WriteLine(method.Name);
}

//如果方法存在重载  用Type数组表示参数类型
Type strType = typeof(string);

MethodInfo subStr = testType.GetMethod("Substring", new Type[] { typeof(int), typeof(int) });

string str = "hello world";
//调用方法
object subStr1 = subStr.Invoke(str, new object[] { 0, 5 });
Console.WriteLine(subStr1); //hello

//其他的还有
//获得枚举
EnumInfo[] enums = testType.GetEnums();

//获得事件
EventInfo[] events = testType.GetEvents();

//获得属性
PropertyInfo[] properties = testType.GetProperties();

//获得接口
InterfaceInfo[] interfaces = testType.GetInterfaces();
```

#### Activator

> 用于快速实例化对象的类。
> 用于将 Type 对象快捷实例化为对象。
> 先得到 Type，然后快速实例化一个对象。

```csharp
class Test
{
    public int a = 1;
    public string b = "hello";
    private int c = 2;

    public Test()
    {

    }

    public Test(int a)
    {
        this.a = a;
    }

    public Test(int a, string b):this(a)
    {
        this.b = b;
    }

    public void Func()
    {

    }
}
//1.无参构造
Type testType = typeof(Test);
Test testObj = Activator.CreateInstance(testType) as Test;

//2.有参构造
testObj = Activator.CreateInstance(testType,99) as Test; //调用一个参数构造函数
testObj = Activator.CreateInstance(testType,99,"hello") as Test; //调用两个参数构造函数
```

#### Assembly

> 程序集类。
> 主要用来加载其它程序集，加载后才能用 Type 来使用其它程序集中的信息。
> 如果想要使用不是自己程序集中的内容需要先加载程序集。
> 比如 dll 文件（库文件）。
> 简单的把库文件看成一种代码仓库，它提供给使用者一些可以直接拿来用的变量、函数或类。

**三种加载程序集的函数：**

| 方法 | 使用场景 |
|------|------|
| `Assembly.Load("程序集名称")` | 一般用来加载在同一文件下的其它程序集 |
| `Assembly.LoadFrom("路径")` | 一般用来加载不在同一文件下的其它程序集 |
| `Assembly.LoadFile("完全限定路径")` | 加载指定完全路径的程序集 |

```csharp
// 需要引用 System.Reflection 命名空间
using System.Reflection;

// ========== 1. Assembly.Load — 加载当前目录下的程序集 ==========
// 假设有一个外部类库项目 MyLibrary，编译后生成 MyLibrary.dll
// MyLibrary.dll 中定义了一个 Calculator 类：
//
// namespace MyLibrary
// {
//     public class Calculator
//     {
//         public int Add(int a, int b) => a + b;
//         public int Multiply(int a, int b) => a * b;
//     }
// }

Assembly asm1 = Assembly.Load("MyLibrary"); // 通过程序集名称加载（不含 .dll 后缀）

// ========== 2. Assembly.LoadFrom — 通过路径加载 ==========
Assembly asm2 = Assembly.LoadFrom(@"C:\Libs\MyLibrary.dll");

// ========== 3. Assembly.LoadFile — 加载指定路径的程序集 ==========
Assembly asm3 = Assembly.LoadFile(@"C:\Libs\MyLibrary.dll");

// ========== 使用加载的程序集中的类型 ==========

// 获取程序集中的类型（通过完整类名：命名空间.类名）
Type calcType = asm1.GetType("MyLibrary.Calculator");

// 通过 Activator 创建该类型的实例
object calcObj = Activator.CreateInstance(calcType);

// 获取方法信息
MethodInfo addMethod = calcType.GetMethod("Add");
MethodInfo multiplyMethod = calcType.GetMethod("Multiply");

// 调用方法（Invoke：参数1为实例对象，参数2为方法参数数组）
object result1 = addMethod.Invoke(calcObj, new object[] { 10, 20 });
Console.WriteLine($"10 + 20 = {result1}"); // 输出：10 + 20 = 30

object result2 = multiplyMethod.Invoke(calcObj, new object[] { 5, 6 });
Console.WriteLine($"5 * 6 = {result2}"); // 输出：5 * 6 = 30

// ========== 遍历程序集中所有公开类型 ==========
Type[] types = asm1.GetExportedTypes();
foreach (Type t in types)
{
    Console.WriteLine($"类型名: {t.FullName}");

    // 遍历该类型的所有公开方法
    foreach (MethodInfo method in t.GetMethods())
    {
        Console.WriteLine($"  方法: {method.Name}, 返回值: {method.ReturnType.Name}");
    }
}

// ========== 获取并设置属性 ==========
// 假设 Calculator 类有一个属性：public string Name { get; set; }
PropertyInfo nameProp = calcType.GetProperty("Name");
if (nameProp != null)
{
    nameProp.SetValue(calcObj, "我的计算器");
    Console.WriteLine($"Name = {nameProp.GetValue(calcObj)}");
}
```

---

## 🏷️ 特性

> 特性是一种允许我们向程序的程序集添加元数据的语言结构。
> 它是用于保存程序结构信息的某种特殊类型的类。
> 特性提供功能强大的方法以将声明信息与代码（类型、方法、属性等）相关联。
> 特性与程序实体关联后，即可在运行时使用反射查询特性信息。
> 特性的目的是告诉编译器把程序结构的某组元数据嵌入程序集中。
> 它可以放置在几乎所有的声明中（类、变量、函数等等声明）。

### 自定义特性

```csharp
//继承特性基类 Attribute
public class MyAttribute : Attribute
{
    //特性中的成员
    public string name;

    //特性中的构造函数
    public MyAttribute(string name)
    {
        this.name = name;
    }
}
```

### 特性的使用

```csharp
//基本语法
//[特性名(参数列表)]
//写在哪里
//类、函数、变量上一行 表示他们具有该特性信息

[MyAttribute("这是一个测试类")]
class Test
{
    [MyAttribute("这是一个测试成员变量")]
    public int a = 100;

    [MyAttribute("这是一个测试方法")]
    public void TestMethod([MyAttribute("这是一个测试方法参数")] int a)
    {

    }
}

//判断是否有特性信息
class Program
{
    static void Main(string[] args)
    {
        Type testType = typeof(Test);
        //IsDefined 第一个参数为特性类型，第二个参数为是否继承特性
        if(testType.IsDefined(typeof(MyAttribute), false))
        {
            Console.WriteLine("测试类具有 MyAttribute 特性");
        }

        //获取元数据中的所有特性
        object[] attributes = testType.GetCustomAttributes(typeof(MyAttribute), false);
        foreach (object attribute in attributes)
        {
            if(attribute is MyAttribute)
            {
                Console.WriteLine((attribute as MyAttribute).name);
            }
        }


    }
}
```

### 限制自定义特性的使用范围

```csharp
//通过为特性类 加特性 来限制特性使用范围
[AttributeUsage(AttributeTargets.Class | AttributeTargets.Struct, AllowMultiple = true, Inherited = true)]
//参数一：AttributeTargets 特性使用范围
//参数二：AllowMultiple 是否允许重复使用特性
//参数三：Inherited 是否继承特性

```

### 系统自带的特性

**过时特性 `Obsolete`：**

```csharp
//过时特性
//Obsolete
//用于标记过时的代码，提示开发者使用新的代码 一般加在函数前

class Test
{
    [Obsolete("请使用 New 方法"),false]
    //参数一：提示信息
    //参数二：true 使用Old会直接报错，false 不会报错
    public void Old()
    {

    }

    public void New()
    {

    }

    public void TestMethod([CallerMemberName] string methodName, [CallerLineNumber] int lineNumber, [CallerFilePath] string filePath)
    {
        Console.WriteLine($"调用者信息：{methodName}，{lineNumber}，{filePath}");
    }
}

//调用者信息特性
//哪个文件调用
//CallerFilePath
//哪一行调用
//CallerLineNumber
//哪个函数调用
//CallerMemberName

//需要引用 System.Runtime.CompilerServices 命名空间
//一般作为函数参数的特性

class Program
{
    static void Main(string[] args)
    {
        Test test = new Test();
        test.TestMethod(); // 输出：调用者信息：Main，行号，文件路径
    }
}
```

**条件编译特性 `Conditional`：**

```csharp
//条件编译特性
//Conditional
//他会和预处理器指令 #define 一起使用，来根据不同的编译环境来编译不同的代码

class Test
{
    [Conditional("DEBUG")]
    public void TestMethod()
    {
        Console.WriteLine("DEBUG 环境下编译");
    }
}

class Program
{
    static void Main(string[] args)
    {
        Test test = new Test();
        test.TestMethod(); // 需要由#define DEBUG 才会输出：DEBUG 环境下编译
    }
}
```

**外部 DLL 包函数特性 `DllImport`：**

```csharp
//外部DLL包函数特性
//DllImport
//用来标记非.Net（非C#）函数，表明该函数在一个外部DLL中定义
//一般用来调用C或C++的DLL包编写好的方法
//需要引用命名空间 System.Runtime.InteropServices 命名空间

[DllImport("msvcrt.dll")] // 调用C++的printf函数
public static extern int printf(string format, params object[] args);

```

---

## 🔁 迭代器

> 迭代器（Iterator）有时又称光标（Cursor），是程序设计的软件设计模式。
> 迭代器模式提供一个方法顺序访问一个聚合对象中的各个元素，而又不暴露其内部的标识。

在表现效果上看：
- 迭代器是可以在容器对象（例如链表或数组）上遍历访问的接口
- 设计人员无需关心容器对象的内存分配的实现细节
- ==可以用 `foreach` 遍历的类，都是实现了迭代器的==

### 标准迭代器的实现方法

```csharp
//关键接口：IEnumerator,IEnumerable
//命名空间：System.Collections.Generic
//可以通过同时继承IEnumerator,IEnumerable 来实现其中的方法

class CustomList:IEnumerable,IEnumerator
{
    private int[] list;
    private int currentIndex = -1;

    public CustomList()
    {
        list = new int[10] {1,2,3,4,5,6,7,8,9,10};
    }

    public IEnumerator GetEnumerator()
    {
        Reset();
        return this;
    }

    public object Current
    {
        get
        {
            return list[currentIndex];
        }
    };

    public bool MoveNext()
    {
        //移动光标
        ++currentIndex;
        //判断是否溢出 如果溢出 则返回false
        return currentIndex < list.Length;
    }

    //重置光标 一般写在IEnumerator对象的函数中 用于第一次重置光标位置
    public void Reset()
    {
        currentIndex = -1;
    }
}

CustomList customList = new CustomList();

//foreach本质
//1.先获取in后面对象的IEnumerator接口 会调用对象中的GetEnumerator方法 来获取
//2.执行IEnumerator对象中的 MoveNext方法
//3.只要MoveNext返回值是true 就会去得到Current 然后复制给Item
foreach (var item in customList)
{
    Console.WriteLine(item);
}
```

### 用 yield return 实现迭代器

> `yield return` 是 C# 提供给我们的语法糖。
> 所谓语法糖，也称糖衣语法。
> 主要作用就是将复杂逻辑简单化，可以增加程序的可读性。
> 从而减少程序代码出错的机会。

```csharp
class CustomList2:IEnumerable
{
    private int[] list;
    private int currentIndex = -1;

    public CustomList2()
    {
        list = new int[10] {1,2,3,4,5,6,7,8,9,10};
    }

    public IEnumerator GetEnumerator()
    {
        for (int i = 0; i < list.Length; i++)
        {
            //yield关键字 配合迭代器使用
            //可以理解为 暂时返回 保留当前状态
            //一会还会回来当前状态
            yield return list[i];
        }
    }
}
```

---

## ✨ 特殊语法

### var 隐式类型

> `var` 是一种特殊的变量类型，它可以用来表示任意类型的变量。

> [!WARNING] 注意
> 1. `var` 不能作为类的成员，只能用于临时变量声明时（一般写在函数语句块中）
> 2. `var` 必须初始化

```csharp
var a = 10;
var b = "hello";
var c = new int[] {1,2,3,4,5};
var d = new List<int>();
```

### 设置对象初始值

```csharp
//申明对象时 可以直接通过写大括号的形式来初始化公共成员变量和属性
class Person
{
    public int age;
    public string name;
}

Person person = new Person {age = 18, name = "张三"};

//集合也可以用大括号申明
 var list = new List<int> {1,2,3,4,5};

 //匿名类型
 //var 变量可以申明为自定义的匿名类型
 var anonymous = new {age = 18, name = "张三"};

 //可空类型
 //值类型是不可空的 但是可以使用可空类型来表示可空的值
 int? a = 10;   //a可以是10 也可以是null
 int? b = null;

 //判断是否可空类型是否有值
 if(a.HasValue)
 {
    Console.WriteLine(a.Value);
 }

 //安全获取可空类型的值
 Console.WriteLine(a.GetValueOrDefault());
 Console.WriteLine(b.GetValueOrDefault(100)); //指定默认值为100

 //引用类型判断是否为空
 object obj = null;

 obj?.ToString(); //如果obj为空 则不会调用ToString方法

```

### 空合并操作符

```csharp
//空合并操作符 ??
//左边值 ?? 右边值
//如果左边值为null 则返回右边值
//如果左边值不为null 则返回左边值
//只要是可以为null的类型都能用
int? a = null;
int b = 100;
int c = a ?? b;
Console.WriteLine(c); //100
```

### 内插字符串

```csharp
//关键符号 $
//用$来构造字符串 让字符串可以拼接变量

string str = "hello";
Console.WriteLine($"hello {str}"); //输出hello hello
```

---

## 📊 值类型和引用类型

| 分类 | 类型 |
|------|------|
| **值类型** | `byte`, `ushort`, `uint`, `ulong`, `sbyte`, `short`, `int`, `long`, `float`, `double`, `decimal`, `char`, `bool`, `enum`, `struct` |
| **引用类型** | `string`, 数组, `class`, `interface`, 委托 |

> [!IMPORTANT] 本质区别
> ==值类型的具体内容存在栈内存上，引用类型的具体内容存在堆内存上。==

---

## 📐 各种排序算法

```csharp
using System;

namespace Lesson
{
    class Program
    {
        static void Main(string[] args)
        {
            int[] arr = new int[] { 4, 2, 7, 2, 6, 9, 5, 1, 2, 4, 7 };
            SelectionSort(arr);

            for (int i = 0; i < arr.Length; i++)
            {
                Console.WriteLine(arr[i]);
            }
        }

        //冒泡排序
        //将数组的元素进行两两比较 大的元素放在小的后面
        //循环进行 找出最大元素放在最大位置
        //本质上是一次循环确认一个元素的最终位置
        static void BubbleSort(int[] arr)
        {
            for(int i = 0; i < arr.Length; i++)
            {
                for(int j = 1; j < arr.Length - i; j++)
                {
                    if (arr[j] < arr[j - 1])
                    {
                        int temp = arr[j];
                        arr[j] = arr[j - 1];
                        arr[j - 1] = temp;
                    }
                }
            }
        }

        //选择排序
        //拿一个max指针记录最大元素值
        //在每一次比较中更新这个max指针
        //内层循环结束后将他与数组的最大位置互换
        //本质上也是是一次循环确认一个元素的最终位置
        static void SelectionSort(int[] arr)
        {
            for(int i=0; i<arr.Length; i++)
            {
                int max = 0;
                for(int j = 0; j < arr.Length-i; j++)
                {
                    if(arr[j] > arr[max])
                    {
                        max = j;
                    }
                }

                int temp = arr[arr.Length - i - 1];
                arr[arr.Length - i-1] = arr[max];
                arr[max] = temp;
            }
        }

        //插入排序
        //将数组分为已排序区和待排序区
        //从待排序中取出首个元素与已排序区元素进行比较放在对应位置、
        //重复n次即可完成排序
        //本质上是小有序数组=>大有序数组
        static void InsertSort(int[] arr)
        {
            //重复n次
            for (int i = 1; i < arr.Length; i++)
            {
                //记录已排序区索引
                int sortIndex = i - 1;
                //取出未排序区元素
                int noSortNum = arr[i];

                //升序排序 找到待排序元素应该插入的位置
                while (sortIndex >= 0 && arr[sortIndex] > noSortNum)
                {
                    //将已排序元素后移 给待排序元素腾出位置
                    arr[sortIndex + 1] = arr[sortIndex];
                    --sortIndex;
                }
                //放入正确位置
                arr[sortIndex + 1] = noSortNum;
            }

        }

        //希尔排序
        //本质上还是插入排序 只是引入了步长的概念
        //将间隔步长相等的元素作为一组 在组内进行插入排序
        //循环缩小步长 再次进行插入排序
        //本质上也是小有序数组=>大有序数组
        static void ShellSort(int[] arr)
        {
            int step = arr.Length;
            //设置步长 每一次循环都除以2
            while ((step /= 2) != 0)
            {
                //这一段就是插入排序
                for (int i = step; i < arr.Length; i++)
                {
                    int noSortNum = arr[i];
                    int sortIndex = i - step;

                    while (sortIndex >= 0 && arr[sortIndex] > noSortNum)
                    {
                        arr[sortIndex + 1] = arr[sortIndex];
                        sortIndex -= step;
                    }
                    arr[sortIndex + step] = noSortNum;
                }
            }
        }

        //归并排序
        //将数组从中间进行递归划分 直到划分的数组两边都只有一个元素
        //将分成两份的左右数组内元素进行依次比较 按大小放入新数组中
        //递归进行 本质上也是小有序数组=>大有序数组
        static int[] Merge(int[] arr)
        {
            //递归结束条件
            if (arr.Length < 2)
            {
                return arr;
            }

            //从中间将数组划为两部分
            int mid = arr.Length / 2;

            int[] leftArr = new int[mid];
            int[] rightArr = new int[arr.Length - mid];

            //左右数组初始化
            for(int i = 0; i < arr.Length; i++)
            {
                if (i < mid)
                {
                    leftArr[i] = arr[i];
                }
                else
                {
                    rightArr[i-mid] = arr[i];
                }
            }
            //递归进行
            return Sort(Merge(leftArr), Merge(rightArr));
        }

        static int[] Sort(int[] leftArr, int[] rightArr)
        {
            //建立新的数组
            int[] newArr = new int[leftArr.Length + rightArr.Length];
            int left = 0;
            int right = 0;

            for(int i = 0; i < newArr.Length; i++)
            {
                //当左数组全部放进去后 说明右数组内的数全部大于左数组依次放入即可
                if (left >= leftArr.Length)
                {
                    for(int j = i; j < newArr.Length; j++)
                    {
                        newArr[j] = rightArr[right];
                        right++;
                    }
                    //中断最外层for循环以免越界
                    break;
                }

                //当右数组全部放进去后 说明左数组内的数全部大于右数组依次放入即可
                if (right >= rightArr.Length)
                {
                    for (int j = i; j < newArr.Length; j++)
                    {
                        newArr[j] = leftArr[left];
                        left++;
                    }
                    break;
                }

                //比较左右数组 小的放入新数组中
                if (leftArr[left] <= rightArr[right])
                {
                    newArr[i] = leftArr[left];
                    left++;
                }
                else if(leftArr[left] > rightArr[right])
                {
                    newArr[i] = rightArr[right];
                    right++;
                }
            }

            return newArr;
        }

        //快速排序
        //选出一个基准元素（一般是数组的第一个元素）
        //设置左右两个指针 将指针所指到的数组元素与基准元素比较
        //（升序排序）右指针元素小于基准元素则将其放入左指针所指位置中 不满足则指针左移
        //注意这里由于左指针一开始所指位置为基准元素的位置 故可认为左指针所指位置一开始为空
        //左指针元素大于基准元素则将其放入右指针所指位置中 不满足则指针右移
        //直到左右指针所指位置重合 将基准元素放入重合位置 确认了一个元素的最终位置
        //从基准元素那将数组分为左右两个数组一次进行递归上述步骤 即可
        //本质上是依次将元素放到最终的正确位置上 一次递归放一个元素
        static void QuickSort(int[] arr, int low, int high)
        {
            //递归终止条件
            if (low >= high)
            {
                return;
            }

            //  确认左右指针 基准元素
            int left = low;
            int right = high - 1 ;
            int baseNum = arr[low];

            while (left < right)
            {
                //判断右指针所指元素是否大于基准元素 满足则将右指针左移
                while (left < right && arr[right] >= baseNum) right--;
                //找到不满足的位置 将其放入左指针所指位置中
                if (left < right) arr[left] = arr[right];

                //判断左指针所指元素是否小于基准元素 满足则将左指针右移
                while (left < right && arr[left] <= baseNum) left++;
                //找到不满足的位置 将其放入右指针所指位置中
                if (left < right) arr[right] = arr[left];
            }

            //将基准元素放入最终位置
            arr[right] = baseNum;

            //左右数组递归进行排序
            QuickSort(arr, low, left-1);
            QuickSort(arr, left + 1, high);
        }


        //堆排序
        //依赖于最大堆性质
        //将数组递归建立为最大堆 每次建立完成后取出堆顶的元素放入数组的n-i位置中
        //n为数组长度 i为递归次数
        //本质上是每一次递归中确立一个元素的最终位置
        static void Heapify(int[] arr, int n, int i)
        {
            int largest = i;     // 假设当前节点是最大的
            int left = 2 * i + 1; // 左子节点
            int right = 2 * i + 2; // 右子节点

            // 如果左子节点存在且大于当前最大，则更新
            if (left < n && arr[left] > arr[largest])
                largest = left;

            // 如果右子节点存在且大于当前最大，则更新
            if (right < n && arr[right] > arr[largest])
                largest = right;

            // 如果最大值不是父节点本身，说明需要交换
            if (largest != i)
            {
                // 交换父节点和较大的子节点
                int temp = arr[i];
                arr[i] = arr[largest];
                arr[largest] = temp;

                // 递归下沉，确保交换后的子树依然满足堆性质
                Heapify(arr, n, largest);
            }
        }
        static void HeapSort(int[] arr)
        {
            int n = arr.Length;

            // ---------- 阶段一：构建最大堆 ----------
            // 从最后一个非叶子节点开始（n/2 - 1），从下往上 Heapify
            for (int i = n / 2 - 1; i >= 0; i--)
            {
                Heapify(arr, n, i);
            }

            // ---------- 阶段二：排序（提取最大值） ----------
            // 将堆顶（最大值）依次移到数组末尾，然后缩小堆大小
            for (int i = n - 1; i > 0; i--)
            {
                // 将当前堆顶（最大值）与堆末尾元素交换
                int temp = arr[0];
                arr[0] = arr[i];
                arr[i] = temp;

                // 缩小堆范围，重新调整堆顶（0 索引）使其满足最大堆
                Heapify(arr, i, 0);
            }
        }
    }
}
```

> [!TIP] 排序算法总结

| 算法 | 核心思想 | 时间复杂度 |
|------|------|:--:|
| **冒泡排序** | 两两比较，每轮确认一个最大元素位置 | O(n²) |
| **选择排序** | 每轮选出最大元素放到最终位置 | O(n²) |
| **插入排序** | 将元素插入已排序区的正确位置 | O(n²) |
| **希尔排序** | 引入步长的插入排序优化 | O(n log n) ~ O(n²) |
| **归并排序** | 递归分治，合并有序数组 | O(n log n) |
| **快速排序** | 基准元素分区，递归排序 | O(n log n) |
| **堆排序** | 构建最大堆，依次提取堆顶 | O(n log n) |
