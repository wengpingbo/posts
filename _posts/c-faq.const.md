title: 11.8-const
author: WEN Pingbo <wengpingbo AT gmail.com>
date: 2013/08/13
tags: [c, c-faq]
categories: [language, c]
---

const 关键字用来修饰一个对象，使其成为一个只读对象。比如：

```
const int n=10;
```

但是 const 所修饰的对象并不等于一个常量表达式。const 修饰的对象是一个运行时对象，而一个常量表达式是一个编译时对象。所以这也可以用来解释为什么用 const 修饰的整数来初始化一个数组是不行的。比如:

```
int a[n]; //error
```

const 总是修饰离它最近的对象，比如：

```
const char *p;
char const *p;
char * const p;
```

第一个和第二个是一样的，因为 const 修饰的是 char 和 *，都表示指针 p 指向的是一个只读字符，但是对于 p 本身，确是可以被改变的。而第 3 个 const 修饰的是指针 p 本身，表示指针 p 是一个只读的对象，但是对于 p 所指向的字符，是可修改的。
这和复杂指针声明是一样的，都是依靠操作符的结合性来判断 const 是修饰那个对象的。

而当你用 const 修饰一个自定义类型的时候，和普通类型是没有区别的。比如：

```
#define x_ptr int *
typedef y_ptr int *;
```

当你进行如下声明的时候：

```
const x_ptr x;
const y_ptr y;
```

这时候，第一个 const 修饰的是x所指向的整数，因为 x_ptr 只是一个宏，并不是一个类型，宏是在编译时就确定的，所以第一个声明和 `const int *x;` 是一样。但是第二个 const 修饰的却是 y，而不是 y所指向的整数，因为 y_ptr 是一个用户自定义类型。这就像你在声明基本类型变量的时候 `const int y` 一样，const 修饰的是 y。
