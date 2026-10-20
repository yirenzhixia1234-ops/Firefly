---
title: Unity 学习2 - Json学习
published: 2026-07-27
pinned: false
description: Unity
tags: [Unity,学习,数据持久化]
author: boluobao
draft: true
---

# Unity基础

## Mathf
Mathf 是 Unity 提供的一个数学结构体，包含了数学相关的静态方法和属性。   
Math 是 C# 提供的一个数学类，包含了数学相关的静态方法和属性。

```csharp
//PI
Mathf.PI;

//取绝对值 - Abs
Mathf.Abs(-10);

//向上取整 - CeilToInt
Mathf.CeilToInt(3.14);

//向下取整 - FloorToInt
Mathf.FloorToInt(3.14);

//钳制函数 - Clamp
Mathf.Clamp(5, 0, 10); //参数一：要钳制的值，参数二：下限，参数三：上限 如果参数一小于下限，返回下限；如果参数一大于上限，返回上限；否则返回参数一

//获取最大值 - Max
Mathf.Max(10,20); //可以传多个参数，返回最大的那个

//获取最小值 - Min
Mathf.Min(10,20); //可以传多个参数，返回最小的那个

//一个数的n次方 - Pow
Mathf.Pow(2,3); //返回2的3次方，即8

//四舍五入 - RoundToInt
Mathf.RoundToInt(3.14); //返回3

//返回一个数的平方根 - Sqrt
Mathf.Sqrt(9); //返回3

//判断一个数是否是2的整数次方 - IsPowerOfTwo
Mathf.IsPowerOfTwo(8); //返回true

//判断正负数 - Sign
Mathf.Sign(10); //返回1，因为10是正数，所以返回1表示正数
Mathf.Sign(-10); //返回-1，因为-10是负数，所以返回-1表示负数
Mathf.Sign(0); //返回0，因为0是0，所以返回0表示0

//插值运算 - Lerp
public float result = 0;
public float start = 0;
public float time = 0;

result = Mathf.Lerp(start, end, t); //返回start到end的time比例
//t为插值系数 取值范围为0~1
//result = start + (end - start) * t;

//插值运算用法一
//每帧改变start的值——变化速度先快后慢，位置无限接近，但是不会得到end的值
start = Mathf.Lerp(start, end, Time.deltaTime);

//插值运算用法二
//每帧改变t的值——变化速度匀速，位置每帧接近，当t>=1时得到结果
time += Time.deltaTime;
result = Mathf.Lerp(start, end, time);
```

### 三角函数

#### 弧度角度转换

```csharp
//弧度转换角度
public float rad = 1;
public float angle = Mathf.Rad2Deg * rad;

//角度转换弧度
public float angle = 1;
public float rad = Mathf.Deg2Rad * angle;
```

#### 三角函数使用

```csharp
//Mathf中的三角函数相关函数在使用时需要传入弧度
//例如：
Mathf.Sin(30 * Mathf.Deg2Rad);
Mathf.Cos(30 * Mathf.Deg2Rad);
```

#### 反三角函数

```csharp
//反三角函数得到的结果是 正弦或者余弦对应的弧度
rad = Mathf.ASin(0.5f);
rad = Mathf.ACos(0.5f);
```

## 向量 

### 向量的模长

```csharp
//向量的模长
Vector3 vector = new Vector3(1, 2, 3);
float magnitude = vector.magnitude;
```

### 单位向量

```csharp
//单位向量
//归一化公式：向量 / 向量的模长
Vector3 unitVector = vector.normalized; //向量归一化
```

### 向量的点乘

向量点乘的意义：    
- 向量点乘的结果是一个标量  
- 向量点乘的结果的大小表示了两个向量之间的夹角的余弦值  
- 向量点乘的结果的符号表示了两个向量之间的关系  
- 向量点乘的结果的符号为正表示两个向量指向相同方向  
- 向量点乘的结果的符号为负表示两个向量指向相反方向  
- 向量点乘的结果的符号为0表示两个向量垂直指向   
```csharp
float dot = Vector3.Dot(vector1.normalized, vector2.normalized);
```
```csharp
//判断两个物体的前后关系
public Transform A;
public Transform B;

//注意这里要用物体A的forward向量来确定A的正面
if(Vector3.Dot(A.forward,B.position - A.position) > 0)
{
    //B在A的前面
}
else
{
    //B在A的后面
}
```

利用点乘可以算出两个向量之间的夹角
```csharp
float angle = Mathf.Acos(dot) * Mathf.Rad2Deg; //两个向量之间的夹角的弧度等于两个向量的点乘的反余弦函数

//也可以用Vector3提供的公式
angle = Vector3.Angle(vector1, vector2)
```

### 向量的叉乘

```csharp
public Vector3 vector1 = new Vector3(1, 2, 3);
public Vector3 vector2 = new Vector3(4, 5, 6);
//向量叉乘
Vector3 cross = Vector3.Cross(vector1.normalized, vector2.normalized);
```
向量叉乘的几何意义：
A x B 得到的向量同时垂直于A和B，即该向量垂直于AB所在的平面      

```csharp
//判断两个物体的左右
public Transform A;
public Transform B;

//注意这里要用物体A的forward向量来确定A的正面
if(Vector3.Cross(A.forward,B.position - A.position).y > 0)
{
    //B在A的右侧
}
else
{
    //B在A的左侧
}
```

### 向量的插值

```csharp
//线性插值
//物体运动的模式与Mathf.Lerp同理 都可以是先快后慢和匀速运动
Vector3 lerp = Vector3.Lerp(vector1, vector2, Time.deltaTime);

//球形插值
Vector3 sphereLerp = Vector3.Lerp(vector1, vector2, Time.deltaTime);
```

## 四元数

使用四元数的原因：  
1.因为欧拉角同一旋转的角度不唯一 
2.使用欧拉角进行旋转时，会存在万向锁问题，即在旋转角度为90度时，两个轴会重合，导致旋转效果异常  
而四元数可以避免这个问题    

### Unity中四元数的使用

```csharp
//四元数Q = [cos(β/2)，sin(β/2)x，sin(β/2)y，sin(β/2)z]
//Quaternion(float x, float y, float z, float w) 
//w为cos(β/2)，x,y,z为sin(β/2)x，sin(β/2)y，sin(β/2)z
Quaternion quaternion = new Quaternion(Mathf.Sin(30*Mathf.Deg2Rad),0,0,Mathf.Cos(30*Mathf.Deg2Rad));//绕x轴旋转30度

//Unity中封装的方法
Quaternion q2 = Quaternion.AngleAxis(60, Vector3.right);//绕x轴旋转60度 等同于上面的公式

//欧拉角转四元数
Quaternion q3 = Quaternion.Euler(30, 0, 0);//绕x轴旋转30度

//四元数转欧拉角
Vector3 euler = q3.eulerAngles;
```

### 四元数的常用方法

```csharp
//单位四元数
Quaternion identity = Quaternion.identity; //表示(1,0,0,0) 即欧拉角(0,0,0) 表示不旋转

//四元数插值运算
//Lerp和Slerp
//在四元数中Lerp和Slerp只有细微差别 由于算法不同Slerp的效果会更好一些
//Lerp的效果相比Slerp更快但是如果旋转的角度范围较大则效果会变差 建议使用Slerp来计算

//插值运算的先快后慢和匀速运动 都与之前一致

//向量指向转换四元数
Quaternion q4 = Quaternion.LookRotation(vector1);

//LookRotation方法可以将传入的面朝向量 转换为对应的四元数角度信息
//例如：当人物的面朝向量需要改变时 只需要将目标面朝向量传入该函数便可得到目标四元数角度信息，
//之后将人物四元数角度信息改为得到的信息即可达到转向 也可以利用Slerp来平滑过渡
```

### 四元数计算
```csharp
//四元数乘四元数
q3 = q1 * q2; //两个四元数相乘得到一个新的四元数 代表两个旋转量的叠加 相当于旋转

//注意旋转相对的坐标系是物体自身的坐标系

//四元数乘向量

v2 = q1 * v1; //四元数q1乘向量v1 得到一个新的向量v2 将向量v1旋转了指定四元数的旋转量 相当于旋转向量

```

## 延迟函数

延迟函数是指在调用时会等待一段时间后再执行的函数    

### 延迟函数的使用

```csharp
//延迟函数的使用
//Invoke方法可以将一个方法延迟一段时间后再执行
//例如：
Invoke("Method", 2f); //2秒后执行Method方法

public void Method()
{
    //2秒后执行的代码
}

//注意：    
//1.延迟函数第一个参数传入的是函数名字符串
//2.延迟函数没有办法传入参数 只能通过包裹一层函数来实现
//3.函数名必须是该脚本上申明的函数

//延迟重复执行函数
//InvokeRepeating方法可以将一个方法延迟一段时间后再重复执行
//例如：
InvokeRepeating("Method", 2f, 1f); //2秒后每1秒执行一次Method方法

//取消延迟函数
//CancelInvoke方法可以取消一个延迟函数
//例如：
CancelInvoke("Method"); //取消Method方法的延迟执行
CancelInvoke(); //取消所有方法的延迟执行

//判断是否有延迟函数
//IsInvoking方法可以判断是否有延迟函数正在执行
//例如：
if(IsInvoking())
{
    //有方法的延迟执行正在执行
}
else
{
    //没有方法的延迟执行正在执行
}

//指定方法是否有延迟执行
if(IsInvoking("Method"))
{
    //有Method方法的延迟执行正在执行
}
else
{
    //没有Method方法的延迟执行正在执行
}
```

当延迟函数所在的脚本对象失活或脚本本身失活时 延迟函数还会继续执行   
只有当延迟函数所在的脚本对象被销毁时或脚本被移除时 才会停止延迟函数的执行       

## 协程

协同程序简称协程 把可能会让主线程卡顿的耗时的逻辑分时分步执行   

主要使用场景：  
1.异步加载文件  
2.异步下载文件  
3.场景异步加载  
4.批量创建时防止卡顿    

### 协程和多线程的区别

新开一个线程是独立的一个管道 和主线程并行执行   
新开一个协程是在原线程之上开启，进行逻辑分时分步执行    

### 协程的使用

继承MonoBehaviour的类都可以使用协程函数

```csharp
//协程函数返回值为IEnumerator及其子类
//函数中通过yield return 返回值进行返回

IEnumerator MyCoroutine()
{
    print("协程开始执行");
    yield return new WaitForSeconds(2f);
    print("协程执行完成");
}

//协程函数的开启

void Start()
{
    Coroutine c = StartCoroutine(MyCoroutine());
    //IEnumerator ie = MyCoroutine();
    //StartCoroutine(ie)
    //StartCoroutine("MyCoroutine"); 
    //这几种方式都可以开启协程函数

    //协程函数的关闭

    StopAllCoroutines(); //关闭所有协程函数

    StopCoroutine("MyCoroutine"); //关闭指定协程函数

    StopCoroutine(c); //关闭指定协程函数 这个是最准确的 
}

```

### yield return 的使用

1.下一帧执行    
yield return 数字;  
yield return null;  
在Update和LateUpdate之间执行    
2.等待指定秒后执行  
yield return new WaitForSeconds(秒);    
在Update和LateUpdate之间执行    
3.等待下一个固定物理帧更新时执行    
yield return new WaitForFixedupdate();  
在Fixedupdate和碰撞检测相关函数之后执行 
4.等待摄像机和GUI渲染完成后执行 
yield return new WaitForEndofFrame();   
在LateUpdate之后的渲染相关处理完毕后之后    
5.一些特殊类型的对象比如异步加载相关函数返回的对象  
之后讲解异步加载资源异步加载场景网络加载时再讲解    
一般在Update和LateUpdate之间执行    
6.跳出协程  
yield break;    

### 协程受对象和组件失活销毁的影响

协程开启后 组件和物体销毁 协程不执行 物体失活协程不执行 组件失活 协程仍然执行


### 协程的原理

#### 协程的本质

协程可以分为两部分  
1.协程函数本体  
2.协程调度器    

协程本体就是一个能够中间暂停返回的函数  
协程调度器是Unity内部实现的，会在对应的时机帮助我们继续执行协程函数 

Unity只实现了协程调度部分       
协程的本体本质上就是一个C#的迭代器方法  

#### C# 迭代器与状态机

协程能够"暂停再恢复"的核心在于 C# 的 `yield return` 语法。编译器在遇到 `yield return` 时，会将这个方法编译成一个**状态机类**。

以一个简单的协程为例：

```csharp
IEnumerator MyCoroutine()
{
    Debug.Log("第一步");
    yield return new WaitForSeconds(2f);
    Debug.Log("第二步");
    yield return null;
    Debug.Log("第三步");
}
```

编译器会将上面的代码转换为类似下面的结构（简化版）：

```csharp
// 编译器自动生成的状态机类（简化示意）
class MyCoroutineStateMachine : IEnumerator
{
    private int _state = 0;  // 当前执行到哪个状态
    
    public object Current { get; private set; }  // 当前 yield return 返回的值
    public bool MoveNext()
    {
        switch (_state)
        {
            case 0:
                Debug.Log("第一步");
                Current = new WaitForSeconds(2f);
                _state = 1;
                return true;  // 返回 true 表示还有后续代码要执行
                
            case 1:
                Debug.Log("第二步");
                Current = null;
                _state = 2;
                return true;
                
            case 2:
                Debug.Log("第三步");
                _state = -1;
                return false; // 返回 false 表示协程结束
        }
        return false;
    }
    
    public void Reset() { /* ... */ }
}
```

关键点：
- `_state` 字段记录了当前执行到的位置（状态编号）
- 每次调用 `MoveNext()` 时，通过 `switch` 跳转到对应的状态继续执行
- `Current` 属性保存了 `yield return` 返回的值（即等待条件）
- `MoveNext()` 返回 `true` 表示协程还有后续代码，返回 `false` 表示执行完毕

#### 协程调度器的运行流程

Unity 的协程调度器在每一帧中会遍历所有活跃的协程，对每个协程执行以下判断：

```
1. 获取协程的 IEnumerator（即状态机实例）
2. 检查 IEnumerator.Current 的值：
   - 如果是 null 或 数字 → 等待到下一帧（Update 之后执行 MoveNext()）
   - 如果是 WaitForSeconds → 等待指定时间后执行 MoveNext()
   - 如果是 WaitForFixedUpdate → 等待到 FixedUpdate 之后执行 MoveNext()
   - 如果是 WaitForEndOfFrame → 等待到帧末尾执行 MoveNext()
   - 如果是 AsyncOperation → 等待异步操作完成后执行 MoveNext()
3. 当条件满足时，调用 MoveNext()
   - 返回 true → 读取新的 Current，继续等待
   - 返回 false → 协程结束，从调度列表中移除
```

所以协程的完整本质可以总结为：

> **协程 = C# 迭代器状态机（提供暂停/恢复能力）+ Unity 调度器（决定何时恢复执行）**

Unity 并没有在底层实现什么特殊的线程或纤程机制——它只是利用了 C# 语言层面 `yield return` 生成的迭代器状态机，再配合自己的调度系统，在合适的时机调用 `MoveNext()`，从而实现了"分时分步执行"的效果。这也是为什么协程仍然运行在主线程上、会受到物体/组件的生命周期影响的原因。

#### 手动模拟协程

理解了协程的本质后，完全可以手动模拟一个简易的协程系统：

```csharp
// 手动驱动迭代器的示例
IEnumerator ie = MyCoroutine();

// 第一次调用 MoveNext，执行到第一个 yield return
ie.MoveNext();  // 输出 "第一步"

// ie.Current 此时是 WaitForSeconds(2f)
// 等待 2 秒后...

// 第二次调用 MoveNext，从上次暂停处继续
ie.MoveNext();  // 输出 "第二步"

// ie.Current 此时是 null

// 第三次调用
ie.MoveNext();  // 输出 "第三步"
// MoveNext() 返回 false，协程结束
```

Unity 做的事情就是把这个"手动调用"自动化了——它在每帧的适当时机，自动帮你调用 `MoveNext()`。  

#### 自己实现简易协程系统

基于以上原理，下面从零实现一个支持**等待指定秒后执行**的简易协程系统。

**1. 自定义等待秒数的 yield 指令**

```csharp
using UnityEngine;

/// <summary>
/// 简易的等待秒数指令，替代 Unity 的 WaitForSeconds
/// </summary>
public class SimpleWaitForSeconds
{
    // 需要等待的总秒数
    public float waitTime;
    // 已经等待的秒数（调度器每帧累加 Time.deltaTime）
    public float elapsedTime;

    public SimpleWaitForSeconds(float seconds)
    {
        waitTime = seconds;
        elapsedTime = 0f;
    }

    /// <summary>
    /// 每帧调用，累加时间，返回是否等待完毕
    /// </summary>
    public bool Tick(float deltaTime)
    {
        elapsedTime += deltaTime;
        return elapsedTime >= waitTime;
    }
}
```

**2. 协程调度器**

维护所有活跃协程，在 `Update` 中检查等待条件，满足时推动迭代器前进一步。

```csharp
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

/// <summary>
/// 简易协程调度器 —— 自己实现协程的核心
/// </summary>
public class SimpleCoroutineRunner : MonoBehaviour
{
    // 存储所有正在运行的协程信息
    private List<CoroutineData> _coroutines = new List<CoroutineData>();

    /// <summary>
    /// 协程数据：记录迭代器 + 当前的等待条件
    /// </summary>
    private class CoroutineData
    {
        public IEnumerator enumerator;            // 协程迭代器实例
        public SimpleWaitForSeconds waitForSeconds; // 当前等待的 SimpleWaitForSeconds（如果有）
        public bool isWaitingNextFrame;            // 是否在等待下一帧（yield return null）
    }

    /// <summary>
    /// 开启一个协程，传入迭代器方法
    /// </summary>
    public void StartSimpleCoroutine(IEnumerator coroutine)
    {
        // 调用 MoveNext() 立即执行到第一个 yield return
        if (coroutine.MoveNext())
        {
            var data = new CoroutineData { enumerator = coroutine };
            // 解析第一个 yield return 返回的是什么
            ParseYieldReturn(data);
            _coroutines.Add(data);
        }
        // 如果 MoveNext() 返回 false，说明协程里没有 yield return，已经执行完毕，不需要加入列表
    }

    /// <summary>
    /// 停止所有协程
    /// </summary>
    public void StopAllSimpleCoroutines()
    {
        _coroutines.Clear();
    }

    private void Update()
    {
        // 倒序遍历方便在迭代中移除元素
        for (int i = _coroutines.Count - 1; i >= 0; i--)
        {
            var data = _coroutines[i];
            bool shouldAdvance = false;

            // 判断当前等待条件是否满足
            if (data.isWaitingNextFrame)
            {
                // yield return null —— 等一帧，Update 中直接满足
                shouldAdvance = true;
            }
            else if (data.waitForSeconds != null)
            {
                // yield return SimpleWaitForSeconds —— 累加时间，检查是否到时间
                shouldAdvance = data.waitForSeconds.Tick(Time.deltaTime);
            }

            if (shouldAdvance)
            {
                // 条件满足，推动迭代器前进一步
                if (data.enumerator.MoveNext())
                {
                    // 还有下一步，解析新的 yield return 值
                    ParseYieldReturn(data);
                }
                else
                {
                    // 协程执行完毕，从列表中移除
                    _coroutines.RemoveAt(i);
                }
            }
        }
    }

    /// <summary>
    /// 解析 yield return 返回的值，决定下次恢复的条件
    /// </summary>
    private void ParseYieldReturn(CoroutineData data)
    {
        // 重置等待状态
        data.waitForSeconds = null;
        data.isWaitingNextFrame = false;

        object current = data.enumerator.Current;

        if (current == null)
        {
            // yield return null → 等待一帧
            data.isWaitingNextFrame = true;
        }
        else if (current is SimpleWaitForSeconds)
        {
            // yield return new SimpleWaitForSeconds(n) → 等待 n 秒
            data.waitForSeconds = current as SimpleWaitForSeconds;
        }
        // 可以扩展更多类型：yield return 数字 → 等一帧
        else if (current is int || current is float)
        {
            data.isWaitingNextFrame = true;
        }
    }
}
```

**3. 使用示例**

```csharp
using UnityEngine;

public class DemoCoroutine : MonoBehaviour
{
    private SimpleCoroutineRunner _runner;

    void Start()
    {
        // 挂载调度器
        _runner = gameObject.AddComponent<SimpleCoroutineRunner>();

        // 开启自定义协程
        _runner.StartSimpleCoroutine(MySimpleCoroutine());
    }

    /// <summary>
    /// 使用自定义 SimpleWaitForSeconds 的协程
    /// </summary>
    IEnumerator MySimpleCoroutine()
    {
        Debug.Log("协程开始 - " + Time.time);

        // 等待 2 秒（使用自己实现的等待指令）
        yield return new SimpleWaitForSeconds(2f);
        Debug.Log("2 秒后执行 - " + Time.time);

        // 等待 1 帧
        yield return null;
        Debug.Log("再等一帧后执行 - " + Time.time);

        // 等待 1.5 秒
        yield return new SimpleWaitForSeconds(1.5f);
        Debug.Log("再等 1.5 秒后执行 - " + Time.time);

        Debug.Log("协程结束");
    }
}
```

**4. 执行流程示意**

以 `MySimpleCoroutine` 为例，整个调度过程如下：

```
帧 0 (Start):
  → StartSimpleCoroutine() 被调用
  → myEnumerator.MoveNext() 执行到第一个 yield return
    → 输出 "协程开始"
    → Current = SimpleWaitForSeconds(2f)
  → ParseYieldReturn 检测到 SimpleWaitForSeconds，记录等待条件
  → 协程加入调度列表

帧 N (每帧 Update):
  → 遍历调度列表
  → data.waitForSeconds.Tick(Time.deltaTime) 累加时间
  → 还没到 2 秒 → 跳过，什么都不做

帧 M (累计 ≥ 2 秒的那一帧 Update):
  → Tick() 返回 true
  → myEnumerator.MoveNext() 从上次暂停处继续执行
    → 输出 "2 秒后执行"
    → Current = null
  → ParseYieldReturn 检测到 null，设置 isWaitingNextFrame = true

帧 M+1 (下一帧 Update):
  → isWaitingNextFrame == true
  → myEnumerator.MoveNext()
    → 输出 "再等一帧后执行"
    → Current = SimpleWaitForSeconds(1.5f)
  → ParseYieldReturn 记录新的等待条件

...以此类推，直到 MoveNext() 返回 false，协程移除
```

**要点总结：**

| 概念 | 对应实现 |
|------|---------|
| 协程函数 | C# 中带 `yield return` 返回 `IEnumerator` 的方法 |
| 暂停点 | `yield return` 语句，编译器在此处切分状态 |
| 恢复执行 | 调用 `MoveNext()`，从上一状态继续 |
| 等待条件 | `IEnumerator.Current` 的值（`null` / `SimpleWaitForSeconds`） |
| 调度器 | `Update` 中检查条件 → 满足时调用 `MoveNext()` |
| 生命周期管理 | 协程列表 + `MoveNext()` 返回 `false` 时移除 |

Unity 的协程系统在底层做的事情和上面的代码完全一致，只是它支持更多的等待条件（`WaitForFixedUpdate`、`WaitForEndOfFrame`、`AsyncOperation` 等），并且调度时机与 Unity 的事件函数执行顺序紧密集成。

## Unity 特殊文件夹

### Resources资源文件夹

```csharp
//路径获取
string pathPath = Application.dataPath + "/Resources";
```

该文件夹需要我们自己来创建  
作用：1.需要通过Resources相关API动态加载的资源需要放在其中    
2.该文件夹下所有文件都会被打包出去  
3.打包时Unity会对其压缩加密 
4.该文件夹打包后只读 只能通过Resources相关API加载   

### StreamingAssets流式资产文件夹

```csharp
//路径获取
string pathPath = Application.streamingAssetsPath;
```

该文件夹需要自己创建    
作用：流文件夹  
1.打包时该文件夹下的文件不会被压缩加密      
2.移动平台只读 PC平台可读写 
3.可放入一些需要自定义动态加载的初始资源    

### PersistentData持久数据文件夹

```csharp
//路径获取
string pathPath = Application.persistentDataPath;
```

不需要自己创建
作用：固定数据文件夹    
1.所有平台都可读可写    
2.一般用于放置动态下载或者动态创建的文件，游戏中创建或者获取的文件都放在其中

### Plugins插件文件夹

一般不获取路径  

需要自己创建
作用：插件文件夹    
1.不同平台的插件相关文件放在其中  比如IOS和安卓平台


### Editor编辑文件夹

一般不获取路径      

需要我们自己创建    
作用：编辑文件夹    
1.开发Unity编辑器时，编辑器相关脚本放在该文件夹中   
2.该文件夹中内容不会被打包出去  

### 默认资源文件夹 Standard Assets

一般不获取路径  

需要我们自己创建    
作用：默认资源文件夹    
1.一般Unity自带资源都放在这个文件夹下   代码和资源优先被编译


## Resources同步加载

### Resources资源动态加载的作用

通过代码动态加载Resources文件夹下指定路径资源 避免繁琐的拖曳操作


```csharp
//加载资源
//现在Resources文件夹下有一个文件夹为MyResources，文件夹下有一个文件为MyResource.txt
//加载MyResource.txt文件
TextAsset textAsset = Resources.Load("MyResources/MyResource") as TextAsset;
textAsset = Resources.Load<TextAsset>("MyResources/MyResource"); //泛型最好用
textAsset = Resources.Load("MyResources/MyResource", typeof(TextAsset)) as TextAsset; //这种一般是在有同名资源时使用 指定类型加载
```

## Resources异步加载

```csharp
//异步加载资源
//加载MyResource.txt文件 多开线程加载
AsyncOperation asyncOperation = Resources.LoadAsync<TextAsset>("MyResources/MyResource");
asyncOperation.completed += (operation) =>
{
    TextAsset textAsset = operation.asset as TextAsset;
    //使用textAsset
    Debug.Log(textAsset.text);
};

//使用协程加载 使用协程更加方便多个资源的加载 同时也可适合进度条画面的使用
IEnumerator LoadMyResource()
{
    AsyncOperation asyncOperation = Resources.LoadAsync<TextAsset>("MyResources/MyResource");
    yield return asyncOperation;
    TextAsset textAsset = asyncOperation.asset as TextAsset;
    //使用textAsset
    Debug.Log(textAsset.text);
}

//加载多个资源
IEnumerator LoadMyResource2()
{
    AsyncOperation asyncOperation = Resources.LoadAsync<TextAsset>("MyResources/MyResource");

    AsyncOperation asyncOperation2 = Resources.LoadAsync<TextAsset>("MyResources/MyResource2");
    while (!asyncOperation.isDone || !asyncOperation2.isDone)
    {
        print(asyncOperation.progress); //打印加载进度
        yield return null;
    }
    TextAsset textAsset = asyncOperation.asset as TextAsset;
    //使用textAsset
    Debug.Log(textAsset.text);
}
```

自定义Resources异步加载管理器
```csharp
using System;
using System.Collections;
using System.Collections.Generic;
using UnityEngine;
using UnityEngine.Events;

public class ResourcesManager : MonoBehaviour
{
    private static ResourcesManager _instance;
    public static ResourcesManager Instance 
    { 
        get 
        { 
            return _instance; 
        }
    }

    private void Awake()
    {
        _instance = this;
    }

    public void LoadResourceAsync<T>(string path , Action<T> callBack) where T : UnityEngine.Object
    {
        ResourceRequest resource = Resources.LoadAsync<T>(path);

        //当资源加载完成时 会触发completed事件 将operation作为参数传递给委托
        resource.completed +=((operation) =>
        {
            //在委托内调用回调方法 并将加载完成的资源作为参数传递给回调方法
            callBack((operation as ResourceRequest).asset as T); //这里opreation的类型为AsyncOperation 所以需要强制转换为ResourceRequest后才能获取asset
        });
    }

}
```

## Resources资源卸载

Resources在一次加载资源以后 该资源就一直存放在内存中作为缓存    
在第二次加载时发现缓存中存在该资源就会直接取出来使用    
所以多次重复加载不会浪费内存 但是会浪费性能 

### 为什么需要卸载资源

虽然缓存机制避免了重复内存占用，但也带来问题：  

1. **资源加载后常驻内存** —— 即使不再使用，资源也不会自动释放，导致内存持续增长
2. **切换场景时缓存不会自动清理** —— 旧场景的资源仍然占用内存
3. **大型资源（纹理、模型、音频）累积** —— 可能导致内存不足、卡顿甚至闪退

因此需要在合适的时机主动释放不再使用的资源。

### 卸载方式一：Resources.UnloadAsset

卸载指定的单个资源对象。

```csharp
// 加载一个资源
TextAsset textAsset = Resources.Load<TextAsset>("MyResources/MyResource");

// 使用完毕后，卸载该资源
Resources.UnloadAsset(textAsset);
```

**注意事项：**
- `UnloadAsset` 只能卸载**存储在硬盘上的资源**（如纹理、音频、文本等），不能卸载纯内存中创建的对象
- 卸载后，该资源的所有引用会变为 `null`（在 Editor 中表现为 Missing）
- 如果资源还被其他地方引用，卸载后那些引用会丢失，表现为粉红色 Missing 材质或空引用
- 参数传 `null` 不会报错，但不执行任何操作
- 该方法**不可以**卸载 `GameObject`、`Component` 或 `AssetBundle`

**适用场景：**  
明确知道某个特定资源不再需要，且确认没有其他地方引用它时使用。例如：过场动画播放完毕后卸载对应的视频资源。

### 卸载方式二：Resources.UnloadUnusedAssets

卸载所有**未被引用**的资源，即当前没有任何活跃对象持有其引用的资源。

```csharp
// 卸载所有未被引用的资源
Resources.UnloadUnusedAssets();
```

**工作原理：**
1. Unity 遍历所有已加载的资源
2. 检查每个资源是否有活跃引用（场景中的物体、脚本中的变量等）
3. 没有被任何地方引用的资源会被卸载回收
4. 该操作是**异步**的，会在后台执行，通常需要多帧才能完成

**注意事项：**
- **非常耗时** —— 这是一个全局扫描操作，可能造成卡顿，不建议每帧调用
- 内部实际上是通过 GC（垃圾回收）来清理的，所以卸载的资源不会立即释放内存，需要等 GC 触发
- 如果资源虽然当前没在用，但**即将**使用，则不要调用此方法，否则会导致重复加载
- 在 Editor 中调用时效果可能与真机不同（Editor 有额外的资源引用）

**适用场景：**
- 场景切换后清理上一场景的残留资源（常见用法）：
  ```csharp
  using UnityEngine.SceneManagement;
  
  IEnumerator LoadNewScene()
  {
      // 1. 异步加载新场景
      AsyncOperation op = SceneManager.LoadSceneAsync("NewScene");
      yield return op;
      
      // 2. 场景加载完后，清理旧资源
      Resources.UnloadUnusedAssets();
      
      // 3. 建议配合 GC 使用，确保内存彻底释放
      System.GC.Collect();
  }
  ```
- 结束一场战斗或关闭一个大型 UI 界面后
- 内存告警时做应急清理

### 卸载方式三：AssetBundle.Unload

如果是通过 AssetBundle 加载的资源，需要使用 AssetBundle 的卸载方法。

```csharp
// 卸载 AssetBundle，参数控制是否同时卸载从中加载的资源
assetBundle.Unload(false);  // 仅卸载 Bundle 本身，已加载出的资源保留
assetBundle.Unload(true);   // 卸载 Bundle 及所有从中加载的资源
```

`Unload(true)` 会使所有从该 Bundle 加载的资源变为 `null`，使用时需格外小心。

### 三种卸载方式对比

| 方法 | 卸载范围 | 是否异步 | 性能消耗 | 使用场景 |
|------|---------|---------|---------|---------|
| `Resources.UnloadAsset(obj)` | 单个指定资源 | 同步 | 低 | 精确释放某个不再需要的资源 |
| `Resources.UnloadUnusedAssets()` | 所有未被引用的资源 | 异步 | **高** | 场景切换后、内存清理 |
| `AssetBundle.Unload(bool)` | AssetBundle 及（可选）其资源 | 同步 | 中 | 卸载 AssetBundle 包 |

### 资源卸载的完整实践

```csharp
using System.Collections;
using UnityEngine;
using UnityEngine.SceneManagement;

public class ResourceCleanupExample : MonoBehaviour
{
    // 示例一：精确卸载 —— 加载并释放单个资源
    void LoadAndUnloadSingleAsset()
    {
        // 加载
        Texture2D tex = Resources.Load<Texture2D>("Textures/BigTexture");
        
        // 使用...
        GetComponent<Renderer>().material.mainTexture = tex;
        
        // 使用完毕，替换掉引用
        GetComponent<Renderer>().material.mainTexture = null;
        
        // 精确卸载
        Resources.UnloadAsset(tex);
    }

    // 示例二：场景切换时的完整内存清理流程（推荐）
    IEnumerator SwitchSceneAndCleanup(string sceneName)
    {
        // 第一步：切断不需要的引用
        // 将引用置空，让资源变成"未被引用"状态
        _cachedReferences.Clear();  // 清空缓存
        _pooledObjects.ForEach(obj => Destroy(obj));  // 销毁对象池
        _pooledObjects.Clear();
        
        // 第二步：加载新场景
        AsyncOperation loadOp = SceneManager.LoadSceneAsync(sceneName);
        loadOp.allowSceneActivation = true;
        yield return loadOp;
        
        // 第三步：卸载无引用资源
        AsyncOperation unloadOp = Resources.UnloadUnusedAssets();
        yield return unloadOp;  // 等待卸载完成
        
        // 第四步：强制 GC 回收内存
        System.GC.Collect();
        
        Debug.Log("内存清理完成");
    }

    // 示例三：内存告警时自动清理
    void Update()
    {
        // 当内存超过阈值时主动清理（不推荐每帧判断，这里仅为示例）
        if (System.GC.GetTotalMemory(false) > 500 * 1024 * 1024) // 500MB
        {
            StartCoroutine(EmergencyCleanup());
        }
    }

    IEnumerator EmergencyCleanup()
    {
        Debug.LogWarning("内存告警，开始清理...");
        Resources.UnloadUnusedAssets();
        yield return null;
        System.GC.Collect();
        Debug.Log("应急清理完成");
    }
}
```

### 内存管理的核心原则

1. **加载自己管** —— 谁加载谁负责释放，或者统一由一个管理器管理
2. **切场景必清** —— 场景切换后调用 `UnloadUnusedAssets` 是最基本的做法
3. **引用置空再卸载** —— `UnloadUnusedAssets` 只会卸载"无引用"的资源，如果脚本变量、静态变量、事件等还持有引用，资源不会被释放
4. **避免频繁调用** —— `UnloadUnusedAssets` 开销大，不要放在 `Update` 中每帧调用
5. **精确优先** —— 如果能确定具体资源不再使用，优先使用 `UnloadAsset` 精确卸载，而不是动辄全量扫描
6. **注意静态引用** —— 静态变量持有的资源引用永远不会被 GC，需要手动置 `null`
7. **关注 DontDestroyOnLoad** —— 跨场景物体持有的资源引用在场景切换后仍然有效，需要特别注意


## 场景异步加载

在切换场景时 Unity会删除当前场景上的所有对象 并且去加载下一个场景的相关信息 
如果当前场景 对象过多或者下一个场景对象过多 
这个过程会非常的耗时 会让玩家感觉到卡顿 
所以异步切换就是来解决该问题的  

1.通过事件回调进行场景加载
```csharp
//异步加载场景
void Start()
{
   
    AsyncOperation loadOp = SceneManager.LoadSceneAsync("NextScene");
    loadOp.completed +=((operation) =>
    {
        Debug.Log("场景加载完成");
    });
}
```

2.通过协程进行场景加载
```csharp
//2.通过协程进行场景加载
//为避免在场景加载完成后 挂载了该脚本的对象被销毁导致不能执行场景加载完后的逻辑需要使用DontDestroyOnLoad方法
IEnumerator LoadNextScene()
{
    AsyncOperation loadOp = SceneManager.LoadSceneAsync("NextScene");
    loadOp.allowSceneActivation = true; // 允许场景激活
    yield return loadOp; // 等待场景加载完成 可以在加载完成后执行后续逻辑
    Debug.Log("场景加载完成");
}

void Start()
{
    StartCoroutine(LoadNextScene());
    DontDestroyOnLoad(this.gameObject); // 防止挂载了加载场景的脚本的物体在场景切换后被销毁 
}

```

## LineRenderer 

LineRenderer 是Unity提供的一个用于画线的组件 使用它我们可以在场景中绘制线段     
一般用于：  
1.绘制攻击范围  
2.武器红外线    
3.辅助功能  
4.其他画线功能  

### 组件参数详解

挂载 LineRenderer 组件后，在 Inspector 中可以看到以下参数：

#### Positions —— 线段的顶点位置

LineRenderer 通过一系列顶点来定义线段的形状。最少需要 2 个点才能画出一条线。

```csharp
LineRenderer lr = GetComponent<LineRenderer>();

// 设置顶点数量
lr.positionCount = 3;

// 设置每个顶点的位置
lr.SetPosition(0, new Vector3(0, 0, 0));
lr.SetPosition(1, new Vector3(1, 1, 0));
lr.SetPosition(2, new Vector3(2, 0, 0));

// 也可以直接赋值数组
lr.SetPositions(new Vector3[] 
{
    new Vector3(0, 0, 0),
    new Vector3(1, 1, 0),
    new Vector3(2, 0, 0)
});

// 批量获取所有顶点位置
Vector3[] positions = new Vector3[lr.positionCount];
lr.GetPositions(positions);
```

#### Loop —— 是否闭合线段

```csharp
lr.loop = true;  // 将首尾相连形成闭环（如画一个圆形警戒圈）
```

- `false`（默认）：首尾不相连，是一条开放线
- `true`：自动连接最后一个点和第一个点，形成闭合图形

#### Width —— 线段的宽度

控制线段从头到尾的宽度变化，使用 `AnimationCurve` 曲线来定义。

```csharp
// 设置整体统一宽度
lr.startWidth = 0.1f;
lr.endWidth = 0.1f;

// 使用曲线控制沿线的宽度变化（更灵活）
AnimationCurve widthCurve = new AnimationCurve();
widthCurve.AddKey(0.0f, 1.0f);   // 线起点处：宽度 * 1.0
widthCurve.AddKey(0.5f, 0.5f);   // 线中部：宽度 * 0.5
widthCurve.AddKey(1.0f, 1.0f);   // 线终点处：宽度 * 1.0
lr.widthCurve = widthCurve;       // 曲线定义的宽度会乘上 startWidth/endWidth
lr.widthMultiplier = 1.0f;        // 整体宽度倍增器
```

**关键点：**
- `startWidth` / `endWidth` 是最简单的控制方式，分别控制首尾宽度，中间线性过渡
- `widthCurve` 可以精确控制线上每个位置的宽度比例（0=起点，1=终点）
- `widthMultiplier` 是一个全局倍增器，修改它不会丢失曲线数据，适合做动画

#### Color —— 线段的颜色

控制线段从头到尾的颜色变化，使用 `Gradient` 渐变来定义。

```csharp
// 简单设置单一颜色
lr.startColor = Color.red;
lr.endColor = Color.blue;

// 使用 Gradient 渐变（更灵活，支持多个颜色关键点）
Gradient gradient = new Gradient();
gradient.SetKeys(
    new GradientColorKey[] 
    { 
        new GradientColorKey(Color.red, 0.0f),      // 0% 处：红色
        new GradientColorKey(Color.green, 0.5f),     // 50% 处：绿色
        new GradientColorKey(Color.blue, 1.0f)       // 100% 处：蓝色
    },
    new GradientAlphaKey[] 
    { 
        new GradientAlphaKey(1.0f, 0.0f),            // 0% 处：不透明
        new GradientAlphaKey(1.0f, 0.5f),            // 50% 处：不透明
        new GradientAlphaKey(0.0f, 1.0f)             // 100% 处：完全透明
    }
);
lr.colorGradient = gradient;
```

**注意：** `colorGradient` 和 `startColor`/`endColor` 是互斥的——如果设定了 `colorGradient`，则以渐变模式运行，`startColor`/`endColor` 不再生效。

#### Materials —— 线段的材质

```csharp
// LineRenderer 需要至少一个材质
lr.material = new Material(Shader.Find("Sprites/Default"));

// 也可以使用默认线材质
lr.material = new Material(Shader.Find("Legacy Shaders/Particles/Alpha Blended Premultiply"));

// 多个材质的情况（较少用）
lr.sharedMaterials = new Material[] { material1, material2 };
```

通常使用默认的 `Sprites/Default` Shader 即可，如果需要发光效果可以选择 `Particles/Additive`。

#### Texture Mode —— 纹理平铺模式

控制纹理如何沿线段应用。

```csharp
// 四种模式：
lr.textureMode = LineTextureMode.Stretch;       // 拉伸：纹理拉伸覆盖整条线
lr.textureMode = LineTextureMode.Tile;          // 平铺：纹理沿线段重复平铺
lr.textureMode = LineTextureMode.DistributePerSegment; // 逐段分布：每段之间独立分布纹理
lr.textureMode = LineTextureMode.RepeatPerSegment;     // 逐段重复：每段内部纹理重复
```

默认是 `Stretch`。如果线段上有重复纹理需求（如虚线效果），选择 `Tile`。

#### Use World Space —— 坐标系模式

```csharp
lr.useWorldSpace = true;   // 使用世界坐标（默认）：顶点位置是绝对世界坐标
lr.useWorldSpace = false;  // 使用局部坐标：顶点位置相对于挂载物体
```

- 世界坐标：物体移动时线条不会跟着动（除非每帧更新位置）
- 局部坐标：线条随物体一起移动旋转缩放

#### Alignment —— 对齐方向

控制线段面向的方向，即线段横截面的朝向。

```csharp
lr.alignment = LineAlignment.View;       // 面对摄像机（默认），始终面向玩家视角
lr.alignment = LineAlignment.TransformZ; // 面向物体 Z 轴（常用在 2D 或固定视角）
lr.alignment = LineAlignment.Local;      // 面向本地坐标系（新版 Unity）
```

`View` 是 3D 场景中最常见的，让线条始终面对摄像机以保证可见。

#### Corner Vertices —— 拐角顶点数

```csharp
lr.numCornerVertices = 5;  // 拐角处的额外顶点数，值越大拐角越圆滑
```

- 默认值 0：拐角是尖角
- 值越大拐角越圆润，但顶点数增多带来性能开销
- 在画平滑曲线时很有用

#### End Cap Vertices —— 端点顶点数

```csharp
lr.numCapVertices = 5;  // 线段两端的额外顶点数，值越大端点越圆滑
```

- 默认值 0：端点是平的（方形）
- 值越大端点越圆，类似给线头加上圆角效果

#### Shadow Bias —— 阴影偏移

```csharp
lr.shadowBias = 0.5f;  // 阴影偏移值，避免阴影与线段重叠产生闪烁（Z-Fighting）
```

一般保持默认值 0.5 即可，仅在线段投射阴影时需要关注。

#### Sorting Layer / Order in Layer —— 排序层级

```csharp
lr.sortingLayerID = SortingLayer.NameToID("Foreground");
lr.sortingOrder = 5;  // 在同一 Sorting Layer 中的渲染顺序
```

与 SpriteRenderer 的排序机制一致，用于 2D 场景中控制绘制顺序。

### 常用方法

```csharp
LineRenderer lr = GetComponent<LineRenderer>();

// ---- 简化设置（快速画线） ----
// 直接设置两个端点
lr.SetPosition(0, startPoint);
lr.SetPosition(1, endPoint);

// ---- 批量设置 ----
lr.positionCount = points.Length;
lr.SetPositions(points);

// ---- 烘焙网格 ----
// 将 LineRenderer 的线烘焙成静态 Mesh（用于特效、碰撞等）
Mesh mesh = new Mesh();
lr.BakeMesh(mesh, Camera.main, true); // true = 使用世界空间坐标

// ---- 获取包围盒 ----
Bounds bounds = lr.bounds; // 获取线的包围盒

// ---- 简化器 ----
lr.Simplify(tolerance); // 简化顶点，用更少的点逼近原曲线，tolerance 控制误差容忍度
```

### 代码绘制示例

```csharp
using UnityEngine;

public class LineRendererDemo : MonoBehaviour
{
    private LineRenderer _lr;

    void Start()
    {
        // 1. 挂载并初始化 LineRenderer
        _lr = gameObject.AddComponent<LineRenderer>();

        // 2. 设置材质（必须有材质才能渲染）
        _lr.material = new Material(Shader.Find("Sprites/Default"));

        // 3. 基本参数
        _lr.startWidth = 0.2f;
        _lr.endWidth = 0.2f;
        _lr.loop = false;
        _lr.useWorldSpace = true;
        _lr.alignment = LineAlignment.View;

        // 4. 设置颜色
        _lr.startColor = Color.red;
        _lr.endColor = Color.yellow;

        // 5. 设置顶点
        _lr.positionCount = 4;
        _lr.SetPositions(new Vector3[]
        {
            new Vector3(0, 0, 0),
            new Vector3(1, 1, 0),
            new Vector3(0, 2, 0),
            new Vector3(-1, 1, 0)
        });
    }
}
```

### 实用案例：绘制攻击范围圆

用循环给 LineRenderer 设置圆周上的顶点来画一个圆：

```csharp
void DrawCircle(LineRenderer lr, Vector3 center, float radius, int segments = 50)
{
    lr.positionCount = segments + 1;  // +1 是为了闭合（回到起点）
    lr.loop = true;                   // 或者用 Loop 闭合

    float angleStep = 360f / segments;
    for (int i = 0; i <= segments; i++)
    {
        float angle = i * angleStep * Mathf.Deg2Rad;
        float x = center.x + Mathf.Cos(angle) * radius;
        float z = center.z + Mathf.Sin(angle) * radius;
        lr.SetPosition(i, new Vector3(x, center.y, z));
    }
}
```

### 实用案例：绘制射线/红外线

从一个点射出一条射线，碰到障碍物后终点变为碰撞点：

```csharp
void DrawRayLine(LineRenderer lr, Vector3 origin, Vector3 direction, float maxDistance)
{
    lr.positionCount = 2;
    lr.SetPosition(0, origin);

    if (Physics.Raycast(origin, direction, out RaycastHit hit, maxDistance))
    {
        lr.SetPosition(1, hit.point);   // 碰到物体，终点是碰撞点
    }
    else
    {
        lr.SetPosition(1, origin + direction * maxDistance); // 没碰到，终点是最远距离
    }
}
```

## 范围检测

必备条件：想要被范围检测到的对象必须具备碰撞器

范围检测API
1.盒状范围检测
```csharp
//参数一：立方体中心点
//参数二：立方体三边大小
//参数三：立方体角度
//参数四：检测指定层级(不填指定所有层)
//参数五：是否忽略触发器 UseGlobal-使用全局设置 Collide-检测触发器 Ignore-忽略触发器 默认为UseGlobal
//返回值：在该范围内的碰撞器
Collider[] colliders = Physics.OverlapBox(Vector3.zero, new Vector3(1, 1, 1), Quaternion.identity, 
1 << LayerMask.NameToLayer("Player"), QueryInteraction.UseGlobal);

for(int i = 0; i < colliders.Length; i++)
{
    Debug.Log(colliders[i].gameObject.name);
}

//重要知识点：
//关于层级
//通过名字得到层级编号LayerMask.NameToLayer
//我们需要通过编号左移构建二进制数
//这样每一个编号的层级都是对应位为1的2进制数
//我们通过位运算可以选择想要检测层级
//好处一个int就可以表示所有想要检测的层级信息
//层级编号是0~31刚好32位
//是一个int数
//每一个编号代表的都是二进制的一位
//0-1 << 0-0000 0000 0000 0000 0000 0000 0000 0001 = 1
//1-1 << 1-0000 0000 0000 0000 0000 0000 0000 0010 = 2
//2-1 << 2-0000 0000 0000 0000 0000 0000 0000 0100 = 4
//3-1 << 3-0000 0000 0000 0000 0000 0000 0000 1000 = 8
//4-1 << 4-0000 0000 0000 0000 0000 0000 0001 0000 = 16
//5-1 << 5-0000 0000 0000 0000 0000 0000 0010 0000 = 32

//在Unity判断是否为指定层级则是遍历所有层级将每个层级的二进制数和传进去的值进行与运算判断是否不为零 
//如果不为零则说明该层级是想要检测的层级

//另一个API返回在该范围内碰撞器的数量
//参数：传入一个数组进行存储
int count = Physics.OverlapBoxCount(Vector3.zero, new Vector3(1, 1, 1), colliders);
Debug.Log(count);
```

2.球形范围检测

```csharp
//参数一：球心
//参数二：球半径
//参数三：检测指定层级(不填指定所有层)
//参数四：是否忽略触发器 UseGlobal-使用全局设置 Collide-检测触发器 Ignore-忽略触发器 默认为UseGlobal
//返回值：在该范围内的碰撞器
Collider[] colliders = Physics.OverlapSphere(Vector3.zero, 1, 1 << LayerMask.NameToLayer("Player"),
QueryInteraction.UseGlobal);

for(int i = 0; i < colliders.Length; i++)
{
    Debug.Log(colliders[i].gameObject.name);
}

//另一个API返回在该范围内碰撞器的数量
//参数：传入一个数组进行存储
int count = Physics.OverlapSphereCount(Vector3.zero, 1, colliders);
Debug.Log(count);
```

3.胶囊体范围检测

```csharp
//参数一：半圆1中心点
//参数二：半圆2中心点
//参数三：半圆半径
//参数四：检测指定层级(不填指定所有层)
//参数五：是否忽略触发器 UseGlobal-使用全局设置 Collide-检测触发器 Ignore-忽略触发器 默认为UseGlobal
//返回值：在该范围内的碰撞器
Collider[] colliders = Physics.OverlapCapsule(Vector3.zero, Vector3.up, 1, 1 << LayerMask.NameToLayer("Player"),
QueryInteraction.UseGlobal);

for(int i = 0; i < colliders.Length; i++)
{
    Debug.Log(colliders[i].gameObject.name);
}

//另一个API返回在该范围内碰撞器的数量
int count = Physics.OverlapCapsuleCount(Vector3.zero, Vector3.up, 1, colliders); 
Debug.Log(count);
```

## 射线检测

建立一条射线
```csharp
//参数一：起点
//参数二：方向（方向向量）
Ray ray = new Ray(Vector3.zero, Vector3.up); // 从原点射出一条射线，方向向上
```

摄像机发射出的射线
```csharp
//参数一：屏幕上点的位置
//方向为摄像机视口方向
Ray ray = Camera.main.ScreenPointToRay(Input.mousePosition);
```

### 射线检测的碰撞检测

1.最原始的射线检测
```csharp
//参数一：射线
//参数二：检测的最大距离（超出这个范围不检测）
//参数三：是否检测指定层级(不填指定所有层)
//参数四：是否忽略触发器 UseGlobal-使用全局设置 Collide-检测触发器 Ignore-忽略触发器 默认为UseGlobal
//返回值bool 当碰撞到对象时返回true，否则返回false

if(Physics.Raycast(ray, 100, 1 << LayerMask.NameToLayer("Player"), QueryInteraction.UseGlobal))
{
    Debug.Log("碰撞到玩家");
}

//还有一种重载是把射线变成了起点和方向

```

2.获取相交的单个物体信息   
```csharp
//参数一：射线
//参数二：存储碰撞信息的结构体
//参数三：检测的最大距离（超出这个范围不检测）
//参数四：是否检测指定层级(不填指定所有层)
//参数五：是否忽略触发器 UseGlobal-使用全局设置 Collide-检测触发器 Ignore-忽略触发器 默认为UseGlobal
RaycastHit hitInfo;
if(Physics.Raycast(ray, out hitInfo, 100, 1 << LayerMask.NameToLayer("Player"), QueryInteraction.UseGlobal))
{
    //检测到的物体名
    Debug.Log(hitInfo.collider.gameObject.name);

    //碰撞点
    Debug.Log(hitInfo.point);

    //法线
    Debug.Log(hitInfo.normal);

    //位置
    Debug.Log(hitInfo.transform.position);

    //碰撞到对象离自己的距离
    Debug.Log(hitInfo.distance);

}
```

3.获取相交的所有物体信息
```csharp
//参数一：射线
//参数二：检测的最大距离（超出这个范围不检测）
//参数三：是否检测指定层级(不填指定所有层)
//参数四：是否忽略触发器 UseGlobal-使用全局设置 Collide-检测触发器 Ignore-忽略触发器 默认为UseGlobal
RayCastHit hitInfo = Physics.RaycastAll(ray, 100, 1 << LayerMask.NameToLayer("Player"), QueryInteraction.UseGlobal);
for(int i = 0; i < hitInfo.Length; i++)
{
    Debug.Log(hitInfo[i].collider.gameObject.name);
}

//还有一种是直接把碰撞信息数组传入RaycastAll方法中作为参数的
if(Physics.RaycastAll(ray, hitInfo, 100, 1 << LayerMask.NameToLayer("Player"), QueryInteraction.UseGlobal) > 0)
{
    for(int i = 0; i < hitInfo.Length; i++)
    {
        Debug.Log(hitInfo[i].collider.gameObject.name);
    }
}
```



