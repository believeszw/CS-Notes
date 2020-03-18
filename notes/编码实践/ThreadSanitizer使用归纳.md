# ThreadSanitizer 使用归纳

## 支持的平台
* Linux: x86_64, mips64 (40-bit VMA), aarch64 (39/42-bit VMA), powerpc64 (44/46/47-bit VMA)

* Mac: x86_64, aarch64 (39-bit VMA)

* FreeBSD: x86_64

* NetBSD: x86_64

查看最新支持的平台，点[这里](https://github.com/llvm-mirror/compiler-rt/blob/master/lib/tsan/rtl/tsan_platform.h)



## 用法
tsan 主要是检查并发的一些问题.

>注：tsan 是平台无关的，对于简单的赋值操作，都假设它没有原子性，除非加了编译器支持的原子语义。比如 int a；a = 100 ，tsan 并不认为里面的赋值操作是原子的(x86硬件是可是保证内存中四节点对齐的赋值操作是原子的，但是程序员保证不了编译器一定把一个赋值操作编译成四节点对齐的指令)，对于平台支持的原子主义，如 int a ; `__atomic_load_n(&a,__ATOMIC_SEQ_CST )`，认为是一个原子的赋值操作。

只需使用 -fsanitize=thread 编译程序并将其与 -fsanitize=thread 链接即可。要获得合理的性能，请添加 -O2。使用 -g 在警告消息中获取文件名和行号。

运行该程序时，如果 TSan 找到数据竞争，它将打印报告。这是一个例子：
```Cpp
$ cat simple_race.cc
#include <pthread.h>
#include <stdio.h>

int Global;

void *Thread1(void *x) {
  Global++;
  return NULL;
}

void *Thread2(void *x) {
  Global--;
  return NULL;
}

int main() {
  pthread_t t[2];
  pthread_create(&t[0], NULL, Thread1, NULL);
  pthread_create(&t[1], NULL, Thread2, NULL);
  pthread_join(t[0], NULL);
  pthread_join(t[1], NULL);
}
```
```Cpp
$ clang++ simple_race.cc -fsanitize=thread -fPIE -pie -g
$ ./a.out
==================
WARNING: ThreadSanitizer: data race (pid=26327)
  Write of size 4 at 0x7f89554701d0 by thread T1:
    #0 Thread1(void*) simple_race.cc:8 (exe+0x000000006e66)

  Previous write of size 4 at 0x7f89554701d0 by thread T2:
    #0 Thread2(void*) simple_race.cc:13 (exe+0x000000006ed6)

  Thread T1 (tid=26328, running) created at:
    #0 pthread_create tsan_interceptors.cc:683 (exe+0x00000001108b)
    #1 main simple_race.cc:19 (exe+0x000000006f39)

  Thread T2 (tid=26329, running) created at:
    #0 pthread_create tsan_interceptors.cc:683 (exe+0x00000001108b)
    #1 main simple_race.cc:20 (exe+0x000000006f63)
==================
ThreadSanitizer: reported 1 warnings
```
[报告格式分析](https://github.com/google/sanitizers/wiki/ThreadSanitizerReportFormat)


>有一些data race 的 bug 非常难解决，一、因为是并发导致的问题，简单的测试用例很难测试出来这样的 bug 。二、它出现的概率非常小，即使出现了 bug 发生了，根据 bug 发现的现象很难分析到 bug 所在的代码位置，而且如果 data race 与其它一些类型的 bug 组合，会导致程序出现一些千奇百怪的现象，根据这些现象无法逆向分析出来根因所在。三、并发编程难度比较大，用文字描述比较困难，我们的编程规范对于并发编程也没有明确的要求，很容易引用 data race 的 bug 。所以感觉用工具来检查代码中是否有潜藏的 data race 非常必要。
