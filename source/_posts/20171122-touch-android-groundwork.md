---
title: 浅谈Android底层
date: 2017-11-22 20:24:35
tags: [底层, Android]
categories: Android
---


# 学习的原因
> 很多应用开发者对Android底层望而却步，主要有两个原因：
> 1.底层太难，我看不懂
> 2.学习底层知识对目前开发没有帮助


## 对于太难的这点
我觉得是因为一开始直接扑进源码里面，看到那些一长串一长串的函数，很容易看得晕头转向，恶心想吐。如果掌握合适的学习方法，会发现其实并没有这么困难。
我建议，在看源码前先理清Android底层的主干逻辑，再拆解出各个模块，各个击破。

## 对于无帮助这点
我觉得Android底层知识的即时回报非常小，但是长期回报是巨大的。
如果能熟悉Android底层的原理，当在开发中遇到一些奇奇怪怪的问题时，我们可以通过debug和查看log等方式，结合底层原理去发现蛛丝马迹，真正解决一部分烦人的小概率bug。而且熟悉android底层的设计架构，在未来做软件架构设计时，可以参考借鉴，甚至可以在此基础上设计出更棒的架构。
这话听上去觉得特别夸张，其实不然，有两点原因：
1.Android底层的架构也在不断调整和优化中，这说明目前的不是最优解；
2.Android更新迭代了这么多版本，需要兼容旧版本，有些地方不能完全放开去设计，需要在兼容和完全优化中做选择。


---





作者在学习过程中，尝试按照自己的思路总结Android底层主干逻辑。肯定会有许多不足之处，希望大家多多指出。
由表到底，分成三层：
> 1.应用程序背后：Android的各大系统服务
> 2.如何获取这些系统服务：ServiceManager
> 3.如何通信：Binder体系

# 应用程序背后：Android的各大系统服务
这里首先要说明下，ActivityManagerService等各种应用服务，虽然说以Service结尾，但是这与Android四大组件Service并无关系。四大组件中的Service，主要提供需要在后台长期运行的服务（如复杂计算、下载等等）；这里的Service代表Client/Server架构中的Server。

Client/Server架构简称为C/S架构，也是客户端/服务器端架构。服务器端主要提供数据管理、数据共享、数据及系统维护和并发控制等，客户端程序主要完成用户的具体的业务。
在Android系统中，Client就是我们写的各种应用程序，Server实现页面跳转，屏幕展示等功能细节。Client向Server发出命令，Server去实现完整的功能。
> 注：Client/Server架构是Server，我们说的Android系统服务是Service。虽然说都是表示“服务”，建议还是注意下拼写，方便更好区分。


## 我举一个简单的例子
> 应用开发者常见的工作是，去实现一个Activity并且显示在手机屏幕上。
对于最简单的页面，开发者只要做三步：
1.AndroidManifest文件加入声明
2.Activity中设置setContentView
3.调用startActivity()去启动（发出指令，后续系统Service去实现）

在背后辛勤工作的就是Android的各大系统服务，例如ActivityManagerService（后续简称AMS）主要管理Activity运行状态，WindowManagerService（后续简称WMS）主要负责控制手机屏幕显示内容。


# 如何获取这些系统服务：ServiceManager

上一节简单的描述了下应用通过各大系统服务去完成Activity生成和屏幕显示。
这里就会有个问题，我们如何去获取这些服务。在应用开发中，如果我们需要使用第三方控件OkHttp，我们需要导入okhttp包，或者在gradle中写入对其的依赖，之后我们才可以调用OkHttp中的对象和方法。

在Android底层也是类似，其中有一个ServiceManager在统筹管理所有的服务。
还是采用上节说的C/S架构，应用程序是客户端，向ServiceManager服务端发起请求获取指定name的服务，要求服务端给与AMS的访问引用。
应用程序持有AMS的引用后，继续采用C/S架构。应用本身还是客户端，此时AMS充当服务端，处理服务端发起的各种Activitiy请求。

{% asset_img 1122_01.png Activity&AMS&WMS %}

这个时候就继续思考一步，AMS、WMS这些系统服务如何和ServiceManager建立联系的呢？
这是在Android手机开机时，AMS、WMS会向ServiceManager注册，将自己的name和实体传给ServiceManager，ServiceManager中会有专门的数据结构（红黑树）去记录这些数据。
注：这里还是采用C/S架构，不过AMS变成了客户端，ServiceManager变成服务端。

{% asset_img 1122_02.png AMS&ServerManager&APP %}


再再深入一步，我们需要通过ServiceManager获取其他服务，那我们怎么获取ServiceManager呢？
这里ServiceManager充当大管家的角色，是在开机时最先被创造的服务，并且被赋予0的代号。所有的服务都要先请示ServiceManager。
注：匿名服务除外，匿名服务不需要注册在ServiceManager。当前连接的服务直接传递匿名服务给应用。
后续启动的服务都可以根据0去找到ServiceManager，并且把自己注册进去。
注：这里还是采用C/S架构，不过AMS变成了客户端，ServiceManager变成服务端。

{% asset_img 1122_03.png AMS&system %}


至此我们把获取系统服务，从表到里分析了一遍。我们再换一个维度，以时间发展表述下。
注：学习的时候要时刻记住，所有的对象都不是直接持有，需要通过各种请求获取后才持有。
再注：这里的持有不一定是持有实体，可能是种引用。

{% asset_img 1122_04.png AMS&ServerManager&system %}


# 如何通信：Binder体系
上面我们了解了系统服务的作用和如何获取系统服务，还有一个更加基础的问题，应用如何和这些系统服务通信。
> 在Android中，各个应用和各个服务处于不同的进程。就不能像进程内编程一样直接调用其他类的函数，需要进程间的通信（IPC：Inter-Process Communication）
这里Android采用Binder方式。

这里思考下，Linux IPC常见的有pipe、socket、共享内存等等，为什么最后Android会选择Binder呢？
Android之间有大量的跨进程通信，对性能、安全性、易用性要求都很高，综合考虑后选择了Binder方式。

## 继续问，Binder的性能优势？

socket主要用于网络通信，以TCP/IP作为基础，需要分包、重组等工作，所以效率递比较底下。
注：Android有采用Unix Domain Socket(UDS)，针对进程间通信优化。在Android中也有使用，这里暂不讨论。

pipe采用消息转发机制，需要两次拷贝。

{% asset_img 1122_05.png pipe %}

Binder在数据传输过程中，只需要一次拷贝。

{% asset_img 1122_06.png Binder %}

经过上述操作后，服务端地址s和客户端地址c指向同一块内存。
服务端想与客户端通信时，就将本地内容拷贝到地址s中，客户端在同时监听地址c中的内存变化，及时获取新信息。

## Binder的安全性优势？

一般的Linux IPC在通信时，请求方会发送user id(uid)和process id(pid)，服务方后根据此去检查请求方的权限，判断后续是否给与服务。
看上去特别安全，但是根源上出现了问题，uid和pid是请求方添加的。这意味着，请求方A可以修改uid和pid，设置成B一样。服务端是没有办法识别出来，这样安全性就无法保障了。


{% asset_img 1122_07.png 传统IPC %}

Binder优化了uid和pid机制，不再由请求方自己添加，而是由内核自动添加。

{% asset_img 1122_08.png Binder安全性 %}

## Binder的易用性优势？

采用C/S架构，应用和服务分离，逻辑清晰。
应用持有一个服务的引用，向该引用发起各种请求，引用内部在通过Binder的细节传输给正式的服务，应用开发者不需要管通信的细节。反之，像Linux IPC的共享内存，虽然不需要拷贝，性能特别高。但是使用起来特别复杂，应用开发者需要控制管理服务的内存。


至此，简单的讲述下了《Android的各大系统服务》，《如何获取这些系统服务：ServiceManager》，《如何通信：Binder体系》。当然，本篇只是描述下系统服务的最表层，背后还有许许多多的代码细节和设计者巧妙构思，后续出相关的文章和大家展示。
> 里面出现最多的字样就是C/S架构，大家在学习中牢记这个架构，Android底层所有的细节都围绕这个展开。



