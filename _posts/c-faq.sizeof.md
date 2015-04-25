title: 1.24-关于sizeof
author: WEN Pingbo <wengpingbo@gmail.com>
date: 2013/07/24
tags: [c, c-faq]
categories: [language, c]
---

sizeof 是 C 中的一个关键字，用来计算类型或者表达式的大小。

今天在看 faq 的时候，碰到一个问题，是说 sizeof 无法计算一个数组的大小，在编译的时候，就出现错误。不解，自己尝试着编写了一个例子，果然出现这个错误：
```
Error： invalid application of ‘sizeof’ to incomplete type ‘int[]’
```

<!-- more -->

例子如下：

```
// file f1.h
extern int array[];
extern int size;
int array2[10];
```

```
// file f1.c
#include "f1.h"

int array[]={1,2,3};
int size=sizeof(array);
```

```
// file f2.c
#include "f1.h"
#include <stdio.h>
#define TEST

int main(int argc, char *argv[])
{
printf("array: %d\n",size);
//printf("array: %d\n",sizeof(array));
printf("array2: %d\n",sizeof(array2));

return 0;
}
```

可以看到，在 f2.c 中，第二个 printf 是编译不过去的，第三个正常。

WHY？

因为 sizeof 是在编译的时候来处理每个对象的大小，而编译的时候是以一个文件(编译单元)为单位进行编译的。所以在处理一个编译单元的时候，编译器是无法从另外一个文件来获得一个非完全声明类型的大小的。这也是为什么错误提示中有一个 incomplete type 的字眼了。

那么，什么是 incomplete type?

在ISO-IEC 9899的标准文档的6.2.5中，是这样来定义类型的：

> The meaning of a value stored in an object or returned by a function is determined by the
> type of the expression used to access it. (An identifier declared to be an object is the
> simplest such expression; the type is specified in the declaration of the identifier.) Types
> are partitioned into object types (types that describe objects) and function types (types
> that describe functions). At various points within a translation unit an object type may be
> incomplete (lacking sufficient information to determine the size of objects of that type) or
> complete (having sufficient information)

从上可以看出类型被分为：对象类型、函数类型。而在一个翻译单元中，对象类型可能是完全的，或者非完全的。

也就是说非完全类型是对象类型的一个特殊类，而什么时候，一个对象类型是不完全的呢？

标准文档是这么说的：缺乏足够的信息来决定一个对象类型的大小

可能说这么多，大家有点迷糊了，下面引用一下 IBM 在 XL C/C++ (V6.0) 的规范文档中，把如下的类型归类为非完全类型：

* void 类型 eg. void *ptr;
* 没有指明大小的数组 eg. extern int array[];
* 数组中的元素是非完全类型
* 没有定义的结构体、union 和枚举类型 eg. struct test;
* 没有定义的类指针 
* 只声明，无定义的类 eg. class test;

其实，incomplete type 一个经常的用处，那就是隐藏一些类型的细节，比如，在头文件中声明一个 struct test;，然后其他程序如果要使用这个结构体，那就包含这个头文件，但是它们无法知道这个结构体内部的细节，也就无法访问结构体内部成员，当然，更加不能用 sizeof。但是对于那些，只是做一个传递的外部函数，这些足够了。

所以，以后用 sizeof 的时候，还是注意一下咯。
