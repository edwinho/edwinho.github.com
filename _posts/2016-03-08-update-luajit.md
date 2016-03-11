---
layout: post
title: "luajit升级到2.1版本，支持64位"
category : game
tags : [cocos2dx, Android, iOS, game]
---

#### 问题：

　　由于从2015.2.1开始所有App Store商店上的应用都必须支持64位，所以星域帝尊的Cocos2d-x版本升级到2.2.6的同时，用于编码的luajit也要升级到支持64位。iOS版本的星域帝尊中用到的luajit升级到2.1后，发现发布机上（即更新游戏前端模块代码和资源）仍旧是使用旧版本的luajit，导致游戏不能读取更新的差异包中不兼容的字节码。

* 32位的luajit和64位的luajit生成的字节码不兼容，所以无法使用一份字节码同时运行在iOS的32位和64位设备。
* 因为Android应用商城没有强制要求必须上传64位的程序，所以没有在Android系统使用luajit arm64版本，因为他们的字节码是不兼容的。

<!-- more -->

#### 修复方案：

　　升级发布机上的luajit，支持64位字节编码。


#### 过程:

　　从http://luajit.org/官网上下载2.1版本的luajit，在发布机上安装64位的luajit环境。但是发布机是同时处理iOS和Android的，所以升级后，导致Android版本的不能读取差异包（cannot load incompatible bytecode），所以后来修改处理方案：加上升级Android版本的luajit。不过Android的有点麻烦，执行build_android.sh生成新的libluajit.a要有gcc、ndk、GUN什么鬼环境，最后生成好，重新编译Cocos2d-x，再生成新的APK包。
