# Linux执行时寻找symbol的流程以及shared_library相关知识

声明：本文在大佬[原文基础](https://medium.com/fcamels-notes/linux-%E5%9F%B7%E8%A1%8C%E6%99%82%E5%B0%8B%E6%89%BE-symbol-%E7%9A%84%E6%B5%81%E7%A8%8B%E4%BB%A5%E5%8F%8A-shared-library-%E7%9B%B8%E9%97%9C%E7%9F%A5%E8%AD%98-b0cf1e19cbf3)上结合自己遇到的问题，做了一个整理。


* 为何需要共享库？

* 如何解决执行时找不到某个符号/库？

* 如何在执行时替换特定的符号？

* 有多个重覆的符号时，怎么确保执行时用到期望的符号？

symbol 可以是变数或函式。

## 静态库和共享库

静态库只是将一堆目标文件包在一个档案中，然后在产生可执行文件时，从中取出用到的一部分，加入可执行文件。由于符号是直接加入可执行文件里，执行时没什么需要担心的问题。

相较于静态库，共享库具有以下优点：

* 节省硬盘空间（避免多个可执行文件有重复的程序和资料）

* 节省系统内存（被多个进程公用时，程序只占用一份空间）

* 方便替换 **library** 而不用重编程序（有时无法重编）

* 开发时节省 **link** 的时间

* 设定 **symbol visibility** 避免使用模组的 **private API** ，确保模组的封裝效果。

* 符合 **LGPL**

缺点就是行为变得复杂很多

另外，若要最佳的执行效率以及安全性（避免被替换函数），将全部符号编成一个执行档是最安全的。以 **Chrome** 的方式， **Chrome** 正式版会编成一个执行档，称为 **static build**；平时开发则用 **component build**（各个模块编成各自的共享库），节省开发时间。在 **static build** 的情况下，链接 **Chrome** 执行档要 8G 以上的内存花费数分钟时间。反观 **component build** 的情况，多数组件只需几秒钟完成链接，只有 **Blink** 需要更长久的时间。

## 基本观念

对有使用共享库的可执行文件，程序内部包含一串未定义的 **symbol** 以及一串使用到的 **shared library** 。要注意的是，**ELF** 没有指定每个符号要去那个共享库找，两个列表独立使用。这个设计提供执行时替换符号实作的弹性。

可以用 **nm -Du** 列出未定义需要外部提供的符号：
```shell
$ nm -Du /bin/ls
U abort
U __assert_fail
U bindtextdomain
U calloc
U clock_gettime
U close
U closedir
U __ctype_b_loc
U __ctype_get_mb_cur_max
U __ctype_tolower_loc
...
```

可以用 **ldd** 列出需要的 **shared library** :

```shell
$ ldd /bin/ls
linux-vdso.so.1 =>  (0x00007ffe7a5fe000)
libselinux.so.1 => /lib/x86_64-linux-gnu/libselinux.so.1 (0x00007fd64b7a1000)
libc.so.6 => /lib/x86_64-linux-gnu/libc.so.6 (0x00007fd64b3d7000)
libpcre.so.3 => /lib/x86_64-linux-gnu/libpcre.so.3 (0x00007fd64b167000)
libdl.so.2 => /lib/x86_64-linux-gnu/libdl.so.2 (0x00007fd64af63000)
/lib64/ld-linux-x86-64.so.2 (0x00007fd64b9c3000)
libpthread.so.0 => /lib/x86_64-linux-gnu/libpthread.so.0 (0x00007fd64ad46000)
```

或是用 **objdump** :

```shell
$ objdump -p /bin/ls
...
Dynamic Section:
  NEEDED               libselinux.so.1
  NEEDED               libc.so.6
  ...
```

**ldd** 是个 **shell** 脚本，会递回回列出全部用到的共享库，比较好用。

>另外，对于执行中的过程，可以查看 **/proc/[PID]/maps** 发现它目前用到的共享库。 **man proc** 可知晓 **/proc/[PID]/maps** 更多资讯，像是如何获取 **PID** 当前自己使用到的内存数量（私有内存）。

## 执行期寻找共享库的流程

执行程序的时候，动态（运行时）链接器（ld.so）会依以下顺序查找（man ld.so有详细说明）可执行文件内列的共享库：

* 若 **shared library** 名称内有“ /”，表示它是路径，直接用这个路径找。

* 若可执行文件内有定义 **DT_RPATH** 没定义 **DT_RUNPATH** ，

* 从 **DT_RPATH** 列的目录里找。

* 从 **LD_LIBRARY_PATH** 列的目录里找。

* 从 **DT_RUNPATH** 列的目录里找。

* 从产生 **ldconfig** 的 **cache** 内找（/etc/ld.so.cache）。

* 从 **OS** 的预设位置找：先找 **/lib** 再找 **/usr/lib** 。

用 **[LD_DEBUG=libs]()** 可以看找的过程。

例子：
```Cpp
LD_DEBUG=libs ./RTSAQuickstart
      9184:	find library=libasan.so.4 [0]; searching
      9184:	 search path=/tmp/tmp.FPJ1Snt7Un/libs/agora_media_sdk/tls/x86_64/x86_64:/tmp/tmp.FPJ1Snt7Un/libs/agora_media_sdk/tls/x86_64:/tmp/tmp.FPJ1Snt7Un/libs/agora_media_sdk/tls/x86_64:/tmp/tmp.FPJ1Snt7Un/libs/agora_media_sdk/tls:/tmp/tmp.FPJ1Snt7Un/libs/agora_media_sdk/x86_64/x86_64:/tmp/tmp.FPJ1Snt7Un/libs/agora_media_sdk/x86_64:/tmp/tmp.FPJ1Snt7Un/libs/agora_media_sdk/x86_64:/tmp/tmp.FPJ1Snt7Un/libs/agora_media_sdk:/tmp/tmp.FPJ1Snt7Un/cmake-build-debug/bin/tls/x86_64/x86_64:/tmp/tmp.FPJ1Snt7Un/cmake-build-debug/bin/tls/x86_64:/tmp/tmp.FPJ1Snt7Un/cmake-build-debug/bin/tls/x86_64:/tmp/tmp.FPJ1Snt7Un/cmake-build-debug/bin/tls:/tmp/tmp.FPJ1Snt7Un/cmake-build-debug/bin/x86_64/x86_64:/tmp/tmp.FPJ1Snt7Un/cmake-build-debug/bin/x86_64:/tmp/tmp.FPJ1Snt7Un/cmake-build-debug/bin/x86_64:/tmp/tmp.FPJ1Snt7Un/cmake-build-debug/bin:/tmp/tmp.FPJ1Snt7Un/cmake-build-debug/opusfile-build/tls/x86_64/x86_64:/tmp/tmp.FPJ1Snt7Un/cmake-build-debug/opusfile-build/tls/x86_64:/tmp/tmp.FPJ1Snt7Un/cmake-build-debug/opusfile-build/tls/x86_64:/tmp/tmp.FPJ1Snt7Un/cmake-build-debug/opusfile-build/tls:/tmp/tmp.FPJ1Snt7Un/cmake-build-debug/opusfile-build/x86_64/x86_64:/tmp/tmp.FPJ1Snt7Un/cmake-build-debug/opusfile-build/x86_64:/tmp/tmp.FPJ1Snt7Un/cmake-build-debug/opusfile-build/x86_64:/tmp/tmp.FPJ1Snt7Un/cmake-build-debug/opusfile-build:/tmp/tmp.FPJ1Snt7Un/cmake-build-debug/opus-build/tls/x86_64/x86_64:/tmp/tmp.FPJ1Snt7Un/cmake-build-debug/opus-build/tls/x86_64:/tmp/tmp.FPJ1Snt7Un/cmake-build-debug/opus-build/tls/x86_64:/tmp/tmp.FPJ1Snt7Un/cmake-build-debug/opus-build/tls:/tmp/tmp.FPJ1Snt7Un/cmake-build-debug/opus-build/x86_64/x86_64:/tmp/tmp.FPJ1Snt7Un/cmake-build-debug/opus-build/x86_64:/tmp/tmp.FPJ1Snt7Un/cmake-build-debug/opus-build/x86_64:/tmp/tmp.FPJ1Snt7Un/cmake-build-debug/opus-build:/tmp/tmp.FPJ1Snt7Un/cmake-build-debug/ogg-build/tls/x86_64/x86_64:/tmp/tmp.FPJ1Snt7Un/cmake-build-debug/ogg-build/tls/x86_64:/tmp/tmp.FPJ1Snt7Un/cmake-build-debug/ogg-build/tls/x86_64:/tmp/tmp.FPJ1Snt7Un/cmake-build-debug/ogg-build/tls:/tmp/tmp.FPJ1Snt7Un/cmake-build-debug/ogg-build/x86_64/x86_64:/tmp/tmp.FPJ1Snt7Un/cmake-build-debug/ogg-build/x86_64:/tmp/tmp.FPJ1Snt7Un/cmake-build-debug/ogg-build/x86_64:/tmp/tmp.FPJ1Snt7Un/cmake-build-debug/ogg-build		(RUNPATH from file ./RTSAQuickstart)
      9184:	  trying file=/tmp/tmp.FPJ1Snt7Un/libs/agora_media_sdk/tls/x86_64/x86_64/libasan.so.4
      9184:	  trying file=/tmp/tmp.FPJ1Snt7Un/libs/agora_media_sdk/tls/x86_64/libasan.so.4
      ...
      9184:	find library=librt.so.1 [0]; searching
      9184:	 search cache=/etc/ld.so.cache
      9184:	  trying file=/lib/x86_64-linux-gnu/librt.so.1
      9184:
      9184:	find library=libm.so.6 [0]; searching
      9184:	 search cache=/etc/ld.so.cache
      9184:	  trying file=/lib/x86_64-linux-gnu/libm.so.6
      9184:
      9184:	find library=libopusfile.so [0]; searching
      9184:	 search path=/tmp/tmp.FPJ1Snt7Un/libs/agora_media_sdk:/tmp/tmp.FPJ1Snt7Un/cmake-build-debug/opusfile-build:/tmp/tmp.FPJ1Snt7Un/cmake-build-debug/opus-build:/tmp/tmp.FPJ1Snt7Un/cmake-build-debug/ogg-build		(RUNPATH from file ./RTSAQuickstart)
      9184:	  trying file=/tmp/tmp.FPJ1Snt7Un/libs/agora_media_sdk/libopusfile.so
      9184:	  trying file=/tmp/tmp.FPJ1Snt7Un/cmake-build-debug/opusfile-build/libopusfile.so
      9184:
      9184:	find library=libopus.so.0 [0]; searching
      9184:	 search path=/tmp/tmp.FPJ1Snt7Un/libs/agora_media_sdk:/tmp/tmp.FPJ1Snt7Un/cmake-build-debug/opusfile-build:/tmp/tmp.FPJ1Snt7Un/cmake-build-debug/opus-build:/tmp/tmp.FPJ1Snt7Un/cmake-build-debug/ogg-build		(RUNPATH from file ./RTSAQuickstart)
      9184:	  trying file=/tmp/tmp.FPJ1Snt7Un/libs/agora_media_sdk/libopus.so.0
      9184:	  trying file=/tmp/tmp.FPJ1Snt7Un/cmake-build-debug/opusfile-build/libopus.so.0
      9184:	  trying file=/tmp/tmp.FPJ1Snt7Un/cmake-build-debug/opus-build/libopus.so.0
      9184:
      9184:	find library=libogg.so.0 [0]; searching
      9184:	 search path=/tmp/tmp.FPJ1Snt7Un/libs/agora_media_sdk:/tmp/tmp.FPJ1Snt7Un/cmake-build-debug/opusfile-build:/tmp/tmp.FPJ1Snt7Un/cmake-build-debug/opus-build:/tmp/tmp.FPJ1Snt7Un/cmake-build-debug/ogg-build		(RUNPATH from file ./RTSAQuickstart)
      9184:	  trying file=/tmp/tmp.FPJ1Snt7Un/libs/agora_media_sdk/libogg.so.0
      9184:	  trying file=/tmp/tmp.FPJ1Snt7Un/cmake-build-debug/opusfile-build/libogg.so.0
      9184:	  trying file=/tmp/tmp.FPJ1Snt7Un/cmake-build-debug/opus-build/libogg.so.0
      9184:	  trying file=/tmp/tmp.FPJ1Snt7Un/cmake-build-debug/ogg-build/libogg.so.0
      9184:
      9184:	find library=libagora_rtc_sdk.so [0]; searching
      9184:	 search path=/tmp/tmp.FPJ1Snt7Un/libs/agora_media_sdk:/tmp/tmp.FPJ1Snt7Un/cmake-build-debug/opusfile-build:/tmp/tmp.FPJ1Snt7Un/cmake-build-debug/opus-build:/tmp/tmp.FPJ1Snt7Un/cmake-build-debug/ogg-build		(RUNPATH from file ./RTSAQuickstart)
      9184:	  trying file=/tmp/tmp.FPJ1Snt7Un/libs/agora_media_sdk/libagora_rtc_sdk.so
```
可以清楚的看到他的寻找路径。

执行期间寻找 **symbol** 的流程

有了 **shared library** 的路径列表后，每看到一個 symbol，先从 **executable** 找，找不到再照 **shared library** 的顺序找，找到第一个符合的 **symbol** 就用。**shared library** 的顺序是链接 **executable** 时決定的，和 **ldd** 列的顺序一致。注意，**executable** 可以替换掉 **shared library** 內使用的 **global symbol** ，甚至是 **shared library** 自己定义的 **global symbol** (例如标准函数库內使用自己定义的 **malloc**)。编译 **shared library** 时添加参数改变此行为，详情可参考[传送门]()。

例子：
```shell
ldd RTSAQuickstart
	linux-vdso.so.1 (0x00007ffc8f79c000)
	libasan.so.4 => /usr/lib/x86_64-linux-gnu/libasan.so.4 (0x00007ff5ec9ae000)
	libpthread.so.0 => /lib/x86_64-linux-gnu/libpthread.so.0 (0x00007ff5ec78f000)
	libAgoraRTSAWrapper.so => /tmp/tmp.FPJ1Snt7Un/cmake-build-debug/bin/libAgoraRTSAWrapper.so (0x00007ff5ec37e000)
	libstdc++.so.6 => /usr/lib/x86_64-linux-gnu/libstdc++.so.6 (0x00007ff5ebff5000)
	libgcc_s.so.1 => /lib/x86_64-linux-gnu/libgcc_s.so.1 (0x00007ff5ebddd000)
	libc.so.6 => /lib/x86_64-linux-gnu/libc.so.6 (0x00007ff5eb9ec000)
	libdl.so.2 => /lib/x86_64-linux-gnu/libdl.so.2 (0x00007ff5eb7e8000)
	librt.so.1 => /lib/x86_64-linux-gnu/librt.so.1 (0x00007ff5eb5e0000)
	libm.so.6 => /lib/x86_64-linux-gnu/libm.so.6 (0x00007ff5eb242000)
	/lib64/ld-linux-x86-64.so.2 (0x00007ff5edbc9000)
	libopusfile.so => /tmp/tmp.FPJ1Snt7Un/cmake-build-debug/opusfile-build/libopusfile.so (0x00007ff5eb031000)
	libopus.so.0 => /tmp/tmp.FPJ1Snt7Un/cmake-build-debug/opus-build/libopus.so.0 (0x00007ff5eadaa000)
	libogg.so.0 => /tmp/tmp.FPJ1Snt7Un/cmake-build-debug/ogg-build/libogg.so.0 (0x00007ff5eaba0000)
	libagora_rtc_sdk.so => /tmp/tmp.FPJ1Snt7Un/libs/agora_media_sdk/libagora_rtc_sdk.so (0x00007ff5ea103000)
```

可以用 **[LD_DEBUG=symbols]()** 看找 **symbol** 的流程.

```Cpp
LD_DEBUG=symbols ./RTSAQuickstart
      9242:	symbol=_res;  lookup in file=./RTSAQuickstart [0]
      9242:	symbol=_res;  lookup in file=/usr/lib/x86_64-linux-gnu/libasan.so.4 [0]
      9242:	symbol=_res;  lookup in file=/lib/x86_64-linux-gnu/libpthread.so.0 [0]
      9242:	symbol=_res;  lookup in file=/tmp/tmp.FPJ1Snt7Un/cmake-build-debug/bin/libAgoraRTSAWrapper.so [0]
      9242:	symbol=_res;  lookup in file=/usr/lib/x86_64-linux-gnu/libstdc++.so.6 [0]
      9242:	symbol=_res;  lookup in file=/lib/x86_64-linux-gnu/libgcc_s.so.1 [0]
      9242:	symbol=_res;  lookup in file=/lib/x86_64-linux-gnu/libc.so.6 [0]
      9242:	symbol=stderr;  lookup in file=./RTSAQuickstart [0]
      9242:	symbol=stderr;  lookup in file=/usr/lib/x86_64-linux-gnu/libasan.so.4 [0]
      9242:	symbol=stderr;  lookup in file=/lib/x86_64-linux-gnu/libpthread.so.0 [0]
      9242:	symbol=stderr;  lookup in file=/tmp/tmp.FPJ1Snt7Un/cmake-build-debug/bin/libAgoraRTSAWrapper.so [0]
      9242:	symbol=stderr;  lookup in file=/usr/lib/x86_64-linux-gnu/libstdc++.so.6 [0]
      9242:	symbol=stderr;  lookup in file=/lib/x86_64-linux-gnu/libgcc_s.so.1 [0]
      9242:	symbol=stderr;  lookup in file=/lib/x86_64-linux-gnu/libc.so.6 [0]
      9242:	symbol=error_one_per_line;  lookup in file=./RTSAQuickstart [0]
      9242:	symbol=error_one_per_line;  lookup in file=/usr/lib/x86_64-linux-gnu/libasan.so.4 [0]
      9242:	symbol=error_one_per_line;  lookup in file=/lib/x86_64-linux-gnu/libpthread.so.0 [0]
      9242:	symbol=error_one_per_line;  lookup in file=/tmp/tmp.FPJ1Snt7Un/cmake-build-debug/bin/libAgoraRTSAWrapper.so [0]
      9242:	symbol=error_one_per_line;  lookup in file=/usr/lib/x86_64-linux-gnu/libstdc++.so.6 [0]
      9242:	symbol=error_one_per_line;  lookup in file=/lib/x86_64-linux-gnu/libgcc_s.so.1 [0]
      9242:	symbol=error_one_per_line;  lookup in file=/lib/x86_64-linux-gnu/libc.so.6 [0]
      9242:	symbol=__morecore;  lookup in file=./RTSAQuickstart [0]
      9242:	symbol=__morecore;  lookup in file=/usr/lib/x86_64-linux-gnu/libasan.so.4 [0]
      9242:	symbol=__morecore;  lookup in file=/lib/x86_64-linux-gnu/libpthread.so.0 [0]
      9242:	symbol=__morecore;  lookup in file=/tmp/tmp.FPJ1Snt7Un/cmake-build-debug/bin/libAgoraRTSAWrapper.so [0]
      9242:	symbol=__morecore;  lookup in file=/usr/lib/x86_64-linux-gnu/libstdc++.so.6 [0]
      9242:	symbol=__morecore;  lookup in file=/lib/x86_64-linux-gnu/libgcc_s.so.1 [0]
      9242:	symbol=__morecore;  lookup in file=/lib/x86_64-linux-gnu/libc.so.6 [0]
      9242:	symbol=__key_encryptsession_pk_LOCAL;  lookup in file=./RTSAQuickstart [0]
      9242:	symbol=__key_encryptsession_pk_LOCAL;  lookup in file=/usr/lib/x86_64-linux-gnu/libasan.so.4 [0]
      9242:	symbol=__key_encryptsession_pk_LOCAL;  lookup in file=/lib/x86_64-linux-gnu/libpthread.so.0 [0]
      9242:	symbol=__key_encryptsession_pk_LOCAL;  lookup in file=/tmp/tmp.FPJ1Snt7Un/cmake-build-debug/bin/libAgoraRTSAWrapper.so [0]
      9242:	symbol=__key_encryptsession_pk_LOCAL;  lookup in file=/usr/lib/x86_64-linux-gnu/libstdc++.so.6 [0]
      9242:	symbol=__key_encryptsession_pk_LOCAL;  lookup in file=/lib/x86_64-linux-gnu/libgcc_s.so.1 [0]
      9242:	symbol=__key_encryptsession_pk_LOCAL;  lookup in file=/lib/x86_64-linux-gnu/libc.so.6 [0]
      9242:	symbol=__progname_full;  lookup in file=./RTSAQuickstart [0]
      9242:	symbol=__progname_full;  lookup in file=/usr/lib/x86_64-linux-gnu/libasan.so.4 [0]
      9242:	symbol=__progname_full;  lookup in file=/lib/x86_64-linux-gnu/libpthread.so.0 [0]
      9242:	symbol=__progname_full;  lookup in file=/tmp/tmp.FPJ1Snt7Un/cmake-build-debug/bin/libAgoraRTSAWrapper.so [0]
      9242:	symbol=__progname_full;  lookup in file=/usr/lib/x86_64-linux-gnu/libstdc++.so.6 [0]
      9242:	symbol=__progname_full;  lookup in file=/lib/x86_64-linux-gnu/libgcc_s.so.1 [0]
```

## 用 LD_PRELOAD 替换 symbol 操作

如前所述，可以通过调整 **shared library** 在命令列的顺序決定先使用谁的操作。若不方便调顺序或者不想重新 **link** 的话，可以用 **LD_PRELOAD**。

若有定义环境变量 **LD_PRELOAD** 或 **/etc/ld.so.preload** 内有列 **shared library** 位置时，**ld.so** 会先从这里找 **symbol** (详细格式见 **man ld.so**)。
下面是替换标准函数库 **putchar** 的例子:

```shell
$ cat mylib.c
#include <stdio.h>
int putchar(int c) {
    printf("call putchar() with %d\n", c);
    return c;
}
$ cat main.c
#include <stdio.h>
int main(void) {
    putchar('X');
    putchar('\n');
    return 0;
}
$ gcc -Wall -fPIC -shared -o libmylib.so mylib.c
$ gcc -o main main.c
$ ./main
X
$ LD_PRELOAD=./libmylib.so ./main
call putchar() with 88
call putchar() with 10
```
这里先着重在 **LD_PRELOAD** 的效果

若想操作 **wrapper** (例如追踪 **malloc/free** 使用情況)，需要调用原本的函式，要用到 **GNU** 的延伸功能 **RTLD_NEXT** ，表示载入「原本规则找到的 **symbol** 」的下一个 **symbol**。

下面是替换 **malloc** 的例子:

```cpp
$ cat mem.c
// RLTD_NEXT is only supported in _GNU_SOURCE.
#define _GNU_SOURCE
#include <stdio.h>
#include <dlfcn.h>
void *malloc(size_t size) {
  static int s_calling = 0;
  void* (*m)(size_t);
// 「标准」取得 function pointer 的写法. 见 TLPI 42.1.2 p863 的说明
  *(void**)(&m) = dlsym(RTLD_NEXT, "malloc");
  if (s_calling) {
    // Avoid recursion.
    return m(size);
  } else {
    s_calling = 1;
    printf("malloc size=%zu\n", size);
    s_calling = 0;
    return m(size);
  }
}
$ cat main2.c
#include <stdio.h>
#include <stdlib.h>
int main(void) {
  char* s = malloc(10);
  return 0;
}
$ gcc -Wall -fPIC -shared -o libmem.so mem.c -ldl
$ gcc -g -o main2 main2.c
$ LD_PRELOAD=./libmem.so ./main2
malloc size=10
```

**dlsym()** 用來从 **shared library** 动态载入 **symbol** ，更多的说明见 [The Linux Programming Interface (TLPI) 42.1](http://man7.org/tlpi/)。

## Recap: 如何解決执行期间找不到 shared library？

先用 **LD_LIBRARY_PATH** 指定 **shared library** 所在的位置，确保可以正常执行。
正解则是在用到目标 **shared library X** 的 **executable / shared library** 里加入 **rpath** 指定 **X** 的位置。或是复制 **X** 到 **ldconfig** 扫描的位置。

## Recap:如何解決执行期间找不到 symbol？

先了解 **symbol** 定义在那個 **library** 里 (这里假设在 **X** 里)，然后用 **ldd** 或者 **LD_DEBUG=libs** 看是否有载入目标 **X**。沒有的話，可以先用 **LD_LIBRARY_PATH** 指定 **X** 的位置，借此确认「有载入目标 **library** 的情況，可以解決问题。」

载入 **X** 沒解決问题的话，用 **nm -D** 检查 **symbol** 是否有在 **library** 內。沒有的話要先解決這個問題。

确认载入 **X** 后可解決問題，可以在用到 **symbol** 的 **executable / shared library** 內加入 **rpath** 指定 **X** 的位置。或是复制 **X** 到 **ldconfig** 扫描的位置。

## Recap: 如何确定执行期间用对 symbol？

用 **LD_DEBUG_OUTPUT=/path/to/log LD_DEBUG=symbols PROG**，查看结果。关于 **LD_DEBUG** 的其它用法，可以看 **[man ld.so](http://man7.org/linux/man-pages/man8/ld.so.8.html)** 还有 **LD_DEBUG=help PROG**。
