---
title: C#学习笔记1
published: 2026-07-01
pinned: false
description: C#学习笔记1
author: boluobao
draft: false
---

# C#学习笔记1


## 面向对象编程

用中文去形容一类对象把一类对象的共同点提取出来然后用程序语言把他翻译过来，带着对象的概念才程序中使用它们。

### 为什么要学习面向对象编程？

1.提高代码复用率
2.提高开发效率
3.提高程序可拓展性
4.清晰的逻辑关系

## 类和对象

1.类的申明和类对象的申明是两个概念  
2.类的申明是申明对象的模板 用来抽象（形容）显示事物的 
3.类对象的申明是用来表现现实中的对象个体的  

类是一个自定义的变量类型 是引用类型的变量 
实例化一个类对象 是在申明变量


### 实例化对象
```csharp
// 定义一个类
public class Person
{
    public string Name { get; set; }
    public int Age { get; set; }
}
```
实例化对象时，需要提供类的名称和括号，例如：
```csharp
// 实例化一个对象
Person p ; // 定义一个对象p，但是不初始化
Person p1 = null; // 定义一个对象p1，但是不初始化，值为null，不分配堆内存空间
Person p2 = new Person(); // 相当于一个Person类型的对象
```

若在类中申明了一个与类同名的变量例如
```csharp
public class Person
{
    public Person p = new Person(); //这样是错的!!!
}
```
此时p不能在类的声名中实例化，否则会报错。

## 成员方法

1.成员方法不要加static关键字  
2.成员方法必须实例化出对象 再通过对象来调用，相当于该对象执行了某个行为  
3.成员方法收到访问修饰符的影响

## 构造函数 析构函数 垃圾回收

### 构造函数
概念：在实例化对象时，自动调用的特殊方法，用来初始化对象的属性。（如果不写构造函数，默认会有一个无参数的构造函数）  
⚠ 注意：如果不自己实现无参构造函数而自己实现了有参构造函数，那么默认的无参构造函数就会被隐藏。

### 析构函数
概念：在对象被销毁时，自动调用的特殊方法，用来释放对象占用的资源。（如果不写析构函数，默认会有一个无参数的析构函数），对于需要手动管理内存的语言（如C++）,需要在析构函数中做一些内存回收处理，但是在C#中存在自动垃圾回收机制GC,所以我们几乎不会怎么使用析构函数，除非你想在某一个对象被垃圾回收时做一些特殊处理。   

析构函数是在垃圾真正被回收时才会调用的函数

### 垃圾回收机制

概念：垃圾回收，英文简写GC（Garbage Collection），垃圾回收的过程是在遍历堆（Heap）中的对象，找到没有被引用的对象，然后释放它们占用的内存空间。  
垃圾回收有多种算法，例如：引用计数，标记清除，标记整理，复制集合等。   
注意：GC只负责回收堆中的对象，不会回收栈中的对象。引用类型都是存在堆中的，所以他的分配和释放都通过垃圾回收机制来管理。栈上的内存是由系统自动管理的，值类型在栈中分配内存的，他们有自己的生命周期，不用对他们进行管理，会自动分配和释放。  
  

C#中内存回收机制的大概原理  
0代内存  1代内存  2代内存  
代的概念：代是垃圾回收机制使用的一种算法（分代算法），新分配的对象会被配置在第0代内存中，每次分配都可能会进行垃圾回收以释放内存（0代内存满时）。  
在一次内存回收过程开始时，垃圾回收器会认为堆中全是垃圾，会进行以下两步：  
1.标记对象 从根（静态字段，方法参数）开始检查引用对象，标记后为可达对象，未标记后为不可达对象  
不可达对象就认为是垃圾  
2.搬迁对象压缩堆  （挂起执行托管代码线程）  释放未标记的对象  搬迁可达对象  修改引用地址（目的是让搬迁后的对象在堆中的地址连续）  
  
大对象总会被认为是第二代内存  目的是减少性能损耗，提高性能  
不会对大对象进行搬迁压缩  850000字节（约83kb）以上的对象为大对象

## 成员属性

概念：  
1.用于保护成员变量  
2.为成员属性的获取和赋值添加逻辑处理  
3.解决3P的局限性，属性可以让成员变量在外部只能获取不能修改或只能修改不能获取。  
  
``` csharp  
public class Person
{
    private string name;
    private int age;
    public string Name
    {
        get
        {
            // 为name添加逻辑处理
            return name;
        }
        set
        {
            // 为name添加逻辑处理
            name = value;
        }
    }
}
```  
当然成员属性中也有一种自动属性，自动属性是指在属性的定义中，没有提供get和set方法，而是直接使用属性名来访问和赋值。例如：

``` csharp  
public class Person
{
    public string Name { get; set; }
    public int Age { get; set; }
}
```  

## 索引器  
概念：让对象可以像数组一样通过索引访问其中元素，使程序看起来更直观更容易编写。  
索引器的申明：
``` csharp  
public class Person
{
    private int[] scores ;
    public int this[int index]
    {
        get
        {
            //依旧是与成员属性一样可以在索引器中添加逻辑处理，例如：
            if(scores == null || scores.Length <= index || index < 0)
            {
                return 0;
            }

            return scores[index];
        }
        set
        {
            if(scores == null)
            {
                scores = new int []{value};;
            }
            else if(index >= scores.Length)
            {
                Array.Resize(ref scores, index + 1);
                scores[index] = value;
            }
            scores[index] = value;
        }
    }
}
```   

索引器的使用：   
``` csharp  
Person p = new Person();
p[0] = 100;
p[1] = 200;
p[2] = 300;
int score = p[0];
Console.WriteLine(score);
```  

索引器的重载：
``` csharp  
public class Person
{
    private int[] scores ;
    private int[,] scores2;
    public int this[int index]
    {
        get;
        set;
    }
    public int this[int index1, int index2]
    {
        get;
        set;
    }
}
```








