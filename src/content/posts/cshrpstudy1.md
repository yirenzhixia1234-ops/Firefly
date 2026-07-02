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
概念：在实例化对象时，自动调用的特殊方法，用来初始化对象的属性。（如果不写构造函数，默认会有一个无参数的构造函数，构造函数一般是public的）  
==注意==：如果不自己实现无参构造函数而自己实现了有参构造函数，那么默认的无参构造函数就会被隐藏。

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

## 静态成员 

概念：静态成员是指在类的定义中，没有被实例化，而是直接通过类名来访问的成员。用static关键字来修饰。  
``` csharp  
public class Person
{
    // 静态成员
    public static int Count = 0;
    //成员变量
    private string name;
}
```  

静态成员被分配在静态内存中，静态内存是程序启动时分配的，程序结束时释放。

const和static的区别：  
1.const和static都是静态成员，但是const是常量，static是变量。  
2.const只能在编译时确定，static可以在运行时确定。  
3.const只能在类中使用，static可以在类中和方法中使用。

## 静态类和静态构造函数 

### 静态类  

概念：静态类是指在类的定义中，没有被实例化，而是直接通过类名来访问的类。用static关键字来修饰。  
``` csharp  
public static class Math
{
    public static int Add(int a, int b)
    {
        return a + b;
    }
}
```  
静态类只能包含静态成员，静态类不能被实例化  

作用：  
1.将常用的静态成员写在静态类中方便使用
2.静态类不能被实例化，更能体现工具类的唯一性    

    
### 静态构造函数  

概念：静态构造函数是指在类的定义中，没有被实例化，而是直接通过类名来访问的构造函数。用static关键字来修饰。  

特点：  
1.静态类和普通类都可以有    
2.不能使用访问修饰符  
3.不能有参数    
4.只会自动调用一次  

作用：在静态类中初始化==静态成员==，或者在静态类中执行一些静态操作。  

静态类中的静态构造函数：  
``` csharp  
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

普通类中的静态构造函数：  
``` csharp  
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

## 拓展方法 
概念：为现有非静态变量类型添加新方法    

作用：  
1.提升程序拓展性
2.不需要再在对象中重新写方法    
3.不需要继承来添加方法  
4.为别人封装的类型写额外的方法  

特点：  
1.一定是写在静态类中
2.一定是静态函数    
3.第一个参数为拓展目标  
4.第一个参数用this修饰  

基本语法：  
``` csharp  
//访问修饰符  static  返回值  函数名(this  拓展类名  参数名，参数类型  参数名，参数类型  参数名)
```  

例如：  
``` csharp  
public static class Tool
{
    //相当于给Person类添加了一个方法GetAge
    public static int GetAge(this Person p)
    {
        return p.Age;
    }
}
``` 

## 运算符重载   

概念：让自定义类和结构体能够使用运算符  

关键字：operator  

特点：  
1.一定是一个公共的静态方法  
2.返回值写在operator之前    
3.逻辑处理自定义    

作用：让自定义类和结构体对象可以进行运算    

注意：  
1.条件运算需要成对实现  
2.一个符号可以多个重载  
3.不能用ref和out    

基本语法：  
``` csharp  
//public static 返回类型 operator 运算符(参数类型  参数名，参数类型  参数名)
```  

实例：
``` csharp  
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

## 内部类和分部类   

### 内部类  

概念：内部类是指在类的定义中，嵌套了一个类。用class关键字来修饰。  

特点：  
1.内部类只能在外部类中实例化  
2.内部类可以访问外部类的成员  
3.内部类可以作为外部类的嵌套类型  

==注意== 访问修饰符作用很大，不能省略。 

实例：  
``` csharp  
public class Person
{
    public int Age;
    public class Inner
    {
        public int Count = 0;
    }
}
```  

### 分部类  

概念：分部类是指在类的定义中，将类的定义分成多个文件。用partial关键字来修饰。  

特点：  
1.分部类可以包含多个文件  
2.每个文件只能包含一个类  
3.每个文件的类名必须相同  
4.分部类中不能有相同的成员  

实例：  
``` csharp  
//在Person.cs文件中定义
public partial class Person
{
    public int Age;
}
```

``` csharp  
//在Person2.cs文件中定义
public partial class Person
{
    public int Count;
}
``` 

## 继承 
概念：一个类A可以继承另一个类B，B被称为基类，A被称为派生类。类A可以访问基类的成员。类A将拥有类B的所有特征和行为。类A可以有自己的特征和行为。    

特点：  
1.单根性 子类只能有一个父类 
2.传递性 子类可以简介继承父类的父类 

语法：  
``` csharp  
//派生类
public class Child : Parent
{
    // 子类的成员
    public int ChildAge;
}
```  

## 里氏替换原则       

概念：任何父类都可以被子类替换，而程序的正确性不会受到影响。父类容器可以存储子类对象，而不会出现错误。    

作用：方便进行对象的存储和管理。    

实例：
``` csharp  
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

## 继承中的构造函数 
特点：
1.当申明一个子类对象时，先执行父类的构造函数，再执行子类的构造函数。
==注意== ：
1.父类的无参构造很重要，因为子类实例化时默认是调用父类的无参构造函数。
实例：
```csharp
public class Person
{
    public int Age;
    public Person(int age)
    {
        Age = age;
    }
}

//若此时父类的无参构造函数被顶替掉时子类在实例化时会报错
public class Student:Person //此时会报错
{

}
``` 

2.子类可以通过base关键字代表父类 调用父类的构造函数。   
实例：
```csharp
public class Student:Person
{
    public Student(int age) : base(age) //这样就不会报错了
    {
    }
}
```     

## 万物之父和装箱拆箱

### 万物之父

概念：object类是所有类的基类，所有的类都是object类的子类。object类的所有方法都可以被其他类调用。
作用：  
1.可以利用里氏替换原则，用object类来存储所有类型的对象  
2.可以用来表示不确定类型，作为函数参数类型  

实例：
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

### 装箱拆箱    

发生条件：用object存==值类型==时（装箱） 再把object转换成==值类型==时（拆箱）   

装箱：把值类型用引用类型来存储，栈内存会迁移到堆内存中  

拆箱：把引用类型存储的值类型取出来，堆内存会迁移到栈内存中  

装箱和拆箱存在内存迁移，增加性能损耗    

## 密封类

概念：密封类不能被继承，只能被实例化。密封类的成员只能在密封类中被访问，不能在子类中被访问。    

关键字：sealed   

实例：
```csharp
//在加上sealed关键字后，Person类就不能被继承了
sealed public class Person
{

}
```













