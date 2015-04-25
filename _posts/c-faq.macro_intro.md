title: 10.20-宏
author: WEN Pingbo <wengpingbo AT gmail.com>
date: 2013/08/13
tags: [c, c-faq]
categories: [language, c]
---

在 C 语言中。宏分为两种：对象式宏和函数式宏。

对象式宏就是不带参数的宏，只是进行一些简单的替换。而函数式宏是带函数的宏，外表看上去就像是一个函数。
比如：

```
#define TEST "this is a test"
#define PRINT(int) printf("PRINT: %d", int)
```

函数式宏还支持变长参数，比如：

```
#define debug(...)   fprintf(stderr, __VA_ARGS__)
```

当定义一个宏之后，还可以通过 #undef 来删除这个宏，比如：

```
#undef debug
```

在宏中，有两个操作符：\# 和 ##。

# 是用来把紧跟该操作符的参数变成一个字符串。比如我们定义一个这样的宏：

```
#define STR(x) #x
```

那么，当你调用 STR(this is a test) 的时候，预编译器就会把这个宏替换为：

```
"this is a test"
```

如果我这样调用：

```
STR("this is \a test")
```

那结果是什么？在 ANSI C 中是这样规定的：当带有 # 的参数中有 " 符号和 \ 符号的时候，会自动在它的前面加上一个反斜杠。所以上面那个例子的答案是：

```
"\"this is \\a test\""
```

另外一个操作符 ## 是用来连接两个参数的。比如有这样一个宏：

```
#define XY(x,y) x##y
```

如果我这样调用这个宏：

```
XY(abc,def)
```

这这个宏调用最后会替换为：

```
abcdef
```

注意操作符 ## 不能出现替换列表的开头和结尾，像：

```
#define X(x) ##x //error
#define Y(y) y## //error
```

是错误的声明，编译器会报错的。

操作符 ## 可以很容易的生成一些新的变量和函数。但是如果过多的用这种宏，也会给代码阅读带来一定的麻烦。
在阅读 kernel 源码的时候，经常发现有一些函数，或者结构体无法被 grep 到，好像这个函数或者结构体没有定义似的。如果出现这种情况，那么这个函数或者结构体极有可能是由带操作符 ## 的宏来生成的。
比如 kernel 中的 MACHINE_START 和 MACHINE_END 的定义：

```
#define MACHINE_START(_type,_name) \ 
static const struct machine_desc __mach_desc_##_type \
__used \
__attribute__((__section__(".arch.info.init"))) = { \
.nr = MACH_TYPE_##_type, \
.name = _name,

#define MACHINE_END \
};
```

可以看到 __mach_desc_##_type 就是通过宏来生成的，如果 _type 是 arm，那么这个 machine_desc 类型的结构体就是 __mach_desc__arm。如果直接搜索这个字符，是搜索不到的。这种带有 ## 的宏在 kernel 中经常出现。


如果多于一个 ## 或者 # 操作符，则这种情况在 ANSI C 中是未定义的，不同的编译器可能会得到不一样的结果。另外，还需注意一点，宏并不会处理字符串里面的替换。比如：

```
#define TRACE(var, fmt) printf("TRACE: var = fmt\n", var)
```

你想通过调用 TRACE(i,%d) 来得到：

```
printf("TRACE: i = %d\n", i)
```

但是，这并不能如你所愿。编译器会报错：

```
warning: macro replacement within a string literal
```

你应该这样来定义你的宏：

```
#define TRACE(var, fmt) \
		printf("TRACE: " #var " = " #fmt "\n", var)
```
