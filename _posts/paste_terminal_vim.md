title: 在LINUX终端和VIM下复制粘贴
author: WEN Pingbo <wengpingbo@gmail.com>
date: 2014/07/21
tags: [linux, vim]
categories: linux
---

在 GUI 界面下，我们可以很自由的复制粘贴。但是在字符界面下，我们不得不用鼠标选定，然后单击右健，选择复制，再到别处去 Ctrl-v。并且对于那些用没有配置过的 VIM 来说，VIM 的粘贴板和 X Window 的粘贴板还不共享。这在码字的过程中，感觉非常不流畅。下面，我们就尝试解决这个问题。

首先我们得让 VIM 和 X Window 共享一个粘贴板，这样我们就可以像在 GUI 界面下一样去复制粘贴了。我们可以在自己的 VIM 配置文件 .vimrc 里添加这么一行：

```
set clipboard=unamedplus
```

这行配置的意思是让 VIM 把 `+` 这个寄存器(粘贴板)设置为平常 yank 和 p 操作的默认粘贴板，而 `+` 寄存器在 VIM 里就是代表 X Window 的粘贴板。这样我们就让 VIM 和 X Window 共享一个粘贴板，再也不用担心 VIM 里复制的东西，不能在 VIM 外去粘贴。

但是这里要注意，如果你下载的是基本 VIM 的话，按照上面的设置是无法实现预期的效果的。因为 VIM 基本版默认不支持 X Window 的粘贴板，所以你得安装 VIM 完全版，或者巨型版。你可以执行如下命令去判断你的 VIM 是否支持 X Window 的粘贴板：

```
vim --version | grep clipboard
```

如果 clipboard 和 xterm_clipboard 带有加号，那么就表示支持这个特性，减号就表示不支持。

在 Ubuntu 下面，你应该安装 vim-gnome，而在 fedora 下面，你需要安装 vim-X11。

这都做完后，你会发现 VIM 在每次退出的时候都会清空粘贴板，而这并不是我们想要的。我们可以在 VIM 配置文件里添加下面一行配置，来让 VIM 在退出的时候，保留粘贴板中的内容：

```
autocmd VimLeave * call system("xsel -ib", getreg('+'))
```

这个配置其实就是在 VIM 每次退出的时候，运行 xsel 命令来把`+`寄存器中的内容保存到系统粘贴板中，所以这个配置要求你安装 xsel。

现在，假设我们从 VIM 中 yank 一些内容，然后退出 VIM，粘贴到终端命令行上，这个时候我们可能还是得拿起鼠标，右键粘贴。其实在大多数 terminal 中都有一个快捷键： Ctrl-Shift-v，把内容粘贴到命令行中。这样我们就解决了在终端下面粘贴的问题。

可能有人会问，在终端下面复制怎么办？这个，暂时还没有找到很满意的解决方案。
