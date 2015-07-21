---
author: edwin
comments: true
date: 2013-04-30 02:55:24+00:00
layout: post
slug: opencl-kernel-module
title: "OpenCL内核模块"
categories:
- codes
tags:
- OpenCL
---

首先介绍一下什么是[OpenCL](http://www.khronos.org/opencl/)，Open Computing Language C程序语言简称OpenCL C，是用来编写运行在OpenCL设备上的kernel程序的一门语言。OpenCL C基于ISO/IEC 9899:1999 C标准做了一定的扩展和限制。

<!-- more -->

以下面的kernel函数作为入门例子：

    
    __kernel void hellocl ( __global uint * buffer) {
         size_t gidx = get_global_id(0);
         size_t gidy = get_global_id(1);
         buffer [( gidx +4* gidy )]=((1 << gidx ) |(0 x10 << gidy));
    }


OpenCL C比C多出一些函数限定符，例子中用到的 \__kernel是所有OpenCL C中必须用到的。函数加注 \__kernel表示其只能运行在OpenCL设备上，并且该函数可以被host调用，kernel中的其它函数也可以像调用普通函数一样调用含有该前缀的函数。

OpenCL C还增加了一些地址空间限定符包括 \__global，\__local，\__constant和 \__private。这些地址限定符可以用来声明变量，以表明该变量对象所使用的内存区域。以上述地址空间限定符声明的对象会被分配到指定地址空间，否者会被分配到通用地址空间。

\__global或简写为 global，表示该声明内存对像使用的是从全局内存空间分配出来的内存，工作空间内所有工作节点都可以读 /写这块被声明的内存。在上面的例子中 buffer就被加注了global限定符，所以 kernel内部可以自由访问buffer。

\__constant或简写为 constant，与global相同，使用从全局内存空间分配出来的内存，可以在所有工作节点做只读访问。在OpenCL C中试图对constant对象进行写操作会产生编译错误。

\__local或简写为 local，声明内存对像使用的是 local memory pool中的内存，仅能在同一个工作组的不同工作节点之间访问。

\__private或简写为 private，所有在 OpenCL C函数内部使用的变量或者传入函数内部的参数都是 private类型，用户在声明 private类型的变量时可以省略此限定符（因此无地址空间限定符的变量都属于此类型）。private类型的对象仅可以在其所在的工作节点内部使用。例如例中的 gidx在每个节点上都是独立存储使用的，不能交互使用。

OpenCL C中支持 C语言中原有的一些数据类型如 bool，char，float等。其中 double和longlong等数据类型可以不支持或是选择性支持。OpenCL C中还增加了一些新的数据类型，其中包括：
half类型。OpenCL C中的 half类型是和 IEEE 754\-2008规范兼容的，使用16bits来表示浮点数据。将 half类型的数据转换成 float型是无损的，但将float类型转化为 half类型则可能损失精度。

除了 half类型，OpenCL还支持内建的矢量数据类型（Vector Data Types），可以建立 vector类型的数据类型包括 char，unsigned char，short，unsigned short，integer，unsigned integer，long，unsigned long和float，在 OpenCL C中用户可以定义charn，ucharn，shortn，ushortn，integern，uintegern，longn，ulongn和floatn型的矢量数据，其中 n为2，4，8或16。这些矢量类型在OpenCL API中可以使用cl_charn，cl_charn，cl_shortn，cl_ushortn，cl_integern，cl_uintegern，cl_longn，cl_ulongn和cl_floatn来声明。用户可以利用矢量数据类型文字来初始化相应的矢量数据或者在执行语句中定义该类型的常量。例如下面的例子：

    
    float4 f = (float4)(1.0f, 2.0f, 3.0f, 4.0f);
    uint8 u = (uint8)(1); //这样是正确的，u的各个分量被定义为1
    float4 f = (float4)( (float2)(1.0f, 2.0f), (float2)(3.0f, 4.0f) )   
    float4 f = (float4)(1.0f, 2.0f); //这是错误的，表达式中的数据个数与声明不匹配


用户可以使用 . 操作访问一个vector数据的各个分量。在 OpenCL中vector数据的前四个分量依次标识为 x，y，z，w分量，如果该矢量数据只有两个分量，例如 uchar2，则只有 .xy分量是可以有效访问的，具有四个或四个以上分量的 vector数据可访问所有 .xyzw分量。OpenCL C允许用户将不同分量的 名叠加在 .操作符后来一次访问多个分量，并且多个分量不必须按 xyzw顺序排列。请看下面的例子：

    
    float4 a;
    a.xyzw = (float4)(1.0f, 2.0f, 3.0f, 4.0f);
    a.z = 3.0f; //可以单独给一个分量赋值
    a.xy = (float2)(4.0, 3.0); // a = (4.0f, 3.0f, 3.0f, 4.0f)
    a.zw = (float2)(2.0f, 1.0f); // a= (4.0f, 3.0f, 2.0f, 1.0f)
    float4 b = a.wzyx; // b = (1.0f, 2.0f, 3.0f, 4.0f)
    float4 c = a.xxyy; // c = (4.0f, 4.0f, 3.0f, 3.0f)
    c = a.xx; //这是非法的，左边的float4和右边的float2不匹配




用户也可以使用各个分量的数字坐标来访问 vector数据的分量，其第5-16个分量只能通过这种方法访问。vector分量的数值坐标依次为 0， 1， 2， 3， 4， 5， 6， 7， 8， 9， a， b， c， d， e， f
不区分大小写，访问时必须加入前缀 s（或S）。用户同样可以通过在 s后叠加不同分量的数字坐标来同时访问多个分量，但数字坐标不能和字母坐标xyzw混用。请看下面的例子：

    
    float16 d;
    d.x = 0.0f; // 等同d.s0
    d.sa = 10.0f; // 为第11个分量赋值
    d.s2cfd = (float4){2.0f, 12.0f, 15.0f, 13.0f}; // 为第2、13、16、14个分量赋值
    d.x2 = (float2){0.0f, 2.0f} // 非法，x 和 2不能同时使用


OpenCL C提供了其他高效的方法来访问和使用 vector数据，它们包括.lo用来访问 vector中前半段数据，.hi用来用来访问 vector后半段数据，.odd用来访问 vector中所有坐标为奇数的数据，.even用来访问 vector中所有坐标为偶数的数据。如下例:

    
    float16 v;
    float8 low = v.lo; // low = v.s01234567
    float8 high = v.hi; // high = v.s89abcdef
    float4 leven = low.even; // leven = low.s0246
    float4 lodd = low.odd; // lodd = s1357
    float8 e = low.lo; // 非法，low.lo的长度为4个float


C语言中大部分的操作符都可以在 OpenCL C中直接使用，但 half类型的变量只能使用 sizeof操作。对于 vector变量，所有操作都是按每个分量进 行的操作的。所以要求操作数据的分量个数相同，scalar （纯量数据即单分量数据）在运算是可以被自动转化成相应的各分量值相同的 vector变量，下例：

    
    float4 v, u, w;
    float f;
    v = u + f;
    /*
    OpenCL C会自动将+右侧的 f 转化为(float4)(f, f, f, f)
    所以 v = u+f 就等同于：
    v.x = u.x + f;
    v.y = u.y + f;
    v.z = u.z + f;
    v.w = u.w + f;
    */
    
    w = v + u;
    /*
    w = v + u 同样等同于下面按分量分别相加：
    w.x = v.x + u.x;
    w.y = v.y + u.y;
    w.z = v.z + u.z;
    w.w = v.w + u.w;
    */
    
    f = w + f; // 非法，OpenCL C不能把 = 右侧的float4 转换为float


关于运算操作的细节描述请参看[OpenCL Specifications](http://www.khronos.org/registry/cl/specs/opencl-1.0.29.pdf)
