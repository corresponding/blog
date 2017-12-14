---
title: 追溯Android的根源
date: 2017-11-28 19:33:32
tags: [底层, Android]
categories: Android
---


{% asset_img android-initiate_head.jpg 创世纪 %}

某日进去一小区被保安拦住，被问了三个哲学问题：
“你是谁？”
“你从哪里来？”
“你要到哪里去？”
于是我陷入了深深的沉思。

同样，学习Android也有三个终极问题：
什么是Android？
Android世界的起源？
Android中APP是怎么运行的？

对于第一个问题，大家都能很快的回答出来。简单来说，Android是一种基于Linux，主要用于移动设备的操作系统。
第三个问题这样回答：APP的屏幕显示、输入事件获取、视频、音频等各个功能，在Android底层都有其对应的系统服务。
当然其中涉及到ActivityManagerService，WindowManagerService等等，后续有空会展开分析。
今天，我们来好好地看下第二个问题，Android系统是如何启动的，探讨下Android系统的起源。

---

# 启动

## main()
就像我们编写的第一行代码helloworld一样，main()是helloworld世界的入口。
在Android中，main()也是Android世界的第一个入口，这个入口的位置在system\core\init\init.cpp。

main()主要工作：
    1.参数校验
    2.挂载一些设备，设置权限（与传统Linux程序相同）
    3.初始化环境
    4.LoadBootScripts()加载待启动项
    5.处理待启动项
    
## LoadBootScripts()
作用就是，加载引导的脚本init.rc，所有开机需要启动的配置读写在这里面。
像Android这样完善而又庞大的系统，在开机时肯定要启动有无数的服务。
如果全部都写到函数中，想必会变成硬编码，不方便于配置，所以这里用脚本方式配置启动内容。
```
static void LoadBootScripts(ActionManager& action_manager, ServiceList& service_list) {
    // 生成解析器
    Parser parser = CreateParser(action_manager, service_list);
    // 解析文件内容后，放入action_manager和service_list中
    parser.ParseConfig("/init.rc");
}
```

## init.rc
该文件放在system\core\rootdir目录下
节选下其中重要的片段
```
service zygote /system/bin/app_process -Xzygote /system/bin --zygote --start-system-server
    class main
    priority -20
    user root
    group root readproc
    socket zygote stream 660 root system
    onrestart write /sys/android_power/request_state wake
    onrestart write /sys/power/state on
    onrestart restart audioserver
    onrestart restart cameraserver
    onrestart restart media
    onrestart restart netd
    onrestart restart wificond
    writepid /dev/cpuset/foreground/tasks
```
这段代码表示：启动名为zygote的Service。zygote翻译过来是“受精卵”，可以孕育出新生命。
在Android中，zygote也是万物的起源。
这段代码清晰的指出了Zygote的代码入口：app_process。


## app_process
上面同样表示app_process是在目录/system/bin/中，不过因为这应该是软链接后的位置，
真实的位置在frameworks\base\cmds\app_process\App_main.cpp，而且入口也是main()。
这里截取其中的重要代码
```
int main(int argc, char* const argv[]) {
    // 这里有两个变量去控制新进程的诞生
    bool zygote = false;                // 是否为zygote进程
    bool startSystemServer = false;     // 是否启动系统服务，例如ActivityManagerService等

    while (i < argc) {
        const char* arg = argv[i++];
        if (strcmp(arg, "--zygote") == 0) {
            zygote = true;
        } else if (strcmp(arg, "--start-system-server") == 0) {
            startSystemServer = true;
        } 
    }
    // 按照之前的入参--zygote --start-system-server分析，发现zygote和startSystemServer都被设置为true
    
    if (zygote) {   
        runtime.start("com.android.internal.os.ZygoteInit", args, zygote);  // 终于接近zygote真实本体了！
    } 
}
```
这里看下runtime，是AppRuntime类。我们赶紧去寻找下它的start方法吧。
发现AppRuntime继承于AndroidRuntime，AndroidRuntime中提供start()方法


## AndroidRuntime
位于frameworks\base\core\jni\AndroidRuntime.cpp中
```
void AndroidRuntime::start(const char* className, const Vector<String8>& options, bool zygote) {
    /*
    * 开启virtual machine，这里env是JNIEnv*类型，指向整个Android的JNI世界
    */
    startVm(&mJavaVM, &env, zygote);
    /*
    * 给虚拟机注册JNI的函数
    */
    startReg(env);
    /*
    * 调用JNI static方法，启动zygote
    * 这里startClass是com.android.internal.os.ZygoteInit类
    * startMeth是startClass中的main()方法
    */
    env->CallStaticVoidMethod(startClass, startMeth, strArray);
}
```

### startVm()
这个函数有一超级长的check，最关键是最后一句JNI_CreateJavaVM(pJavaVM, pEnv, &initArgs);
完成这个函数后，VM正式创建成功，后续可以开始通过JNI对native底层代码进行调用了。

### startReg()
这个函数中，最重要的是register_jni_procs(gRegJNI, NELEM(gRegJNI), env)
其中gRegJNI表示待注册的JNI函数，截取一段给大家看看
```
static const RegJNIRec gRegJNI[] = {
    REG_JNI(register_android_os_Process),
    REG_JNI(register_android_os_SystemProperties),
    REG_JNI(register_android_os_Binder),
    REG_JNI(register_android_os_Parcel),
```
发现原来Parcel、Binder等等Android独有的特性都在此处被注册，后面就可以通过JNI方式直接调用。

### env->CallStaticVoidMethod
根据这个函数的入参，可以得知最后调用com.android.internal.os.ZygoteInit的main()函数。
从类名上可以推测出，这负责zygote的生成。

## zygoteInit.main()
对应的文件为frameworks\base\core\java\com\android\internal\os\ZygoteInit.java
```
public static void main(String argv[]) {
    // 注册server socket，这里采用socket方式和其他进程通信
    zygoteServer.registerServerSocket(socketName);
    // 加载classes,opengl,textsource等各种资源
    preload(bootTimingsTraceLog);
    // 开启新进程，用以启动system server
    if (startSystemServer) {
        Runnable r = forkSystemServer(abiList, socketName, zygoteServer);
        r.run();
        return;
    }
    // 初始化zygote
    caller = zygoteServer.runSelectLoop(abiList);
    caller.run();
}
```
说了这么多，大家一定被绕晕了吧，这里整理一张流程图，大家可以对着看。
{% asset_img android_initiate.png Android启动流程图 %}

这里走向两个分支，我们接下去也分开去介绍zygote和System server初始化。


# zygote
zygote，这是一个平凡而又伟大的进程，以后Android的所有进程都由他诞生。
我们继续来看下zygoteInit.main()中走向zygote分支的流程，后续调用zygoteServer.runSelectLoop()。

## zygoteServer.runSelectLoop()
对应的文件为frameworks\base\core\java\com\android\internal\os\zygoteServer.java
其主要功能是，不断接受请求并作出响应。
```
Runnable runSelectLoop(String abiList) {
    // 不断循环，接受并处理新请求
    while (true) {
        // 监听数据源
        Os.poll(pollFds, -1);
        for (int i = pollFds.length - 1; i >= 0; --i) {
            // 解析出数据源的命令
            ZygoteConnection connection = peers.get(i);
            final Runnable command = connection.processOneCommand(this);
            return commond;
        }
    }
}
```
zygote是采用poll的方式去监听数据源。
注：poll 是 Linux API 提供的复用方式。IO多路复用是指内核一旦发现进程指定的一个或者多个IO条件准备读取，它就通知该进程。
至此，zygote会处于runSelectLoop()的循环中，不断监听外部请求并作出响应，fork出进程。



# system server
在android的启动过程中，我们也需要启动各种应用服务，我们来找下有他们是怎样诞生的。

## forkSystemServer()
这个也在zygoteInit.main()中被调用到，上面已经有提及，这里就不重复列出了。
在这里发现了zygote的作用，zygote把自己的进程fork一份，用来启动各大系统服务。
zygote进程就像受精卵一样，慢慢分裂诞生出整个Android系统，并且在后续中，新进程的诞生也依赖与zygote的fork。
截取其中的重要代码
```
private static Runnable forkSystemServer(）{
    pid = Zygote.forkSystemServer(
                    parsedArgs.uid, parsedArgs.gid,
                    parsedArgs.gids,
                    parsedArgs.runtimeFlags,
                    null,
                    parsedArgs.permittedCapabilities,
                    parsedArgs.effectiveCapabilities);
                    
    if (pid == 0) {
        return handleSystemServerProcess(parsedArgs);
    }
}
```

## handleSystemServerProcess()
handleSystemServerProcess()这个函数完成了fork后的剩余工作
```
private static Runnable handleSystemServerProcess(ZygoteConnection.Arguments parsedArgs) {
    // 加载system server dex的option
    performSystemServerDexOpt(systemServerClasspath);
    // 启动system server 
    return ZygoteInit.zygoteInit(parsedArgs.targetSdkVersion, parsedArgs.remainingArgs, cl);
}
```

## ZygoteInit.zygoteInit()
从函数命名来看，这是各大应用服务的初始化地址。
```
public static final Runnable zygoteInit(int targetSdkVersion, String[] argv, ClassLoader classLoader) {
    // native层的初始化system server
    ZygoteInit.nativeZygoteInit();
    // 返回java层初始化system server的runnable，后续必然有run()的操作
    return RuntimeInit.applicationInit(targetSdkVersion, argv, classLoader);
}
```

## RuntimeInit.applicationInit()
越来越接近真相了，再坚持一会。
```
protected static Runnable applicationInit(int targetSdkVersion, String[] argv, ClassLoader classLoader) {
    // 未来app调用exit()会直接退出，不会清理进程
    nativeSetExitWithoutCleanup(true);
    // 寻找最终的类以及方法
    return findStaticMain(args.startClass, args.startArgs, classLoader);
}
```

## findStaticMain()
从函数命名可以看出，这回找到服务的main()函数并调用。
这里说明良好的说明有多重要，可以让其他开发者在阅读纯代码时就能get到函数的含义。
这里列下findStaticMain()的重要片段
```
private static Runnable findStaticMain(String className, String[] argv, ClassLoader classLoader) {
    Class<?> cl = Class.forName(className, true, classLoader);          // 找到对应的类SystemServer
    Method m = cl.getMethod("main", new Class[] { String[].class });    // 找到其中的main()函数
    return new MethodAndArgsCaller(m, argv);     
}
```

## MethodAndArgsCaller
这是一个实现runnable的类，里面主要的run()就是去执行之前找到的method
```
static class MethodAndArgsCaller implements Runnable {
    public void run() {
        mMethod.invoke(null, new Object[] { mArgs });  
    }
}
```

## ZygoteInit.main()
我们继续回到ZygoteInit.main()
```
public static void main(String argv[]) {
    // 回到最开始出发的地方，这里r
    Runnable r = forkSystemServer(abiList, socketName, zygoteServer);
    // 接下去果然是run，其中运行SystemServer.main()
    if (r != null) {
        r.run();
        return;
    }
}
```

## SystemServer.main()
这个函数里面就一句话，调用同一个类的run()
```
public static void main(String[] args) {
    new SystemServer().run();
}
```

## SystemServer().run()
看来这个是SystemServer的启动地了，看下其中的关键代码
```
private void run() {
    // 准备主线程的Looper
    Looper.prepareMainLooper();
    // 加载native的服务
    System.loadLibrary("android_servers");
    // 开启各项系统服务
    startBootstrapServices();
    startCoreServices();
    startOtherServices();
    // 开启Looper循环，不断监听事件
    Looper.loop();
}
            
```
至此，system server全部启动成功，并且开启Looper循环，会不断监听外来的请求并响应。

这里也梳理下system server整个启动过程。
{% asset_img android_systemserver.png system server 启动流程图 %}


---

我们已经清晰的梳理出Android的大致的启动流程。
可以结合上一篇文章《浅谈Android底层》一起看。上篇文章，在用户角度从表面慢慢推进到底层system server（包括ServiceManager、AMS、WMS等等）。再进一步，就是追溯这些zygote和system server的起源。

作为应用开发者，只需要完成两部：
1.配置AndroidManifest.xml的启动项
2.实现对应的Activity
就能完成最简单的应用。但是我始终会好奇其背后的神秘而又精密的机制。

我们身上始终留着追溯根源的血液。就像在遥远的古代，刀耕火种的人类在漫天星空下去追溯祖先的由来，构想了无数种文明的起源。此时，作为应用开发者，我们也在追溯Android系统的起源。



