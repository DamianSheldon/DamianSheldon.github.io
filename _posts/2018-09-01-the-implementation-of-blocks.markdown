---
layout: post
title: "Blocks 的实现"
date: 2018-09-01 15:42:34 +0800
comments: true
categories: [Archives, iOS Development]
keywords: Block, Implementation, 实现, 本质
description: Blocks 的实现
---
Block 的实现是面试中高频出现的问题，背后的原因我想是希望借此考察面试者对 Block 的掌握程度，在日后的工作中能够用好它；同时能从侧面反映面试者有没有深入钻研技术，以及独立思考能力如何，可谓一举多得。

下面我们就来看看 ObjC 中的 Blocks 是如何实现。Clang 的 `-rewrite-objc` 选项可以将含有 Block 语法的源代码转换为 C++，说是 C++，其实也仅使用了 struct 结构，其本质是 C 语言。

下面我们先转换一个简单的文件试试:

```objc
#import <Foundation/Foundation.h>

int main(int argc, char ** argv)
{
    @autoreleasepool {
        void (^blk)(void) = ^{
            printf("Block\n");
        };

        blk();
    }    

    return 0;
}

// 使用命令:
$ clang -fobjc-arc -ObjC -rewrite-objc -mios-version-min=6.0.0 -fobjc-runtime=ios-6.0.0 -isysroot /Applications/Xcode.app/Contents/Developer/Platforms/iPhoneOS.platform/Developer/SDKs/iPhoneOS11.3.sdk -arch arm64 block-essense.m  -o block-essense-in-c.c

//限于篇幅，省略不相关的部分，结果如下:
struct __block_impl {
  void *isa;
  int Flags;
  int Reserved;
  void *FuncPtr;
};

struct __main_block_impl_0 {
  struct __block_impl impl;
  struct __main_block_desc_0* Desc;
  __main_block_impl_0(void *fp, struct __main_block_desc_0 *desc, int flags=0) {
    impl.isa = &_NSConcreteStackBlock;
    impl.Flags = flags;
    impl.FuncPtr = fp;
    Desc = desc;
  }
};
static void __main_block_func_0(struct __main_block_impl_0 *__cself) {

            printf("Block\n");
        }

static struct __main_block_desc_0 {
  size_t reserved;
  size_t Block_size;
} __main_block_desc_0_DATA = { 0, sizeof(struct __main_block_impl_0)};
int main(int argc, char ** argv)
{
    /* @autoreleasepool */ { __AtAutoreleasePool __autoreleasepool; 
        void (*blk)(void) = ((void (*)())&__main_block_impl_0((void *)__main_block_func_0, &__main_block_desc_0_DATA));

        ((void (*)(__block_impl *))((__block_impl *)blk)->FuncPtr)((__block_impl *)blk);
    }

    return 0;
}
```
<!--more-->
可以看到持有 block 的变量实际上就是指针，而 block 本身则是结构体，在我们的例子中对应的是 `__main_block_impl_0`,功能代码则是通过函数来实现的，block 结构体内有成员变量指向该函数，这样我们对 block 的实现渐渐清晰起来了。

Block 有一个重要的特性--自动捕获变量。这又是怎么实现的呢？我们同样可以使用上述的方法来得到答案。我们构造一个捕获变量的例子，然后来查看它的结果：

```objc
#import <Foundation/Foundation.h>

int main(int argc, char ** argv)
{
    @autoreleasepool {
        
        BOOL flag = YES;
        int i = 28;
        float pi = 3.1415;
        char c = 'x';
        
        void (^blk)(void) = ^{
            printf("Block\n");
            printf("flag:%d\n", flag);
            printf("i:%d\n", i);
            printf("pi:%d\n", pi);
            printf("c:%d\n", c);
        };
        
        blk();
    }
    
    return 0;
}

// 转换之后相关部分
struct __main_block_impl_0 {
  struct __block_impl impl;
  struct __main_block_desc_0* Desc;
  BOOL flag;
  int i;
  float pi;
  char c;
  __main_block_impl_0(void *fp, struct __main_block_desc_0 *desc, BOOL _flag, int _i, float _pi, char _c, int flags=0) : flag(_flag), i(_i), pi(_pi), c(_c) {
    impl.isa = &_NSConcreteStackBlock;
    impl.Flags = flags;
    impl.FuncPtr = fp;
    Desc = desc;
  }
};
static void __main_block_func_0(struct __main_block_impl_0 *__cself) {
  BOOL flag = __cself->flag; // bound by copy
  int i = __cself->i; // bound by copy
  float pi = __cself->pi; // bound by copy
  char c = __cself->c; // bound by copy

            printf("Block\n");
            printf("flag:%d\n", flag);
            printf("i:%d\n", i);
            printf("pi:%d\n", pi);
            printf("c:%d\n", c);
        }

static struct __main_block_desc_0 {
  size_t reserved;
  size_t Block_size;
} __main_block_desc_0_DATA = { 0, sizeof(struct __main_block_impl_0)};
int main(int argc, char ** argv)
{
    /* @autoreleasepool */ { __AtAutoreleasePool __autoreleasepool; 

        BOOL flag = ((bool)1);
        int i = 28;
        float pi = 3.1415;
        char c = 'x';

        void (*blk)(void) = ((void (*)())&__main_block_impl_0((void *)__main_block_func_0, &__main_block_desc_0_DATA, flag, i, pi, c));

        ((void (*)(__block_impl *))((__block_impl *)blk)->FuncPtr)((__block_impl *)blk);
    }

    return 0;
}
```

可以看到自动捕获的标量数据是直接声明为 block 结构体的成员变量。

除了读取捕获自动变量的值，block 还支持使用 `__block` 修饰符来修改自动捕获的变量。我们同样来看个例子：

```objc
#import <Foundation/Foundation.h>

int main(int argc, char ** argv)
{
    @autoreleasepool {
        
        __block BOOL flag = YES;
        __block int i = 28;
        __block float pi = 3.1415;
        __block char c = 'x';
        
        void (^blk)(void) = ^{
            printf("Block\n");
            
            flag = NO;
            i = 88;
            pi = 3.1415926;
            c = 'a';
            
            printf("flag:%d\n", flag);
            printf("i:%d\n", i);
            printf("pi:%f\n", pi);
            printf("c:%d\n", c);
        };
        
        blk();
    }
    
    return 0;
}

// 转换之后相关部分的结果:
struct __Block_byref_flag_0 {
  void *__isa;
__Block_byref_flag_0 *__forwarding;
 int __flags;
 int __size;
 BOOL flag;
};
struct __Block_byref_i_1 {
  void *__isa;
__Block_byref_i_1 *__forwarding;
 int __flags;
 int __size;
 int i;
};
struct __Block_byref_pi_2 {
  void *__isa;
__Block_byref_pi_2 *__forwarding;
 int __flags;
 int __size;
 float pi;
};
struct __Block_byref_c_3 {
  void *__isa;
__Block_byref_c_3 *__forwarding;
 int __flags;
 int __size;
 char c;
};

struct __main_block_impl_0 {
  struct __block_impl impl;
  struct __main_block_desc_0* Desc;
  __Block_byref_flag_0 *flag; // by ref
  __Block_byref_i_1 *i; // by ref
  __Block_byref_pi_2 *pi; // by ref
  __Block_byref_c_3 *c; // by ref
  __main_block_impl_0(void *fp, struct __main_block_desc_0 *desc, __Block_byref_flag_0 *_flag, __Block_byref_i_1 *_i, __Block_byref_pi_2 *_pi, __Block_byref_c_3 *_c, int flags=0) : flag(_flag->__forwarding), i(_i->__forwarding), pi(_pi->__forwarding), c(_c->__forwarding) {
    impl.isa = &_NSConcreteStackBlock;
    impl.Flags = flags;
    impl.FuncPtr = fp;
    Desc = desc;
  }
};
static void __main_block_func_0(struct __main_block_impl_0 *__cself) {
  __Block_byref_flag_0 *flag = __cself->flag; // bound by ref
  __Block_byref_i_1 *i = __cself->i; // bound by ref
  __Block_byref_pi_2 *pi = __cself->pi; // bound by ref
  __Block_byref_c_3 *c = __cself->c; // bound by ref

            printf("Block\n");

            (flag->__forwarding->flag) = ((bool)0);
            (i->__forwarding->i) = 88;
            (pi->__forwarding->pi) = 3.1415926;
            (c->__forwarding->c) = 'a';

            printf("flag:%d\n", (flag->__forwarding->flag));
            printf("i:%d\n", (i->__forwarding->i));
            printf("pi:%f\n", (pi->__forwarding->pi));
            printf("c:%d\n", (c->__forwarding->c));
        }
static void __main_block_copy_0(struct __main_block_impl_0*dst, struct __main_block_impl_0*src) {_Block_object_assign((void*)&dst->flag, (void*)src->flag, 8/*BLOCK_FIELD_IS_BYREF*/);_Block_object_assign((void*)&dst->i, (void*)src->i, 8/*BLOCK_FIELD_IS_BYREF*/);_Block_object_assign((void*)&dst->pi, (void*)src->pi, 8/*BLOCK_FIELD_IS_BYREF*/);_Block_object_assign((void*)&dst->c, (void*)src->c, 8/*BLOCK_FIELD_IS_BYREF*/);}

static void __main_block_dispose_0(struct __main_block_impl_0*src) {_Block_object_dispose((void*)src->flag, 8/*BLOCK_FIELD_IS_BYREF*/);_Block_object_dispose((void*)src->i, 8/*BLOCK_FIELD_IS_BYREF*/);_Block_object_dispose((void*)src->pi, 8/*BLOCK_FIELD_IS_BYREF*/);_Block_object_dispose((void*)src->c, 8/*BLOCK_FIELD_IS_BYREF*/);}

static struct __main_block_desc_0 {
  size_t reserved;
  size_t Block_size;
  void (*copy)(struct __main_block_impl_0*, struct __main_block_impl_0*);
  void (*dispose)(struct __main_block_impl_0*);
} __main_block_desc_0_DATA = { 0, sizeof(struct __main_block_impl_0), __main_block_copy_0, __main_block_dispose_0};
int main(int argc, char ** argv)
{
    /* @autoreleasepool */ { __AtAutoreleasePool __autoreleasepool; 

        __attribute__((__blocks__(byref))) __Block_byref_flag_0 flag = {(void*)0,(__Block_byref_flag_0 *)&flag, 0, sizeof(__Block_byref_flag_0), ((bool)1)};
        __attribute__((__blocks__(byref))) __Block_byref_i_1 i = {(void*)0,(__Block_byref_i_1 *)&i, 0, sizeof(__Block_byref_i_1), 28};
        __attribute__((__blocks__(byref))) __Block_byref_pi_2 pi = {(void*)0,(__Block_byref_pi_2 *)&pi, 0, sizeof(__Block_byref_pi_2), 3.1415};
        __attribute__((__blocks__(byref))) __Block_byref_c_3 c = {(void*)0,(__Block_byref_c_3 *)&c, 0, sizeof(__Block_byref_c_3), 'x'};

        void (*blk)(void) = ((void (*)())&__main_block_impl_0((void *)__main_block_func_0, &__main_block_desc_0_DATA, (__Block_byref_flag_0 *)&flag, (__Block_byref_i_1 *)&i, (__Block_byref_pi_2 *)&pi, (__Block_byref_c_3 *)&c, 570425344));

        ((void (*)(__block_impl *))((__block_impl *)blk)->FuncPtr)((__block_impl *)blk);
    }

    return 0;
}
```

可以看到使用 `__block` 修饰的变量实际上转换成了结构体变量，同样在 block 结构体中有成员变量指向它们。

上面我们看过了使用 block 时的几种情况，我们可以尝试来总结使用 block 的情况，然后查看各种情况转换之后的代码来进一步探索 block 的实现，进而得到比较完善的答案。

首先 block 可以按是否捕获变量分为两大类，其次捕获变量时根据是否支持修改又可以分为两类，最后捕获变量又可以分为程序的数据区域、栈上和堆上三种情况。综上，我们可以得到得到如下的 block 分类列表:

* 不捕获变量(1)
* 捕获变量
	
	* 不修改捕获的变量
		
		* 存在程序数据区的变量(2)
		* 存在栈上的变量(3)
		* 存在堆上的变量(4)
		
	* 修改捕获的变量(`__block` 修饰的变量)

		* 存在程序数据区的变量(5)
		* 存在栈上的变量(6)
		* 存在堆上的变量(7)

这样算下来应该是存在七种情况，我们可以分别构造各种情况的例子，然后得到 block 的实现全貌。

全局变量和 static 变量是程序数据区变量，block 中访问全局变量和在其他地方没有什么不同，所以 block 的实现中不需要对它进行特别考虑。Static 变量在捕获时会在 block 结构体中有对应的成员变量，可以用该成员变量来读写。由于它在程序的生命周期中一直存在，所以当 block 捕获并修改它时，不需要生成对应的结构体变量，这和其他 `__block` 修饰的变量不同。

情况三和四比较类似，它们都会在 block 结构体中增加相应的成员变量，不同之处是捕获堆上的变量， block 的描述结构体变量中会增加 copy 和 dipose 函数，用来管理对应的内存。

情况六和七也类似，它们都是将变量转换为结构体，然后在 block 结构体增加成员变量指向它们。捕获堆上的变量时，block 内的成员变量指向变量，而这个变量是指向堆上分配的一块内存的，也就是一个对象，对象就是一块内存区域嘛，用代码示例如下：

```objc
blk_t blk;
   
{
  __block id __strong array = [[NSMutableArray alloc] init];
    
  blk = [^(id obj){
      
      [array addObject:obj];
      NSLog(@"array count = %ld", [array count]);
      
  } copy];
}

// __block 修饰指向 array 的变量
struct __Block_byref_array_0 {
  void *__isa;
__Block_byref_array_0 *__forwarding;
 int __flags;
 int __size;
 void (*__Block_byref_id_object_copy)(void*, void*);
 void (*__Block_byref_id_object_dispose)(void*);
 __strong id array;
};

// 表示 block 的结构体
struct __main_block_impl_0 {
  struct __block_impl impl;
  struct __main_block_desc_0* Desc;
  __Block_byref_array_0 *array; // by ref
  __main_block_impl_0(void *fp, struct __main_block_desc_0 *desc, __Block_byref_array_0 *_array, int flags=0) : array(_array->__forwarding) {
    impl.isa = &_NSConcreteStackBlock;
    impl.Flags = flags;
    impl.FuncPtr = fp;
    Desc = desc;
  }
};
```

看到这里，我们有了 block 捕获变量出了作用域后还能存在原因的线索，当表示 block 的结构体从栈上拷贝到堆上，如果是只读变量，它的值赋值给 block 结构体的成员变量了；如果是 `__block` 修饰的变量，表示该变量的结构体也会一并拷贝到堆上，并由 block 持有和管理。

至此，我们应该对 block 的实现比较清晰了。

##Reference

* Objective-C 高级编程

