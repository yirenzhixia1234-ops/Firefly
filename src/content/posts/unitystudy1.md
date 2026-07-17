---
title: Unity学习笔记1
published: 2026-07-14
pinned: false
description: Unity学习笔记1 - Unity编辑器、GameObject、组件系统等基础
tags: [Unity,学习]
author: boluobao
draft: false
---

# Unity学习笔记1

## GameObject

### GameObject的成员变量

```csharp
this.gameObject.name; // 获取游戏对象的名称

this.gameObject.activeSelf; // 获取游戏对象是否激活

this.gameObject.isStatic; // 获取游戏对象是否为静态对象

this.gameObject.layer; // 获取游戏对象的图层

this.gameObject.tag; // 获取游戏对象的标签名

this.gameObject.transform; // 获取游戏对象的Transform组件 但是不如this.transform方便
```

### GameObject的静态方法

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

//还有几个方法用的比较少 是GameObject父类 Object提供的方法
//Unity里面的Object和万物之父object不同
//Unity里的Object类在UnityEngine命名空间下 Object类是集成了object类的一个自定义类
//C#的Object类在System命名空间下 Object类是C#的万物之父类
GameObject.FindObjectOfType<GameObject>(); //找到场景中挂载有某个脚本的游戏对象


//克隆游戏对象
GameObject cubeClone = GameObject.Instantiate(cube); // 克隆一个游戏对象 如果类继承了MonoBehaviour 则直接用Instantiate方法即可

//删除游戏对象
GameObject.Destroy(cubeClone); // 删除一个游戏对象
GameObject.Destroy(cubeClone, 2f); // 删除一个游戏对象 2秒后删除
GameObject.Destroy(this); //删除对象上的脚本
//Destroy 不会马上移除游戏对象 只是给这个游戏对象加了一个移除标识
//一般情况下他会在游戏的下一帧将这个对象移除并从内存中删除

GameObject.DestroyImmediate(cubeClone); // 立即删除游戏对象

//切换场景不移除游戏对象
//如果在场景切换时 不希望游戏对象被销毁 可以使用DontDestroyOnLoad方法
GameObject.DontDestroyOnLoad(cubeClone); // 不在场景切换时销毁游戏对象
```

### GameObject的成员方法

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

## 时间相关Time

时间相关内容主要用于游戏中参与计时、位移、时间暂停等逻辑

### 时间缩放比例

```csharp
Time.timeScale = 0.5f; // 时间缩放比例 0.5倍
Time.timeScale = 1f; // 时间缩放比例 1倍 正常时间
Time.timeScale = 2f; // 时间缩放比例 2倍 快进时间
Time.timeScale = 0f; // 时间缩放比例 0倍 暂停时间
```

### 帧间隔时间

```csharp

//帧间隔时间 主要用于计算位移
//路程 = 速度 * 时间间隔
//根据需求选择参与计算的间隔时间
//如果希望 游戏暂停就不动的 那就使用deltaTime
//如果希望 不受timeScale影响的 那就使用unscaledDeltaTimeTime

//最近的一帧用了多长时间 单位为秒
float deltaTime = Time.deltaTime; //受timeScale影响

float uncaledDeltaTime = Time.unscaledDeltaTime; //不受timeScale影响

```

### 游戏开始到现在的时间

```csharp
//获取游戏开始到现在的时间
float time = Time.time; //主要用于计时 受timeScale影响

float uncaledTime = Time.unscaledTime; //不受timeScale影响
```

### 物理帧间隔时间

```csharp
//物理帧间隔时间 主要用于计算物理模拟
//物理模拟 = 物理力 * 物理时间间隔
//根据需求选择参与计算的物理帧间隔时间
//unity中物理模拟间隔时间默认是0.02f 单位为秒 FixedUpdate方法调用
float physicsDeltaTime = Time.fixedDeltaTime; //物理帧间隔时间 受timeScale影响

float uncaledPhysicsDeltaTime = Time.unscaledFixedDeltaTime; //不受timeScale影响
```

### 帧数

从游戏开始到现在跑了多少次循环  
```csharp
int frameCount = Time.frameCount; //获取游戏开始到现在跑了多少次循环
```

## Transform

### Vector3

Vector3主要用来表示三维空间中的一个点或者一个向量   

```csharp
//申明
Vector3 pos = new Vector3();
pos.x = 1f; //x轴坐标
pos.y = 2f; //y轴坐标
pos.z = 3f; //z轴坐标
//或
pos = new Vector3(1f, 2f, 3f); //直接赋值
Vector3 pos2 = new Vector3(1f,2f); //不传z轴坐标 默认是0 常用于2D游戏

//Vector3的基本运算
//加法
Vector3 pos3 = pos + new Vector3(1f, 2f, 3f); //将pos3的坐标设置为pos的坐标加上(1,2,3)
//减法
Vector3 pos4 = pos - new Vector3(1f, 2f, 3f); //将pos4的坐标设置为pos的坐标减去(1,2,3)
//乘法
Vector3 pos5 = pos * 2f; //将pos5的坐标设置为pos的坐标乘以2
//除法
Vector3 pos6 = pos / 2f; //将pos6的坐标设置为pos的坐标除以2

//常用向量
Vector3.zero; //0,0,0
Vector3.one; //1,1,1
Vector3.right; //1,0,0
Vector3.up; //0,1,0
Vector3.forward; //0,0,1
Vector3.back; //0,0,-1

//计算两个点之间的距离
float distance = Vector3.Distance(pos, pos2); //计算pos和pos2之间的距离
```

### 位置

```csharp
//获取相对于世界坐标的位置的位置
Vector3 worldPos = this.transform.position; //获取对象在世界坐标的位置

//获取相对于父对象的位置的位置
Vector3 localPos = this.transform.localPosition; //获取对象在父对象坐标的位置

//位置的修改
//位置的赋值不能单个修改xyz某一个值 只能通过传入完整的Vector3来修改
this.transform.position = new Vector3(1f, 2f, 3f); //将对象的位置设置为(1,2,3)

//也可以
Vector3 vPos = this.transform.position;
vPos.x = 1f; //通过拿到Vector3的x属性 来修改x轴坐标
this.transform.position = vPos; //再将对象的位置设置为vPos

//对象当前的朝向
Vector3 forward = this.transform.forward; //获取对象的前向向量
Vector3 up = this.transform.up; //获取对象的上向向量
Vector3 right = this.transform.right; //获取对象的右向向量
```

### 位移

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

### 角度

```csharp
//获取相对世界坐标的角度
Vector3 worldAngle = this.transform.eulerAngles; //获取对象在世界坐标的角度

//获取相对于父对象的角度
Vector3 localAngle = this.transform.localEulerAngles; //获取对象在父对象坐标的角度

//设置角度与位移同理

//API计算
//参数一：每一帧旋转的角度
//参数二：表示相对坐标系 默认是自己的坐标系
this.transform.Rotate(new Vector3(1f, 2f, 3f)*Time.deltaTime, Space.World); //将对象向世界坐标系的x轴旋转1度 向y轴旋转2度 向z轴旋转3度

//相对于某个轴转
//参数一：旋转的轴
//参数二：转动的角度
//参数三：表示相对坐标系 默认是自己的坐标系
this.transform.Rotate(Vector3.up, new Vector3(1f, 2f, 3f), 1f*Time.deltaTime); 

//相对于某个点转
//参数一：旋转的点
//参数二：点上的哪一个轴
//参数三：转动的角度
this.transform.RotateAround(Vector3.zero, Vector3.up, 1f*Time.deltaTime); 
```

### 缩放和看向

```csharp
public GameObject obj;
void Update()
{
    //获取相对于世界坐标缩放
    Vector3 worldScale = this.transform.localScale; //获取对象在世界坐标缩放

    //获取相对于父对象缩放
    Vector3 localScale = this.transform.localScale; //获取对象在父对象坐标缩放

    //注意：
    //1.同样 游戏对象的缩放不能单独改xyz 不过世界坐标系的缩放是不能修改的
    //所以要修改对象的缩放只能修改本地的缩放大小

    //2.Unity没有提供修改缩放相关的API 只能通过修改Vector3来修改
    this.transform.localScale += Vector.one * Time.deltaTime; //将对象的缩放大小增加1 每帧增加1

    //看向
    //让一个对象的面朝向 可以一直看向某一个点或者某一个对象
    this.transform.LookAt(Vector3.zero); //传入Vector3表示看向点
    this.transform.LookAt(obj.transform); //传入对象的Transform表示看向对象
} 
```

### 父子关系

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
    //       true 会保留 世界坐标系下的位置角度缩放信息 并和父对象进行计算再得到本地坐标系的信息
    //       false 不会保留 世界坐标系下的位置角度缩放信息 直接将对象的位置角度缩放设置为父对象的位置角度缩放
    this.transform.SetParent(obj.transform, true); //将对象的父对象设置为obj
}
```

#### 子对象
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

### 坐标转换

#### 世界坐标转本地坐标

```csharp
//将坐标从世界坐标系转换为本地坐标系
Vector3 localPos = this.transform.InverseTransformPoint(Vector3.zero); //将世界坐标系的点转换为本地坐标系的点

Vector3 localDir = this.transform.InverseTransformDirection(Vector3.right); //将世界坐标系的向量转换为本地坐标系的向量 不受缩放影响
Vector3 localDirScale = this.transform.InverseTransformDirection(Vector3.right); //将世界坐标系的向量转换为本地坐标系的向量 受缩放影响
```

#### 本地坐标转世界坐标

```csharp
//将坐标从本地坐标系转换为世界坐标系
Vector3 worldPos = this.transform.TransformPoint(Vector3.zero); //将本地坐标系的点转换为世界坐标系的点 受缩放影响

Vector3 worldDir = this.transform.TransformDirection(Vector3.right); //将本地坐标系的向量转换为世界坐标系的向量 不受缩放影响
Vector3 worldDirScale = this.transform.TransformVector(Vector3.right); //将本地坐标系的向量转换为世界坐标系的向量 受缩放影响
```

## Input和Screen

### Input输入

```csharp
void Update()
{
    //获取鼠标点击位置
    Vector3 mousePos = Input.mousePosition; //获取鼠标点击位置
    
    //检测鼠标输入 0表示左键 1表示右键 2表示中键
    if(Input.GetMouseButtonDown(0)) //如果鼠标点击了左键
    {
        //执行操作
    }

    if(Input.GetMouseButtonUp(0)) //如果鼠标松开了左键
    {
        //执行操作
    }

    if(Input.GetMouseButton(0)) //如果鼠标按下左键 当鼠标长按不放时会一直进入这个判断
    {
        //执行操作
    }

    //中键滚动
    Vector3 scroll = Input.mouseScrollDelta; //获取鼠标滚动的距离 y轴表示滚动方向 -1向下 1向上 0不滚动

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

    //检测默认轴输入
    float axisH = Input.GetAxis("Horizontal"); //获取水平轴的输入 -1到1之间
    float axisV = Input.GetAxis("Vertical"); //获取垂直轴的输入 -1到1之间

    float mAxisH = Input.GetAxis("Mouse X"); //获取鼠标水平轴的输入 -1到1之间
    float mAxisV = Input.GetAxis("Mouse Y"); //获取鼠标垂直轴的输入 -1到1之间

    //如果是使用Input.GetAxisRaw() 会返回0 1 -1 而不是-1到1之间的浮点数

    //是否有任意键长按
    if(Input.anyKey)
    {
        //执行操作
    }

    //是否有任意键被按下
    if(Input.anyKeyDown)
    {
        //执行操作
    }

    //这一帧的键盘输入
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
    Vector3 gravity = Input.acceleration.gravity;

    //旋转速度
    Vector3 gyroRotation = Input.gyro.rotationRate;

    //陀螺仪当前旋转的四元数
    Quaternion gyroRotation = Input.gyro.attitude;
}
```

### 屏幕相关

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
    //全屏窗口  FullScreenMode.FullScreen
    //最大化窗口  FullScreenMode.MaximizedFullScreen
    //窗口模式  FullScreenMode.Windowed
    Screen.fullScreenMode = FullScreenMode.Windowed;

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

## Camera

### 重要静态成员
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

### 重要成员

```csharp
void Start()
{
    //界面上的参数都可以在Camera中获取到
    //例如
    Camera.main.depth = 10; //得到主摄像机的深度进行设置

    //世界坐标转屏幕坐标
    //返回的是一个Vector3 含量的x y z坐标 
    //x y值代表对象在屏幕上的位置
    //z值代表对象离摄像机的距离
    Vector3 screenPos = Camera.main.WorldToScreenPoint(transform.position); 

    //屏幕坐标转世界坐标
    //需要改变z轴的值 因为z值代表对象离摄像机的距离
    //如果z值为0 则对象一直处于摄像机的原点上
    //给z赋值后 可以理解为物体会存在于距离摄像机z值的距离的面上
    Vector3 v = Input.mousePosition;
    v.z = 10;
    Vector3 worldPos = Camera.main.ScreenToWorldPoint(v);

}
```

## 光源

### 光源模式(Mode)

Realtime 实时模式 每帧实时计算，效果好，性能消耗大  
Baked 烘焙光源 事先计算好，无法动态变化 
Mixed 混合光源 实时计算+事先计算    