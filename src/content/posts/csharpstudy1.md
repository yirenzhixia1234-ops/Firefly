---
title: C#学习笔记1
published: 2026-07-01
pinned: false
description: C#学习笔记1 - 面向对象编程、类与对象、继承、多态、接口等核心概念
tags: [C#,学习]
author: boluobao
draft: false
---

# 🎯 C#学习笔记1

本笔记涵盖 C# 面向对象编程的核心知识，从基础的 SOLID 设计原则到高级的接口、多态等概念，帮你系统地掌握 C# 的面向对象编程思想。

---

## 📐 面向对象编程

用中文去形容一类对象，把一类对象的共同点提取出来然后用程序语言把它翻译过来，带着对象的概念在程序中使用它们。

> [!IMPORTANT] 七大设计原则概览
> 面向对象设计的七大原则是编写可维护、可扩展代码的基石。下面逐一展开说明。

### 1. 开闭原则（OCP）

> 对扩展开放，对修改关闭。

开闭原则是说我们应该努力设计不需要修改的模块。在实际应用中：

- 将变化的代码和不需要变化的代码进行隔离
- 将变化的代码抽象成稳定接口，针对接口进行编程
- 在扩展系统的行为时，只需要添加新的代码，而不需要修改已有的代码
- 一般可以通过添加新的子类和重写父类的方法来实现

> [!TIP] 核心价值
> 开闭原则是面向对象设计的**核心**，满足该原则可以达到最大限度的复用性和可维护性。

---

### 2. 单一职责原则（SRP）

单一原则表明，如果你有多个原因去改变一个类，那么应该把这些引起变化的原因分离开，把这个类分成多个类，每个类只负责处理一种改变。当你做出某种改变时，只需要修改负责处理该改变的类。

> [!WARNING] 注意
> 当我们去改变一个具有多个职责的类时，可能会影响该类的其他功能。

单一职责原则代表了设计应用程序时一种很好的识别类的方式，并且它提醒你思考一个类的所有演化方式。只有对应用程序的工作方式有了很好的理解，才能很好地分离职责。

---

### 3. 接口隔离原则（ISP）

接口隔离原则表明客户端不应该被强迫实现一些他们不会使用的接口，应该把肥胖接口中的方法分组，然后用多个接口替代它，每个接口服务于一个子模块。

> [!TIP] 实践建议
> 如果已经设计成了胖接口，可以使用**适配器模式**隔离它。不过过度使用会产生大量包含单一方法的接口，所以需要根据经验，识别出那些将来需要扩展的代码来使用它。

---

### 4. 里氏替换原则（LSP）

里氏替换原则是对开闭原则的扩展，它表明我们在创建基类的新的子类时，不应该改变基类的行为。

> 当我们设计程序模块时，我们会创建一些类层次结构，然后通过扩展一些类来创建它们的子类。我们必须确保子类只是扩展而没有替换父类的功能，否则当我们在已有程序模块中使用它们时将会产生不可预料的结果。

里氏替换原则表明：==当一个程序模块使用基类时，基类的引用可以被子类替换而不影响模块的功能。==

---

### 5. 依赖倒转原则（DIP）

> 上层模块不应该依赖于底层模块，它们都应该依赖于抽象。
> 抽象不应该依赖于细节，细节应该依赖于抽象。

应用该原则意味着上层类不直接使用底层类，他们使用接口作为抽象层。这种情况下上层类中创建底层类的对象的代码不能直接使用 `new` 操作符。

可以使用一些创建型设计模式，例如：
- 工厂方法（Factory Method）
- 抽象工厂（Abstract Factory）
- 原型模式（Prototype）
- 模版设计模式（Template Method）

> [!WARNING] 不要滥用
> 如果我们有一个类的功能很有可能在将来不会改变，那么我们就不需要使用该原则。

---

### 6. 迪米特法则（LoD）

> 迪米特法则（Law of Demeter）又叫**最少知识原则**（Least Knowledge Principle LKP），就是说一个对象应当对其他对象有尽可能少的了解。

迪米特法则的目的在于降低类之间的耦合。由于每个类尽量减少对其他类的依赖，因此，很容易使得系统的功能模块相互独立，相互之间不存在依赖关系。

> [!NOTE] 设计模式中的应用
> 门面模式（Facade）和中介模式（Mediator）都是迪米特法则的应用例子。

**狭义的迪米特法则的缺点：**

- 在系统里面造出大量的小方法，这些方法仅仅是传递间接的调用，与系统的业务逻辑无关
- 遵循类之间的迪米特法则会使一个系统的局部设计简化，因为每一个局部都不会和远距离的对象有直接的关联。但也会造成系统的不同模块之间的通信效率降低

**广义的迪米特法则在类的设计上的体现：**

- 优先考虑将一个类设置成不变类
- 尽量降低一个类的访问权限
- 尽量降低成员的访问权限

---

### 7. 组合/聚合复用原则（CARP）

| 关系 | 说明 |
|------|------|
| **聚合（Aggregation）** | 整体与部分的关系，表示"含有"，部分可以脱离整体作为一个独立的个体存在 |
| **组合（Composition）** | 更强的聚合，部分组成整体且不可分割，部分不能脱离整体而单独存在。整体完全支配其组成部分，包括创建和销毁 |

组合/聚合和继承是实现复用的两个基本途径。==合成复用原则是指尽量使用组合/聚合，而不是使用继承。==

> [!IMPORTANT] 何时才使用继承？
> 只有当以下条件**全部**被满足时，才应当使用继承关系：
> 1. 子类是超类的一个特殊种类，而不是超类的一个角色 — 区分 "Is-A" 和 "Has-A"
> 2. 永远不会出现需要将子类换成另外一个类的子类的情况
> 3. 子类具有扩展超类的责任，而不是置换掉或注销掉超类的责任

---

### 🤔 为什么要学习面向对象编程？

| 优势 | 说明 |
|------|------|
| 🔄 提高代码复用率 | 减少重复代码 |
| ⚡ 提高开发效率 | 更快地构建应用 |
| 🔧 提高程序可拓展性 | 轻松应对需求变更 |
| 🧠 清晰的逻辑关系 | 代码更易于理解和维护 |

---

## 📦 类和对象

> [!IMPORTANT] 核心理解
> 1. 类的声明和类对象的声明是两个概念
> 2. 类的声明是声明对象的模板，用来抽象（形容）现实事物的
> 3. 类对象的声明是用来表现现实中的对象个体的
>
> ==类是一个自定义的变量类型，是引用类型的变量。实例化一个类对象就是在声明变量。==

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

> [!CAUTION] 常见错误
> 若在类中声明了一个与类同名的变量，不能在类的声明中实例化，否则会报错。
>
> ```csharp
> public class Person
> {
>     public Person p = new Person(); // ❌ 这样是错的!!!
> }
> ```

---

## ⚙️ 成员方法

> [!NOTE] 三个要点
> 1. 成员方法不要加 `static` 关键字
> 2. 成员方法必须实例化出对象，再通过对象来调用，相当于该对象执行了某个行为
> 3. 成员方法受到访问修饰符的影响

---

## 🏗️ 构造函数 · 析构函数 · 垃圾回收

### 构造函数

> 在实例化对象时，自动调用的特殊方法，用来初始化对象的属性。
> 如果不写构造函数，默认会有一个无参数的构造函数，构造函数一般是 `public` 的。

> [!WARNING] 重要
> 如果不自己实现无参构造函数而自己实现了有参构造函数，那么默认的无参构造函数就会被隐藏。

实例：

```csharp
// 定义一个类
public class Person
{
    public string Name { get; set; }
    public int Age { get; set; }
    // 无参构造函数
    public Person()
    {
        Name = "默认姓名";
        Age = 0;
    }
    // 有参构造函数
    public Person(string name)
    {
        Name = name;
    }
    //通过this关键字调用其他构造函数
    public Person(string name, int age) : this(name)
    {
        Age = age;
    }
}
```

---

### 析构函数

> 在对象被销毁时，自动调用的特殊方法，用来释放对象占用的资源。

对于需要手动管理内存的语言（如C++），需要在析构函数中做一些内存回收处理。但是在C#中存在自动垃圾回收机制GC，所以我们几乎不会怎么使用析构函数，除非你想在某一个对象被垃圾回收时做一些特殊处理。

> [!NOTE] 时机
> 析构函数是在垃圾**真正被回收时**才会调用的函数。

---

### 垃圾回收机制

> 垃圾回收，英文简写 GC（Garbage Collection）。垃圾回收的过程是在遍历堆（Heap）中的对象，找到没有被引用的对象，然后释放它们占用的内存空间。

垃圾回收有多种算法，例如：==引用计数、标记清除、标记整理、复制集合==等。

> [!IMPORTANT] GC 的重要限制
> GC **只负责回收堆中的对象**，不会回收栈中的对象。
> - 引用类型都是存在堆中的，所以它的分配和释放都通过垃圾回收机制来管理
> - 栈上的内存是由系统自动管理的，值类型在栈中分配内存，它们有自己的生命周期，会自动分配和释放

#### C# 中内存回收机制的原理

| 内存代 | 说明 |
|--------|------|
| **0代内存** | 新分配的对象配置在第0代内存中，每次分配都可能触发GC（0代内存满时） |
| **1代内存** | 从0代存活下来的对象晋升到1代 |
| **2代内存** | 从1代存活下来的对象晋升到2代 |

> 代的概念是垃圾回收机制使用的一种**分代算法**。

在一次内存回收过程开始时，垃圾回收器会认为堆中全是垃圾，会进行以下两步：

1. **标记对象** — 从根（静态字段、方法参数）开始检查引用对象，标记后为可达对象，未标记后为不可达对象。不可达对象就认为是垃圾
2. **搬迁对象压缩堆** — 挂起执行托管代码线程 → 释放未标记的对象 → 搬迁可达对象 → 修改引用地址（目的是让搬迁后的对象在堆中的地址连续）

> [!TIP] 关于大对象
> - 大对象总会被认为是第2代内存，目的是减少性能损耗，提高性能
> - 不会对大对象进行搬迁压缩
> - ==85000字节（约83KB）以上的对象为大对象==

---

## 🔒 成员属性

> 三大作用：
> 1. 用于保护成员变量
> 2. 为成员属性的获取和赋值添加逻辑处理
> 3. 解决3P的局限性，属性可以让成员变量在外部**只能获取不能修改**或**只能修改不能获取**

```csharp
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

> [!TIP] 自动属性
> 自动属性是指在属性的定义中，不提供具体的 get/set 实现，而是直接使用属性名来访问和赋值：
>
> ```csharp
> public class Person
> {
>     public string Name { get; set; }
>     public int Age { get; set; }
> }
> ```

---

## 📇 索引器

> 让对象可以像数组一样通过索引访问其中元素，使程序看起来更直观、更容易编写。

### 索引器的声明

```csharp
public class Person
{
    private int[] scores ;
    public int this[int index]
    {
        get
        {
            // 依旧是与成员属性一样可以在索引器中添加逻辑处理，例如：
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

### 索引器的使用

```csharp
Person p = new Person();
p[0] = 100;
p[1] = 200;
p[2] = 300;
int score = p[0];
Console.WriteLine(score);
```

### 索引器的重载

```csharp
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

---

## 📌 静态成员

> 静态成员是指在类的定义中，没有被实例化，而是直接通过类名来访问的成员。用 `static` 关键字来修饰。

```csharp
public class Person
{
    // 静态成员
    public static int Count = 0;
    //成员变量
    private string name;
    //静态函数
    public static void SetCount(int count) //注意在静态函数中只能访问静态成员（方法） 或者作为参数传进来的变量
    {
        Count = count;
    }
}
```

==静态成员被分配在静态内存中，静态内存是程序启动时分配的，程序结束时释放。==

### const 和 static 的区别

| 特性 | `const` | `static` |
|------|---------|----------|
| 本质 | 常量 | 变量 |
| 确定时机 | 只能在编译时确定 | 可以在运行时确定 |
| 使用范围 | 只能在类中使用 | 可以在类中和方法中使用 |

---

## 🧱 静态类和静态构造函数

### 静态类

> 静态类是指在类的定义中，用 `static` 关键字来修饰，不能被实例化，只能直接通过类名来访问。

```csharp
public static class Math
{
    public static int Add(int a, int b)
    {
        return a + b;
    }
}
```

> [!IMPORTANT] 静态类的特点
> - 静态类只能包含静态成员
> - 静态类不能被实例化
> - 作用：将常用的静态成员写在静态类中方便使用，更能体现工具类的唯一性

---

### 静态构造函数

> 静态构造函数是指在类的定义中，用 `static` 关键字来修饰的构造函数。

特点：

| 特点 | 说明 |
|------|------|
| 适用范围 | 静态类和普通类都可以有 |
| 访问修饰符 | 不能使用 |
| 参数 | 不能有参数 |
| 调用次数 | 只会自动调用一次 |

作用：在静态类中初始化 ==静态成员==，或者在静态类中执行一些静态操作。

**静态类中的静态构造函数：**

```csharp
public static class Math
{
    public static int Count ;
    // 静态构造函数
    static Math()
    {
        // 初始化静态成员
        Count = 0;
    }

    public static int Add(int a, int b)
    {
        return a + b;
    }
}
```

**普通类中的静态构造函数：**

```csharp
public class Person
{
    // 静态构造函数
    static Person()
    {
        // 初始化静态成员
        Count = 0;
    }
    public static int Count = 0;
}
```

---

## 🔌 拓展方法

> 为现有非静态变量类型添加新方法。

作用：
1. 提升程序拓展性
2. 不需要再在对象中重新写方法
3. 不需要继承来添加方法
4. 为别人封装的类型写额外的方法

特点：
1. 一定是写在静态类中
2. 一定是静态函数
3. 第一个参数为拓展目标
4. 第一个参数用 `this` 修饰

**基本语法：**

```csharp
//访问修饰符  static  返回值  函数名(this  拓展类名  参数名，参数类型  参数名，参数类型  参数名)
```

**示例：**

```csharp
public static class Tool
{
    //相当于给Person类添加了一个方法GetAge
    public static int GetAge(this Person p)
    {
        return p.Age;
    }
}
```

---

## ➕ 运算符重载

> 让自定义类和结构体能够使用运算符。

| 要点 | 说明 |
|------|------|
| 关键字 | `operator` |
| 方法类型 | 一定是一个公共的静态方法 |
| 返回值 | 写在 `operator` 之前 |
| 逻辑 | 自定义处理 |

> [!WARNING] 注意事项
> 1. 条件运算需要成对实现
> 2. 一个符号可以多个重载
> 3. 不能用 `ref` 和 `out`

**基本语法：**

```csharp
//public static 返回类型 operator 运算符(参数类型  参数名，参数类型  参数名)
```

**实例：**

```csharp
public class Point
{
    public int X;
    public int Y;
    public Point(int x, int y)
    {
        X = x;
        Y = y;
    }

    public static Point operator +(Point p1, Point p2)
    {
        return new Point(p1.X + p2.X, p1.Y + p2.Y);
    }

    //重载
    public static Point operator +(Point p1, int value)
    {
        return new Point(p1.X + value, p1.Y + value);
    }
}
```

---

## 🪆 内部类和分部类

### 内部类

> 内部类是指在类的定义中，嵌套了一个类。用 `class` 关键字来修饰。

特点：
1. 内部类只能在外部类中实例化
2. 内部类可以访问外部类的成员
3. 内部类可以作为外部类的嵌套类型

> [!CAUTION] 注意
> 访问修饰符作用很大，不能省略。

```csharp
public class Person
{
    public int Age;
    public class Inner
    {
        public int Count = 0;
    }
}
```

---

### 分部类

> 分部类是指在类的定义中，将类的定义分成多个文件。用 `partial` 关键字来修饰。

特点：
1. 分部类可以包含多个文件
2. 每个文件只能包含一个类
3. 每个文件的类名必须相同
4. 分部类中不能有相同的成员

**Person.cs：**

```csharp
public partial class Person
{
    public int Age;
}
```

**Person2.cs：**

```csharp
public partial class Person
{
    public int Count;
}
```

---

## 🧬 继承

> 一个类A可以继承另一个类B，B被称为基类，A被称为派生类。类A可以访问基类的成员。类A将拥有类B的所有特征和行为，同时可以有自己的特征和行为。

| 特点 | 说明 |
|------|------|
| **单根性** | 子类只能有一个父类 |
| **传递性** | 子类可以间接继承父类的父类 |

**语法：**

```csharp
//派生类
public class Child : Parent
{
    // 子类的成员
    public int ChildAge;
}
```

---

## 🔄 里氏替换原则

> 任何父类都可以被子类替换，而程序的正确性不会受到影响。父类容器可以存储子类对象，而不会出现错误。

作用：方便进行对象的存储和管理。

```csharp
public class GameObject
{

}

public class Preson:GameObject
{
    public void Move();
}

public class Monster:GameObject
{
    public void Attack();
}

public class Program
{
    public static void Main()
    {
        //父类容器可以存储子类对象的表现
        GameObject person = new Preson();
        GameObject monster = new Monster();
        GameObject[] objects = new GameObject[] {new Preson(), new Monster()};

        //通过is运算符判断对象是否是子类对象
        if(person is Preson)
        {
            //如果是子类对象，就可以调用子类的方法
            //as：将一个对象转换成指定类对象 如果成功则返回指定类对象，失败则返回null
            (person as Preson).Move();
        }

        if(monster is Monster)
        {
            (monster as Monster).Attack();
        }

        //当父类容器中存储多个子类对象时，需要根据对象的类型来调用不同的方法，常见于背包管理中
        if(objects[0] is Preson)
        {
            (objects[0] as Preson).Move();
        }
        else if(objects[0] is Monster)
        {
            (objects[0] as Monster).Attack();
        }
    }
}
```

---

## 🧱 继承中的构造函数

特点：
1. 当声明一个子类对象时，先执行父类的构造函数，再执行子类的构造函数。

> [!WARNING] 关键注意
> 1. 父类的无参构造很重要，因为子类实例化时默认是调用父类的无参构造函数。
> 2. 子类可以通过 `base` 关键字代表父类，调用父类的构造函数。

```csharp
public class Person
{
    public int Age;
    public Person(int age)
    {
        Age = age;
    }
}

// 若此时父类的无参构造函数被顶替掉时，子类在实例化时会报错
public class Student:Person // ❌ 此时会报错
{

}
```

**解决方案 — 使用 `base` 关键字：**

```csharp
public class Student:Person
{
    public Student(int age) : base(age) // ✅ 这样就不会报错了
    {
    }
}
```

---

## 📦 万物之父和装箱拆箱

### 万物之父（object）

> `object` 类是所有类的基类，所有的类都是 `object` 类的子类。`object` 类的所有方法都可以被其他类调用。

作用：
1. 可以利用里氏替换原则，用 `object` 类来存储所有类型的对象
2. 可以用来表示不确定类型，作为函数参数类型

```csharp
public class Program
{
    public static void Main()
    {
        //object类可以存储所有类型的对象
        object obj = new int();
        obj = new string();
        obj = new bool();

        //引用类型转换
        object str = "hello";
        string str2 = str as string;

        //值类型转换（强转）
        int num = 10;
        object numObj = num;
        int num2 = (int)numObj;
    }
}
```

---

### 装箱拆箱

> [!IMPORTANT] 核心概念
> 发生条件：用 `object` 存 ==值类型== 时（装箱），再把 `object` 转换成 ==值类型== 时（拆箱）。

| 操作 | 说明 |
|------|------|
| **装箱（Boxing）** | 把值类型用引用类型来存储，栈内存会迁移到堆内存中 |
| **拆箱（Unboxing）** | 把引用类型存储的值类型取出来，堆内存会迁移到栈内存中 |

> [!WARNING] 性能提示
> 装箱和拆箱存在==内存迁移==，增加性能损耗。应尽量避免频繁的装箱拆箱操作。

---

## 🔐 密封类

> 密封类不能被继承，只能被实例化。密封类的成员只能在密封类中被访问，不能在子类中被访问。

关键字：`sealed`

```csharp
//在加上sealed关键字后，Person类就不能被继承了
sealed public class Person
{

}
```

---

## 🎭 多态（VOB）

> 让继承同一父类的子类们，在执行相同方法时有不同的表现（状态）。

| 目的 | 解决的问题 |
|------|------------|
| 同一父类对象，执行相同行为（方法）有不同的表现 | 让同一个对象有唯一行为的特征 |

### 虚函数

> 在父类中定义的方法，子类可以重写。子类重写的方法，就叫虚函数。

```csharp
public class GameObject
{
    public virtual void Move()
    {
        Console.WriteLine("移动");
    }
}

public class Preson:GameObject
{
    //子类重写父类的方法，就叫重写虚函数
    //重写虚函数时，需要在方法前添加override关键字
    public override void Move()
    {
        //base的作用：代表父类，通过base来保留父类的行为
        base.Move();
        Console.WriteLine("人移动");
    }
}
```

---

## 🎨 抽象类和抽象函数

> 被 `abstract` 关键字修饰的类，就叫抽象类。抽象类不能被实例化，只能被继承。

```csharp
public abstract class GameObject
{
    //抽象方法不能有方法体，只能在子类中实现
    public abstract void Move();
}

public class Preson:GameObject
{
    public override void Move()
    {
        Console.WriteLine("人移动");
    }
}
```

| 对比 | 虚方法（virtual） | 抽象方法（abstract） |
|------|-------------------|----------------------|
| 实现方式 | 子类选择性实现 | 子类**必须**实现 |
| 方法体 | 父类中可以有默认实现 | 父类中不能有方法体 |

> [!TIP] 选择指南
> - 不希望被实例化的类，相对比较抽象的类可以使用抽象类
> - 父类中的行为不太需要被实现的，只希望子类去定义具体的规则的，可以选择抽象类然后使用其中的抽象方法

---

## 🔌 接口

> 被 `interface` 关键字修饰的类，就叫接口。接口不能被实例化，只能被实现。

### 接口声明的规范

1. 不包含成员变量
2. 只包含方法、属性、索引器、事件
3. 成员不能被实现
4. 成员可以不用写访问修饰符，不能是私有的
5. 接口不能继承类，但是可以继承另一个接口

### 接口的使用规范

1. 类可以继承多个接口
2. 类继承接口后，必须实现接口中的所有成员

### 接口的特点

1. 它和类的声明类似
2. 接口是拿来继承的
3. 接口不能被实例化，但是可以作为容器存储对象

**接口的声明：**

```csharp
public interface IGameObject
{
    void Move();

    string Name { get; set; }

    int this[int index] { get; set; }

    public event Action OnMove;
}
```

**接口的使用：**

1. 类只能继承一个类，但是可以实现多个接口
2. 类实现接口后，必须实现接口中的所有成员，并且必须是 `public` 的
3. 实现的接口函数可以加 `virtual` 关键字在子类重写
4. 接口也遵循里氏替换原则

```csharp
public class GameObject:IGameObject
{
    public void Move()
    {
        Console.WriteLine("移动");
    }
    public string Name { get; set; } = "游戏对象";
    public int this[int index] { get; set; } = 0;
    public event Action OnMove;
    public void OnMove()
    {
        OnMove?.Invoke();
    }
}
```

> [!NOTE]
> 接口可以继承接口。接口继承接口时不需要实现，待类继承接口后自己去实现所有内容。

### 显示实现接口

当一个类实现了多个接口，且接口中包含相同的方法时，需要在类中显示实现接口的方法。

```csharp
interface IAtk
{
    void Attack();
}

interface ISuperAtk
{
    void Attack();
}

class Person:IAtk,ISuperAtk
{
    //显示实现接口就是用接口名.方法名() 去实现
    void IAtk.Attack()
    {
        Console.WriteLine("普通攻击");
    }
    void ISuperAtk.Attack()
    {
        Console.WriteLine("超级攻击");
    }
}

class Program
{
    static void Main(string[] args)
    {
        Person p = new Person();
        //调用普通攻击方法
        (p as IAtk).Attack();
        //调用超级攻击方法
        (p as ISuperAtk).Attack();
    }
}
```

---

## 🔒 密封方法

> 被 `sealed` 关键字修饰的方法，就叫密封方法。让虚方法或者抽象方法不能被子类重写。

---

## 📂 命名空间

> 命名空间是用来组织和重构代码的。

**基本语法：**

```csharp
namespace 命名空间名
{
    //命名空间中的代码
    //类、结构、枚举、委托、接口等
}
```

> [!WARNING] 注意
> 同一命名空间不允许有相同的类名。

**实例：**

```csharp
namespace MyGame
{
    public class GameObject
    {

    }
}

//相同命名空间的类可以互相使用
namespace MyGame
{
    public class Preson:GameObject
    {

    }
}

//若是不同命名空间的类，需要使用using关键字来引入命名空间
using MyGame;

namespace MyGame2
{
    class Program
    {
        static void Main(string[] args)
        {
            //添加了using关键字后，就可以直接使用MyGame命名空间中的类
            GameObject g = new GameObject();
            //或者MyGame.Peson p = new MyGame.Peson(); 也可以使用
        }
    }
}
```

> [!NOTE]
> - 不同命名空间中允许有相同的类名，但需要引用时必须指明出处。
> - 命名空间可以包裹其他命名空间。
> - `internal` 命名空间：只能在当前程序集中使用。

---

## 🔍 万物之父中的方法

```csharp
public class Object
{
    //静态方法
    public static bool Equals(object obj,object obj2); //判断两个对象是否相等
    public static bool ReferenceEquals(object obj,object obj2); //判断两个对象是否引用同一个对象   如果传入的是值对象则始终返回false

    //成员方法
    public Type GetType(); //获取对象的类型
    protected object MemberwiseClone(); //获取对象的浅拷贝对象  浅拷贝会返回一个新的对象 但是新对象中的引用变量会和原对象中的引用变量指向同一个对象

    //虚方法
    public virtual string ToString(); //返回对象的字符串表示
    public virtual bool Equals(object obj); //默认实现还是比较两者是否为同一引用  但微软在System.ValueType中重写了这个方法，用来比较值是否相等
    public virtual int GetHashCode(); //返回对象的哈希码  重写时一般返回对象的成员变量的哈希码的组合

}
```

---

## 📝 String

> [!CAUTION] 重要特性
> `string` 类型虽然是引用类型，但是当字符串被重新赋值时，会创建一个新的字符串对象，而不是修改原字符串。
>
> ```csharp
> string str = "1313333";
> str = "4444444"; //会创建一个新的字符串对象  而不是修改原字符
> //所以频繁的修改字符串时  会消耗大量的内存空间
> ```

### string 常用方法

```csharp
//字符串指定位置获取
string str = "1313333";
char c = str[0]; //获取字符串的第一个字符 是用string类中的索引器实现的

//字符串转char数组
char[] arr = str.ToCharArray();

//字符串拼接
string str2 = str + "123";
string str3 = string.Format("{0}{1}",str,"123"); //用Format方法拼接字符串  可以用{0}、{1}等占位符来表示参数的顺序

//字符串正向查找字符位置
int index = str.IndexOf("313"); //返回字符串中第一个字符的索引  如果没有则返回-1

//反向查找字符位置
int index2 = str.LastIndexOf("313"); //返回字符串中最后一个字符的索引  如果没有则返回-1

//移除指定位置后的字符
string str4 = str.Remove(index,3); //从索引为index开始移除3个字符 并不会改变原字符串

//替换指定字符串
string str5 = str.Replace("313","123"); //将字符串中的313替换为123 并不会改变原字符串

//大小写转换  并不会改变原字符串
string str6 = str.ToLower(); //将字符串转换为小写
string str7 = str.ToUpper(); //将字符串转换为大写

//字符串截取  并不会改变原字符串
string str8 = str.Substring(index,3); //从索引为index开始截取3个字符  参数一为开始索引  参数二为截取的字符数 若索引超出范围则会报错

//字符串切割  并不会改变原字符串
string[] str9 = str.Split("313"); //将字符串按313切割  返回一个字符串数组
```

### 🔄 字符串反转（双指针 + XOR 交换）

```csharp
//编写一个函数，将输入的字符串反转，不要使用中间商，你必须原地修改输入数组，交换过程中不使用额外空间
//用双指针+XOR异或交换字符
void ReverseString(char[] s)
{
    int left = 0;
    int right = s.Length - 1;
    while (left < right)
    {
        s[left] ^= s[right];
        s[right] ^= s[left];
        s[left] ^= s[right];
        left++;
        right--;
    }
}
```

---

## 🧵 StringBuilder

> 可以解决频繁修改字符串时内存空间浪费的问题。使用前需引用 `System.Text` 命名空间。

```csharp
StringBuilder sb = new StringBuilder("1313333");

//StringBuilder存在容量的问题 每次往里面增加时会自动扩容
//获得容量
int capacity = sb.Capacity;
//获得字符串长度
int length = sb.Length;

//增加字符
sb.Append("123");
sb.AppendFormat("{0}",123);

//插入
sb.Insert(index,"123");

//删除
sb.Remove(index,3);

//清空
sb.Clear();

//查
Console.WriteLine(sb[1]);

//改
sb[1] = '4';

//替换
sb.Replace("313","12313"); //会修改原字符串 不会产生垃圾

//修改
sb.Clear();
sb.Append("123");
```

---

## ⚖️ 结构体和类的区别

结构体和类的最大区别是存储空间的区别：结构体是值类型，类是引用类型。它们的存储位置一个在栈上，一个在堆上。

结构体具备面向对象思想中封装的特性，但是不具备继承和多态的特性。由于结构体不具备继承的特性，所以它不能用 `protected` 来修饰成员变量。

### 细节对比

| 特性 | 结构体（struct） | 类（class） |
|------|:--:|:--:|
| 类型 | 值类型 | 引用类型 |
| 存储位置 | 栈 | 堆 |
| `protected` 修饰符 | ❌ 不支持 | ✅ 支持 |
| 成员变量初始值 | ❌ 不能指定 | ✅ 可以 |
| 无参构造函数 | ❌ 不能声明 | ✅ 可以 |
| 有参构造后无参构造 | 不会被顶掉 | 会被顶掉 |
| 析构函数 | ❌ 不能声明 | ✅ 可以 |
| 继承 | ❌ 不能被继承 | ✅ 可以 |
| 初始化成员变量 | 构造函数中必须初始化所有 | 随意 |
| `static` 修饰 | ❌ 不存在静态结构体 | ✅ 可以 |
| 自引用 | ❌ 不能声明同类变量 | ✅ 可以 |
| 实现接口 | ✅ 可以 | ✅ 可以 |

### 如何选择？

| 场景 | 选择 |
|------|------|
| 想要用继承和多态时（玩家、怪物等） | 🟢 **类** |
| 对象是数据集合时（位置、坐标等） | 🟢 **结构体** |
| 经常赋值传递，且改变赋值对象时原对象不想跟着变化（坐标、向量、旋转等） | 🟢 **结构体** |

---

## 🆚 抽象类和接口的区别

| 特性 | 抽象类（abstract） | 接口（interface） |
|------|:--:|:--:|
| 构造函数 | ✅ 可以有 | ❌ 不能 |
| 继承数量 | 只能单一继承 | 可以被继承多个 |
| 成员变量 | ✅ 可以有 | ❌ 不能 |
| 方法类型 | 成员方法、虚方法、抽象方法、静态方法 | 只能声明没有实现的抽象方法 |
| 访问修饰符 | ✅ 可以使用 | 建议不写，默认 `public` |

### 如何选择？

> 表示对象的用**抽象类**，表示行为拓展的用**接口**。
> 不同对象拥有的共同行为，我们往往可以使用接口来实现。

| 例子 | 类型 | 原因 |
|------|------|------|
| 🐾 动物 | 抽象类 | 动物是一类对象 |
| 🕊️ 飞翔 | 接口 | 飞翔是一个行为 |
