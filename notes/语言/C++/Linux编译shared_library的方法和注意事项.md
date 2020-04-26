# Linux编译shared_library的方法和注意事项

## static linker 和 dynamic (runtime) linker

* static linker 负责 link 产生 shared library 和 executable，在 Linux 上预设是 ld (ld.bfd)， Google 有另写一套 gold 取代 ld，两者的预设行为略有不同。gcc 是 compiler，但它會使用 static linker 产生 shared library / executable。有时需要透过 gcc 传参数给 static linker (后述)。

* dynamic (runtime) linker 负责在执行期间载入 shared library ，Linux 上是ld.so，这在[前篇文章](https://github.com/believeszw/CS-Notes/blob/master/notes/语言/C++/Linux执行时寻找symbol的流程以及shared_library相关知识.md)介绍过了。

这篇将重心放在 static linker。

## 基本例子

代码：
```shell
$ cat foo.h
#ifndef FOO_H
#define FOO_H
void foo();
#endif
$ cat foo.c
#include "foo.h"
#include <stdio.h>
void foo() {
  printf("call foo\n");
}
$ cat main.c
#include "foo.h"
int main(void) {
  foo();
  return 0;
}
```

编译 shared library:
```shell
$ gcc -g -fPIC -c foo.c
$ gcc -shared foo.o -o libfoo.so
```
-fPIC 表示要编成 position-independent code，这样不同 process 载入 shared library 时，library 的程序和信息才能放到内存不同位置。

编译可执行程序
```shell
$ gcc -g -o main main.c libfoo.so
$ gcc -g -o main main.c -lfoo -L.
```
两个指令产生一样的结果，但第二个作法比价常见:

* -lfoo: 表示要链接 libfoo.so

* -L.: 表示搜寻 libfoo.so 时，除了预设目录外，也要找目前的位置 ( . )。可以指定多次 -LDIR。

执行程序
```shell
$ LD_LIBRARY_PATH=. ./main
call foo
```
LD_LIBRARY_PATH 是必要的，因为 libfoo.so 不在 ld.so 搜寻的路径里。后面会介绍更好的作法。

用 ldd 查看 main 用到的 shared library:
```shell
$ ldd main
  linux-vdso.so.1 =>  (0x...)
  libfoo.so => not found
  libc.so.6 => /lib/x86_64-linux-gnu/libc.so.6 (0x...)
  /lib64/ld-linux-x86-64.so.2 (0x...)
$ LD_LIBRARY_PATH=. ldd main
  linux-vdso.so.1 =>  (0x...)
  libfoo.so => ./libfoo.so (0x...)
  libc.so.6 => /lib/x86_64-linux-gnu/libc.so.6 (0x...)
  /lib64/ld-linux-x86-64.so.2 (0x...)
```
预设 ldd 找不到 libfoo.so，加上 LD_LIBRARY_PATH 后就找到了。

## 参数顺序的影响
需要多個 object file (* .o)、static library (* .a，只是一包 object file 的集合)、shared library 時，它們之间的順序会有影响。ld 每个库只会看一次，丟掉没用到的 symbol 和记录 undefined symbol，预期后面的库会提供 undefined symbol。看完全部库仍有 undefined symbol，就会 link error。

man gcc 对 -l 的说明如下:
>It makes a difference where in the command you write this option; the linker searches and processes libraries and object files in the order they are specified. Thus, foo.o -lz bar.o searches library z after file foo.o but before bar.o. If bar.o refers to functions in z, those functions may not be loaded.

若出現 A.a 用到 B.a、B.a 用到 A.a 这种 circular dependency，需要多提一次库名:
```shell
$ gcc -g -o main main.o A.a B.a A.a
```
或是用 ld 的参数--start-group 和--end-group 包住 A.a 和 B.a:
```shell
$ gcc -g -o main main.o -Wl,--start-group A.a B.a -Wl,--end-group
```
这样 ld 会重复查看 A.a 和 B.a，缺点是比较沒有效率。

安裝 shared library 到系系统目录

* 若 **shared library** 名称内有“ /”，表示它是路径，直接用这个路径找。

* 若可执行文件内有定义 **DT_RPATH** 没定义 **DT_RUNPATH** ，

* 从 **DT_RPATH** 列的目录里找。

* 从 **LD_LIBRARY_PATH** 列的目录里找。

* 从 **DT_RUNPATH** 列的目录里找。

* 从产生 **ldconfig** 的 **cache** 内找（/etc/ld.so.cache）。

* 从 **OS** 的预设位置找：先找 **/lib** 再找 **/usr/lib** 。

所以复制 shared library 到 /lib 或 /usr/lib 或是 ldconfig 扫描的位置，就不用 LD_LIBRARY_PATH 了。可以从 /etc/ld.so.conf 得知 ldconfig 会扫描的位置。

在 Ubuntu 18.04 上，/usr/local/lib 有在 ldconfig 扫描的位置里。所以前面的例子可以这么改:
```shell
$ gcc -fPIC -g -shared -o libfoo.so foo.c
$ sudo cp libfoo.so /usr/local/lib
$ gcc -g -o main main.c -lfoo # 註1
```
然后更新 ldconfig 的 cache，这样执行 main 的時候才会找到新加人的 libfoo.so:
```shell
$ sudo ldconfig
$ ldd main
  ...
  libfoo.so => /usr/local/lib/libfoo.so (0x...)
  ...
$ ./main
call foo
```
因为 /etc/ld.so.cache 有 libfoo 的记录，就不需要 LD_LIBRARY_PATH 了。
[注 1] Link main 的時候不用加 -L 是因为 /usr/local/lib 已经在 ld 的预设搜索路径里了:
```shell
$ ld --verbose | grep SEARCH
SEARCH_DIR("=/usr/local/lib/x86_64-linux-gnu");
SEARCH_DIR("=/lib/x86_64-linux-gnu");
SEARCH_DIR("=/usr/lib/x86_64-linux-gnu");
SEARCH_DIR("=/usr/local/lib64");
SEARCH_DIR("=/lib64");
SEARCH_DIR("=/usr/lib64");
SEARCH_DIR("=/usr/local/lib");
SEARCH_DIR("=/lib");
SEARCH_DIR("=/usr/lib");
SEARCH_DIR("=/usr/x86_64-linux-gnu/lib64");
SEARCH_DIR("=/usr/x86_64-linux-gnu/lib");
```

## linker name, real name 和 soname
有时会升级 shared library，所以要有方法辨认 shared library 的版本，还有可能需要同时并存同一套但不同版本的 shared library。
对于 library X，Linux 的命名惯例是:

* libX.so.A.B.C: real name。表示版号 A.B.C，A 是 major number，惯例是 A 一样的情況会向前兼容 (library 的开发者会遵守此项惯例)。比方说原本用 libX 1.0.0 的程式，改用 libX 1.2.3 也不会有问题。但用 libX 2.0.0 可能会有问题。

* libX.so.A: soname。供 dynamic linker 使用。

* libX.so: linker name，供 static linker 使用。用 gcc -lX 的时候，会先找 libX.so，找不到再找 libX.a (static library) (见 man ld 的 -l namespec )。

制作 shared library 時，需要:

* (real name) 产生 libX.A.B.C

* (soname) 产生 soft link libX.so.A → libX.A.B.C

* (linker name) 产生 soft link libX.so → libX.so.A

ldconfig 可以帮我們产生 soname，linker name 则要自己产生。
综合上面所知，更正式的产生以及安裝 shared library 流程是:
```shell
$ gcc -fPIC -g -shared -Wl,-soname,libfoo.so.1 -o libfoo.so.1.0.0 foo.c
$ sudo cp libfoo.so.1.0.0 /usr/local/lib
# 在 /usr/local/lib 产生 soname libfoo.so.1 -> libfoo.so.1.0.0
$ sudo ldconfig
# 产生 linker name
$ cd /usr/local/lib && sudo ln -s libfoo.so.1 libfoo.so
```
gcc 用 -Wl,NAME,VALUE 的方式传递参数给 ld，所以 -Wl,-soname,libfoo.so.1 是 gcc 传递参数 -soname=libfoo.so.1 给 ld，这个参数的意思是写入 soname “libfoo.so.1” 到产生的 shared library 里。gcc 的参数说明见 man gcc，ld 的参数则是看 man ld。
Link main 的方法一样:
```shell
$ gcc -g -o main main.c -lfoo
# ldd 的結果略有不同，變成剛才指定的 soname "libfoo.so.1"
$ ldd main
  ...
  libfoo.so.1 => /usr/local/lib/libfoo.so.1 (0x...)
  ...
```
注意 executable main 里是注明需要 shared library “libfoo.so.1”。不是 “libfoo.so.1.0.0”，也不是 “libfoo.so”。因为 ld 是从 libfoo.so.1.0.0 这个库里读出 soname “libfoo.so.1”，然后写入 main。

日后需要更新 libfoo 的话，只要有向前兼容，可以产生不同的 real name 但维持一样的 soname，然后一样复制到 /usr/local/lib，再重跑 ldconfig。ldconfig 会更新 soname 指向新版的 real name。
像这样:
```shell
$ gcc -fPIC -g -shared -Wl,-soname,libfoo.so.1 -o libfoo.so.1.2.3 foo.c
$ sudo cp libfoo.so.1.2.3 /usr/local/lib
# 在 /usr/local/lib 产生 soname libfoo.so.1 -> libfoo.so.1.2.3
$ sudo ldconfig
```
这样旧的程序不用重新 link，它会直接使用新版的 libfoo.so.1.2.3。
若需要产生不向前兼容的 library，需要用不同的 soname:
```shell
# soname 由 libfoo.so.1 改为 libfoo.so.2
$ gcc -fPIC -g -shared -Wl,-soname,libfoo.so.2 -o libfoo.so.2.0.0 foo.c
$ sudo cp libfoo.so.2.0.0 /usr/local/lib
# 在 /usr/local/lib 产生 soname libfoo.so.2 -> libfoo.so.2.0.0
$ sudo ldconfig
# 更新 linker name
$ cd /usr/local/lib && sudo ln -fs libfoo.so.2 libfoo.so
```
之后用 gcc -lfoo 時，会用到 libfoo.so.2.0.0。若还想要用旧版 1.x.x.，就得在 gcc 参数里直接指定在 /usr/local/lib 下的库，而不能用 -lfoo 了。

另外，先前产生的 executable 内有注明是使用 libfoo.so.1，执行旧版程序时，dynamic linker 会找到 libfoo.so.1 → libfoo.so.1.2.3，所以不用担心安裝 libfoo.so.2.x.x 后会用错版本。

## Recap: ld, ld.so, ldconfig 的关系

以 Ubuntu 18.04 上的 libfontconfig 为例说明。

1. 安裝 libfontconfig1:amd64 会产生
```shell
# real name
/usr/lib/x86_64-linux-gnu/libfontconfig.so.1.9.0
# soname
/usr/lib/x86_64-linux-gnu/libfontconfig.so.1
```
* ibfontconfig.so.1.9.0 是目标 shared library。

* ldconfig 产生 soft link libfontconfig.so.1 指向 libfontconfig.so.1.9.0。

2. 对于有使用 libfontconfig 的 executable 来说，ld.so 从 executable 的 DT_NEEDED 位置读到字串 “libfontconfig.so.1” (可用 objdump -p PROG | grep NEEDED 列出此位置)，用这个字串顺着搜寻路径找到 /usr/lib/x86_64-linux-gnu/libfontconfig.so.1，然后载入 /usr/lib/x86_64-linux-gnu/libfontconfig.so.1.9.0。

3. 安裝 libfontconfig1-dev:amd64 会产生
```shell
# linker name
/usr/lib/x86_64-linux-gnu/libfontconfig.so
```
libfontconfig.so 是 soft link 指向 libfontconfig.1.9.0，供开发者使用，这样 gcc -lfontconfig 才能找到它 (告知 ld 去找 libfontconfig.so)。

## rpath
若不想裝到系统路径 ( ldconfig 扫描的位置、/lib、/usr/lib )，也可以在 executable / shared library 内写入 rpath:
```shell
$ gcc -fPIC -g -shared  -o libfoo.so foo.c
$ gcc -g -o main main.c -lfoo -L. -Wl,-rpath,`pwd`
$ ldd main
  ...
  libfoo.so => /home/believe/test/libfoo.so
  ...
$ ./main
call foo
```
main 里多了 DT_RPATH 的信息:
```shell
$ objdump -p main | grep RPATH
  RPATH                /home/believe/test
```

## rpath 和 shared library
假设 libfoo.so 会用到 libbar.so，程序更改如下 :
```Cpp
$ cat foo.c
#include "foo.h"
#include "util/bar.h"
#include <stdio.h>
void foo() {
  printf("call foo\n");
  bar();
}
$ cat util/bar.h
#ifndef BAR_H
#define BAR_H
void bar();
#endif
$ cat util/bar.c
#include "bar.h"
#include <stdio.h>
void bar() {
  printf("call bar\n");
}
```
程序的关系是 main → libfoo.so → libbar.so。可以将 libbar.so 的位置用 rpath 写到 libfoo.so 里，这样产生 main 的时候，不需再指定 libbar.so 的位置:
```shell
$ gcc -fPIC -g -shared -o util/libbar.so util/bar.c
$ gcc -fPIC -g -shared -o libfoo.so foo.c -lbar -Lutil -Wl,-rpath,`pwd`/util
$ gcc -g -o main main.c -lfoo -L. -Wl,-rpath,`pwd`
$ ./main
call foo
call bar
```
用 objdump 和 ldd 检查产生的库:
```shell
$ objdump -p main | grep RPATH
  RPATH                /home/fcamel/test
$ objdump -p libfoo.so | grep RPATH
  RPATH                /home/fcamel/test/util
$ ldd main
  linux-vdso.so.1 =>  (0x...)
  libfoo.so => /home/fcamel/test/libfoo.so (0x...)
  libc.so.6 => /lib/x86_64-linux-gnu/libc.so.6 (0x...)
  libbar.so => /home/fcamel/test/util/libbar.so (0x...)
  /lib64/ld-linux-x86-64.so.2 (0x...)
```
可看出 main 本身不知道 libbar.so 的位置，不过直接使用 libbar.so 的 libfoo.so 知道。
需要注意的是这里 rpath 都用绝对路径，目的是确保 main 移到別的位置后仍能找到 shared library。但使用绝对路径没那么弹性，不方便复制到不同机器使用。

## rpath 的 $ORIGIN

若能将 executable 和用到的 shared library 放在一個目录里，整个目录传到別的机器上就能使用的话，会很方便。也就是说，我们希望以 executable 所在的目录为根，用相对路径來找目录下的 shared library。rpath 的 $ORIGIN 提供这个语意。

$ORIGIN 表示 executable 所在的目录，所以可以将 shared library 放在 lib 下，然後指定 rpath 为 $ORIGIN/lib:
```shell
$ gcc -Wl,rpath,'$ORIGIN'/lib ...
```
这样 executable / shared library 内会记录:
```shell
RPATH                $ORIGIN/lib
```
只要维持 executable 和 shared library 的相对位置，整个目录移到不同位置仍能正常执行。

## Recap: 该用何种方法安裝 shared library？
* 系统共用的 shared library，裝到 ldconfig 扫描的路径，用 ldconfig 产生 soname 并自己产生 linker name。省硬盘空间和内存。

* 开发时会经常重编，将各个模组编成独立的 shared library 可以节省 link 时间。用 rpath=$ORIGIN，比 LD_LIBRARY_PATH 干净。

* 出错时可用 LD_LIBRARY_PATH 检查。

* 部署 executable 到使用者环境时，从安全性和出错 (避免被替换库) 以及效率来看，全部合在一個 executable 较佳。

* 承上，有用到 LGPL 的 library 或是希望提供弹性让使用者可以自行替换不同操作 (例如升级 shared library)，可将相关程序另编成 shared library，然后在 executable 使用rpath=$ORIGIN。

* 部署 executable 到 server 环境时，有很多选择。我偏向使用 rpath=$ORIGIN，这样 server 环境和开发者环境一样，减少环境差异造成不同的行为，还有开发者方便替换自己的出错版本到 server 环境。

## Symbol Interposition 和 Symbol Visibility
symbol interposition (function interposition) 是指替换 symbol 为不同的操作。在前篇提到的 LD_PRELOAD 是最容易使用的方法。预设编译 shared library 后，shared library 内全部 symbol 都是 global symbol，都能被替换。

如前篇所言，ld.so 会先从 executable 内找 symbol，找不到才会从 shared library 找。所以若 executable 有和 shared library 内一样的 symbol，执行 shared library 内程序时，反而会用到 executable 内的 symbol，这违背常理。

编译 shared library 时可加参数 -Bsymbolic 让外部无法替换 shared library 的 symbol (参数细节见 man ld) 。但这个作法是全部一起封杀，失去日后用 LD_PRELOAD 替换部份 public API 的弹性。

所以比较好的作法是将 shared library 的 public API 设为 global symbol，剩下的设为 local symbol，这样外部无法使用也无法替换 local symbol，让 shared library 有更好的封裝效果。开发时将各个模组编成 shared library 并设好 visibility，可以避免使用到 private API (会造成 link error)。symbol visibility 测试的例子见这里。

<details><summary><code>测试的例子</code></summary><br>

```shell
$ cat prog.c
#include <stdio.h>

void func();

void xyz() {
  printf("main-xyz\n");
}

void xyz2() {
  printf("main-xyz2\n");
}

int main(void) {
  func();
  return 0;
}

$ cat foo.c
#include <stdio.h>

void xyz() {
  printf("foo-xyz\n");
}

void xyz2() {
  printf("foo-xyz2\n");
}

void func() {
  xyz();
  xyz2();
}

#
# Default behavior
#

$ gcc -g -c -fPIC -Wall -c foo.c
$ gcc -g -shared -o libfoo.so foo.o
$ nm libfoo.so | grep xyz
0000000000000730 T xyz
0000000000000743 T xyz2
$ gcc -g -o prog prog.c libfoo.so
$ LD_LIBRARY_PATH=. ./prog
main-xyz
main-xyz2

#
# Use -Bsymbolic to reference global symbols internally.
#

$ gcc -g -shared -o libfoo.so foo.o -Wl,-Bsymbolic
$ nm libfoo.so | grep xyz
00000000000006e0 T xyz
00000000000006f3 T xyz2
$ LD_LIBRARY_PATH=. ./prog
foo-xyz
foo-xyz2


#
# Use version script to hide local symbols.
#

$ cat libfoo.map
FOO {
  global: func; xyz2; # explicitly list symbols to be exported.
  local: *;           # hide the rest.
};
$ gcc -g -shared -o libfoo.so foo.o -Wl,--version-script=libfoo.map
$ nm libfoo.so | grep xyz  # T/t means global/local symbol
0000000000000690 t xyz
00000000000006a3 T xyz2
$ LD_LIBRARY_PATH=. ./prog
foo-xyz
main-xyz2
```

</details>

刚才的例子是用 Version Script 设定 global/local symbol。symbol visibility 独立于程序，比较不好维护。比较好的作法是用 gcc 的属性 visibility。基本用法是编译全部程序时都加上 -fvisibility=hidden -fvisibility-inlines-hidden，预设改成产生 local symbol。然后在 public API 前面加上 __attribute__ ((visibility (“default”)))，标示为 global symbol。
像是这样 :
```Cpp
$ cat foo.c
#include <stdio.h>
void xyz() {
  printf("foo-xyz\n");
}
void xyz2() {
  printf("foo-xyz2\n");
}
__attribute__ ((visibility ("default"))) void func() {
  xyz();
  xyz2();
}
$ gcc -g -fPIC -c foo.c -fvisibility=hidden
$ gcc -shared foo.o -o libfoo.so
$ nm libfoo.so
...
00000000000006c6 T func
...
00000000000006a0 t xyz
00000000000006b3 t xyz2
```
xyz() 和 xyz2() 都变成 local symbol 了 (T/t 分別表示 global/local symbol)。
由于 visibility 是 gcc 扩充功能，在不同平台语法有些不同，参考官方文件 visibility 了解跨平台的写法。或是參考 Chromium 的用法，像是 base/base_export.h 定义的 BASE_EXPORT 以及使用 BASE_EXPORT 的例子。

## 参考资料
* [The Linux Programming Interface: ch41 Fundamentals of Shared Libraries](http://man7.org/tlpi/)

* [The Linux Programming Interface: ch42 Advanced Features of Shared Libraries](http://man7.org/tlpi/)

* [man ld.so](http://man7.org/linux/man-pages/man8/ld.so.8.html)

* [man ld](http://man7.org/linux/man-pages/man1/ld.1.html)

## 相關資料
* [《在 Linux 下開發 C/C++ 的新手指南》](https://medium.com/fcamels-notes/%E5%9C%A8-linux-%E4%B8%8B%E9%96%8B%E7%99%BC-c-c-%E7%9A%84%E6%96%B0%E6%89%8B%E6%8C%87%E5%8D%97-735fcd960b0)

* [《Linux 執行時尋找 symbol 的流程以及 shared library 相關知識》](https://medium.com/@fcamel/linux-%E5%9F%B7%E8%A1%8C%E6%99%82%E5%B0%8B%E6%89%BE-symbol-%E7%9A%84%E6%B5%81%E7%A8%8B%E4%BB%A5%E5%8F%8A-shared-library-%E7%9B%B8%E9%97%9C%E7%9F%A5%E8%AD%98-b0cf1e19cbf3)

* [《Linux 編譯 shared library 的方法和注意事項》](https://medium.com/@fcamel/linux-%E7%B7%A8%E8%AD%AF-shared-library-%E7%9A%84%E6%96%B9%E6%B3%95%E5%92%8C%E6%B3%A8%E6%84%8F%E4%BA%8B%E9%A0%85-cb35844ef331)

* [《(C/C++ ) 如何在 Linux 上使用自行編譯的第三方函式庫》](https://medium.com/@fcamel/c-c-%E5%A6%82%E4%BD%95%E5%9C%A8-linux-%E4%B8%8A%E4%BD%BF%E7%94%A8%E8%87%AA%E8%A1%8C%E7%B7%A8%E8%AD%AF%E7%9A%84%E7%AC%AC%E4%B8%89%E6%96%B9%E5%87%BD%E5%BC%8F%E5%BA%AB-1f19c3abaebe)

* [《Chromium 在 link 時用的參數》](https://medium.com/@fcamel/chromium-%E5%9C%A8-link-%E6%99%82%E7%94%A8%E7%9A%84%E5%8F%83%E6%95%B8-e301a1f1d908)
