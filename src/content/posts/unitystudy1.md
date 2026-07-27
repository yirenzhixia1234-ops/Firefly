---
title: Unity学习笔记1
published: 2026-07-14
pinned: false
description: Unity学习笔记1 - Unity编辑器、GameObject、组件系统等基础
tags: [Unity,学习]
author: boluobao
draft: false
---

# 🎮 Unity学习笔记1

> [!NOTE] 学习概览
> 本篇笔记涵盖 Unity 最核心的基础知识：**GameObject（游戏对象）**、**Time（时间系统）**、**Transform（变换组件）**、**Input & Screen（输入与屏幕）**、**Camera（摄像机）**以及**光源系统**。这些是 Unity 开发的基石。

---

## 🧩 GameObject

GameObject 是 Unity 中**所有实体的基类**，场景中的每一个物体（角色、摄像机、灯光、UI 等）都是 GameObject。它本身不执行功能，而是通过**挂载组件（Component）**来赋予行为。

### 📋 GameObject 的成员变量

```csharp
this.gameObject.name; // 获取游戏对象的名称

this.gameObject.activeSelf; // 获取游戏对象是否激活

this.gameObject.isStatic; // 获取游戏对象是否为静态对象

this.gameObject.layer; // 获取游戏对象的图层

this.gameObject.tag; // 获取游戏对象的标签名

this.gameObject.transform; // 获取游戏对象的Transform组件 但是不如this.transform方便
```

### 🛠️ GameObject 的静态方法

#### 创建与查找

```csharp
GameObject.CreatePrimitive(PrimitiveType.Cube); // 创建一个立方体游戏对象

//查找对象相关 无法找到失活的游戏对象
//如果场景中存在多个游戏对象 且都有相同的名字或标签名 则无法确定返回的是哪个游戏对象
//1.查找单个对象 通过名字查找
GameObject cube = GameObject.Find("Cube"); // 查找一个名字为Cube的游戏对象
if(cube != null)
{
    //做出相应逻辑
}
//不过该查找效率较低 因为会遍历所有游戏对象 找到第一个匹配的即可停止
//通过tag查找
GameObject cube = GameObject.FindWithTag("Cube");
if(cube != null)
{
    //做出相应逻辑
}

//2.查找多个对象 无法找到失活的游戏对象
GameObject[] cubes = GameObject.FindGameObjectsWithTag("Cube");
if(cubes.Length > 0)
{
    //做出相应逻辑
}
```

> [!WARNING] 查找注意事项
> - `Find` 系列方法 ==无法找到失活（SetActive(false)）的游戏对象==
> - 遍历所有对象，**效率较低**，==不要在 Update 中频繁调用==
> - 同名/同标签时返回结果不确定，尽量避免重名

```csharp
//还有几个方法用的比较少 是GameObject父类 Object提供的方法
//Unity里面的Object和万物之父object不同
//Unity里的Object类在UnityEngine命名空间下 Object类是集成了object类的一个自定义类
//C#的Object类在System命名空间下 Object类是C#的万物之父类
GameObject.FindObjectOfType<GameObject>(); //找到场景中挂载有某个脚本的游戏对象
```

| 对比 | UnityEngine.Object | System.Object |
|------|:--:|:--:|
| 命名空间 | `UnityEngine` | `System` |
| 角色 | Unity 引擎对象的基类 | C# 万物之父 |
| 继承关系 | 继承自 System.Object | 最顶层基类 |

#### 克隆与删除

```csharp
//克隆游戏对象
GameObject cubeClone = GameObject.Instantiate(cube); // 克隆一个游戏对象 如果类继承了MonoBehaviour 则直接用Instantiate方法即可

//删除游戏对象
GameObject.Destroy(cubeClone); // 删除一个游戏对象
GameObject.Destroy(cubeClone, 2f); // 删除一个游戏对象 2秒后删除
GameObject.Destroy(this); //删除对象上的脚本
//Destroy 不会马上移除游戏对象 只是给这个游戏对象加了一个移除标识
//一般情况下他会在游戏的下一帧将这个对象移除并从内存中删除

GameObject.DestroyImmediate(cubeClone); // 立即删除游戏对象
```

| 方法 | 执行时机 | 适用场景 |
|------|------|------|
| **Destroy(obj)** | 当前帧结束时标记，下一帧移除 | 运行时删除（推荐） |
| **Destroy(obj, delay)** | 延迟指定秒数后移除 | 延时销毁（如粒子效果） |
| **DestroyImmediate(obj)** | 立即删除 | 编辑器脚本中必须使用 |

```csharp
//切换场景不移除游戏对象
//如果在场景切换时 不希望游戏对象被销毁 可以使用DontDestroyOnLoad方法
GameObject.DontDestroyOnLoad(cubeClone); // 不在场景切换时销毁游戏对象
```

### 🔧 GameObject 的成员方法

```csharp
//创建空物体
GameObject empty = new GameObject("Empty"); //参数为游戏对象的名称
GameObject empty2 = new GameObject("Empty2", typeof(MonoBehaviour)); //参数为游戏对象的名称 以及要挂载的脚本 可以挂载多个脚本

//为对象添加组件
Collider collider = empty.AddComponent<Collider>(); // 为对象添加一个Collider组件
//通过返回值可以调用组件的方法
Collider collider = empty.GetComponent<Collider>(); // 获取对象上的Collider组件

//标签比较
bool isCube = empty.CompareTag("Cube"); // 比较对象的标签名是否为Cube
if(isCube)
{
    //做出相应逻辑
}

//设置失活激活
empty.SetActive(true); // 激活对象
empty.SetActive(false); // 失活对象
```

> [!IMPORTANT] AddComponent vs GetComponent
> - `AddComponent<T>()` 会**创建并返回**新组件引用，多次调用会添加多个相同组件
> - `GetComponent<T>()` 只**获取**已有的组件，不存在则返回 null
> - `CompareTag()` ==比 `tag == "xxx"` 更高效==（避免了字符串比较的额外开销）

---

## ⏱️ 时间相关 Time

Time 类提供了游戏中**计时、帧间隔、时间缩放**等核心功能，是驱动游戏逻辑（移动、计时器、动画）的基础。

### 🎛️ 时间缩放比例

```csharp
Time.timeScale = 0.5f; // 时间缩放比例 0.5倍（慢放）
Time.timeScale = 1f;   // 时间缩放比例 1倍（正常时间）
Time.timeScale = 2f;   // 时间缩放比例 2倍（快进）
Time.timeScale = 0f;   // 时间缩放比例 0倍（暂停）
```

| timeScale | 效果 | 使用场景 |
|:--:|------|------|
| **1** | 正常速度 | 默认状态 |
| **0** | ==完全暂停== | 暂停菜单、游戏结束 |
| **0.5** | 慢动作 | 子弹时间、技能特写 |
| **> 1** | 加速 | 快进、调试加速 |

### 📏 帧间隔时间

```csharp
//帧间隔时间 主要用于计算位移
//路程 = 速度 × 时间间隔
//根据需求选择参与计算的间隔时间
//如果希望 游戏暂停就不动的 那就使用deltaTime
//如果希望 不受timeScale影响的 那就使用unscaledDeltaTime

//最近的一帧用了多长时间 单位为秒
float deltaTime = Time.deltaTime; //受timeScale影响

float unscaledDeltaTime = Time.unscaledDeltaTime; //不受timeScale影响
```

> [!IMPORTANT] deltaTime 的核心作用
> `路程 = 速度 × Time.deltaTime` —— 用帧间隔时间乘速度，使移动**与帧率无关**。无论 30fps 还是 120fps，物体每秒移动的距离一致。

| 属性 | 受 timeScale 影响 | 典型用途 |
|------|:--:|------|
| **deltaTime** | ✅ 是 | 游戏内移动、计时（暂停时停止） |
| **unscaledDeltaTime** | ❌ 否 | UI 动画、加载进度条（暂停时也运行） |

### ⏳ 游戏开始到现在的时间

```csharp
//获取游戏开始到现在的时间
float time = Time.time; //主要用于计时 受timeScale影响

float unscaledTime = Time.unscaledTime; //不受timeScale影响
```

### 🔬 物理帧间隔时间

```csharp
//物理帧间隔时间 主要用于计算物理模拟
//物理模拟 = 物理力 × 物理时间间隔
//Unity中物理模拟间隔时间默认是0.02f 单位为秒 FixedUpdate方法调用
float physicsDeltaTime = Time.fixedDeltaTime; //物理帧间隔时间 受timeScale影响

float unscaledFixedDeltaTime = Time.fixedUnscaledDeltaTime; //不受timeScale影响
```

### 🔢 帧数

> 从游戏开始到现在跑了多少次循环。

```csharp
int frameCount = Time.frameCount; //获取游戏开始到现在跑了多少次循环
```

---

## 🔄 Transform 组件

Transform 是每个 GameObject 都**必须拥有**的组件，控制物体的**位置（Position）、旋转（Rotation）、缩放（Scale）**以及**父子层级关系**。

### 📐 Vector3 基础

> Vector3 主要用来表示三维空间中的一个**点**或者一个**向量**。

```csharp
//声明
Vector3 pos = new Vector3();
pos.x = 1f; //x轴坐标
pos.y = 2f; //y轴坐标
pos.z = 3f; //z轴坐标
//或
pos = new Vector3(1f, 2f, 3f); //直接赋值
Vector3 pos2 = new Vector3(1f,2f); //不传z轴坐标 默认是0 常用于2D游戏
```

#### 基本运算

```csharp
//加法
Vector3 pos3 = pos + new Vector3(1f, 2f, 3f); //将pos3的坐标设置为pos的坐标加上(1,2,3)
//减法
Vector3 pos4 = pos - new Vector3(1f, 2f, 3f); //将pos4的坐标设置为pos的坐标减去(1,2,3)
//乘法
Vector3 pos5 = pos * 2f; //将pos5的坐标设置为pos的坐标乘以2
//除法
Vector3 pos6 = pos / 2f; //将pos6的坐标设置为pos的坐标除以2
```

#### 常用静态向量

| 向量 | 值 | 方向 |
|------|:--:|------|
| `Vector3.zero` | (0, 0, 0) | 原点 |
| `Vector3.one` | (1, 1, 1) | 单位向量 |
| `Vector3.right` | (1, 0, 0) | 右（X 轴正方向） |
| `Vector3.up` | (0, 1, 0) | 上（Y 轴正方向） |
| `Vector3.forward` | (0, 0, 1) | 前（Z 轴正方向） |
| `Vector3.back` | (0, 0, -1) | 后（Z 轴负方向） |

```csharp
//计算两个点之间的距离
float distance = Vector3.Distance(pos, pos2); //计算pos和pos2之间的距离
```

---

### 📍 位置（Position）

```csharp
//获取相对于世界坐标的位置
Vector3 worldPos = this.transform.position; //获取对象在世界坐标的位置

//获取相对于父对象的位置
Vector3 localPos = this.transform.localPosition; //获取对象在父对象坐标的位置
```

| 属性 | 参照系 | 用途 |
|------|------|------|
| **position** | 世界坐标系 | 判断物体在世界中的绝对位置 |
| **localPosition** | 父对象坐标系 | 子对象相对父对象的位置偏移 |

```csharp
//位置的修改
//位置的赋值不能单个修改xyz某一个值 只能通过传入完整的Vector3来修改
this.transform.position = new Vector3(1f, 2f, 3f); //将对象的位置设置为(1,2,3)

//也可以
Vector3 vPos = this.transform.position;
vPos.x = 1f; //通过拿到Vector3的x属性 来修改x轴坐标
this.transform.position = vPos; //再将对象的位置设置为vPos
```

> [!WARNING] 位置修改陷阱
> `this.transform.position.x = 1f;` ==这样直接赋值会编译报错！== 因为 `position` 返回的是 Vector3 的**值拷贝**（struct），必须整体赋值。

```csharp
//对象当前的朝向
Vector3 forward = this.transform.forward; //获取对象的前向向量
Vector3 up = this.transform.up; //获取对象的上向向量
Vector3 right = this.transform.right; //获取对象的右向向量
```

---

### 🚀 位移（Translate）

```csharp
void Update()
{
    float speed = 1f; //速度
    //自己计算
    this.transform.position += this.transform.forward * speed * Time.deltaTime; 

    //利用API
    //参数一：位移的距离 
    //参数二：表示相对坐标系 默认是自己的坐标系
    this.transform.Translate(this.transform.forward * speed * Time.deltaTime, Space.World); //将对象向前进1米 相对于世界坐标系

    //一般使用API进行移动
}
```

| Space 参数 | 参照系 | 说明 |
|:--:|------|------|
| **Space.World** | 世界坐标系 | 无视物体自身旋转，始终沿世界轴移动 |
| **Space.Self**（默认） | 自身坐标系 | 沿物体当前朝向移动 |

---

### 🔁 角度与旋转（Rotation）

```csharp
//获取相对世界坐标的角度
Vector3 worldAngle = this.transform.eulerAngles; //获取对象在世界坐标的角度

//获取相对于父对象的角度
Vector3 localAngle = this.transform.localEulerAngles; //获取对象在父对象坐标的角度

//设置角度与位移同理
```

#### 旋转 API

```csharp
//API计算
//参数一：每一帧旋转的角度
//参数二：表示相对坐标系 默认是自己的坐标系
this.transform.Rotate(new Vector3(1f, 2f, 3f)*Time.deltaTime, Space.World); 

//相对于某个轴转
//参数一：旋转的轴
//参数二：转动的角度
//参数三：表示相对坐标系 默认是自己的坐标系
this.transform.Rotate(Vector3.up, 1f*Time.deltaTime, Space.World); 

//相对于某个点转
//参数一：旋转的点
//参数二：点上的哪一个轴
//参数三：转动的角度
this.transform.RotateAround(Vector3.zero, Vector3.up, 1f*Time.deltaTime); 
```

| 旋转方式 | 说明 | 适用场景 |
|------|------|------|
| **Rotate(角度)** | 绕自身/世界轴旋转 | 物体自转、朝向调整 |
| **Rotate(轴, 角度)** | 绕指定轴旋转 | 绕特定方向旋转 |
| **RotateAround(点, 轴, 角度)** | 绕指定点公转 | 行星公转、围绕目标旋转 |

---

### 📏 缩放与看向

```csharp
public GameObject obj;
void Update()
{
    //获取相对于世界坐标缩放
    Vector3 worldScale = this.transform.localScale; //获取对象在世界坐标缩放（实际上是lossyScale）

    //获取相对于父对象缩放
    Vector3 localScale = this.transform.localScale; //获取对象在父对象坐标缩放

    //注意：
    //1.同样 游戏对象的缩放不能单独改xyz 不过世界坐标系的缩放是不能修改的
    //所以要修改对象的缩放只能修改本地的缩放大小

    //2.Unity没有提供修改缩放相关的API 只能通过修改Vector3来修改
    this.transform.localScale += Vector3.one * Time.deltaTime; //将对象的缩放大小增加1 每帧增加1

    //看向
    //让一个对象的面朝向 可以一直看向某一个点或者某一个对象
    this.transform.LookAt(Vector3.zero); //传入Vector3表示看向点
    this.transform.LookAt(obj.transform); //传入对象的Transform表示看向对象
} 
```

| 属性 | 可读 | 可写 | 说明 |
|------|:--:|:--:|------|
| **localScale** | ✅ | ✅ | 相对于父对象的缩放 |
| **lossyScale** | ✅ | ❌ | 世界坐标系下的实际缩放（只读） |

---

### 🌳 父子关系

#### 获取和设置父对象

```csharp
GameObject obj;
void Start()
{
    //获取父对象
    Transform parent = this.transform.parent; //获取对象的父对象
    //设置父对象
    this.transform.parent = null; //将对象的父对象设置为null
    this.transform.parent = obj.transform; //将对象的父对象设置为obj

    //通过API设置父对象
    //参数一：父对象的Transform
    //参数二：是否保持世界坐标的位置角度缩放信息 
    //       true  会保留世界坐标系下的位置角度缩放信息 并和父对象进行计算再得到本地坐标系的信息
    //       false 不会保留世界坐标系下的位置角度缩放信息 直接将对象的位置角度缩放设置为父对象的位置缩放
    this.transform.SetParent(obj.transform, true); //将对象的父对象设置为obj
}
```

| worldPositionStays | 效果 |
|:--:|------|
| **true** | 保持世界位置不变（自动调整 localPosition 补偿） |
| **false** | 世界位置可能发生变化（localPosition 保持不变） |

#### 子对象操作

```csharp
Transform child;
GameObject obj;
void Start()
{
    //移除子对象
    //将挂载在本对象的所有子对象移除
    this.transform.DetachChildren();

    //获取子对象
    //按名字查找 即使对象失活了也能找到 不过只能找到自己的儿子对象 挂载在自己儿子对象上的对象找不到
    Transform child = this.transform.Find("子对象"); //获取子对象的Transform 效率比GameObject.Find高很多

    //遍历子对象
    for(int i =0;i<this.transform.childCount;i++)
    {
        Transform child = this.transform.GetChild(i); //通过索引号来获取子对象的Transform 如果索引超出范围 会报错
    }

    //子对象判断自己的父对象是否是某个对象
    child.IsChildOf(obj.transform);

    //得到自己作为子对象的编号
    int index = this.transform.GetSiblingIndex(); //获取对象作为子对象的编号

    //把自己设置为第一个子对象
    child.SetAsFirstSibling(); //将对象设置为第一个子对象

    //把自己设置为最后一个子对象
    child.SetAsLastSibling(); //将对象设置为最后一个子对象

    //把自己设置为指定编号的子对象
    child.SetSiblingIndex(1); //将对象设置为第二个子对象 如果索引超出范围 会变为最后一个子对象
}
```

| 查找方式 | 效率 | 能否找到失活对象 | 查找范围 |
|------|:--:|:--:|------|
| **transform.Find("name")** | ⭐⭐⭐ 高 | ✅ 能 | 仅直接子对象 |
| **GameObject.Find("name")** | ⭐ 低 | ❌ 不能 | 全局搜索 |
| **transform.GetChild(i)** | ⭐⭐⭐ 最高 | ✅ 能 | 按索引取子对象 |

---

### 🔀 坐标转换

#### 世界坐标 → 本地坐标

```csharp
//将坐标从世界坐标系转换为本地坐标系
Vector3 localPos = this.transform.InverseTransformPoint(Vector3.zero); //将世界坐标系的点转换为本地坐标系的点

Vector3 localDir = this.transform.InverseTransformDirection(Vector3.right); //将世界坐标系的向量转换为本地坐标系的向量 不受缩放影响
Vector3 localDirScale = this.transform.InverseTransformVector(Vector3.right); //将世界坐标系的向量转换为本地坐标系的向量 受缩放影响
```

#### 本地坐标 → 世界坐标

```csharp
//将坐标从本地坐标系转换为世界坐标系
Vector3 worldPos = this.transform.TransformPoint(Vector3.zero); //将本地坐标系的点转换为世界坐标系的点 受缩放影响

Vector3 worldDir = this.transform.TransformDirection(Vector3.right); //将本地坐标系的向量转换为世界坐标系的向量 不受缩放影响
Vector3 worldDirScale = this.transform.TransformVector(Vector3.right); //将本地坐标系的向量转换为世界坐标系的向量 受缩放影响
```

| 方法 | 转换内容 | 受缩放影响 | 用途 |
|------|:--:|:--:|------|
| **TransformPoint / InverseTransformPoint** | 点坐标 | ✅ 是 | 计算物体相对位置 |
| **TransformDirection / InverseTransformDirection** | 方向向量 | ❌ 否 | 计算物体朝向 |
| **TransformVector / InverseTransformVector** | 向量 | ✅ 是 | 考虑缩放的向量转换 |

---

## 🖱️ Input 输入 & 📺 Screen 屏幕

### ⌨️ Input 输入系统

Input 类用于检测玩家的**键盘、鼠标、触屏、手柄**等输入。

#### 鼠标输入

```csharp
void Update()
{
    //获取鼠标位置
    Vector3 mousePos = Input.mousePosition; //获取鼠标在屏幕上的像素坐标
    
    //检测鼠标输入 0表示左键 1表示右键 2表示中键
    if(Input.GetMouseButtonDown(0)) //如果鼠标点击了左键（按下瞬间）
    {
        //执行操作
    }

    if(Input.GetMouseButtonUp(0)) //如果鼠标松开了左键（松开瞬间）
    {
        //执行操作
    }

    if(Input.GetMouseButton(0)) //如果鼠标按下左键（持续按住）
    {
        //执行操作
    }

    //中键滚动
    Vector3 scroll = Input.mouseScrollDelta; //获取鼠标滚动的距离 y轴表示滚动方向 -1向下 1向上 0不滚动
```

> 三阶段检测模式：

| 方法 | 触发时机 | 适用场景 |
|------|------|------|
| **GetMouseButtonDown** | 按下的那一帧 | 单击、射击、UI 点击 |
| **GetMouseButtonUp** | 松开的那一帧 | 蓄力释放、拖拽结束 |
| **GetMouseButton** | 按住期间每帧 | 连射、拖拽中、持续旋转 |

#### 键盘输入

```csharp
    //检测键盘按键是否按下
    if(Input.GetKeyDown(KeyCode.Space)) //如果按下了空格键
    {
        //执行操作
    }

    if(Input.GetKeyUp(KeyCode.Space)) //如果松开了空格键
    {
        //执行操作
    }

    if(Input.GetKey(KeyCode.Space)) //如果按下了空格键 当空格键被按下时会一直进入这个判断
    {
        //执行操作
    }
```

#### 轴输入

```csharp
    //检测默认轴输入
    float axisH = Input.GetAxis("Horizontal"); //获取水平轴的输入 -1到1之间（平滑过渡）
    float axisV = Input.GetAxis("Vertical"); //获取垂直轴的输入 -1到1之间（平滑过渡）

    float mAxisH = Input.GetAxis("Mouse X"); //获取鼠标水平移动量
    float mAxisV = Input.GetAxis("Mouse Y"); //获取鼠标垂直移动量

    //如果是使用Input.GetAxisRaw() 会返回0 1 -1 而不是-1到1之间的浮点数（无平滑）
```

| 方法 | 返回值 | 特点 |
|------|------|------|
| **GetAxis()** | -1 ~ 1 浮点数 | ==平滑过渡==，适合移动控制 |
| **GetAxisRaw()** | -1, 0, 1 整数 | ==无平滑==，适合格斗游戏等需要精确输入的场合 |

#### 其他输入

```csharp
    //是否有任意键按住
    if(Input.anyKey)
    {
        //执行操作
    }

    //是否有任意键被按下（该帧按下）
    if(Input.anyKeyDown)
    {
        //执行操作
    }

    //这一帧的键盘输入字符串
    string key = Input.inputString;

    //手柄输入
    //得到手柄所有按键名
    string[] keys = Input.GetJoystickNames();

    //手柄A键被按下
    if(Input.GetButtonDown("A"))
    {
        //执行操作
    }

    //手柄A键被松开
    if(Input.GetButtonUp("A"))
    {
        //执行操作
    }

    //手柄A键长按
    if(Input.GetButton("A"))
    {
        //执行操作
    }
```

> [!NOTE] Input Manager 配置
> 按键映射（"Horizontal", "Vertical", "A" 等）在 `Edit → Project Settings → Input Manager` 中配置，可自定义轴名称、按键绑定和灵敏度。

#### 移动设备触摸

```csharp
    //移动设备触摸
    if(Input.touchCount > 0)
    {
        Touch t1 = Input.touches[0];

        //位置
        Vector3 touchPos = t1.position;
        //相对于上次位置的位移
        Vector3 touchDelta = t1.deltaPosition;
    }

    //是否启用多点触碰
    Input.multiTouchEnabled = true;

    //陀螺仪相关
    //是否开启陀螺仪
    Input.gyro.enabled = true;

    //重力加速度向量
    Vector3 gravity = Input.gyro.gravity;

    //旋转速度
    Vector3 gyroRotation = Input.gyro.rotationRate;

    //陀螺仪当前旋转的四元数
    Quaternion gyroAttitude = Input.gyro.attitude;
}
```

---

### 📺 屏幕相关

```csharp
void Start()
{
    //获取屏幕分辨率
    Resolution res = Screen.currentResolution;

    //获取屏幕宽高
    //得到的是窗口的宽高 而不是设备分辨率的宽高
    int screenWidth = Screen.width;
    int screenHeight = Screen.height;

    //屏幕休眠模式
    Screen.sleepTimeout = SleepTimeout.NeverSleep;
    //如果设置为SleepTimeout.NeverSleep 则屏幕不会休眠

    //运行时是否设置为全屏
    Screen.fullScreen = true;

    //独占全屏  FullScreenMode.ExclusiveFullScreen
    //全屏窗口  FullScreenMode.FullScreenWindow
    //最大化窗口  FullScreenMode.MaximizedWindow
    //窗口模式  FullScreenMode.Windowed
    Screen.fullScreenMode = FullScreenMode.Windowed;
```

| 全屏模式 | 说明 |
|------|------|
| **ExclusiveFullScreen** | 独占全屏（性能最优） |
| **FullScreenWindow** | 全屏窗口（无边框窗口，切换快） |
| **MaximizedWindow** | 最大化窗口 |
| **Windowed** | 窗口模式 |

```csharp
    //移动设备屏幕相关
    //允许自动旋转为左横向 Home键在左
    Screen.autorotateToLandscapeLeft = true;
    //允许自动旋转为右横向 Home键在右
    Screen.autorotateToLandscapeRight = true;
    //允许自动旋转为纵向 Home键在上
    Screen.autorotateToPortrait = true;
    //允许自动旋转为纵向 Home键在下
    Screen.autorotateToPortraitUpsideDown = true;

    //指定屏幕显示方向
    Screen.orientation = ScreenOrientation.Portrait; //固定为纵向

    //设置分辨率 一般移动设备不使用
    Screen.SetResolution(1280, 720, false); //false表示不设置为全屏模式
}
```

---

## 📷 Camera 摄像机

Camera 是 Unity 中玩家"眼睛"的载体，==场景中的内容必须通过 Camera 才能渲染到屏幕上==。

### 📌 重要静态成员

```csharp
void Start()
{
    //获取主相机
    //如果想通过这种方式获取主相机 请确保场景中只有一个主相机 且主相机的tag为MainCamera
    Camera mainCamera = Camera.main;
    //获取摄像机数量
    int cameraCount = Camera.allCamerasCount;
    //得到所有摄像机
    Camera[] cameras = Camera.allCameras;

    //渲染相关委托
    //摄像机剔除前处理的委托函数
    Camera.onPreCull += (c) =>
    {
        //执行操作
    };

    Camera.onPreRender += (c) =>
    {   
        //执行操作
    };

    Camera.onPostRender += (c) =>
    {
        //执行操作
    };
}
```

| 渲染回调 | 执行时机 |
|------|------|
| **onPreCull** | 摄像机剔除前 |
| **onPreRender** | 摄像机开始渲染前 |
| **onPostRender** | 摄像机渲染完成后 |

### ⚙️ 重要成员

```csharp
void Start()
{
    //界面上的参数都可以在Camera中获取到
    //例如
    Camera.main.depth = 10; //得到主摄像机的深度进行设置
```

#### 坐标转换

```csharp
    //世界坐标转屏幕坐标
    //返回的是一个Vector3 包含x y z坐标 
    //x y值代表对象在屏幕上的像素位置
    //z值代表对象离摄像机的距离
    Vector3 screenPos = Camera.main.WorldToScreenPoint(transform.position); 

    //屏幕坐标转世界坐标
    //需要改变z轴的值 因为z值代表对象离摄像机的距离
    //如果z值为0 则对象一直处于摄像机的原点上
    //给z赋值后 可以理解为物体会存在于距离摄像机z值距离的面上
    Vector3 v = Input.mousePosition;
    v.z = 10;
    Vector3 worldPos = Camera.main.ScreenToWorldPoint(v);
}
```

> [!IMPORTANT] 屏幕坐标转世界坐标的关键
> `Input.mousePosition` 的 z 值始终为 0，==必须手动设置 z 值==才能用 `ScreenToWorldPoint` 正确转换，z 值代表距离摄像机的深度距离。

---

## 💡 光源 Light

光源组件用于为场景提供照明，影响物体的明暗、阴影和氛围。

### 光源模式（Mode）

| 模式 | 运算方式 | 效果 | 性能 |
|------|------|------|:--:|
| **Realtime** | 每帧实时计算 | 动态光影，可实时变化 | 🔴 高消耗 |
| **Baked** | 预先烘焙到光照贴图 | 静态光照，无法动态变化 | 🟢 低消耗 |
| **Mixed** | 实时 + 烘焙混合 | 静态物体用烘焙，动态物体用实时 | 🟡 中消耗 |

> [!TIP] 性能优化建议
> - 场景中尽量用 **Baked** 模式处理不动的静态物体光照
> - 仅对需要实时变化的光源使用 **Realtime**
> - **Mixed** 是平衡性能和效果的好选择

---

## 📡 UnityAction 事件委托

> `UnityAction` 是 Unity 事件系统（`UnityEngine.Events`）中的**泛型委托类型**，是所有 UGUI 交互事件（Button.onClick、Slider.onValueChanged、Toggle.onValueChanged、InputField.onValueChanged 等）的**回调签名基础**。

> [!IMPORTANT] 一句话理解
> `UnityAction` = C# `Action` 的 Unity 版本，专门为 UnityEvent 系统设计，支持**0~3 个参数**的泛型重载。

---

### 📋 UnityAction 与 C# Action 对比

| 对比维度 | C# `Action` | Unity `UnityAction` |
|------|:--:|:--:|
| 命名空间 | `System` | `UnityEngine.Events` |
| 参数数量 | 0~16 个 | 0~3 个 |
| 序列化 | ❌ 不可序列化 | ✅ 可在 Inspector 中序列化显示 |
| UnityEvent 兼容 | ❌ 不能直接用于 UnityEvent | ✅ 专为 UnityEvent 设计 |
| 用途 | 通用委托 | UI 事件回调（Button / Slider / Toggle 等） |

---

### 声明与定义

UnityAction 有 **4 种泛型重载**：

```csharp
namespace UnityEngine.Events
{
    // 无参数
    public delegate void UnityAction();

    // 1 个参数
    public delegate void UnityAction<T0>(T0 arg0);

    // 2 个参数
    public delegate void UnityAction<T0, T1>(T0 arg0, T1 arg1);

    // 3 个参数
    public delegate void UnityAction<T0, T1, T2>(T0 arg0, T1 arg1, T2 arg2);
}
```

| 重载 | 签名 | 典型使用场景 |
|------|------|------|
| `UnityAction` | `void ()` | `Button.onClick`、`EventTrigger` |
| `UnityAction<T0>` | `void (T0)` | `Slider.onValueChanged(float)`、`Toggle.onValueChanged(bool)`、`InputField.onValueChanged(string)` |
| `UnityAction<T0, T1>` | `void (T0, T1)` | 自定义 UnityEvent<string, int> |
| `UnityAction<T0, T1, T2>` | `void (T0, T1, T2)` | 自定义 UnityEvent 三参（较少用） |

---

### 💻 使用方式

#### 方式1：作为方法直接绑定（最常用）

```csharp
using UnityEngine;
using UnityEngine.UI;
using UnityEngine.Events;

public class UnityActionDemo : MonoBehaviour
{
    public Button button;
    public Slider slider;
    public Toggle toggle;
    public InputField inputField;

    void Start()
    {
        // UnityAction（无参）→ Button.onClick
        button.onClick.AddListener(OnButtonClick);

        // UnityAction<float> → Slider.onValueChanged
        slider.onValueChanged.AddListener(OnSliderValueChanged);

        // UnityAction<bool> → Toggle.onValueChanged
        toggle.onValueChanged.AddListener(OnToggleValueChanged);

        // UnityAction<string> → InputField.onValueChanged
        inputField.onValueChanged.AddListener(OnInputValueChanged);
    }

    void OnButtonClick() { }
    void OnSliderValueChanged(float value) { }
    void OnToggleValueChanged(bool isOn) { }
    void OnInputValueChanged(string text) { }
}
```

#### 方式2：显式声明 UnityAction 变量

```csharp
using UnityEngine;
using UnityEngine.UI;
using UnityEngine.Events;

public class UnityActionVariableDemo : MonoBehaviour
{
    public Button button;
    public Slider slider;

    void Start()
    {
        // 无参 UnityAction 的多种声明写法
        UnityAction clickAction1 = new UnityAction(OnButtonClick);
        UnityAction clickAction2 = OnButtonClick;          // 隐式转换
        UnityAction clickAction3 = delegate { Debug.Log("匿名方法"); };
        UnityAction clickAction4 = () => Debug.Log("Lambda");

        button.onClick.AddListener(clickAction1);

        // 带参数 UnityAction<T> 的多种声明写法
        UnityAction<float> sliderAction1 = new UnityAction<float>(OnSliderChanged);
        UnityAction<float> sliderAction2 = OnSliderChanged;
        UnityAction<float> sliderAction3 = delegate(float val) { Debug.Log(val); };
        UnityAction<float> sliderAction4 = (float val) => Debug.Log(val);

        slider.onValueChanged.AddListener(sliderAction1);
    }

    void OnButtonClick() { }
    void OnSliderChanged(float value) { }
}
```

#### 方式3：动态组合多个 UnityAction（多播委托）

```csharp
using UnityEngine;
using UnityEngine.UI;
using UnityEngine.Events;

public class UnityActionCombineDemo : MonoBehaviour
{
    public Button button;

    void Start()
    {
        UnityAction action1 = () => Debug.Log("播放音效");
        UnityAction action2 = () => Debug.Log("播放动画");
        UnityAction action3 = () => Debug.Log("发送消息");

        // C# 委托 += 组合（多播委托）
        UnityAction combined = action1 + action2 + action3;
        button.onClick.AddListener(combined);

        // 移除其中一个
        combined -= action2; // 现在只剩下 action1 + action3
        // 注意：此时按钮上绑定的仍然是 3 个监听
        // 需要 RemoveListener + AddListener 才能更新按钮
    }
}
```

#### 方式4：UnityAction 作为参数传递（封装通用方法）

```csharp
using UnityEngine;
using UnityEngine.UI;
using UnityEngine.Events;

public class UnityActionParameterDemo : MonoBehaviour
{
    public Button btn1, btn2, btn3;
    public Slider slider1;

    // 封装通用的按钮设置方法
    public void SetupButton(Button btn, UnityAction onClickCallback)
    {
        btn.onClick.RemoveAllListeners();
        btn.onClick.AddListener(onClickCallback);
    }

    // 封装带参数的 Slider 设置方法
    public void SetupSlider(Slider sld, UnityAction<float> onValueChangedCallback)
    {
        sld.onValueChanged.RemoveAllListeners();
        sld.onValueChanged.AddListener(onValueChangedCallback);
    }

    void Start()
    {
        SetupButton(btn1, () => Debug.Log("按钮1"));
        SetupButton(btn2, () => Debug.Log("按钮2"));
        SetupButton(btn3, OnComplexAction);
        SetupSlider(slider1, (float v) => UpdateVolume(v));
    }

    void OnComplexAction() { }
    void UpdateVolume(float volume) { }
}
```

---

### 🔗 UnityAction 与 UnityEvent 的关系

> `UnityEvent` 内部使用 `UnityAction` 作为回调的存储和调用机制。

```csharp
// UnityEvent 的简化源码结构
public class UnityEvent : UnityEventBase
{
    public void AddListener(UnityAction call) { ... }
    public void RemoveListener(UnityAction call) { ... }
    public void Invoke() { ... }
}

// UnityEvent<T0> 使用 UnityAction<T0>
public class UnityEvent<T0> : UnityEventBase
{
    public void AddListener(UnityAction<T0> call) { ... }
    public void RemoveListener(UnityAction<T0> call) { ... }
    public void Invoke(T0 arg0) { ... }
}
```

| UnityEvent 类型 | 内部用的 UnityAction | 常见组件 |
|------|------|------|
| `UnityEvent` | `UnityAction` | `Button.onClick`、`EventTrigger` |
| `UnityEvent<float>` | `UnityAction<float>` | `Slider.onValueChanged`、`Scrollbar.onValueChanged` |
| `UnityEvent<bool>` | `UnityAction<bool>` | `Toggle.onValueChanged` |
| `UnityEvent<string>` | `UnityAction<string>` | `InputField.onValueChanged`、`InputField.onEndEdit` |
| `UnityEvent<int>` | `UnityAction<int>` | `Dropdown.onValueChanged` |
| `UnityEvent<Vector2>` | `UnityAction<Vector2>` | `ScrollRect.onValueChanged` |

---

### 🎯 与 Lambda 的配合

```csharp
using UnityEngine;
using UnityEngine.UI;
using UnityEngine.Events;

public class UnityActionLambdaDemo : MonoBehaviour
{
    public Button button;
    public Slider slider;
    public Toggle toggle;

    void Start()
    {
        // Lambda 会被编译器隐式转换为 UnityAction
        button.onClick.AddListener(() => {
            Debug.Log("Lambda -> UnityAction 隐式转换");
        });

        // Lambda -> UnityAction<float>
        slider.onValueChanged.AddListener((float val) => {
            Debug.Log(string.Format("Slider: {0:F2}", val));
        });

        // Lambda -> UnityAction<bool>
        toggle.onValueChanged.AddListener((bool isOn) => {
            Debug.Log(isOn ? "开启" : "关闭");
        });

        // 闭包捕获外部变量
        int clickCount = 0;
        button.onClick.AddListener(() => {
            clickCount++;
            Debug.Log(string.Format("点击了 {0} 次", clickCount));
        });
    }
}
```

> [!TIP] Lambda 本质
> `() => { ... }` 这个 lambda 表达式**被编译器自动转换为** `UnityAction` 委托实例。`button.onClick.AddListener(() => { ... })` 等价于先创建 UnityAction 再传入。

---

### 🔧 自定义 UnityEvent（配合 UnityAction）

```csharp
using UnityEngine;
using UnityEngine.Events;
using System;

// 第1步：继承 UnityEvent，指定参数类型
[Serializable]
public class MyStringIntEvent : UnityEvent<string, int> { }

public class CustomEventDemo : MonoBehaviour
{
    // 第2步：声明为 public → 在 Inspector 中可拖拽绑定
    public MyStringIntEvent onDataReceived;

    void Start()
    {
        // 第3步：代码绑定 UnityAction<string, int>
        onDataReceived.AddListener(OnDataReceived);
        onDataReceived.AddListener((string msg, int code) => {
            Debug.Log(string.Format("[{0}] {1}", code, msg));
        });
    }

    void OnDataReceived(string message, int code)
    {
        Debug.Log(string.Format("收到: {0}, 代码: {1}", message, code));
    }

    void Update()
    {
        if (Input.GetKeyDown(KeyCode.Space))
        {
            onDataReceived.Invoke("测试消息", 200); // 手动触发
        }
    }
}
```

> 自定义 UnityEvent 三步：① 继承 `UnityEvent<T0, T1, ...>` → ② 加 `[Serializable]` → ③ 声明为 `public`。之后既可在 Inspector 拖拽绑定，也可代码 `AddListener`。

---

### 📊 常用 UGUI 事件与对应 UnityAction 速查

| 组件 | 事件属性 | UnityAction 类型 | 触发时机 |
|------|------|------|------|
| **Button** | `onClick` | `UnityAction` | 点击时 |
| **Toggle** | `onValueChanged` | `UnityAction<bool>` | 勾选状态变化时 |
| **Slider** | `onValueChanged` | `UnityAction<float>` | 值变化时（拖动中每帧触发） |
| **Scrollbar** | `onValueChanged` | `UnityAction<float>` | 值变化时 |
| **InputField** | `onValueChanged` | `UnityAction<string>` | 文本内容变化时 |
| **InputField** | `onEndEdit` | `UnityAction<string>` | 编辑结束（回车或失焦） |
| **Dropdown** | `onValueChanged` | `UnityAction<int>` | 选中项变化时 |
| **ScrollRect** | `onValueChanged` | `UnityAction<Vector2>` | 滚动位置变化时 |

> [!NOTE] 命名空间
> `UnityAction` 和 `UnityEvent` 都位于 `UnityEngine.Events` 命名空间，使用前需要 `using UnityEngine.Events;`。
