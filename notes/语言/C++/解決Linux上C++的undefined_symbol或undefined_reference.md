# Linux 上 C / C++ 的未定义符号或未定义引用

使用 C / C ++ 程序分为三个步骤：编译（compile）→ 链接（link）→ 执行（加载符号）。了解每个阶段的行为，才知道如何处理该阶段的未定义符号。

## 基本概念

* 编译 .c / .cpp 为 .o（目标文件）时，需要提供 header 。当 header 不在预测搜寻路径时，可用 -I DIR 增加搜寻目录。事实上，在编译单一文件时，gcc/g++ 不在意 symbol 是否存在，，只要有声明就可以，所以关键是有没有引入正确的 header 。这也是可分散编译的原因（如 [distcc](https://github.com/distcc/distcc) ），因为程序代码编译成 .o 文件时，没有相关性。

* 用静态链接器（ld.bfd，gold，lld）将 * .o 链接成动态库或执行程序时，通常需要确认查找得到所有的符号，需要提供要链接的库。用 -L DIR 指定目录位置，用 -l 指定函数库名称。

* 执行的时候，动态链接程序（又称运行时链接程序）会加载共享库中的符号。前一个步骤只是检查是否有符号。检查通过链接成可执行文件或共享库后，若执行时路径不对，或库不存在，则会在执行期间找不到符号。

## 常用工具

* 查看 BINARY 内含或包含的符号
用 nm -D BINARY 列出 BINARY （目标文件，静态/动态库，可执行文件）动态部分的符号：
  * T/t 分别表示全球/本地符号。

  * U 表示未定义符号。

>用 nm -Du BINARY 只会显示使用到但未定义的符号。

* 用 pkg-config 列出编译/连结用的参数

>系统内部安全的开发套件，可用 pkg-config --cflags LIB 列出编译时使用 LIB 需要的参数， pkg-config --libs LIB 列出关联时使用的参数。

以 Gtk+ 首先，安装开发套件后，用 dpkg -L 找 pkg-config 的设定档：

```shell
$ dpkg -L libgtk-3-dev：amd64 | grep“ pc $”
/usr/lib/x86_64-linux-gnu/pkgconfig/gtk+-unix-print-3.0.pc
/usr/lib/x86_64-linux-gnu/pkgconfig/gdk-x11-3.0.pc
/ usr / lib / x86_64-linux-gnu / pkgconfig / gdk-broadway-3.0.pc
/usr/lib/x86_64-linux-gnu/pkgconfig/gdk-wayland-3.0.pc
/ usr / lib / x86_64-linux-gnu / pkgconfig / gtk + -broadway-3.0.pc
/usr/lib/x86_64-linux-gnu/pkgconfig/gtk+-wayland-3.0.pc
/usr/lib/x86_64-linux-gnu/pkgconfig/gtk+-mir-3.0.pc
/ usr / lib / x86_64-linux-gnu / pkgconfig / gdk-mir-3.0.pc
/usr/lib/x86_64-linux-gnu/pkgconfig/gtk+-3.0.pc
 / usr / lib / x86_64-linux-gnu / pkgconfig / gdk- 3.0.pc
/usr/lib/x86_64-linux-gnu/pkgconfig/gtk+-x11-3.0.pc
```

想使用 Gtk+ -3.0 的话，列出编译时用到的参数：

```shell
$ pkg-config --cflags gtk + -3.0
-pthread -I / usr / include / gtk-3.0 -I / usr / include / at-spi2-atk / 2.0 -I / usr / include / at / spi-2.0 -I /usr/include/dbus-1.0 -I / usr / lib / x86_64-linux-gnu / dbus-1.0 / include -I / usr / include / gtk-3.0 -I / usr / include / gio-unix-2.0 /- I / usr / include / mirclient -I / usr / include / mircore -I / usr / include / mircookie -I / usr / include / cairo -I / usr / include / pango-1.0 -I / usr / include / harfbuzz- I / usr / include / pango-1.0 -I / usr / include / atk-1.0 -I / usr / include / cairo -I / usr / include / pixman-1 -I / usr / include / freetype2 -I / usr / include / libpng12 -I / usr / include / gdk-pixbuf-2.0 -I / usr / include / libpng12 -I / usr / include / glib-2.0 -I / usr / lib / x86_64-linux-gnu / glib-2.0 /包括
```

列出连结时用到的参数：

```shell
$ pkg-config --libs gtk + -3.0
-lgtk-3 -lgdk-3 -lpangocairo-1.0 -lpango-1.0 -latk-1.0 -lcairo-gobject -lcairo -lgdk_pixbuf-2.0 -lgio-2.0 -lgobject-2.0 -lglib -2.0
```

## 解决编译原始码的问题（ compiler 的工作）

若使用的是系统装的函式库 X，装上 X-dev 后会有标题，然后看有没有带 pkg-config 的设定档。有的话用 pkg-config 可以知道该下什么参数找到 header 。没有的话，用 dpkg -L X-dev | grep include 找出 header 位置。若是另外下载的程式码，在里面找找 header 也没什么困难。

下面列一些相关的资料。

1. 若是网路上看到别人的范例，里面有正确的 include 位置，可以用 header 位置寻找需要安装的套件，像是使用 GDK event 要包含 “gdk/gdkevents.h”：
```shell
$ apt-file search gdk / gdkevents.h
libgtk-3-dev：/usr/include/gtk-3.0/gdk/gdkevents.h
libgtk2.0-dev：/usr/include/gtk-2.0/gdk/gdkevents.h
```

2. 若使用的函式有附的手册页，则可能会提到需要的编译/连结参数。以 man 3 sqrt 为例：
```shell
SQRT(3)     Linux Programmer,s Manual     SQRT(3)
NAME
       sqrt, sqrtf, sqrtl - square root function
SYNOPSIS
       #include <math.h>
double sqrt(double x);
       float sqrtf(float x);
       long double sqrtl(long double x);
Link with -lm.
Feature   Test   Macro   Requirements  for  glibc  (see  fea‐
   ture_test_macros(7)):
sqrtf(), sqrtl():
           _BSD_SOURCE || _SVID_SOURCE || _XOPEN_SOURCE >= 600
           || _ISOC99_SOURCE || _POSIX_C_SOURCE >= 200112L;
           or cc -std=c99
```
* #include 表示需要的 header。

* 接着是函式的 signature。

* Link with -lm 表示使用编译时要加 -lm ，linker 才會找到 libm.so。

* Feature Test Macro 是 UNIX 跨平台用的一套规范，见 man feature_test_macros 了解作用，以及 glibc 认得的类型。feature test macro 要定义在档案的开头或直接用 gcc -D 定义。

3. 可用 gcc -E 列出前置处理（CPP）展开 #include，#define，#if …等指令后的程式码，查看是否有引入正确的档案。细节说明见“GCC展开前置处理的技巧”。

# 解决连结共享库/可执行文件的问题（static linker的工作）
类似处理编译的作法，用系统附带的套件 X 时，先看有没有 pkg-config 的设定档。没有的话用 dpkg -L X-dev 找位置。

要注意的是静态库和共享库的行为不同。静态库只是一群对象文件的集合。它没有记额外资讯（像是需要那些库）。若执行程序 X 用到静态库 A，而 A 用到库 B 。编 X 时，要加上 -lA 和 -lB 的参数。编 X 的一部分要知道它用到的静态库有那些相依性，而不是 A 自己会搞定自己的相依性。若 A 是共享库，在编出 A 的时候，会记录 A 需要 B，编 X 时不需另外指定 -lB。
更多细节见《 Linux编译共享库的方法和注意事项》。

# 解决执行期的问题（动态/运行时链接程序的工作）
可在执行时设环境 LD_DEBUG=libs,symbols 变数然后执行程序，了解加载的过程。除错的细节见《 Linux执行时寻找符号的流程以及共享库相关知识》。
