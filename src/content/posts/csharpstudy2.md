---
title: C#学习笔记2
published: 2026-07-06
pinned: false
description: C#学习笔记2
author: boluobao
draft: false
---

# C#学习笔记2

## ArrayList

ArrayList 是C#为我们封装好的类 他的本质是一个object类型的数组 ArrayList类帮助我们实现了很多方法 比如数组的增删查改  

申明：
```csharp
ArrayList arrayList = new ArrayList(); //主要要引用System.Collections命名空间
```

增：
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

删：
```csharp
//移除指定元素 从数组头开始遍历 删除第一个匹配的元素
arrayList.Remove(1);
//移除指定索引的元素
arrayList.RemoveAt(0);
//清空数组
arrayList.Clear();
``` 

查：   
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

改：
```csharp
//修改指定位置的元素
arrayList[0] = "hello world";
```

遍历：
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

装箱拆箱：  
ArrayList本质上是一个可以自动扩容的object数组， 
由于用万物之父来存储数据，自然存在装箱拆箱。    
当往其中进行值类型存储时就是在装箱，当将值类型对象取出来转换使用时，就存在拆箱。  
所以ArrayList尽量少用。    

实例：
```csharp
int i =1;
arrayList.Add(i); //装箱

int j = (int)arrayList[0]; //拆箱
```

数组和ArrayList的区别： 
在C#中，`Array`（数组）和`ArrayList`都用于存储数据集合，但它们在设计哲学和特性上存在根本差异。简单来说，`Array`是**类型安全、大小固定**的高性能基础数据结构；而`ArrayList`是**非类型安全、可动态增长**的旧式集合类。    

目前，`ArrayList`在实际开发中已基本被泛型的`List<T>`所取代，但在理解C#集合类型演进时，了解其与`Array`的区别仍然很有价值。   

### 核心区别概览

| 特性 | Array (数组) | ArrayList |
| :--- | :--- | :--- |
| **类型安全** | **是**，存储类型在声明时固定，编译时检查。 | **否**，以`object`类型存储所有元素，可混存不同类型。 |
| **类型** | 强类型 | 非泛型集合 |
| **大小** | **固定**，创建后长度不可变。 | **动态**，可自动扩容。 |
| **性能** | **更高**，无额外类型转换开销。 | **较低**，涉及装箱/拆箱和类型转换。 |
| **内存分配** | 通常在**栈**上 | 在**堆**上 |
| **维度** | 支持**多维**（如二维数组）。 | 始终是**一维**的。 |
| **命名空间** | `System.Array` | `System.Collections` |

---

### 详细对比分析

#### 1. 类型安全与存储内容
*   **`Array`是类型安全的**：它在声明时就确定了元素类型（如 `int[]`, `string[]`），任何类型不匹配的操作都会在编译时报错，这能有效避免运行时的类型转换异常。
*   **`ArrayList`是非类型安全的**：它将所有元素都当作 `object` 类型处理，因此可以在同一个 `ArrayList` 中存储字符串、整数、自定义对象等不同类型的数据。但这种灵活性带来了风险，在取出元素时**必须进行显式类型转换**，如果类型不匹配，会抛出运行时异常。

#### 2. 性能差异：装箱与拆箱
这是两者性能差异的核心原因。
*   **`Array`性能优越**：由于存储的是特定类型（如 `int`），它直接操作值，**没有装箱和拆箱的开销**。
*   **`ArrayList`性能开销大**：当向 `ArrayList` 添加一个值类型（如 `int`）时，该值会被**装箱**（Boxing）为 `object` 类型；当从 `ArrayList` 读取该值并转换为 `int` 时，又会发生**拆箱**（Unboxing）。这个过程涉及内存分配和类型检查，会显著影响性能。

#### 3. 容量与灵活性
*   **`Array`大小固定**：一旦创建，其长度（`Length`）就不可改变。如果需要更大或更小的数组，必须创建一个新数组并复制元素。
*   **`ArrayList`可动态调整**：它就像一个会自动增长的容器，你可以随时添加（`Add`）、插入（`Insert`）或删除（`Remove`）元素，其容量（`Capacity`）会根据需要自动扩充。

#### 4. 功能与使用简便性
*   **`Array`功能基础**：提供基本的索引访问，本身没有提供添加、插入、删除元素的方法。
*   **`ArrayList`功能丰富**：提供了 `Add`、`Insert`、`Remove`、`Sort` 等众多便捷方法来操作集合。

### 总结与建议

1.  **当集合大小固定且元素类型单一时，应优先使用 `Array`**。例如，存储一周七天的名称。
2.  **在现代C#开发中，应避免使用 `ArrayList`**。它存在的类型安全问题和性能缺陷，已由泛型集合 `List<T>` 完美解决。
3.  **对于需要动态增删、且元素类型单一的场景，应使用 `List<T>`**。它兼具了 `ArrayList` 的灵活性和 `Array` 的类型安全与高性能。


## Stack  

Stack的本质：
Stack（栈）是一个c#为我们封装好的类 
它的本质也是object[]数组，只是封装了特殊的存储规则  
Stack是栈存储容器，栈是一种先进后出的数据结构   
先存入的数据后获取，后存入的数据先获取  
栈是先进后出    
