---
layout: post
title: "Cocos2d-x内存管理机制"
category : game
tags : [cocos2dx, C++]
---

[TOC]

　　Cocos2d-x的内存管理机制实际上来源于Objective-C，这套机制几乎贯穿Cocos2d-x引擎中所有对动态分配内存的管理，它使管理动态分配到堆上的对象的工作更加简单和高效。但是，它独特的工作机制也使一些开发者，尤其是不熟悉Objective-C的开发者对其形成一些“误解”。

<!-- more -->

### 1、C++显式内存管理

　　C++使用new关键字在运行时给一个对象动态分配内存，并返回堆上内存的地址供应用程序访问，被动态分配的内存需要在对象不再被使用时通过delete运算符将其内存释放，归还给内存池。

　　显式的内存内存管理在性能上有一定优势，但是极易出错。一方面，直接访问内存地址提高了应用程序的性能及内存使用的灵活性；另一方面，我们总是无法通过人的思维去保证一个逻辑的正确，由于程序没有正确地分配与释放而造成的野指针、重复释放、内存泄漏等问题又严重影响着应用程序的稳定性。 

* 野指针：指针指向的内存单元已经被释放，但是其它指针可能还指向它，这些内存可能已经被重新分配给其他对象，从而导致无法预测的结果。
* 重复释放：重复释放一个已经被释放的内存单元，或者释放一个野指针（也是重复释放）都会导致C++运行时错误。
* 内存泄漏：不再被使用的内存单元如果不被释放，就会一直占用内存单元，如果这些操作不断重复，就会导致内存占用不断增加。在游戏中，内存泄漏引发的问题尤其严重，因为可能每一帧都在创建一个永远不会被回收的对象。

### 2、C++11中的智能指针
　　根据分配内存的方法，C++有3种管理数据内存的方法，分别是自动存储、静态存储和动态存储。其中，静态存储用于存储一些在整个应用程序执行期间都存在的静态变量，动态存储用于存储通过new关键字分配的内存单元。

　　而对于在函数内部定义的常规变量，则使用自动存储空间，其对应的变量称为自动变量。自动变量在所属的函数被调用时自动产生，在该函数结束时消亡。实际上，自动变量是一个局部变量，其作用域包含它的代码块所对应的作用域。自动变量通常存储在栈上，这意味着进入代码块时，其中的变量将依次加入栈中，而在离开该代码块时，按与加入时相反的顺序释放这些变量。所有自动变量的内存总是能够被正确的释放。

　　由于自动变量通常不会导致内存问题，所以智能指针试图通过将一个动态分配的内存单元与一个自动变量关联，让这个自动变量在离开代码块并被自动释放的时候释放其管理指针的内存单元，这使程序员不再不需要显式地调用delete运算符就可以很好地管理动态分配的内存。

　　C++11使用3种不同的智能指针，分别是unique\_ptr、shared\_ptr和weak\_ptr，都属于模板类型，可以通过如下的方式使用它们。

```C++
int main() {
	unique_ptr<int> p1 (new int(11));
	shared_ptr<int> p2 (new int(22));
	weak_ptr<int> p3 = p2;
}
```

　　每个智能指针都重载了“\*”运算符，我们可以使用“\*p1”访问所分配的堆内存。智能指针在析构或者调用reset成员的时候，都可能释放其所拥有的堆内存。三者之间的区别如下。

* unipue\_ptr指针不能与其它智能指针共享所指对象的内存，如将“p1”赋值给“p4”（`uniqui_ptr<int>p4 = p1;`）将导致编译错误。但是，可以通过标准库的move函数来转移unique\_ptr指针对象的“所有权”，一旦转移成功，原来的unique\_ptr指针就失去了对对象内存的所有权，再使用该指针会导致运行时错误。
* 多个shared\_ptr指针可以共享同一堆分配对象的内存，它在实现上采用引用计数，即使一个shared\_ptr指针放弃了所有权（调用了reset成员或离开其作用域），也不会影响其它智能指针对象。只有所有引用计数归零的时候，才会真正释放所占有的堆内存。
* weak\_ptr指针可以用来指向shared\_ptr指针分配的对象内存，但不拥有该内存，我们可以使用其lock成员来访问其指向内存的一个shared\_ptr对象，当其所指向的内存无效时，返回指针空值（nullptr）。weak_ptr指针通常可以用来验证shared\_ptr指针的有效性。

### 3、为什么不使用智能指针
　　看起来，shared\_ptr是一个完美的内存管理方案，然而，实际上至少有以下两个原因使Cocos2d-x不应该使用智能指针。

　　第一，智能指针有比较大的性能损失。shared\_ptr为了保证线程安全，必须使用一定开工的互斥锁来保证所有线程访问时其引用计数正确。这种性能损失对一般的应用程序而言是没有问题的，而对游戏这种实时性要求非常高的应用程序却是不可授受的，游戏需要一种更简单的内存管理模型。

　　第二，虽然智能指针能帮助程序员进行有效的堆内存管理，但它仍然需要程序员显式地声明智能指针。例如，创建一个Node的代码需要这样写。

```C++
	shared_ptr<Node*> node(new Node());
```

　　另外，在需要引用的地方，一般应该使用weak\_ptr指针，否则在Node被移除的时候还要手动减持shared\_ptr指针的引用计数，示例如下。

```C++
    weak_ptr<Node*> refNode = node;
```

　　这些额外的约束让智能指针使用起来很不自然。这种用一种约束的方式来避免逻辑错误的方法虽然可取，却不是一种优雅的方式。开发者每天都要而对大量的代码，他们需要更自然的内存管理方式，就像语言自身的特性一样，甚至几乎察觉不到其背后的机制。

### 4、垃圾回收机制
　　垃圾回收器将之前使用过、现在不再使用或者没有任何指针再指向的内存空间称为“垃圾”，将这些“垃圾”收集起来以便再次利用的机制称为“垃圾回收”。垃圾回收机制在1959年前后由约翰·麦肯锡（John MaCarthy）为Lisp语言发明，在编程语言发展的过程中，垃圾回收机制的堆内存管理也得到了很大的发展。如今流行的一些语言，如Java、C#、Ruby、PHP、Perl等，都支持垃圾回收机制。

　　垃圾回收主要有如下两种方式。

* 基于引用计数：引用计数使用系统记录的一个对象被引用的次数，当对象被引用的次数变为零时，该对象即被视作为垃圾而被回收。这种算法的优点是实现方式比较简单。
* 基于跟踪处理：先产生跟踪对象的关系图，再进行垃圾回收。其算法是首先将程序中正在使用的对象视为根对象，从根对象开始查找它们所引用的堆空间，并在这些堆空间上做标记。做完标记之后，所有未被标记的对象即被视作垃圾，会在第二阶段被清理。在第二阶段可以使用不同的方式进行清理，直接清理可能会产生大量的内存碎片。清理方法是对正在使用的对象进行移动或者复制，从而减少内存碎片的产生。

　　不管采用哪种方式，垃圾回收机制都可以使内存管理变得更自然，更重要的是，程序员几乎不用为此做任何被约束的事情。


### 5、Cocos2d-x内存管理机制
　　垃圾回收机制通常需要语言级的实现。C++目前尚未提供完整的垃圾回收机制，Cocos2d-x中的内存管理机制可以看成基于智能指针的一个变体，但它同时使程序员可以像使用垃圾回收机制那样不需要声明智能指针。

　　**1.引用计数**

　　Cocos2d-x中的所有对象几乎都继承自Ref基类。Ref基类主要的职责就是对对象进行引用计数管理，示例如下。

```C++
class CC_DLL Ref
{
public:
	void retain();
	void release();
	Ref* autorelease();
	unsigned int getReferenceCount() const;

protected:
	Ref();

protected:
	// count of references
	unsigned int  _referenceCount;
	friend class AutoreleasePool;
};
```

　　当一个对象使用由new运算符分配的内存时，其引用计数为1，调用retain()方法会增加其引用计数，调用release()方法则会减少其引用计数，release()方法会在其引用计数为零时自动调用delete运算符删除对象并释放内存。

　　除此之外，retain()方法和release方法没有做任何特别的事情，它们只是帮助我们记录一个对象被引用的次数。实际上，在程序中很少直接单独使用retain()方法和release()方法来管理内存，因为最重要的还是要在设计的时候就明确它们应该在哪个地方被释放，大多数引用只是一种弱引用关系，使用retain()方法和release()方法反而会增加复杂性。

　　来看一下在仅有引用计数的情况下应该怎样管理UI元素，示例如下。

```c++
auto node = new Node();          // 引用计数为1
addChild(node);                  // 引用计数为2
node->removeFromParent();        // 引用计数为1
node->release();                 // 引用计数为0
```

　　显然，这不是我们想要的结果。如果忘记调用release()方法就会导致内存泄漏。

　　**2.用autorelease()方法声明一个“智能指针”**
     
　　回顾前面讲述的智能指针，如果将一个动态分配的内存与一个自动变量关联，那么当这个自动变量的生命周期结束时将释放这块堆内存，从而使程序员不必担心其内存无法释放。我们是否可以借鉴类似的机制来避免手动释放UI元素。
     
　　Cocos2d-x使用autorelease()方法来声明一个对象指针为智能指针，但是这些智能指针并不单关联某个自动变量，而是全部被加入一个AutoreleasePool中，在每一帧结束的时候对加入AutoreleasePool中的对象进行清理，也就是说，在Cocos2d-x中，一个智能指针的生命同期从被创建时开始，到当前帧结束时结束，示例如下。

```c++
Ref* Ref:autorelease()
{
	PoolManager::getInstance()->getCurrentPool()->addObject(this);
	return this;
}
```

　　在以上方法中，Cocos2d-x通过autorelease()方法将一个对象添加到一个AutoreleasePool中。

　　Cocos2d-x在每一帧线束的时候清理当前AutoreleasePool中的对象，示例如下。

```c++
void DisplayLinkDirector::mainLoop()
{
	if(!_invalid){
	    drawScene();
	    // release the object
	    PoolManager::getInstance()->getCurrentPool()->clear();
	}
}

void AutoreleasePool::clear()
{
	for( const auto &obj:_managedObjectArray) {
	    obj->release();
	}
	_managedObjectArray.clear();
}
```

　　实际的实现机制是：AutoreleasePool对池中的每个对象执行一次relase操作，假该对象的引用计数为1，表示其从未被使用，则执行release操作之后引用计数为零，对象将被释放。创建一个不被使用的Node，示例如下。

```c++
auto node = new Node();          //引用计数1
node->autorelease();          //加入智能指针池
```

　　可以预计，在该帧结束的时候Node对象将被自动释放。如果该对象在该帧结束之前被使用，示例如下。

```c++
auto node = new Node();          //引用计数为1
node->autorelease();          //加入智能指针池
addChild(node);          //引用计数为2
```

　　在该帧结束的时候，AutoreleasePool对其执行1次release操作之后，引用计数为1，该对象继续存在。当下次该Node被移除的时候，引用计数为零，引用计数为零，对象就会被自动释放。这样就实现了Ref对象的自动内存管理。
　　
　　然而，这需要程序员手动声明其是“智能”的，示例如下。

```c++
auto node = new Node();
node->autorelease();          //在Cocos2d-x中声明智能指针
```

　　为了简化这种声明，Cocos2d-x使用静态的create()方法来返回一个智能指针对象。Cocos2d-x中大部分的类都可以通过create()方法返回一个智能指针对象，如Node、Action等。同时，自定义的UI元素也应该遵循这样的风格，以简化其声明，示例如下。

```c++
Node * Node::create(void)
{
	Node * ret = new Node();
	if (ret && ret->init() ){
	    ret->autorelease();
	}
	else {
	    CC_SAFE_DELETE(ret);
	}
	return ret;
}
```

　　**3.AutoreleasePool队列**
　　
　　对于一个游戏对象而言，一帧的生命周期显然有些长。假设一帧中会调用100个方法，每个方法创建10个autorelease对象，并且这些对象只在每个方法的作用域内被使用，则在该帧即将结束的时候，内存中的峰值为1000个游戏对象所占用的内存。这样，游戏的平均内存占用会大大增加，而实际上，每帆平均只需要占用10个对象的内存（假设这些方法是顺序执行的）。
　　
　　默认AutoreleasePool一帧被清理一次，主要是用来清理UI元素的。因为UI元素大部分都是添加到UI树中的，会一直占用内存，所以，这种情况下每帧清理并不会对内存占用有太大的影响。
　　
　　显然，对于自定义的数据对象，我们需要能够自定义AutoreleasePool的生命周期。Cocos2d-x通过实现一个AutoreleasePool队列来实现智能指针生命周期的自定义，并由PoolManager来管理这个AutoreleasePool队列，示例如下。

```c++
 class CC_DLL PoolManager
 {
 public:
	static PoolManager* getInstance();
	static void destroyInstance();

	AutoreleasePool  *getCurrentPool() const;
	bool isObjectInPools(Ref* obj) const;

	friend class AutoreleasePool;

 private:
	PoolManager();
	~PoolManager();

	void push(AutoreleasePool *pool);
	void pop();

	static PoolManager* s_singleInstace;

	std::deque<AutoreleasePool *> _releasePoolStack;
	AutoreleasePool *_curReleasePool;
 }
```

　　PoolManager的初始状态默认至少有一个AutoreleasePool，它主要用来存储前面讲述的Cocos2d-x中的UI元素对象。我们可以创建自己的AutoreleasePool对象，将其加入队列尾端。但是，如果我们使用new运算符来创建AutoreleasePool对象，则需要手动释放。为了达到和智能指针使用自动变量来管理内存相同的效果，Cocos2d-x对AutorelesePool的构造和析构函数进行了特殊处理，使我们可以通过自动变量来管理内存的释放。示例如下。

```c++
AutoreleasePool::AutoreleasePool()
: _name("")
{
	_managedObjectArray.reserve(150);
	PoolManager::getInstance()->push(this);
}

AutureleasePool::AutoreleasePool(const std::string &name)
: _name(name)
{
	_managedObjectArray.reserve(150);
	PoolManager::getInstance()->push(this);
}

AutoreleasePool::~AutoreleasePool()
{
	clear();
	PoolManager::getInstance()->pop();
}
```

　　AutoreleasePool在构造函数中将自身指针添加到PoolManager的AutoreleasePool队列中，并在析构函数中从队列中移除自己。Ref::autorelease()方法始终将自己添加到当前AutoreleasePool中，只要当前AutoreleasePool始终为队列尾端的元素，声明一个AutoreleasePool对象就可以影响之后的对象，直到该AutoreleasePool对象被移出队列为止，示例如下。

```c++
Class MyClass: public Ref
{
	static MyClass* create() {
		auto ref = new MyClass();
		return ref->autorelease();
	}
}

void customAutoreleasePool()
{
	AutoreleasePool pool;
	auto ref1 = MyClass::create();
	auto ref2 = MyClass::create();
}
```

　　在该方法开始执行时，声明一个AutoreleasePool类型的自动变量Pool，其构造函数会将自身加入PoolManager的AutoreleasePool队列的尾端。接下来，ref1的ref2都会被添加到pool池中。当该方法结束时，自动变量pool的生命周期结束，其析构函数将释放对象，将从队列中移除自己。
     
　　这样，我们就能够通过自定义AutoreleasePool的生命周期来控制Cocos2d-x中autorelease对象的生命周期了。

### 6、怎样进行内存管理
　　结合Cocos2d-x的内存管理机制及其特点，总结一些使用Cocos2d-x内存管理时的注意事项。

* Ref的引用计数并不是线程安全的。在多线程中，我们需要通过处理互斥锁来保证线程的安全。在Objective-C中，由于AutoreleasePool是语言级别的系统实现，所以每个线程都有自己的AutoreleasePool队列。在Cocos2d-x中，从性能等方面考虑，没有提供现成的安全实现。
* 对自定义Node的子类，为该类添加create()方法，并使该方法返回一个autorelease对象。
* 对自定义的数据类型，如果需要动态分配内存，继承自Ref，使用智能指针RefPtr来管理其内存的释放。
* 对只在一个方法内部使用的Ref对象，需要使用自动回收池的，应使用自定义的AutoreleasePool来即时清理对内存的占用。
* 不要动态分配AutoreleasePool，要始终使用自动变量。
* 不要显式调用RefPtr的构造函数，始终使用隐式方式调用构造函数，因为显式的构造函数会导致同时执行构造函数和赋值操作符，这会造成一次必要的临时智能指针变量的产生。

　　————文章摘自《我所理解的COCOS2D-X》