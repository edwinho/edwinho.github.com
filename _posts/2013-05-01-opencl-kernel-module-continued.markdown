---
author: edwin
comments: true
date: 2013-05-01 05:17:53+00:00
layout: post
slug: opencl-kernel-module-continued
title: OpenCL内核模块(续)
categories:
- 学习笔记
tags:
- OpenCL
---

OpenCL C提供了丰富的内建函数（built-in functions），其中有些内建函数的名称和 C语言的库函数相同，但其实现不同，大部分 OpenCL C built-in functions都是支持纯量 (scalar)变量和矢量 (vector)变量的。built-in functions按其功能可分为十余类，这里只介绍一些常用的以及 OpenCL C中独有的函数。

<!--more-->

在上一篇文章的入门例子hellocl的kernel中，用到了获取工作节点信息的 built-in functions，这种函数统称为work-item functions。其中使用了 get_global_id(uint dimindx)，此函数会返回当前工作节点在 dimindx维度（以 0作为起始值，依次递加1）上的位置（以 0作为起始值，依次递加 1）。OpenCL最多支持三维工作空间，所以 dimindx可以是0，1或2。在 hellocl中，用户通过 clEnqueueNDRangeKernel 创建了一个2维4*4大小的工作空间，可以通过 get_global_id(0)；get_global_id(1)分别得到当然工作节点在第一和第二维度上的坐标。

用户也可以通过 uint get_work_dim ()获得该 kernel工作空间的维度信息，该函数返回的维度以 1为起始值。同样的用户也可以通过 size t get_global_size (uint dimindx)来获得工作空间在每个维度上的长度。请看下面一个稍微复杂例子，它会自动分析 kernel工作空间维度和每个维度的长度，并将当前空间的坐标写入到 unint4类型的 buffer相应位置的 xyz分量上。其kernel函数如下：

    
    kernelSourceCode[] = KERNEL(
       __kernel void hellcl(__global uint4 *buffer){
            uint dim = get_work_dim();
            size_t gidx, gidy, gidz;
            size_t gsizx, gsizy, gsizz;
            if (dim == 1) {
                gidx = get_global_id(0);
                gsizx = get_global_size(0);
                buffer[gidx].x = gidx;
            }
            else 
                  if (dim == 2) {
                     gidx = get_global_id(0);
                     gidy = get_global_id(1);
                     gsizx = get_global_size(0);
                     gsizy = get_global_size(1);
                     buffer[gidx + gidy*gsizx].x = gidx;
                     buffer[gidx + gidy*gsizx].y = gidy;
                  }
                  else {
                     gidx = get_global_id(0);
                     gidy = get_global_id(1);
                     gidz = get_global_id(2);
                     gsizx = get_global_size(0);
                     gsizy = get_global_size(1);
                     gsizz = get_global_size(2);
                     buffer[gidx + gidy*gsizx + gidz*gsizy].x = gidx;
                     buffer[gidx + gidy*gsizx + gidz*gsizy].y = gidy;
                     buffer[gidx + gidy*gsizx + gidz*gsizy].z = gidz;
                  }
       }
    );


主函数(部分)：

    
    int main()
    {
    
    //平台初始化，建立command queue 和 kernel 对象
    
    //用户定义工作空间的大小
    #define XSIZE 16
    #define XSIZE 32
    #define XSIZE 64
    
        cl_uint4 outbuffer[XSIZE][YSIZE][ZSIZE];
        cl_mem outputBuffer = clCreateBuffer(
                                             context,
                                             CL_MEM_USE_HOST_PTR,
                                             XSIZE*YSIZE*ZSIZE*sizeof(cl_uint4),
                                             outbuffer,
                                             &status);
        if (status != CL_SUCCESS)
            return EXIT_FAILURE;
    
        hostmap = clEnqueueMapBuffer(command_queue,
                                 outputBuffer,
                                 CL_TRUE,
                                 CL_MAP_WRITE,
                                 0,
                                 XSIZE*YSIZE*ZSIZE*sizeof(cl_uint4),
                                 NULL,
                                 NULL,
                                 NULL,
                                 &status);
        memset(hostmap, 0, XSIZE*YSIZE*ZSIZE*sizeof(cl_uint4);
        status = clEnqueueUnmapMemObject(
                           command_queue, outputBuffer,
                           hostmap, NULL, NULL, NULL);
    
        size_t globalThreads[] = {XSIZE, YSIZE, ZSIZE};
        size_t localThreads[] = {4, 4, 4};
    
        status = clEnqueueNDRangeKernel (
                            commandQueue, kernel,
                            3, NULL, globalThreads,
                            localThreads, 0, NULL, NULL);
    
        //执行kernel ...
    
        hostmap = clEnqueueMapBuffer (
                              command_queue, outputBuffer,
                              CL_TRUE, CL_MAP_READ, 0, 
                              XSIZE*YSIZE*ZSIZE*sizeof(cl_uint4),
                              NULL, NULL, NULL, &status);
    
        //任选一点进行验证
    
        int xid = 2, yid = 2, zid = 3;
        printf("OutPut(%d, %d, %d).xyz = %d, %d, %d", 
                                     xid, yid, zid,
                                     outbuffer[zid][yid][xid].s[0],
                                     outbuffer[zid][yid][xid].s[1],
                                     outbuffer[zid][yid][xid].s[2]);
    
        status = clEnqueueUnmapMemObject(
                           command_queue, outputBuffer,
                           hostmap, NULL, NULL, NULL);
    
        //回收资源
    
        return 0;
    }


用户可以任意改变例中的维度和每个维度上的长度来测试，本例还指出用户如果需要在OpenCL API中访问一个 vector的分量如本例中 cl_uint4的第一二三个分量，只能使用 .s[n]其中 n为分量的数字坐标，其中 s必须为小写，xyzw访问方式被禁止，用户也不可以在API中一次访问多个分量.

进一步，当用户使用 get_global_id获取非法维度上的 id时，例如工作空间为二维而用户试图获得第三维（或三维以上）的 global id，函数将返回0，使用 get_global_size获取非法维度的长度是会返回 1。而上面例程中在执行kernel之前output初始化为0，因此原kernel等价于如下的简化形式:

    
    __kernel vod hellocl(__global uint4 *buffer)
    {
        size_t gidx, gidy, gidz;
        size_t gsizx, gsizy;
    
        gidx = get_global_id(0);
        gidy = get_global_id(1);
        gidz = get_global_id(2);
        gsizx = get_global_size(0);
        gsizy = get_global_size(1);
        buffer[gidx + gidy*gsizx + gidz*gsizx*gsizy] = (uint4)(gidx, gidy, gidz, 0);
    }




其他work-Item functions包括：<br/>
size_t get_local_size (uint dimindx) 用来得到每个工作组在相应维度上的长度;<br/>
size_t get_local_id (uint dimindx) 用来的到在相应维度上的local id;<br/>
size_t get_num_groups (uint dimindx) 用来得到在该维度上有多少个工作组；<br/>
size_t get_group_id (uint dimindx) 用来的到在相应维度上的group id。<br/>


**第二类 build-in functions是math functions即数学计算类内建函数**

其中包括三角函数，反三角函数，幂、指、对数函数，误差互补函数，比较大小函数等等。这类 math functions的输入值必须为 float 或floatn类型，其中n可以是 2，4，8，16，并且各个输入参数的类型和返回结果的类型除特殊规定外必须相同。其中 vector即floatn类型的数据是按每个分量进行计算 的。例如比较大小函数fmax:

    
    float8 a = (float8)(1.0f, 2.0f, 3.0f, 4.0f, 5.0f, 6.0f, 7.0f, 8.0f);
    float8 b = (float8)(5.0f, 5.0f, 5.0f, 5.0f, 5.0f, 5.0f, 5.0f, 5.0f);
    
    float8 c = fmax(a, b);
    // fmax会比较a, b每个分量的大小， 将大者返回给 c 的相应分量
    // c = ((5.0f, 5.0f, 5.0f, 5.0f, 5.0f, 6.0f, 7.0f, 8.0f)


在以下几种情况下 math functions所使用的变量类型可以或必须不同，请看以下例子:

    
    float4 a;
    float b;
    // fmax 和 fmin 可以将vertor类型和scalar类型的 float 数据进行比较，但scalar数据必须为第二个参数
    float4 c = fmax(a, b);
    //这等同于 c = fmax(a, (float4)(b, b, b, b) );
    
    int4 n;
    float4 d = ldexp(x, n);
    //ldexp(x, n) 返回 x 和 2 的 n次幂的乘积，这里 n 必须为 int或与 x等长的 int vertor
    //与ldexp类似的函数有 rootn(x, n) 返回 x的 n次方根; pown(x, n)返回 x的 n次幂




OpenCL C还在此基础上提供了一些 native math functions，它们都使用统一的前缀 native。它们包括 native\_cos，native\_sin，native\_exp等等，相应OpenCL device会用一条或几条机器指令完成这些函数。native functions一般要远远快于相对应的 math functions，但 native functions对输入参数要求和对应的内建函数不同，且其输出精度一般比对应的非 native函数要低，不同的OpenCL实现精度各不相同。<br/>
此外OpenCL C也提供了一些以half\_为前缀的数学函数，这些函数如同基本的数学函数一样可以使用 float和 floatn最为参数，结果要比对应的数学函数精度低，但相差的ULP value <= 8192 ulp。同时half\_函数可以使用 half类型数据作为输入输出参数。



**第三类为整数类函数（ Integer Functions**）

这里函数的输入输出参数必须为 char， charn，uchar， ucharn， short， shortn， ushort， ushortn， int， intn， uint， uintn， long， longn ulong，或者 ulongn。n为2，4，8或16。这里同样除特殊规定外输入输出参数的类型必须相同，如果是 vector的话，长度也必须相同。

除了基本的整数数学函数例如，绝对值函数 abs(x)，比较大小函数max(x，y)，min(x，y)之外，OpenCL C还新增了如下的内建整数函数（build-in integer functions）：<br/>
hadd (x， y)按每个分量计算(x + y)>>1后返回；<br/>
rhadd(x， y)按每个分量计算(x + y +1)>>1后返回；<br/>
mul_hi(x， y)按每个分量计算 x乘y，并返回乘积的高位部分例如两个ulong(64bits)相乘后的乘积为一个 0-128bit的整数，取其高 64位作为返回值，普通mul(x，y)返回的实际是乘积的低64位。<br/>
mul24 (x， y)，只取 x，y的低 24位相乘，在 x，y高于 24的情况下乘积的结果在不同的OpenCL实现上是不同的；<br/>
rotate(v， i)，将 v中每个分量左移 i中相应分量规定的位数，左移溢出的部分从v中该分量的右侧移入；<br/>
upsample (hi， lo)，按分量返回 hi<<sizeof(hi)|lo。这里要求 hi和lo位数必须相同，其中 lo必须为无符号类型数据，返回的数据类型长度必须为hi或lo长度的两倍，如果 hi为无符号类型则返回值也为无符号类型，反之hi为有符号类型，返回值也为有符号类型，且显然符号与 hi相同。例如，hi为short4，则 lo必须为 ushort4，返回值类型必须为 int4，且返回值按分量的符号与hi相应分量的符号相同。



**第四类为几何计算类函数（Geometric Functions）**

这里函数是在几何计算中经常被用到的，包括：float4 cross (float4 p0， float4 p1)，计算 p0与p1的外积。这里认为 p0，p1的前三个分量组成了两个三维坐标，计算这两个向量的外积，将外积坐标返回到返回值的前三个分量中，返回值的第四个分量是 0，输入值的第四个分量在这个函数中没有意义。这个函数的输入输出必须是float4.<br/>
float dot (p0， p1)为相应的内积函数，其中 p0 p1为float，float2或float4以下几何函数中没有特殊标示的变量与此相同。返回为 p0中各分量与 p1中对应分量乘积求和，故为纯量浮点数。<br/>
float length(p)，返回以 p的各分量为坐标的向量长度，即 p中个分量平方求和再开方。<br/>
float fast_length (p)，使用half_sqrt做开方运算。<br/>
float distance (p0， p1)返回向量p0-p1的长度。<br/>
float fast_distance (p0，p1)使用fast_length计算p0-p1的长度。<br/>
normalize(p)，正规化向量 p，即返回方向与 p相同但长度为 1的向量，即p的各分量除以length(p)后返回。<br/>
fast_normalize(p)使用fast_length计算p的长度。



**第五类向量型数据读取和储存函数（ Vector Data Load and Store Functions）**

OpenCL C提供了 vloadn和vstoren函数来按指针和偏移来读取和储存vector类型的数据。这里n必须为2，4，8，16之一。<br/>
gentypen vloadn (size_t offset，gentype \*p) vloadn所使用的指针 p应该为scalar类型指针，即gentype\*应该是 char\*，uchar\*，short\*，ushort\*，int\*，uint\*，long\*，或者ulong\*。该函数会从 (p + (offset\* n))指针位置抓取n个gentype类型的数据组成gentypen类型的vector数据返回。这里要求 p必须对sizeof(gentype)对齐。如果 p为vector指针，不会产生变异错误，但函数的返回值有可能因为不同的OpenCL实现而不同。<br/>
void vstoren (gentypen data， size_t offset， gentype \*p)该函数将data值储存到(p + (offset * n))指向的位置。p指针类型gentype\*和要求都与vlordn函数相同，不同的是这里的p指向的内存一定不能是__constant类型。<br/>
half目前不要求所有的实现都必须支持 halfn的vector类型和动态的指针赋值，或指针访问。用户可以通过内建的 vload_half，vstore_half，vload_halfn，vstore_halfn及vloada_halfn，vstorea_halfn来进行指针操作。其中，
float vload_half (size_t offset，half \*p)会将 (p + offset)处的 half值转化为float类型后返回。这里要求p必须对16对齐。<br/>
floatn vload_halfn (size_t offset，half \*p)会读取 (p + (offset \* n))位的n个half值并将转化为float值后按顺序组成一个 floatn类型的数据返回， p同样必须对16对齐。<br/>
floatn vloada_halfn (size_t offset，half \*p)与vload halfn所完成的任务相同，但要求p必须16\*n对齐。该函数效率有可能比vload_halfn高。<br/>
store half类的函数大致与load half类函数相同，
float vstore_half (float data，size_t offset，half \*p)会将 data转化成 half类型后写入(p + offset)处，这里要求p必须对16对齐。<br/>
floatn vstore_halfn (floatn data，size_t offset，half \*p)会将data中的每个分量转化成half类型后依次写入(p + offset)处，这里要求p必须对16对齐。<br/>
floatn vstorea_halfn (floatn data，size_t offset，half \*p)与vstore_halfn所完成的任务相同，但要求p必须对16\*n对齐。<br/>
与vload half不同的是 data从float转化为 half是可能损失精度的，上面介绍的是按默认的取最靠近的偶数（ round to nearest even）方法进行转换的。OpenCL C按不同的转化方法又提供了其他的 store half(n)的方法。

**第六类同步函数（Synchronization Functions）**

OpenCL C提供了同步函数来进行一个工作组空间 (work group)上不同节点间的同步，包括：

void barrier (cl_mem_fence_flags flags)，只有工作组中所有的工作节点都执行完该函数之后才能进行后续的操作。这就要求用户不能将该函数放置在条件语句当中，除非用户确保所有的工作节点都满足或都不满足该条件语句，不过如果这样条件语句本质上也就没有意义了。同时由于local buffer是在每个工作组内共享的，global buffer是在所有工作组节点间共享访问的，所以不同节点之间对同一块buffer的操作也可以能需要同步。用户可以将 barrier中的 flag设成 <br/>CLK_LOCAL_MEM_FENCE来同步同一工作组内barrier之前所有对local buffer的读写操作，或设成 <br/>CLK_GLOBAL_MEM_FENCE来同步之前所有对global buffer的读写操作。

OpenCL C同提供了更灵活的内存同步操作。void mem_fence (cl_mem_fence_flags flags)用来同步在此之前flags类型buffer的读写操作。<br/>
void read_mem_fence (cl_mem_fence_flags flags)用来同步在此之前flags类型buffer的读操作。<br/>
void write_mem_fence (cl_mem_fence_flags flags)用来同步在此之前flags类型buffer的写操作。



**第七类是 OpenCL C中提供的用来处理 local buffer与global buffer之间异步拷贝的函数**

event_t async_work_group_copy(*dst，*src，size_t num_elements，event_t event)

其中 des和src可以是 OpenCL C支持的任意数据类型，包括 vector类型的指针，这里要求 des和src的所指向的数据类型必须相同，其所在区域或者为 global或者为 local但必须不同，即 des如果是 global buffer则src必须是local buffer，反之 des是local buffer，src必须为 global buffer。值得注意的是num_ elements并不是 copy内存的长度，而是指有多少个这种类型的数据需要 copy，即实际的 copy的长度应为 sizeof(type of des)\*num elements。其返回值为该异步 copy的事件对象，用户可以使用 wait _group_events (int num_events，event_t *event_list)来同步一项或者多项异步 copy事件。用户也可以将其他异步 copy的事件作为 async_work_group_copy的event，这时event被认为是其他的异步 copy与本次 copy之间公用的事件对象，当该事件结束时所有相关的异步 copy，包括本次 copy都将结束。在此情况下返回的事件对象就是输入的 event。否者用户需将输入 event设为 0。另外必须注意的是在一个工作组内部的所有节点都必须同时调用到该函数，并且各个参数也必须相同，否者copy结果是不保证正确的。

另外OpenCL C也提供了global buffer的缓存函数prefetch (global *p，size_t num_elements)这里p必须是global buffer指针，num_elements同样是copy的数据元素的个数。OpenCL C还提供其他丰富的内建函数，由于和 C语言的库函数类似就不一一介绍了，可以参考[OpenCL Specifications](http://www.khronos.org/registry/cl/specs/opencl-1.0.29.pdf)查看细节。
