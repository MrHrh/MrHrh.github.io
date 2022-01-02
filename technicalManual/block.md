# OC中block的实现原理
## 目录
- [OC中block的实现原理](#oc中block的实现原理)
  - [目录](#目录)
  - [前言](#前言)
  - [OC->C++](#oc-c)
    - [核心代码解读](#核心代码解读)
    - [block的真实身份：对象](#block的真实身份对象)
    - [衍生函数](#衍生函数)
  - [变量捕获](#变量捕获)
  - [__block](#__block)
    - [__forwarding指针](#__forwarding指针)
  - [循环引用](#循环引用)
    - [weakself和strongself](#weakself和strongself)
  - [block的三种类型](#block的三种类型)

## 前言
block是OC语言中常用的一种用来处理异步调用，回调的方式，作为一名iOS开发我们有必要学习block的底层实现原理以便更好的使用它。  
## OC->C++  
大家都知道OC是C的超集，在编译的时候OC代码会被编译器rewrite成C/C++的代码，通过rewrite之后的C++代码，我们可以探究出OC中很多语言特性的底层实现，block的实现就是其中之一：  
` xcrun -sdk iphoneos clang -arch arm64 -rewrite-objc xxx.m `  
通过上面的命令可以将一个OC的代码转换成C语言的代码，我们尝试将下面这段代码进行转换:  

```Objective-C

#import "Hrhblock.h"

@implementation Hrhblock

- (void)blockTest {
  NSInteger test1 = 10;
  void (^testBlock)(NSInteger) = ^(NSInteger test) {
    NSLog(@"%ld", test1 + test);
  };
  testBlock(10);
}

@end
```  

转换之后得到的C++代码如下（只是其中一小部分，整个转换后的文件有3w多行）  
### 核心代码解读
```C++
struct __block_impl {
  void *isa;
  int Flags;
  int Reserved;
  void *FuncPtr;
};

struct __Hrhblock__blockTest_block_impl_0 {
  struct __block_impl impl;
  struct __Hrhblock__blockTest_block_desc_0* Desc;
  NSInteger test1;
  __Hrhblock__blockTest_block_impl_0(void *fp, struct __Hrhblock__blockTest_block_desc_0 *desc, NSInteger _test1, int flags=0) : test1(_test1) {
    impl.isa = &_NSConcreteStackBlock;
    impl.Flags = flags;
    impl.FuncPtr = fp;
    Desc = desc;
  }
};
static void __Hrhblock__blockTest_block_func_0(struct __Hrhblock__blockTest_block_impl_0 *__cself, NSInteger test) {
  NSInteger test1 = __cself->test1; // bound by copy

  NSLog((NSString *)&__NSConstantStringImpl__var_folders_3j_dm_v_0ln3cd41dqq16yyk7lr0000gp_T_Hrhblock_7e0219_mi_0, test1 + test);
}


static struct __Hrhblock__blockTest_block_desc_0 {
  size_t reserved;
  size_t Block_size;
} __Hrhblock__blockTest_block_desc_0_DATA = { 0, sizeof(struct __Hrhblock__blockTest_block_impl_0)};


static void _I_Hrhblock_blockTest(Hrhblock * self, SEL _cmd) {
    NSInteger test1 = 10;
    void (*testBlock)(NSInteger) = ((void (*)(NSInteger))&__Hrhblock__blockTest_block_impl_0((void *)__Hrhblock__blockTest_block_func_0, &__Hrhblock__blockTest_block_desc_0_DATA, test1));
    ((void (*)(__block_impl *, NSInteger))((__block_impl *)testBlock)->FuncPtr)((__block_impl *)testBlock, 10);
}
```  

不难看出我们OC代码中的`blockTest`函数转换之后是如下的部分：  
```C++
static void _I_Hrhblock_blockTest(Hrhblock * self, SEL _cmd) {
    NSInteger test1 = 10;
    void (*testBlock)(NSInteger) = ((void (*)(NSInteger))&__Hrhblock__blockTest_block_impl_0((void *)__Hrhblock__blockTest_block_func_0, &__Hrhblock__blockTest_block_desc_0_DATA, test1));
    ((void (*)(__block_impl *, NSInteger))((__block_impl *)testBlock)->FuncPtr)((__block_impl *)testBlock, 10);
}
```  

### block的真实身份：对象
这个函数内部从上到下依次是test1变量的定义，block的定义以及block的调用，从这段代码中我们可以发现OC中定义的block被转换成了一个名为` __Hrhblock__blockTest_block_impl_0`结构体，而OC代码的定义block的地方就是生成这个对象的地方，在转换之后的代码里我们找到该结构体的定义：  
```C++
struct __block_impl {
  void *isa;
  int Flags;
  int Reserved;
  void *FuncPtr;
};

struct __Hrhblock__blockTest_block_impl_0 {
  struct __block_impl impl;
  struct __Hrhblock__blockTest_block_desc_0* Desc;
  NSInteger test1;
  __Hrhblock__blockTest_block_impl_0(void *fp, struct __Hrhblock__blockTest_block_desc_0 *desc, NSInteger _test1, int flags=0) : test1(_test1) {
    impl.isa = &_NSConcreteStackBlock;
    impl.Flags = flags;
    impl.FuncPtr = fp;
    Desc = desc;
  }
};
```  

从这个结构体中我们可以看到block的真实定义：  
+ impl：保存block基本数据的结构体，包括isa指针和FuncPtr等。
+ Desc：描述信息，主要包含了block的大小
+ test1：被捕获的变量
+ 构造函数：用来初始化block  

### 衍生函数
根据`__Hrhblock__blockTest_block_impl_0`结构体的构造函数的实现中的第一个参数fp可以看出构造函数内部将其赋值给了impl的FuncPtr指针，在`_I_Hrhblock_blockTest`函数中，最终调用的方法也是它，可以说明这个fp传入的就是实际上block的执行函数，从`_I_Hrhblock_blockTest`函数里可以看出fp指针我们传了一个函数指针`__Hrhblock__blockTest_block_func_0`，该函数的实现如下：  
```C++
static void __Hrhblock__blockTest_block_func_0(struct __Hrhblock__blockTest_block_impl_0 *__cself, NSInteger test) {
  NSInteger test1 = __cself->test1; // bound by copy

  NSLog((NSString *)&__NSConstantStringImpl__var_folders_3j_dm_v_0ln3cd41dqq16yyk7lr0000gp_T_Hrhblock_7e0219_mi_0, test1 + test);
}
```  

不难看出这就是我们OC中block体内所要做的事情，打印test1+test的值。  
`_I_Hrhblock_blockTest`函数最后是对block的调用，可以看到这里其实是调用了testBlock中的FuncPtr指针所指向的函数，也就是`__Hrhblock__blockTest_block_func_0`函数，函数的两个参数分别是block对象本身和调用block时传的参数test。  
经过上面的分析过程我们大致对block的底层实现原理有了整体的认识，总结一下：OC代码中定义的block最终会被处理成一个对象和一个函数，这个函数就是block体中要执行的代码，block对象中保存了该函数的地址，block内要用到的变量，构造函数以及其他一些编译所需的信息。在OC代码中定义block的地方其实是构造block对象的地方，对block的调用的代码其实是调用了block对象中所保存的函数指针，并传入了相应的参数。  
接下来我们结合日常工作中常用的知识以及经常出现的case，更深入的了解block的实现：  
## 变量捕获
block拿到并使用外部变量的方式被称为变量捕获，如上面的例子所示，test1就是被block捕获的一个局部变量，在`__Hrhblock__blockTest_block_impl_0`结构体中有该变量的定义。针对不同类型的变量，block有不同的捕获机制，下面我们一起来看一下：  
```C++

#import "Hrhblock.h"

NSInteger test2 = 100;
static NSInteger test3 = 1000;

@implementation Hrhblock

- (void)blockTest {
  NSInteger test1 = 10;
  static NSInteger test4 = 10000;
  void (^testBlock)(NSInteger) = ^(NSInteger test) {
    NSLog(@"%ld", test1 + test);
    NSLog(@"%ld", test2 + test);
    NSLog(@"%ld", test3 + test);
    NSLog(@"%ld", test4 + test);
  };
  testBlock(10);
}

@end
```  

还是上面的例子，我们增加了test2，test3和test4这四个变量，它们的类型分别是全局变量，静态全局变量，静态局部变量。将这个新的函数rewrite之后得到的`__Hrhblock__blockTest_block_impl_0`结构体的定义如下：  
```C++
struct __Hrhblock__blockTest_block_impl_0 {
  struct __block_impl impl;
  struct __Hrhblock__blockTest_block_desc_0* Desc;
  NSInteger test1;
  NSInteger *test4;
  __Hrhblock__blockTest_block_impl_0(void *fp, struct __Hrhblock__blockTest_block_desc_0 *desc, NSInteger _test1, NSInteger *_test4, int flags=0) : test1(_test1), test4(_test4) {
    impl.isa = &_NSConcreteStackBlock;
    impl.Flags = flags;
    impl.FuncPtr = fp;
    Desc = desc;
  }
};
```  

我们会发现虽然block内部用到了四个变量，但是只捕获了两个，而被捕获的都是局部变量；针对普通的局部变量block捕获的是block对象定义时该局部变量的值，而针对静态局部变量来说，block捕获的是它的地址。这个是为什么呢，其实可以通过这些变量的类型来联想一下，先给出结论：  
- __全局变量：不捕获__
不管是普通全局变量还是静态全局变量，block都不会捕获。因为全局变量在哪里都可以访问，block内部是可以直接访问的，所以外部更改全局变量的值时，block内部的值也会自动更新。  
- __普通局部变量：捕获值__
通过`__Hrhblock__blockTest_block_impl_0`对象的实现可以发现，block在捕获普通局部变量的时候是直接赋值的，相当于把外部test1的值赋给了block结构体内的test1，这两个test1其实是没什么关系的，block捕获的普通局部变量的值是代码走到定义block处的时候变量的值，当后面我们在外部再修改test1的值时，block内部的test1不会随之变化，因为根本不是一个变量。  
- __静态局部变量：捕获地址__
我们发现block在捕获静态局部变量test4的时候，捕获的是该变量的地址，block内部可以通过对该地址解引用来修改该局部静态变量的值，因此block内外是互通的。  
不同类型变量使用不同捕获方式的原因是根据其类型的定义而定的：普通局部变量在出了大括号之后就会被释放，如果block捕获它时也捕获它的地址，那么当block在大括号外部调用时，就会访问一个野指针了；虽然静态局部变量一直都不会被释放，但是由于他的作用域只在当前的大括号内，出了大括号就无法访问，因此选择捕获它的地址，方便修改。  

## __block
上面说到普通局部变量在block内外的值是不互通的，但在实际开发中会有很多场景需要处理互通的局部变量又不希望将其处理成静态局部变量，这个时候我们就需要用到__block关键字，该关键字的作用是让局部变量在block内外的值互通。那根据上面介绍的block的实现原理以及普通局部变量的捕获方式是无法实现的，__block做了哪些工作来实现的呢，我们还是用原来的例子看一下：  
```C++

#import "Hrhblock.h"

NSInteger test2 = 100;
static NSInteger test3 = 1000;

@implementation Hrhblock

- (void)blockTest {
  NSInteger test1 = 10;
  static NSInteger test4 = 10000;
  __block NSInteger test5 = 100000;
  void (^testBlock)(NSInteger) = ^(NSInteger test) {
    NSLog(@"%ld", test1 + test);
    NSLog(@"%ld", test2 + test);
    NSLog(@"%ld", test3 + test);
    NSLog(@"%ld", test4 + test);
    test5 += test;
  };
  testBlock(10);
  NSLog(@"%ld", test5);
}

@end
```  

这里我们定义了一个使用__block关键字修饰的局部变量test5，转换成C++代码如下所示：  
```C++
struct __Block_byref_test5_0 {
  void *__isa;
__Block_byref_test5_0 *__forwarding;
 int __flags;
 int __size;
 NSInteger test5;
};


struct __Hrhblock__blockTest_block_impl_0 {
  struct __block_impl impl;
  struct __Hrhblock__blockTest_block_desc_0* Desc;
  NSInteger test1;
  NSInteger *test4;
  __Block_byref_test5_0 *test5; // by ref
  __Hrhblock__blockTest_block_impl_0(void *fp, struct __Hrhblock__blockTest_block_desc_0 *desc, NSInteger _test1, NSInteger *_test4, __Block_byref_test5_0 *_test5, int flags=0) : test1(_test1), test4(_test4), test5(_test5->__forwarding) {
    impl.isa = &_NSConcreteStackBlock;
    impl.Flags = flags;
    impl.FuncPtr = fp;
    Desc = desc;
  }
};
static void __Hrhblock__blockTest_block_func_0(struct __Hrhblock__blockTest_block_impl_0 *__cself, NSInteger test) {
  __Block_byref_test5_0 *test5 = __cself->test5; // bound by ref
  NSInteger test1 = __cself->test1; // bound by copy
  NSInteger *test4 = __cself->test4; // bound by copy


        NSLog((NSString *)&__NSConstantStringImpl__var_folders_3j_dm_v_0ln3cd41dqq16yyk7lr0000gp_T_Hrhblock_83015b_mi_0, test1 + test);
        NSLog((NSString *)&__NSConstantStringImpl__var_folders_3j_dm_v_0ln3cd41dqq16yyk7lr0000gp_T_Hrhblock_83015b_mi_1, test2 + test);
        NSLog((NSString *)&__NSConstantStringImpl__var_folders_3j_dm_v_0ln3cd41dqq16yyk7lr0000gp_T_Hrhblock_83015b_mi_2, test3 + test);
        NSLog((NSString *)&__NSConstantStringImpl__var_folders_3j_dm_v_0ln3cd41dqq16yyk7lr0000gp_T_Hrhblock_83015b_mi_3, (*test4) + test);
        (test5->__forwarding->test5) += test;
    }
    
static void _I_Hrhblock_blockTest(Hrhblock * self, SEL _cmd) {
    NSInteger test1 = 10;
    static NSInteger test4 = 10000;
    __attribute__((__blocks__(byref))) __Block_byref_test5_0 test5 = {(void*)0,(__Block_byref_test5_0 *)&test5, 0, sizeof(__Block_byref_test5_0), 100000};
    void (*testBlock)(NSInteger) = ((void (*)(NSInteger))&__Hrhblock__blockTest_block_impl_0((void *)__Hrhblock__blockTest_block_func_0, &__Hrhblock__blockTest_block_desc_0_DATA, test1, &test4, (__Block_byref_test5_0 *)&test5, 570425344));
    ((void (*)(__block_impl *, NSInteger))((__block_impl *)testBlock)->FuncPtr)((__block_impl *)testBlock, 10);
    NSLog((NSString *)&__NSConstantStringImpl__var_folders_3j_dm_v_0ln3cd41dqq16yyk7lr0000gp_T_Hrhblock_83015b_mi_4, (test5.__forwarding->test5));
}
```  

可以看到我们用__block关键字修饰的变量test5其实不是NSInteger类型的简单变量，而是一个__Block_byref_test5_0类型的结构体对象，该结构体中保存了一个名为test5的简单变量，而block在捕获时其实捕获的是该结构体对象，在block内部修改该变量的值的时候，改变的其实是结构体内部所持有的test5变量的值，这样一来因为我们在block内外拿到的都是同一个结构体对象，所以能够做到在block内外同时保持相同的值。  
### __forwarding指针
可能会有疑问，test5对象访问自己的test5变量的时候为什么要用__forwarding指针指向，这里就要说到__forwarding的实现原理，block很多时候会作为一个参数传递，这个时候会将block拷贝一份到堆上，name这个时候，__block变量也会被拷贝一份，在调用block的时候，block内部改变的值其实是堆上的__block变量，那在栈上的block如果想输出正确的值，必须让自己的__forwarding指针指向堆上的__block变量，确保内外一致。  
![__forwarding指针](/images/blockPic01.png)  
## 循环引用
在日常开发中经常会有这样一种场景，我们需要在block内部调用当前对象的属性或者方法，这时如果当前对象也持有了这个block的话就会造成循环引用。先来看一下当我们直接在block内部使用self的属性时block对当前对象的捕获，oc代码如下：  
```C++

#import "Hrhblock.h"

NSInteger test2 = 100;
static NSInteger test3 = 1000;

@implementation Hrhblock

- (void)blockTest {
  NSInteger test1 = 10;
  static NSInteger test4 = 10000;
  __block NSInteger test5 = 100000;
  void (^testBlock)(NSInteger) = ^(NSInteger test) {
    NSLog(@"%ld", test1 + test);
    NSLog(@"%ld", test2 + test);
    NSLog(@"%ld", test3 + test);
    NSLog(@"%ld", test4 + test);
    test5 += test;
    self.test += test;
  };
  testBlock(10);
  NSLog(@"%ld", test5);
}

@end
```  

rewrite之后的c++代码如下  
```C++
struct __Hrhblock__blockTest_block_impl_0 {
  struct __block_impl impl;
  struct __Hrhblock__blockTest_block_desc_0* Desc;
  NSInteger test1;
  NSInteger *test4;
  Hrhblock *self;
  __Block_byref_test5_0 *test5; // by ref
  __Hrhblock__blockTest_block_impl_0(void *fp, struct __Hrhblock__blockTest_block_desc_0 *desc, NSInteger _test1, NSInteger *_test4, Hrhblock *_self, __Block_byref_test5_0 *_test5, int flags=0) : test1(_test1), test4(_test4), self(_self), test5(_test5->__forwarding) {
    impl.isa = &_NSConcreteStackBlock;
    impl.Flags = flags;
    impl.FuncPtr = fp;
    Desc = desc;
  }
};
```  

可以看到，这里的block直接强引用了Hrhblock这个对象，这会造成Hrhblock的引用计数+1，此时如果self对象也强持有了block的话，那么就会形成一个引用环，两个对象的引用计数永远不为0也就永远不会被释放，就造成了内存泄漏。  
在开发中我们通常都会使用weakself和strongself，或者@weakify和@strongify来解决，这四个宏的定义如下：  
### weakself和strongself
```C++
#define WeakSelf __weak typeof(self) wself = self
#define StrongSelf __strong typeof(wself) self = wself
```  

通过这两个宏的实现可以看到，weakSelf其实是拿到了self的weak引用，而strongSelf是针对weakself对象的强引用。在实际的开发中，weakself和strongself都是成对使用的，在block外部使用weakSelf的时候，self的引用计数并没有增加，而在block内部声明strongSelf的时候，由于指针的连带关系，self的引用计数还是会加1，截止到这里，使用这两个宏的效果和直接在block内部调用self是一样的，那这样做为什么最后不会造成循环引用呢?  
当我们使用了weakself和strongself的时候，首先block在捕获变量时，捕获的就是weakSelf，而不是self，这说明block并没有持有self（引用环此时没有形成）；其次在block内部使用self的时候，由于strongSelf宏的作用，此时block内部的self其实是一个局部变量，这个局部变量引用了self。但是由于局部变量的作用域是在当前的大括号内，也就是说block执行完之后，block内部的self变量就被释放了，所以即便self持有了block也不会造成循环引用。  
其实这里解决循环引用问题的核心思想就是把block持有self对象，转换成一个局部变量持有，并且保证了block未执行完之前self对象不会被释放。  
@weakify和@strongify的实现原理等同于weakself和strongself，这里就不再详述了。  
## block的三种类型
Block有三种不同的类型，他们的区别基本可以用下图来说明，顾名思义也不难看出存储的位置不同因此有不同的名称。  
![内存分配](/images/blockPic02.png)  
- NSConcreteGlobalBlock：全局Block,不访问任何外部变量，不会涉及到任何拷贝.  
- NSConcreteStackBlock：有访问到外部变量的Block,该保存在栈中，当函数返回时被销毁。  
- NSConcreteMallocBlock：保存在堆中的Block,引用计数为0的时候被销毁.该类型的Block是由NSConcreteStackBlock 复制到堆中形成的。  
