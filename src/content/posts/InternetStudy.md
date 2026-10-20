---
title: unity网络开发学习笔记
published: 2026-09-03
pinned: false
description: unity网络开发学习笔记
tags: [网络开发,学习,unity]
author: boluobao
draft: false
---

# unity网络开发学习笔记

## 网络基本概念
- 1. 网络：由若干设备和连接这些设备的链路构成，设备间可以相互通信
- 2. 局域网：指某一个小区域内由多台设备互联成的计算机组
- 3. 以太网：网络连接的一种规则，定义了连接传输规范
- 4. 城域网：是在一个城市范围内所建立的网络，几十到一百公里
- 5. 广域网：是连接不同地区、城市、国家的远程网络，几十到几千公里
- 6. 互联网（因特网）：是目前国际上最大的互联网，定义了通信规则等
- 7. 万维网：是基于因特网的网站和网页的统称

## Ip端口和Mac地址
- 1. IP地址：是网络中设备的唯一标识，由32位二进制数组成，分为四段，每段8位，用点号分隔，如192.168.1.1
- 2. 端口：是设备上用于区分不同应用程序的标识，由16位二进制数组成，范围0-65535，如80端口用于HTTP协议，443端口用于HTTPS协议
- 3. Mac地址：是网络设备上用于唯一标识设备的物理地址，由48位二进制数组成，由16进制数表示，如00:0c:29:8a:5a:5a

## 客户端和服务端
- 1. 客户端：指用户使用的设备，如手机、电脑等运行使用的应用程序，用于发送请求和接收响应 
- 2. 服务端：服务端应用程序运行在远端的一台计算机上，客户端通过网络和服务端进行通讯，服务端为客户端提供各种服务 

## 数据通信模型

- 1.C/S模式：客户端/服务端模式，客户端向服务端发送请求，服务端处理请求并返回响应，客户端接收响应并显示结果

## 网络协议

### OSI模型

- 1. OSI模型：开放式系统互联模型，将网络通信分为七层，从上到下分别是应用层、表示层、会话层、传输层、网络层、数据链路层、物理层
- 2. 应用层：提供应用程序和网络之间的接口，如HTTP、FTP、SMTP等
- 3. 表示层：负责数据格式的转换和加密，如ASCII、JPEG、MPEG等
- 4. 会话层：负责建立、管理和终止会话，如TCP、UDP等
- 5. 传输层：负责数据的传输和可靠性的保证，如TCP、UDP等
- 6. 网络层：负责数据在网络中的路由和转发，如IP、ICMP等
- 7. 数据链路层：负责数据在链路中的传输和错误检测，如以太网、PPP等
- 8. 物理层：负责数据在物理介质上的传输，如光纤、电缆、无线电波等

### TCP/IP协议
- 1. TCP/IP协议：是互联网通信的基础协议，将网络通信分为四层，从上到下分别是应用层、传输层、网络层、数据链路层
- 2. 应用层：提供应用程序和网络之间的接口，如HTTP、FTP、SMTP等
- 3. 传输层：负责数据的传输和可靠性的保证，如TCP、UDP等
- 4. 网络层：负责数据在网络中的路由和转发，如IP、ICMP等
- 5. 数据链路层：负责数据在链路中的传输和错误检测，如以太网、PPP等

### TCP和UDP协议

- 1. TCP协议：传输控制协议，是一种面向连接的、可靠的、基于字节流的传输层通信协议，适用于需要可靠传输的应用场景，如文件传输、邮件发送等
- 2. UDP协议：用户数据报协议，是一种无连接的、不可靠的、基于数据报的传输层通信协议，适用于需要快速传输的应用场景，如视频聊天、实时游戏等

#### TCP 三次握手和四次挥手

**三次握手（建立连接）**

- 第一次握手：客户端向服务端发送 SYN=1，seq=x（自己的初始序号），表示"我要建立连接"，客户端进入 SYN-SENT（同步已发送）状态
- 第二次握手：服务端收到后回复 SYN=1，ACK=1，seq=y，ack=x+1，表示"收到你的请求，同意建立连接"，服务端进入 SYN-RCVD（同步已接收）状态
- 第三次握手：客户端收到后回复 ACK=1，ack=y+1，双方确认后都进入 ESTABLISHED（已建立连接）状态，连接建立成功

> 为什么必须是三次：① 确保双方都能正常发送和接收数据（前两次只能证明服务端收发正常，第三次才证明客户端收发正常）；② 防止"已失效的连接请求"突然到达服务端，导致服务端误以为要建立连接而白白占用资源

**四次挥手（断开连接，以客户端主动关闭为例）**

- 第一次挥手：客户端向服务端发送 FIN=1，seq=u，表示"我的数据发完了，请求断开连接"，客户端进入 FIN-WAIT-1（终止等待1）状态
- 第二次挥手：服务端收到后回复 ACK=1，ack=u+1，表示"收到你的断开请求，但我可能还有数据要发"，服务端进入 CLOSE-WAIT（关闭等待）状态；客户端收到 ACK 后进入 FIN-WAIT-2（终止等待2）状态
- 第三次挥手：服务端把剩余数据发完后，向客户端发送 FIN=1，seq=v，表示"我的数据也发完了，可以断开了"，服务端进入 LAST-ACK（最后确认）状态
- 第四次挥手：客户端收到后回复 ACK=1，ack=v+1，进入 TIME-WAIT（时间等待）状态，等待 2MSL 时间后关闭；服务端收到 ACK 后直接进入 CLOSED（关闭）状态

> 为什么必须是四次：TCP 是全双工通信，两个方向（客户端→服务端、服务端→客户端）的数据通道需要**分别关闭**。被动关闭方（服务端）收到 FIN 后可能还有数据要发，所以 ACK（第二次挥手）和 FIN（第三次挥手）必须分两次发送，不能合并，因此比三次握手多一次

#### UDP协议
- 1. UDP协议：用户数据报协议，是一种无连接的、不可靠的、基于数据报的传输层通信协议，适用于需要快速传输的应用场景，如视频聊天、实时游戏等 
- 2. UDP协议特点：① 无连接，不需要建立连接；② 不可靠，不保证数据包的顺序和完整性；③ 数据报文长度可变，最大长度为65507字节；④ 速度快，不需要建立连接和确认，延迟小；⑤ 一对多、多对一、多对多通信；

#### TCP和UDP协议的区别
- 1. TCP协议：面向连接、可靠、基于字节流、传输效率低、适用于对数据完整性要求高的场景
- 2. UDP协议：无连接、不可靠、基于数据报、传输效率高、适用于对实时性要求高的场景

## IP地址和端口类

### IPAddress类

- IPAddress类：用于表示IP地址，可以表示IPv4和IPv6地址，可以通过字符串或整数表示IP地址，如IPAddress.Parse("192.168.1.1")或IPAddress.Parse("0xC0A80101")

```csharp
//初始化IP地址的方式
IPAddress ipAddress = IPAddress.Parse("192.168.1.1"); //字符串表示 最推荐
IPAddress ipAddress = new IPAddress(new byte[] { 192, 168, 1, 1 }); //字节数组表示
IPAddress ipAddress = new IPAddress(0xC0A80101); //整数表示

//获取可用的IPv6地址
IPAddress.Ipv6Any;
```

### IPEndPoint类

- IPEndPoint类：用于表示IP地址和端口号，可以通过IPAddress和端口号初始化，如IPEndPoint ipEndPoint = new IPEndPoint(IPAddress.Parse("192.168.1.1"), 80)

```csharp
//初始化IPEndPoint的方式
IPEndPoint ipEndPoint = new IPEndPoint(IPAddress.Parse("192.168.1.1"), 8080); //字符串表示
IPEndPoint ipEndPoint = new IPEndPoint(new IPAddress(new byte[] { 192, 168, 1, 1 }), 8080); //字节数组表示
```

## 网络通信

### 域名解析

- 域名解析：将域名转换为IP地址的过程，如将www.baidu.com解析为220.181.38.148

#### IPHostEntry类

- IPHostEntry类：用于表示域名解析的结果，包含域名、IP地址、别名等信息，可以通过Dns.GetHostEntry方法获取，如IPHostEntry hostEntry = Dns.GetHostEntry("www.baidu.com")

```csharp
//获取域名解析结果
IPHostEntry hostEntry = Dns.GetHostEntry("www.baidu.com");
//获取关联IP
IPAddress[] addresses = hostEntry.AddressList;
//获取关联域名
string[] aliases = hostEntry.Aliases;
//获取主机名
string hostName = hostEntry.HostName;
```

#### DNS类
- DNS类：用于域名解析，可以通过Dns.GetHostEntry方法获取域名解析结果，如IPHostEntry hostEntry = Dns.GetHostEntry("www.baidu.com")

```csharp
//获取域名解析结果 
//同步获取 由于获取远程主机信息是需要进行网络通信的 所以可能会阻塞主线程
IPHostEntry hostEntry = Dns.GetHostEntry("www.baidu.com");
//异步获取
private async void GetHostEntryAsync()
{
    Task<IPHostEntry> task = Dns.GetHostEntryAsync("www.baidu.com");
    await task;
}
GetHostEntryAsync();

//获取关联IP
IPAddress[] addresses = hostEntry.AddressList;
//获取关联域名
string[] aliases = hostEntry.Aliases;
//获取主机名
string hostName = hostEntry.HostName;
```

### 序列化

- 在C#中序列化一个类对象可以通过使用BinaryFormatter类来实现，但由于服务器端所用语言可能不是C#，所以我们要自己实现类的序列化

```csharp
//定义一个类
public class Person
{
    public string Name { get; set; }
    public int Age { get; set; }
}

//序列化
Person person = new Person { Name = "张三", Age = 18 };

//先得到字节数组总长度
int bytesLength = sizeof(int) + 
sizeof(int) +  //代表string类型的长度
Encoding.UTF8.GetBytes(person.Name).Length;

//创建字节数组
byte[] bytes = new byte[bytesLength];

int index = 0;
//写入字符串长度
byte[] strBytes = Encoding.UTF8.GetBytes(person.Name);
int nameLength = strBytes.Length;
BitConverter.GetBytes(nameLength).CopyTo(bytes, index);
index += sizeof(int);

//写入字符串
strBytes.CopyTo(bytes, index);
index += strBytes.Length;

//写入整数
BitConverter.GetBytes(person.Age).CopyTo(bytes, index);
index += sizeof(int);
```

### Socket（套接字）

- Socket类：用于网络通信，可以通过创建Socket对象来发送和接收数据，如Socket socket = new Socket(AddressFamily.InterNetwork, SocketType.Stream, ProtocolType.Tcp)

```csharp
//创建Socket对象
Socket socket = new Socket(AddressFamily.InterNetwork, SocketType.Stream, ProtocolType.Tcp);

//参数一：AddressFamily
//AddressFamily.InterNetwork：IPv4
//AddressFamily.InterNetworkV6：IPv6

//参数二：SocketType
//SocketType.Stream：TCP
//SocketType.Dgram：UDP

//参数三：ProtocolType
//ProtocolType.Tcp：TCP
//ProtocolType.Udp：UDP

//建立用于TCP通信的连接
Socket socket_Tcp = new Socket(AddressFamily.InterNetwork, SocketType.Stream, ProtocolType.Tcp);
socket_Tcp.Connect(new IPEndPoint(IPAddress.Parse("127.0.0.1"), 8080));

//建立用于UDP通信的连接
Socket socket_Udp = new Socket(AddressFamily.InterNetwork, SocketType.Dgram, ProtocolType.Udp);
socket_Udp.Connect(new IPEndPoint(IPAddress.Parse("127.0.0.1"), 8080));
```

#### Socket常用属性

```csharp
//套接字的连接状态
socket.Connected;

//套接字的类型
socket.SocketType;

//套接字的协议类型
socket.ProtocolType;

//套接字的寻址方案
socket.AddressFamily;

//从网络中获取准备读取的数据数据量
socket.Available;

//获取本机EndPoint对象
socket.LocalEndPoint as IPEndPoint;

//获取远程EndPoint对象
socket.RemoteEndPoint as IPEndPoint;
```

#### Socket常用方法

```csharp
//1.主要用于服务端
  //绑定IP端口
  socket.Bind(new IPEndPoint(IPAddress.Parse("127.0.0.1"), 8080));
  //监听 设置客户端连接的最大数量
  socket.Listen(10);
  //接受客户端连接
  Socket clientSocket = socket.Accept();
  //接收客户端发送的数据
  byte[] buffer = new byte[1024];
  int length = clientSocket.Receive(buffer);
  //发送数据给客户端
  clientSocket.Send(buffer, length, SocketFlags.None);

//2.主要用于客户端
  //连接服务器
  socket.Connect(new IPEndPoint(IPAddress.Parse("127.0.0.1"), 8080));
  //发送数据给服务器
  socket.Send(Encoding.UTF8.GetBytes("hello world"));
  //接收服务器发送的数据
  byte[] buffer = new byte[1024];
  int length = socket.Receive(buffer);
  //释放连接并关闭套接字 先于Close调用
  socket.Shutdown(SocketShutdown.Both);
  //关闭连接
  socket.Close();
```

### 分包和粘包
- 分包：将数据分成多个包发送，每个包都包含数据长度等信息，接收方根据数据长度来解析数据
- 粘包：将多个包的数据粘在一起发送，接收方需要根据协议来解析数据

### 心跳消息
- 心跳消息：用于检测客户端和服务器之间的连接是否正常，客户端和服务器之间定期发送心跳消息，如果长时间没有收到心跳消息，则认为连接已经断开

### TCP异步通信

#### SocketTCP通信的异步方法(Begin开头)

```csharp
//BeginAccept 异步接受客户端连接
Socket.BeginAccept(AsyncCallback callback, object state);
//EndAccept 异步接受客户端连接的结束方法
Socket.EndAccept(IAsyncResult asyncResult);
```

- BeginConnect 异步接受客户连接
```csharp
Socket socketTCP = new Socket(AddressFamily.InterNetwork, SocketType.Stream, ProtocolType.Tcp);

socketTCP.BeginAccept(BeginAcceptCallback, socketTCP);

private void BeginAcceptCallback(IAsyncResult ar)
{
    Socket socketTCP = (Socket)ar.AsyncState;
    try
    {
        Socket socket = socketTCP.EndAccept(ar);

        socketTCP.BeginAccept(BeginAcceptCallback, socketTCP);
    }
    catch (SocketException e)
    {
        print(e.Message);
    }
}
```

```csharp
//BeginConnect 异步连接服务器
Socket.BeginConnect(EndPoint remoteEP, AsyncCallback callback, object state);
//EndConnect 异步连接服务器的结束方法
Socket.EndConnect(IAsyncResult asyncResult);
```

- BeginConnect 异步连接服务器
```csharp
Socket socketTCP = new Socket(AddressFamily.InterNetwork, SocketType.Stream, ProtocolType.Tcp);

socketTCP.BeginConnect(new IPEndPoint(IPAddress.Parse("127.0.0.1"), 8080), BeginConnectCallback, socketTCP);

private void BeginConnectCallback(IAsyncResult ar)
{
    Socket socketTCP = (Socket)ar.AsyncState;
    try
    {
        socketTCP.EndConnect(ar);
    }
    catch (SocketException e)
    {
        print(e.Message);
    }
}
```

```csharp
//BeginReceive 异步接收数据
Socket.BeginReceive(byte[] buffer, int offset, int size, SocketFlags socketFlags, AsyncCallback callback, object state);
//EndReceive 异步接收数据的结束方法
Socket.EndReceive(IAsyncResult asyncResult);
```

- BeginReceive 异步接收数据
```csharp
Socket socketTCP = new Socket(AddressFamily.InterNetwork, SocketType.Stream, ProtocolType.Tcp);

socketTCP.BeginReceive(buffer, 0, buffer.Length, SocketFlags.None, BeginReceiveCallback, socketTCP);

private void BeginReceiveCallback(IAsyncResult ar)
{
    Socket socketTCP = (Socket)ar.AsyncState;
    try
    {
        int length = socketTCP.EndReceive(ar);
        socketTCP.BeginReceive(buffer, 0, buffer.Length, SocketFlags.None, BeginReceiveCallback, socketTCP);
    }
    catch (SocketException e)
    {
        print(e.Message);
    }
}
```

```csharp
//BeginSend 异步发送数据
Socket.BeginSend(byte[] buffer, int offset, int size, SocketFlags socketFlags, AsyncCallback callback, object state);
//EndSend 异步发送数据的结束方法
Socket.EndSend(IAsyncResult asyncResult);
```

- BeginSend 异步发送数据
```csharp
Socket socketTCP = new Socket(AddressFamily.InterNetwork, SocketType.Stream, ProtocolType.Tcp);

socketTCP.BeginSend(Encoding.UTF8.GetBytes("hello world"), 0, Encoding.UTF8.GetBytes("hello world").Length, SocketFlags.None, BeginSendCallback, socketTCP);

private void BeginSendCallback(IAsyncResult ar)
{
    Socket socketTCP = (Socket)ar.AsyncState;
    try
    {
        socketTCP.EndSend(ar);
    }
    catch (SocketException e)
    {
        print(e.Message);
    }
}
```

#### SocketTCP通信的异步方法(Async开头)

- AsyncAccept 异步接受客户端连接
```csharp
Socket socketTcp = new Socket(AddressFamily.InterNetwork, SocketType.Stream, ProtocolType.Tcp);
SocketAsyncEventArgs e = new SocketAsyncEventArgs();
e.Completed += (socket,args) =>
{
    if(args.SocketError == SocketError.Success)
    {
        Debug.Log("连接成功");
        Socket socketClient = args.AcceptSocket;

        socketTcp.AcceptAsync(e);
    }
    else
    {
        Debug.Log("连接失败");
    }  
};
socketTcp.AcceptAsync(e);
```

- AsyncConnect 异步连接服务器
```csharp
Socket socketTcp = new Socket(AddressFamily.InterNetwork, SocketType.Stream, ProtocolType.Tcp);
SocketAsyncEventArgs e = new SocketAsyncEventArgs();
e.RemoteEndPoint = new IPEndPoint(IPAddress.Parse("127.0.0.1"), 8080);
e.Completed += (socket,args) =>
{
    if(args.SocketError == SocketError.Success)
    {
        Debug.Log("连接成功");
    }
    else
    {
        Debug.Log("连接失败");
    }  
};
socketTcp.ConnectAsync(e);
```

- AsyncReceive 异步接收数据
```csharp
Socket socketTcp = new Socket(AddressFamily.InterNetwork, SocketType.Stream, ProtocolType.Tcp);
SocketAsyncEventArgs e = new SocketAsyncEventArgs();
e.SetBuffer(new byte[1024*1024], 0, 1024*1024);
e.Completed += (socket,args) =>
{
    if(args.SocketError == SocketError.Success)
    {
        Debug.Log("接收成功");
        byte[] buffer = args.Buffer;
        //arg.Offset 表示接收到的数据在 buffer 中的偏移量
        //arg.BytesTransferred 表示接收到的数据长度
        string str = Encoding.UTF8.GetString(buffer, args.Offset, args.BytesTransferred);

        //设置接收缓冲区
        args.SetBuffer(0, 1024*1024);
        socketTcp.ReceiveAsync(e);
    }
    else
    {
        Debug.Log("接收失败");
    }  
};
socketTcp.ReceiveAsync(e);
```

- AsyncSend 异步发送数据

```csharp
Socket socketTcp = new Socket(AddressFamily.InterNetwork, SocketType.Stream, ProtocolType.Tcp);
SocketAsyncEventArgs e = new SocketAsyncEventArgs();
byte[] buffer = Encoding.UTF8.GetBytes("hello world");
e.SetBuffer(buffer, 0, buffer.Length);
e.Completed += (socket,args) =>
{
    if(args.SocketError == SocketError.Success)
    {
        Debug.Log("发送成功");
    }
    else
    {
        Debug.Log("发送失败");
    }
};
socketTcp.SendAsync(e);
```

### UDP通信概述
UDP（User Datagram Protocol，用户数据报协议）是一种无连接的协议，它不保证数据包的顺序、可靠性和完整性。UDP协议的通信过程相对简单，只需要发送和接收数据包即可。

- 1. 服务端和客户端代码
  创建->绑定->收发消息->释放->关闭

- 2. UDP相对TCP的区别
  代码流程更简单，UDP不可靠效率高

- 3. UDP的分包、黏包问题
  UDP不会黏包，要避免UDP自动分包  
  我们可以根据需求自己处理UDP的分包黏包 但是前提是要解决无序和丢包的问题

#### UDP同步通信基本逻辑(服务端和客户端通用)

```csharp
//1.创建套接字
Socket socketUDP = new Socket(AddressFamily.InterNetwork, SocketType.Dgram, ProtocolType.Udp);
//2.绑定端口
socketUDP.Bind(new IPEndPoint(IPAddress.Parse("127.0.0.1"), 8080));
//3.发送到指定目标
IPEndPoint remoteIpPoint = new IPEndPoint(IPAddress.Parse("127.0.0.1"), 8081);
//指定要发送的字节数和远程计算机的IP和端口
socketUDP.SendTo(Encoding.UTF8.GetBytes("hello world"), remoteIpPoint);
//4.接收数据
IPEndPoint remoteIpPoint2 = new IPEndPoint(IPAddress.Any, 0);
byte[] buffer = new byte[1024*1024];
int length = socketUDP.ReceiveFrom(buffer, ref remoteIpPoint2);
string str = Encoding.UTF8.GetString(buffer, 0, length);
//5.关闭套接字
socketUDP.Shutdown(SocketShutdown.Both);
socketUDP.Close();
```

#### UDP异步通信
- BeginReceiveFrom 异步接收数据
```csharp
Socket socketUDP = new Socket(AddressFamily.InterNetwork, SocketType.Dgram, ProtocolType.Udp);
socketUDP.Bind(new IPEndPoint(IPAddress.Parse("127.0.0.1"), 8080));

EndPoint remoteIpPoint = new IPEndPoint(IPAddress.Any, 0);
byte[] buffer = new byte[1024*1024];
socketUDP.BeginReceiveFrom(buffer, 0, buffer.Length, SocketFlags.None, ref remoteIpPoint, BeginReceiveFromCallback, (socketUDP, remoteIpPoint));

private void BeginReceiveFromCallback(IAsyncResult ar)
{
    (Socket s,EndPoint ipPoint) info = ((Socket,EndPoint))ar.AsyncState;
    try
    {
        int length = info.s.EndReceiveFrom(ar, ref remoteIpPoint);
        string str = Encoding.UTF8.GetString(buffer, 0, length);
        info.s.BeginReceiveFrom(buffer, 0, buffer.Length, SocketFlags.None, ref remoteIpPoint, BeginReceiveFromCallback, (socketUDP, remoteIpPoint));
    }
    catch (SocketException e)
    {
        print(e.Message);
    }
}
```

- BeginSendTo 异步发送数据
```csharp
Socket socketUDP = new Socket(AddressFamily.InterNetwork, SocketType.Dgram, ProtocolType.Udp);
socketUDP.Bind(new IPEndPoint(IPAddress.Parse("127.0.0.1"), 8080));

EndPoint remoteIpPoint = new IPEndPoint(IPAddress.Parse("127.0.0.1"), 8081);
socketUDP.BeginSendTo(Encoding.UTF8.GetBytes("hello world"), 0, Encoding.UTF8.GetBytes("hello world").Length, SocketFlags.None, remoteIpPoint, BeginSendToCallback, socketUDP);
private void BeginSendToCallback(IAsyncResult ar)
{
    Socket socketUDP = (Socket)ar.AsyncState;
    try
    {
        socketUDP.EndSendTo(ar);
    }
    catch (SocketException e)
    {
        print(e.Message);
    }
}
```

#### UDP异步通信(基于SocketAsyncEventArgs)
- AsyncReceiveFrom 异步接收数据
```csharp
Socket socketUDP = new Socket(AddressFamily.InterNetwork, SocketType.Dgram, ProtocolType.Udp);
socketUDP.Bind(new IPEndPoint(IPAddress.Parse("127.0.0.1"), 8080));

SocketAsyncEventArgs e = new SocketAsyncEventArgs();
e.RemoteEndPoint = new IPEndPoint(IPAddress.Any, 0);
e.SetBuffer(new byte[1024*1024], 0, 1024*1024);
e.Completed += (socket, args) =>
{
    if(args.SocketError == SocketError.Success)
    {
        IPEndPoint remoteIpPoint = (IPEndPoint)args.RemoteEndPoint;
        string str = Encoding.UTF8.GetString(args.Buffer, args.Offset, args.BytesTransferred);
        socketUDP.ReceiveFromAsync(e);
    }
    else
    {
        Debug.Log("接收失败");
    }
};
socketUDP.ReceiveFromAsync(e);
```

- AsyncSendTo 异步发送数据
```csharp
Socket socketUDP = new Socket(AddressFamily.InterNetwork, SocketType.Dgram, ProtocolType.Udp);
socketUDP.Bind(new IPEndPoint(IPAddress.Parse("127.0.0.1"), 8080));

SocketAsyncEventArgs e = new SocketAsyncEventArgs();
e.RemoteEndP oint = new IPEndPoint(IPAddress.Parse("127.0.0.1"), 8081);
e.SetBuffer(Encoding.UTF8.GetBytes("hello world"), 0, Encoding.UTF8.GetBytes("hello world").Length);
e.Completed += (socket, args) =>
{b  
    if(args.SocketError == SocketError.Success)
    {
        Debug.Log("发送成功");
    }
    else
    {
        Debug.Log("发送失败");
    }
};
socketUDP.SendToAsync(e);
```

### 文件传输FTP

#### FTP工作原理
FTP（File Transfer Protocol，文件传输协议）是一种用于在网络上传输文件的协议。它使用TCP协议作为传输层协议，提供可靠的文件传输服务。

FTP协议的工作原理如下：

1. 客户端与服务器建立TCP连接，使用21端口进行控制连接。
2. FTP有两种传输模式：主动模式和被动模式。
3. FTP有两种传输方式：ASCII模式和二进制模式。
4. FTP传输文件时需要进行身份验证，使用用户名和密码进行登录。如果服务器允许可以进行匿名登录。

#### FTP关键类
C#中实现FTP通信的相关类 ``System.Net.FtpWebRequest`` 、 ``System.Net.FtpWebResponse``、 ``System.Net.NetworkCredential``

- 1. NetworkCredential类
```csharp
//通信凭证类 用于在Ftp文件传输时设置账号密码
NetworkCredential networkCredential = new NetworkCredential("username", "password");
```

- 2. FtpWebRequest类
```csharp
//Ftp文件传输请求类
//主要用于：上传下载删除服务器上的文件

//重要方法
//Create 创建新的WebRequest 用于进行Ftp相关操作
FtpWebRequest ftpWebRequest = (FtpWebRequest)WebRequest.Create("ftp://127.0.0.1/1.txt");
//Abort 如果正在进行文件传输，用此方法可以终止传输
ftpWebRequest.Abort();
//GetRequestStream 获取用于写入数据的流
Stream stream = ftpWebRequest.GetRequestStream();
//GetResponse 获取响应
FtpWebResponse ftpWebResponse = (FtpWebResponse)ftpWebRequest.GetResponse();

//重要成员
//Credentials 设置通信凭证
ftpWebRequest.Credentials = networkCredential;
//KeepAlive 设置当完成请求时是否保持连接 默认为true
ftpWebRequest.KeepAlive = true;
//Method 操作命令设置
// WebRequestMethods.Ftp类中的操作命令属性
// DeleteFile 删除文件
// DownloadFile 下载文件
// UploadFile 上传文件
// ListDirectory 获取目录列表
// ListDirectoryDetails 获取详细目录列表
// CreateDirectory 创建目录
// DeleteDirectory 删除目录
ftpWebRequest.Method = WebRequestMethods.Ftp.DownloadFile;
//UseBinary 设置是否使用二进制传输 默认为true
ftpWebRequest.UseBinary = true;
//RenameTo 重命名文件
ftpWebRequest.RenameTo = "2.txt";
```

- 3. FtpWebResponse类
```csharp
//Ftp文件传输响应类
//主要用于：获取文件传输状态
FtpWebResponse ftpWebResponse = (FtpWebResponse)ftpWebRequest.GetResponse();

//重要方法
//Close 关闭响应
ftpWebResponse.Close();
//GetResponseStream 获取响应流
Stream stream = ftpWebResponse.GetResponseStream();

//重要成员
//ContentLength 获取响应内容长度
ftpWebResponse.ContentLength;
//ContentType 获取响应内容类型
ftpWebResponse.ContentType;
//StatusDescription 获取响应状态描述
ftpWebResponse.StatusDescription;
//StatusCode 获取响应状态码
ftpWebResponse.StatusCode;
//BannerMessage 登录前建立连接时FTP服务器发送的消息
ftpWebResponse.BannerMessage;
//ExitMessage 登录后断开连接时FTP服务器发送的消息
ftpWebResponse.ExitMessage;
//LastModified 获取文件最后修改时间
ftpWebResponse.LastModified;
```
 
#### FTP文件上传下载
- 上传文件
```csharp
using System;
using System.Collections;
using System.Collections.Generic;
using System.IO;
using System.Net;
using System.Net.Sockets;
using UnityEngine;

public class SocketTCPBegin : MonoBehaviour
{
    // Start is called before the first frame update
    void Start()
    {
        try
        {
            NetworkCredential networkCredential = new NetworkCredential("boluobao", "TGYsjy040823");
            FtpWebRequest ftpWebRequest = (FtpWebRequest)WebRequest.Create("ftp://127.0.0.1/pic.jpg");
            ftpWebRequest.Proxy = null;
            ftpWebRequest.Credentials = networkCredential;
            ftpWebRequest.Method = WebRequestMethods.Ftp.UploadFile;
            ftpWebRequest.UseBinary = true;
            Stream upLoadStream = ftpWebRequest.GetRequestStream();

            using (FileStream file = File.OpenRead(Application.streamingAssetsPath + "/pic.jpg"))
            {
                byte[] bytes = new byte[1024];

                int length = file.Read(bytes, 0, bytes.Length);
                //一次只读1kb没读完就一直读
                while (length != 0)
                {
                    upLoadStream.Write(bytes, 0, length);
                    length = file.Read(bytes, 0, bytes.Length);
                }
                file.Close();
                upLoadStream.Close();
                print("上传结束");
            }
        }
        catch (Exception e)
        {
            print("上传失败" + e.Message);
            return;
        }
    }
}
```

- 下载文件
```csharp
using System;
using System.Collections;
using System.Collections.Generic;
using System.IO;
using System.Net;
using System.Net.Sockets;
using UnityEngine;

public class SocketTCPBegin : MonoBehaviour
{
    // Start is called before the first frame update
    void Start()
    {
        try
        {
            NetworkCredential networkCredential = new NetworkCredential("boluobao", "TGYsjy040823");
            FtpWebRequest ftpWebRequest = (FtpWebRequest)WebRequest.Create("ftp://127.0.0.1/pic.jpg");
            ftpWebRequest.Proxy = null;
            ftpWebRequest.Credentials = networkCredential;
            ftpWebRequest.Method = WebRequestMethods.Ftp.DownloadFile;
            ftpWebRequest.UseBinary = true;
            FtpWebResponse res = (FtpWebResponse)ftpWebRequest.GetResponse();
            Stream downLoadStream = res.GetResponseStream();
            print(Application.persistentDataPath);

            using (FileStream file = File.Create(Application.persistentDataPath + "/pic.jpg"))
            {
                byte[] buffer = new byte[1024];
                int readCount = downLoadStream.Read(buffer, 0, buffer.Length);
                while (readCount != 0)
                {
                    file.Write(buffer, 0, readCount);
                    readCount = downLoadStream.Read(buffer, 0, buffer.Length);
                }
                downLoadStream.Close();
                file.Close();
                print("下载结束");
            }
        }
        catch (Exception e)
        {
            print("下载失败" + e.Message);
            return;
        }
    }
}
```

### 超文本传输协议HTTP

- HTTP超文本传输协议就是一个在网络中上传和下载的一套规则

#### HTTP工作原理

- 1. HTTP是以TCP方式工作
  连接-> 请求- >响应- >断开连接
- 2. HTTP是无状态的协议
  每次请求都是独立的，服务器不会记录请求的来源
- 3. HTTP使用元信息作为标头
  主要数据前添加一部分额外信息(元信息)  
  包含传送的对象属于哪种类型，采用的是哪种编码等等

#### HTTP关键类

- 1. HttpWebRequest类
  主要用于发送客户端请求的类 发送HTTP客户端请求给服务器 用于消息通信、上传、下载等

```csharp
//重要方法
//1.Create 创建一个HttpWebRequest对象
HttpWebRequest req = WebRequest.Create(new Uri("http://26.81.20.74:8000/Http_Server/)")) as HttpWebRequest;
//2.Abort 终止请求
req.Abort();
//3.GetRequestStream 获取请求流
Stream stream = req.GetRequestStream();
//4.GetResponse 获取响应
HttpWebResponse res = req.GetResponse() as HttpWebResponse;
//5.BeginGetResponse 异步获取响应
IAsyncResult result = req.BeginGetResponse(new AsyncCallback(AsyncCallback), req);
//6.BeginGetRequestStream 异步获取请求流
IAsyncResult result = req.BeginGetRequestStream(new AsyncCallback(AsyncCallback), req);

//重要成员
//1.Credentials 通信凭证
req.Credentials = new NetworkCredential("boluobao", "TGYsjy040823");
//2.PreAuthenticate 是否随请求发送一个身份验证标头，一般需要进行身份验证
req.PreAuthenticate = true;
//3.Headers 构成标头的名称/值对集合
req.Headers
//4.ContentLength 请求内容长度
req.ContentLength = 1024;
//5.ConetentType 请求内容类型
req.ContentType = "";
//6.Method 请求方法
  //WebRequestMethods.Http类中的操作命令属性
  //Get 获取请求
  //Post 提交请求
  //Head 获取请求头
req.Method = WebRequestMethods.Http.Get;
```
- 2. HttpWebResponse类
  主要用于接收服务器响应的类

```csharp
//重要方法
//1.GetResponseStream 获取响应流
Stream stream = res.GetResponseStream();
//2.Close 释放所有资源
res.Close();


//重要成员
//1.ContentLength 响应内容长度
res.ContentLength;
//2.ContentType 响应内容类型
res.ContentType;
//3.StatusCode 响应状态码
res.StatusCode;
//4.StatusDescription 响应状态描述
res.StatusDescription;
//5.BannerMessage 登录前建立连接时FTP服务器发送的消息
res.BannerMessage;
//6.ExitMessage 登录后断开连接时FTP服务器发送的消息
res.ExitMessage;
//7.LastModified 获取文件最后修改时间
res.LastModified;
```

- 3. NetworkCredentials、Uri、Stream、FileStream类
和FTP处使用类似

#### HTTP下载

```csharp
using System;
using System.Collections;
using System.Collections.Generic;
using System.IO;
using System.Net;
using UnityEngine;

public class Http : MonoBehaviour
{
    // Start is called before the first frame update
    void Start()
    {
        try
        {
            // HttpWebRequest req = WebRequest.Create(new Uri("http://26.81.20.74:8000/Http_Server/)")) as HttpWebRequest;
            // req.Method = WebRequestMethods.Http.Head;
            // HttpWebResponse res = req.GetResponse() as HttpWebResponse;
            // if (res.StatusCode == HttpStatusCode.OK)
            // {
            //     Debug.Log("OK");
            // }
            // else
            // {
            //     Debug.Log("NG");
            // }

            HttpWebRequest req = WebRequest.Create(new Uri("http://26.81.20.74:8000/Http_Server/pic.jpg")) as HttpWebRequest;
            req.Method = WebRequestMethods.Http.Get;
            HttpWebResponse res = req.GetResponse() as HttpWebResponse;
            if(res.StatusCode == HttpStatusCode.OK)
            {
                Debug.Log("OK");
                using (FileStream fileStream = File.Create(Application.persistentDataPath + "/pic.jpg"))
                {
                    Stream downloadStream = res.GetResponseStream();
                    byte[] buffer = new byte[1024];
                    int length = downloadStream.Read(buffer, 0, buffer.Length);
                    while (length != 0)
                    {
                        fileStream.Write(buffer, 0, length);
                        length = downloadStream.Read(buffer, 0, buffer.Length);
                    }
                    fileStream.Close();
                    downloadStream.Close();
                    print(Application.persistentDataPath);
                }
            }
            else
            {
                Debug.Log("NG");
            }
        }
        catch (WebException e)
        {
            Debug.Log(e.Message);
            return;
        }

    }
}
```

#### HTTP上传

- Post相关
  Post方法发送数据时，数据会附加在请求的末尾，数据长度没有限制，数据可以包含文本、图片、视频等  
  Post方法发送的数据不会在浏览器地址栏显示，数据也不会被保存在浏览器历史记录中  
  要注意ContentType，不同的数据类型需要设置不同的Content-Type
  
```csharp
void Start()
    {
        try
        {
            const string boundary = "---------------------------735323031399963166993862150";

            HttpWebRequest req = WebRequest.Create(new Uri("http://26.81.20.74:8000/Http_Server/")) as HttpWebRequest;
            req.Method = WebRequestMethods.Http.Post;
            // 注意（multipart/form-data 的 boundary 规则，踩过坑）：
            // 1. Content-Type 里只声明 boundary 的“值”，body 里用时必须在前面补 "--"：
            //    每段的分隔行 = "--" + boundary，结束行 = "--" + boundary + "--"。
            //    少写开头这两个 "-" 是最常见的坑 —— 服务器能收到 POST，但解析不出任何 part，
            //    于是返回 200、控制台打印 OK，文件却根本没上传成功。
            // 2. boundary 的值必须和 body 里用的完全一致，所以抽成常量，别在两处各写一份硬编码。
            // 3. 分隔行和每个 header 行都必须以 \r\n 结尾，只写 \n 有些服务器会解析失败。
            // 4. header 区和文件数据之间要留一个空行（\r\n\r\n），否则数据会被当成 header。
            // 5. boundary 只能用 ASCII 字符，且要足够随机、不能出现在文件内容里，
            //    否则会在文件中间被误切分（图片是二进制，尤其要注意）。
            // 6. 这里的 name="file" 要和服务器端取值的字段名一致（如 form['file']）。
            req.ContentType = "multipart/form-data; boundary=" + boundary;
            req.Timeout = 500000;

            string head = "--" + boundary + "\r\n" +
            "Content-Disposition: form-data; name=\"file\"; filename=\"pic.jpg\"\r\n" +
            "Content-Type: image/jpeg\r\n\r\n";
            string foot = "\r\n--" + boundary + "--\r\n";

            byte[] headByte = System.Text.Encoding.UTF8.GetBytes(head);

            byte[] footByte = System.Text.Encoding.UTF8.GetBytes(foot);

            using (FileStream fileStream = File.OpenRead(Application.persistentDataPath + "/pic.jpg"))
            {
                int Length = headByte.Length + (int)fileStream.Length + footByte.Length;
                byte[] buffer = new byte[Length];
                int fileLength = fileStream.Read(buffer, headByte.Length, (int)fileStream.Length);
                req.ContentLength = Length;
                Stream uploadStream = req.GetRequestStream();
                uploadStream.Write(headByte, 0, headByte.Length);
                while (fileLength != 0)
                {
                    uploadStream.Write(buffer, headByte.Length, fileLength);
                    fileLength = fileStream.Read(buffer, headByte.Length, (int)fileStream.Length);
                }
                uploadStream.Write(footByte, 0, footByte.Length);
                uploadStream.Close();
                fileStream.Close();
            }
            HttpWebResponse res = req.GetResponse() as HttpWebResponse;
            if (res.StatusCode == HttpStatusCode.OK)
            {
                Debug.Log("OK");
            }
            else
            {
                Debug.Log("NG");
            }
        }
        catch (WebException e)
        {
            Debug.Log(e.Message);
            return;
        }
    }
```

#### HTTP和FTP的区别
- HTTP和FTP都是基于TCP/IP的应用层协议，用于在网络上传输数据，但它们有一些重要的区别
- HTTP（超文本传输协议）主要用于在Web浏览器和Web服务器之间传输超文本数据，如HTML、图片、视频等。HTTP是无状态的协议，每次请求都会建立一个新的连接，请求完成后连接会关闭。HTTP支持GET、POST、PUT、DELETE等请求方法，用于获取、提交、更新和删除数据。
- FTP（文件传输协议）主要用于在客户端和服务器之间传输文件，如文本文件、图片、视频等。FTP是有状态的协议，每次请求都会建立一个新的连接，请求完成后连接会保持一段时间，以便后续的请求可以复用该连接。FTP支持上传、下载、删除、重命名等操作，用于管理文件。
- HTTP和FTP的主要区别在于它们的应用场景和数据传输方式。HTTP主要用于Web应用，而FTP主要用于文件传输。HTTP是无状态的协议，而FTP是有状态的协议。HTTP支持GET、POST、PUT、DELETE等请求方法，而FTP支持上传、下载、删除、重命名等操作。

### Unity网络相关类

#### UnityWebRequest类

- UnityWebRequest类是Unity网络编程的核心类，它封装了HTTP请求和响应，可以发送GET、POST等请求，并处理响应数据

- 1. 获取文本
```csharp
IEnumerator LoadText()
{
    UnityWebRequest www = UnityWebRequest.Get("http://26.81.20.74:8000/Http_Server/test.txt");
    yield return www.SendWebRequest();

    if (www.result != UnityWebRequest.Result.Success)
    {
        Debug.Log("获取失败" + www.error);
    }
    else
    {
        Debug.Log(www.downloadHandler.text);
        byte[] bytes = www.downloadHandler.data;
        Debug.Log(bytes.Length);
    }
}
```

- 2. 获取图片
```csharp
IEnumerator LoadImage()
{
    //http
    UnityWebRequest www = UnityWebRequestTexture.GetTexture("http://26.81.20.74:8000/Http_Server/pic.jpg");
    //ftp
    //UnityWebRequest www = UnityWebRequestTexture.GetTexture("ftp://26.81.20.74:8000/Http_Server/pic.jpg");
    //本地文件
    //UnityWebRequest www = UnityWebRequestTexture.GetTexture("file:///C:/Users/86182/Desktop/UnityWeb/pic.jpg");
    yield return www.SendWebRequest();

    if (www.result != UnityWebRequest.Result.Success)
    {
        Debug.Log("获取失败" + www.error);
    }
    else
    {
        Texture texture = DownloadHandlerTexture.GetContent(www);
    }
}
```

- 3. 获取AB包
```csharp
IEnumerator LoadAB()
{
    UnityWebRequest www = UnityWebRequestAssetBundle.GetAssetBundle("http://26.81.20.74:8000/Http_Server/AssetBundle/AssetBundle.unity3d");
    // yield return www.SendWebRequest();

    while(!www.isDone)
    {
        Debug.Log(www.downloadProgress);
        yield return null;
    }

    if (www.result != UnityWebRequest.Result.Success)
    {
        Debug.Log("获取失败" + www.error);
    }
    else
    {
        AssetBundle bundle = DownloadHandlerAssetBundle.GetContent(www);
    }
}
```

- 4. 上传相关
```csharp
//接口
//IMultipartFormSection
//数据相关类都继承该接口
//我们可以用父类装子类
List<IMultipartFormSection> dataList = new List<IMultipartFormSection>();
//子类数据
//MultipartFormDataSection
//1.二进制字节数组
dataList.Add(new MultipartFormDataSection(Encoding.UTF8.GetBytes("123123123123123")));
//2.字符串
dataList.Add(new MultipartFormDataSection("12312312312312312dsfasdf"));
//3.参数名，参数值（字节数组，字符串），编码类型，资源类型（常用）
dataList.Add(new MultipartFormDataSection("Name", "MrTang", Encoding.UTF8, "application/...."));
dataList.Add(new MultipartFormDataSection("Msg", new byte[1024], "appl....."));
//MultipartFormFileSection
//1.字节数组
dataList.Add(new MultipartFormFileSection(File.ReadAllBytes(Application.streamingAssetsPath + "/test.p")));
//2.文件名，字节数组（常用）
dataList.Add(new MultipartFormFileSection("上传的文件.png", File.ReadAllBytes(Application.streamingAssetsPath + "/test.png")));
//3.字符串数据，文件名（常用）
dataList.Add(new MultipartFormFileSection("12312313212312", "test.txt"));
//4.字符串数据，编码格式，文件名（常用）
dataList.Add(new MultipartFormFileSection("12312313212312", Encoding.UTF8, "test.txt"));
//5.表单名，字节数组，文件名，文件类型
dataList.Add(new MultipartFormFileSection("file", new byte[1024], "test.txt", ""));
//6.表单名，字符串数据，编码格式，文件名
dataList.Add(new MultipartFormFileSection("file", "123123123", Encoding.UTF8, "test.txt"));
```

- 上传文件
```csharp
IEnumerator upLoad()
{
    List<IMultipartFormSection> data = new List<IMultipartFormSection>();
    data.Add(new MultipartFormDataSection("name","12131231", Encoding.UTF8, "text/plain"));
    data.Add(new MultipartFormFileSection("上传的文件.png", File.ReadAllBytes(Application.streamingAssetsPath + "/test.png")));

    UnityWebRequest www = UnityWebRequest.Post("http://26.81.20.74:8000/Http_Server/Upload", data);
    while (!www.isDone)
    {
        Debug.Log(www.uploadProgress);
        yield return null;
    }

    if (www.result != UnityWebRequest.Result.Success)
    {
        Debug.Log("上传失败" + www.error);
    }
    else
    {
        Debug.Log("上传成功");
    }
}
```

`可以通过继承DownloadHandler类，自定义下载处理类，实现下载进度条等功能`

### protobuf

- protobuf是Google开发的一种数据序列化协议，可以用于网络传输、数据存储等场景，具有高效、跨平台、跨语言等特点

#### protobuf配置规则

```proto
//规则一：指定版本
syntax = "proto3";

//规则二：注释方式
//单行注释
//多行注释
/*
多行注释
*/

//规则十：导入其他proto文件
import "other.proto";

//规则三：包名
package com.example;

//规则四：消息类
message TestMsg{
    //规则五：成员类型和唯一编号

    //浮点数
    // = 1不代表默认值 而是唯一编号 用于序列化和反序列化
    float testF = 1;
    double testD = 2;
     
     //变长编码
     //根据数字的大小 来使用对应的字节数来存储
     int32 testInt32 = 3;
     int64 testInt64 = 4; //以上两种都不太适用于表示负数

     sint32 = testSInt32 = 5;
     sint64 = testSInt64 = 6; //适用于表示负数
      
     uint32 = testUInt32 = 7;
     uint64 = testUInt64 = 8; //无符号

    //固定字节数的类型
     sfixed32 = testSFixed32 = 9; //始终是4个字节
     sfixed64 = testSFixed64 = 10;//表示无符号 始终是8个字节

     fixed32 = testFixed32 = 11;
     fixed64 = testFixed64 = 12;

     //字符串
     string testStr = 13;

     //布尔值
     bool testBool = 14;

     bytes testBytes = 15; //字节字符串

    //规则六：特殊标识
    //required：必须赋值
    //optional：可选
    //repeated：数组
    //map：字典

    //数组List
    repeated int32 testList = 16;
    //字典Map
    map<string, int32> testMap = 17;

    //规则七：枚举 可以申明到类里面也可以申明到类外面
    enum TestEnum{
        Normol = 0; //第一个常量必须映射到0
        Boss = 5;
    }

    //枚举成员变量的申明
    TestEnum testEnum = 18;

    //申明自定义类对象
    TestMsg2 testMsg2 = 19;

    //规则九：保留字段
    //1.保留编号
    reserved 20, 21, 22, 23;
    //2.保留字段名
    reserved "testStr", "testBool";
}

//规则八：嵌套类
message TestMsg2{
    TestEnum testEnum = 1;
}
``` 

#### protobuf使用

- 1. 打开cmd窗口
- 2. 进入protobuf.exe所在的目录
- 3. 执行命令`protoc.exe -I=配置路径 --csharp_out=输出路径 ./test.proto`，将test.proto文件编译成c#文件
- 注意路径不要有中文和特殊符号