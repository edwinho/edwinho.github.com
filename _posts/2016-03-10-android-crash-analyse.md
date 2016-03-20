---
layout: post
title: "Android崩溃分析"
category : [Android, game]
tags : [cocos2dx, Android, game]
---

### Crash以及分类

* Crash是指App非正常退出现象
* 分类：
  * Java层Crash
     - ANR(Application Not Responding) 应用无响应
     - Force Close 强制结束
  * Native层Crash
     - Tombstones
  * 系统服务崩溃（System Server Crash）
  * kernel Panics

<!-- more -->

### ANR(Application Not Responding)

* 发生场景
  * 应用长时间无响应
* 崩溃症状
  * 系统弹出窗口询问用户选择"Force Close"或者"Wait"
    "orce Close"将杀掉发生ANR的应用进程
    "Wait"将会等待系统择机恢复此应用进程
* 发生原因
  * 应用主线程卡住，对其他请求响应超时
  * 死锁
  * 系统反应迟钝
  * 内存，CPU，GPU负载过重
* 定位原因
  * 在线日志 DDMS/adb logcat 
  * 离线日志 data/anr/traces.txt 
  * 网络日志 第三方bug收集sdk
    腾讯bugly，Testin云测试
* 解决建议
  * 不要在主线程中进行耗时的操作，特别是IO相关如网络操作，读写DB等操作
  * 在内存和CPU方面遇到瓶颈的话，可以使用android提供的内存和CPU的调优工具进行定位相关代码和业务模块进行定优化 (traceview，systrace，Oprofile等)

### Force Close

* 发生场景
  * 应用程序崩溃
* 崩溃症状
  * 系统弹出窗口提示用户进程崩溃
  * 黑屏
* 发生原因
  * java层代码异常且未捕获
* 定位原因
  * 在线日志 DDMS/adb logcat – 离线日志 bugreport > /sdcard/bugreport.txt – 网络日志 第三方bug收集sdk
    腾讯bugly，Testin云测试
    自己写代码收集或开源项目
* 解决建议
  * 异常处理try catch
  * 非关键流程进行异常处理，出bug也不卡流程
  * 关键流程异常处理，输出日志，友好提示

### Tombstones

* 发生场景
  * Native层崩溃
* 崩溃症状
  * 进程收到错误信号，发生Crash，Android 5.0之前进程直接退出（闪退）, Android 5.0之后会弹“程序已崩溃”的对话框
* 发生原因
  * 由c++/c代码引起（NDK）
* 定位原因
  * 在线日志 DDMS/adb logcat – 离线日志 /data/tombstones/tombstone_nn
  * 网络日志 第三方bug收集sdk
* 腾讯bugly，Testin云测试
* 日志还原
  * 原始日志只有地址信息没有符号信息
  * 发布程序中的so是剥离符号表的(stripped symbols) 
  * 通过带符号表的so和NDK工具还原堆栈信息

### NDK介绍

* NDK（Native Development Kit）
  * 一套工具集(toolset)，允许在Android上进行native开发（即c/c++开发），工具集针对各个cpu架构提供了编译，符号还原和反编译工具
* 使用场景：
  * 已有c+/c++库利用
  * 性能热点（游戏引擎）
  * 代码保护

### NDK编译以及符号还原

* 编译（例子），编译luajit 
  * 先用make+makefile编译出libulua.a
    交叉编译，指定ABI（指令体系架构），以便生成对应的机器指令
    采用库本身的makefile，因为编译有顺序，如果直接用ndk-build编译，要指定文件编译顺序
  * 用Android.mk+ndk-build编译出libulua.so
    会导出不带符号和带符号的so
    ~ libs/armeabi-v7a/libulua.so
    ~ obj/local/armeabi-v7a/libulua.so

### 符号还原

* 用带符号表版本的so进行堆栈还原
  * 所以对于自己项目用到的so，一定要有剥离符号表版本的so和对应的未剥离符号表的so，否则出了问题也无法还原符号信息，无符号表定位比较困难（只能通过看汇编码来定位）
* cat /home/error.log | ndk-stack -sym $(project_loc)/obj/local/armeabi
* 反编译
  * arm-linux-androideabi-objdump

### native常见错误信号分析

* 常见错误信号
  * SIGILL
    信号量 4，机器指令错误，例如给函数指针赋了个非方法来执行（指针跑飞）
  * SIGSEGV
    信号量 11，段错误，非法方式访问了可访问的内存区域SIGBUS
    SEGV_ACCERR，对映射的对象没有权限
    SEGV_MAPERR，地址没有映射到对象
  * SIGBUS
    信号量 7，内存错误，非法方式访问了不可访问的内存区域
  * SIGFPE
    信号量 8，算数错误（除0）
  * SIGABRT
    信号量 6
    主动中止运行

### native常见错误

* 空指针
  * 在进程的地址空间中，从0开始的第一个页面的权限被设置为不可读也不可写
* 野指针
  * 野指针，未初始化，其指向的地址通常是随机的，有可能不会马上Crash，而是破坏了别处的内存
* 数组越界
  * 数据错乱且写了附近的内存区域
* 整数除以零
* 格式化输出参数错误
  * 数据错乱，读了附近的内存区域内容
* 主动抛出异常
  * 捕获到异常，主动报告终止
  * 动态库在内部运行出现错误时，大都会主动abort，终止运行

### 系统服务崩溃（System Server Crash）

* 发生场景
  * 系统服务是Android核心进程，此进程发生崩溃
* 崩溃症状
  * 手机重启到Android启动界面
* 发生原因
  * 系统服务看门狗发现异常
  * 系统服务发生未捕获异常
  * 系统服务Native发生Tombstone

### Kernel Panics

* 发生场景
  * Linux内核发生严重错误
* 崩溃症状
  * 手机从bootloader开始完全重启
* 发生原因
  * Linux内核内存空间发生内存崩溃
  * 内核看门狗发现异常

-------------------------------

注：调试步骤

例如Crash（崩溃）信息上场是：
02-25 20:07:44.612: A/libc(17865): Fatal signal 11 (SIGSEGV) at 0x00000010 (code=1), thread 17883 (Thread-3542)

用cygWin进入Android工程目录，然后用NDK调试
命令及参数：adb logcat|ndk-stack -sym 目录

![ndk-debug](http://edwinho.github.io/images/android/ndk-debug.png)

可以看到哪个文件哪一行发生了错误。