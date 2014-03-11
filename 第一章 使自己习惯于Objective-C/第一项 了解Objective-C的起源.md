## 第一项 了解Objective-C的起源

Objective-C和C++、Java等面向对象语言相似，但在某些方面又存在差异。假如你拥有面向对象开发的经验，相信你已经掌握并使用过很多的范例与模式。不管怎样，由于Objective-C采用的是消息结构体而不是函数调用，为此在语法上可能需要过渡。Objective-C由Smalltalk进化而来，这也是消息传递的起源。函数调用与消息发送的区别如下：

	// 消息发送 Objective-C
	Object *obj = [Object new];
	[obj performWith:parameter1 and:parameter2];
	
	// 函数调用 C++
	Object *obj = new Object;
	obj->perform(parameter1, parameter2);
	
以上最大的不同在于，使用消息发送由运行时(runtime)决定哪段代码将会被执行。而在函数调用时，由编译器(compiler)进行决定。当多态(polymorphism)被引入到函数调用(function-calling)的示例中时，一种运行时查找(runtime looup)方式将会涉及到一个虚拟表。但是在消息发送中查找总是存活在运行时。事实上，编译器并不关心将要被发送的对象类型。它们只需要在运行时通过动态绑定(dynamic binding)被完美的找到即可，所涉及到的详细内容请参照第11项。

Objective-C的运行组件比编译器承担了更重要的责任，运行时包含了所有的Objective-C的面向对象特征的数据结构与函数。例如，运行时涵盖了所有的内存管理方法。从本质上来看的话，运行时时一些代码的集合，它与你的所有代码以及一些被动态连接到你的代码中的动态库(dynamic library)紧密相连。从而无论运行时何时被更新，你的应用程序都会从中获取最大的性能提升，然后一种在编译时做很多工作的语言为了提升性能需要被重新编译。

Objective-C作为C的超集，所有的C的特征在Objective-C中都是可用的。因此，为了编写高性能的Objective-C，你需要了解C与Objective-C的核心内容。尤其在理解了C中的内存管理模型将会帮助你理解Objective-C的内存管理模型，以及为什么要采用引用计数机制。这些涵盖理解在Objective-C中使用指针来指向一个对象。当你声明一个变量时，该变量已经被一个对象所持有，语法如下：

	NSString *someString = @"The string";
	
该语法大部分是从C直接演变而来，声明一个NSString *类型的变量someString，在这里someString是一个指向NSString的指针。所有的Objective-C对象都需要采用此种方式进行声明，因为从内存中来看对象是被分配在堆空间中而不是栈中。从栈中声明Objective-C对象是非法的：

	NSString stackString； // 错误：接口类型不能被静态分配
	
从堆中分配的变量someString指向了一块内存区域，它包含了NSString对象。这也就意味着创建另一个变量指向同一个内存区域而不需要对其进行复制，当然了两个变量指向了同一个对象：
	
	NSString *someString = @"The string";
	NSString *anotherString = someString;

在这里只存在一个NSString实例，但是存在两个NSString *类型的变量同时指向该实例。也就意味着当前的栈结构已经在内存中分配了2个位的指针变量的大小空间(在32位架构中，一个指针变量占据了4个字节，在64位架构中占据了8个字节)，该部分内存保存了NSString实例的内存地址。

图1.1 阐述了以上的布局，其中data存储了NSString实例，包含一组需要展现真实字符串的字节数组。

![内存中的栈分配与堆分配](http://jiwanqiang.com/wp-content/uploads/EffObjc/1.1.png)

从堆中分配的内存可以被直接管理，相反一个变量倘若从栈中分配，那么当该栈内存的帧出栈之后将会被自动清理。

Memory management of the heap memory is abstracted away by Objective-C. You donot need to usemalloc and free to allocate and deallocate the memory for objects. The Objective-C runtime abstracts this out of the way through a memory-management architecture known as reference counting (see Item 29).
Sometimes in Objective-C, you will encounter variables that don’t have a * in the definition and might use stack space. These variables are not holding Objective-Cobjects. An example is CGRect, from the CoreGraphics framework:
	CGRect frame;
	frame.origin.x = 0.f;
	frame.origin.y = 10.f;
	frame.size.width = 100.f;
	frame.size.height = 150.f;
	
CGRect作为一个C结构体，定义如下：

	struct CGRect {
		CGPoint origin;
		CGSize size;
	};

These types of structures are used throughout the system frameworks, where the overhead of using Objective-C objects could affect performance. Creating objects incurs overhead that using structures does not, such as allocating and deallocating heap memory. When nonobject types (int, float,double, char, etc.) are the only data to be held, a structure, such as CGRect, is usually used.Before embarking on writing anything in Objective-C, I encourage you to read texts about the C language and become familiar with the syntax. If you dive straight into Objective-C, you may find certain parts of the syntax confusing.
### 温故而知新
* Objective-C is a superset of C, adding object-oriented features. Objective-C uses a messaging structure with dynamic binding, meaning that the type of an object is discovered at runtime. The runtime, rather than the compiler, works out what code to run for a given message.
* Understanding the core concepts of C will help you write effective Objective-C. In particular, you need to understand the memory model and pointers.