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
## 多态vob  

概念：让继承同一父类的子类们 在执行相同方法时有不同的表现（状态） 

目的：同一父类对象 执行相同行为（方法）有不同的表现 

解决的问题：让同一个对象有唯一行为的特征    

### 虚函数

概念：在父类中定义的方法，子类可以重写。子类重写的方法，就叫虚函数。

实例：  
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

## 抽象类和抽象函数 

概念：被abstract关键字修饰的类，就叫抽象类。抽象类不能被实例化，只能被继承。

实例：
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

虚方法是可以由子类选择性实现的      
抽象方法必须要实现  
不希望被实例化的类，相对比较抽象的类可以使用抽象类  
父类中的行为不太需要被实现的，只希望子类去定义具体的规则的 可以选择抽象类然后使用其中的抽象方法

## 接口

概念：被interface关键字修饰的类，就叫接口。接口不能被实例化，只能被实现。

接口申明的规范：
1.不包含成员变量
2.只包含方法、属性、索引器、事件    
3.成员不能被实现  
4.成员可以不用写访问修饰符，不能是私有的    
5.接口不能继承类，但是可以继承另一个接口    

接口的使用规范：
1.类可以继承多个接口  
2.类继承接口后，必须实现接口中的所有成员    

特点：
1.他和类的申明类似      
2.接口是拿来继承的  
3.接口不能被实例化，但是可以作为容器存储对象

接口的申明：
```csharp   
public interface IGameObject
{
    void Move();

    string Name { get; set; }

    int this[int index] { get; set; }

    public event Action OnMove;
}
``` 

接口的使用：    
1.类只能继承一个类，但是可以实现多个接口    
2.类实现接口后，必须实现接口中的所有成员    并且必须是public的  
3.实现的接口函数可以加virtual关键字在子类重写   
4.接口也遵循里氏替换原则    


实例：
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

接口可以继承接口  接口继承接口时不需要实现，待类继承接口后自己去实现所有内容    

显示实现接口：
当一个类实现了多个接口，且接口中包含相同的方法时，需要在类中显示实现接口的方法。

实例：
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

## 密封方法 

概念：被sealed关键字修饰的方法，就叫密封方法。让虚方法或者抽象方法不能被子类重写。

## 命名空间 

概念：命名空间是用来组织和重构代码的。  

命名空间的使用：
基本语法：
```csharp
namespace 命名空间名
{
    //命名空间中的代码
    //类、结构、枚举、委托、接口等
}
```     
同一命名空间不允许有相同的类名。

实例：
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

不同命名空间中允许有相同的类名。但需要引用必须指明出处。    

命名空间可以包裹其他命名空间。

internal命名空间：只能在当前程序集中使用。

## 万物之父中的方法 

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

## string  

==注意== string类型虽然是引用类型 但是当字符串被重新赋值时  会创建一个新的字符串对象  而不是修改原字符串    
例如：
```csharp
string str = "1313333";
str = "4444444"; //会创建一个新的字符串对象  而不是修改原字符
//所以频繁的修改字符串时  会消耗大量的内存空间
``` 

string类的方法：

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

## StringBuilder

可以解决频繁修改字符串时  内存空间浪费的问题  使用前需引用System.Text命名空间

初始化：
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

## 结构体和类的区别

结构体和类的最大区别是存储空间的区别，结构体是值类型，类是引用类型。他们的存储位置一个在栈上一个在堆上  

结构体和类在使用上很相似，结构体甚至可以用面向对象的思想来形容一类对象。结构体具备面向对象思想中封装的特性，但是他不具备继承和多态的特性，因此大大减少了他的使用频率。
由于结构体不具备继承的特性，所以它不能用protected来修饰成员变量 

细节区别：
1.结构体是值类型，类是引用类型  
2.结构体存在栈中，类存在堆中    
3.结构体成员不能使用protected访问修饰符，而类可以   
4.结构体成员变量申明不能指定初始值，而类可以    
5.结构体不能申明无参的构造函数，而类可以    
6.结构体申明有参构造函数后，无参构造不会被顶掉  
7.结构体不能申明析构函数，而类可以  
8.结构体不能被继承，而类可以    
9.结构体需要在构造函数中初始化所有成员变量，而类随意    
10.结构体不能被静态static修饰（不存在静态结构体），而类可以 
11.结构体不能在自己内部申明和自已一样的结构体变量，而类可以     

但结构体可以实现接口    

如何选择结构体和类：    
1.想要用继承和多态时，直接淘汰结构体，比如玩家、怪物等等
2.对象是数据集合时，优先考虑结构体，比如位置、坐标等等
3.从值类型和引用类型赋值时的区别上去考虑，比如经常被赋值传递的对象，并且改变赋值对象，原对象不想跟着变化时，就用结构体。比如坐标、向量、旋转等等

## 抽象类和接口的区别   

区别：
1.抽象类中可以有构造函数；接口中不能    
2.抽象类只能被单一继承；接口可以被继承多个  
3.抽象类中可以有成员变量；接口中不能    
4.抽象类中可以申明成员方法，虚方法，抽象方法，静态方法：接口中只能申明没有实现的抽象方法    
5.抽象类方法可以使用访问修饰符；接口中建议不写，默认public      

如何选择抽象类和接口：  
表示对象的用抽象类，表示行为拓展的用接口    
不同对象拥有的共同行为，我们往往可以使用接口来实现  
举个例子：  
动物是一类对象，我们自然会选择抽象类：而飞翔是一个行为，我们自然会选择接口。    







































