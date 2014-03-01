---
author: edwin
comments: true
date: 2013-07-25 11:36:36+00:00
layout: post
slug: const-data-summary
title: const型数据的小结
categories:
- 学习笔记
tags:
- C++
---


由于与对象有关的const型数据种类较多，形式又有些相似，往往难以记住，所以正面就归纳了一下。为了便于理解，以具体形式表示，对象名设为Time。

<!--more-->

1. `Time const p`或`const Time p`
	p是常量对象，其值在任何情况下都不能改变

2. `void Time::foo() const`
	foo是Time类中的常量成员函数，可以引用，但不能修改本类中的数据成员

3. `Time *const p`
	p是指向Time对象的常指针，p的值（即p的指向）不能改变

4. `const Time *p`
	p是指向Time类对象的常指针，其指向的类对象的值不能通过指针来改变

5. `Time &p = q`
	p是Time类对象q的引用，q和p指向同事一段内存空间

其中最后一行是对象的引用，不属于const型数据。
