title: MD5 算法实现
author: WEN Pingbo <wengpingbo@gmail.com>
date: 2012/12/08
tags: [security, md5]
categories: security
---

MD5 算法的标准文件见 [RFC1321](http://www.ietf.org/rfc/rfc1321.txt)。这里说一下具体实现时，应该注意的地方。

## LSB 和 MSB
其实，在官方文档中已经提到过这个，但是没注意，现把它摘抄如下：

> In this document a "word" is a 32-bit quantity and a "byte" is an eight-bit quantity. A sequence of bits can be interpreted in a natural manner as a sequence of bytes, where each consecutive group of eight bits is interpreted as a byte with the high-order (most significant) bit of each byte listed first. Similarly, a sequence of bytes can be interpreted as a sequence of 32-bit words, where each consecutive group of four bytes is interpreted as a word with the low-order (least significant) byte given first.

<!-- more -->

翻译过来，就是说在 MD5 算法中一个字节内是按 MSB 存储的，在一个字内是按 LSB 存储的。

例如：

```
00010111 11011001 11100011 10101100(msb)
```

按照文档中的规定，每个字节是按 MSB 存储的，则写成十六进制如下：

```
0x17 d9 e3 ac
```

而在一个字，即 32 位里，是按高字节序存储的，所以最终的存储值如下：

```
0xace3d917
```

另外，在填充长度时，两个字也是按 LSB 存储，每个字内是按上面所说的存储，具体原文如下：

> A 64-bit representation of b (the length of the message before the padding bits were added) is appended to the result of the previous step. In the unlikely event that b is greater than 2^64, then only the low-order 64 bits of b are used. (These bits are appended as two 32-bit words and appended low-order word first in accordance with the previous conventions.)

虽然官方文档规定了数据该怎样存储，但是在真正实现时，就连官方给的 DEMO 也没有完全遵守。以至于自己实现 MD5 算法时，算出来的结果总是不对。

## MD5实现步骤

1. 分割输入数据，填充数据和长度 - 在这一步，原生的数据和填充的数据（0x80 00 00 ...）是按高字节序存储的。而填充的长度须按上面所说的规定存储，即字节内MSB，字内 LSB，双字 LSB。

2. 初始化4字缓存 - 文档中写的是按 LSB 存储，但是真正初始化是却是按 MSB 存储的。

3. 4*16 迭代 - 在迭代中，传进来的 X[i] 是需要进行 LSB 转换的，也就是把原来存储的 MSB 数据转换成 LSB （第一步是原生数据加填充数据（MSB） + 长度填充（LSB），现在需要把全部数据再次进行 LSB 转换，长度填充相当于转换了两次）

4. 输出结果 - 输出的结构，需要进行 LSB 转换，就是把 4 字缓存中每个字进行 LSB 转换，然后输出的结果才是正确的。

要注意 T 表的计算，虽然文档中给出了具体的计算公式 `(2^32*abs(sin(i)))`，但是还是建议直接通过一个数组预先给出，因为这个计算如果处理的不好，会有精度误差，导致最后结果错误，并且这个还与系统类型，编译器版本，CPU 类型和所用语言有关。我想这也是官方的 DEMO 为了程序的可移植性和稳定性，采用的也是预先给出 T 表值，而没有现用现算的原因吧。像这次实现 MD5，有一个人用 Java 实现的，结果没有碰到这个问题，而我用 C 语言实现，这个问题就出现了。
