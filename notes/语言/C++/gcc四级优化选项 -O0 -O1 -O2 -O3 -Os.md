# gcc四级编译优化选项 -O0 -O1 -O2 -O3 -Os

>参考GCC文档 http://gcc.gnu.org/onlinedocs/gcc/Optimize-Options.html.

查看GCC各选项打开的优化项: `gcc -Q --help=optimizers` 。分为如下:

```Cpp
O0   : 默认选项，不做任何优化。

O/O1 : 会尝试减少代码体积和代码运行时间，但是并不执行会花费大量时间的优化操作。

O2   : 显著提升编译时间，提升代码性能，做不包含 `space-speed tradeoff` (空间和速度之间权衡) 的所有优化

O3   : 进一步优化，显著增加可执行文件大小。使用一些向量型算法，提高代码并行执行程度，利用现代 CPU 中的流水线、 Cache 等。

Os   : 针对程序空间大小优化（多用于嵌入式系统）优化性能同时不增加可执行文件大小。包含 O2 选项中不增加代码大小的优化项，同时 -Os 还会执行更加优化程序空间的选项。

Og   : 优化性能同时不损害可调试性。包含 O1 选项中不损害可调试性的优化项

Ofast: 忽视严格的标准编译性。包含 O3 中所有选项及 ` -ffast-math`
```

## O1优化项
```cpp
-fauto-inc-dec
-fbranch-count-reg
-fcombine-stack-adjustments
-fcompare-elim
-fcprop-registers // 使用寄存器之间 copy-propagation 传值;因为在函数中把寄存器分配给变量, 所以编译器执行第二次检查以便减少调度依赖性(两个段要求使用相同的寄存器)并且删除不必要的寄存器复制操作
-fdce
-fdefer-pop // 推迟退出函数调用的参数，对于那些需要在函数调用后必须取出（pop）函数参数的机器而言，打开该项编译器将把函数调用的参数压入栈，等必要时几个函数调用参数一起取出（pop）。这将节省处理时间。
-fdelayed-branch // 如果对目标机支持这个功能,它试图重新排列指令,以便利用延迟分支 (delayed branch) 指令后面的指令空隙.
-fdse
-fforward-propagate
-fguess-branch-probability // 使用启发式算法预测分之指令，增加指令的命中率，提升运行效果。
-fif-conversion
-fif-conversion2
-finline-functions-called-once
-fipa-profile
-fipa-pure-const
-fipa-reference
-fipa-reference-addressable
-fmerge-constants
-fmove-loop-invariants
-fomit-frame-pointer
-freorder-blocks
-fshrink-wrap
-fshrink-wrap-separate
-fsplit-wide-types
-fssa-backprop
-fssa-phiopt
-ftree-bit-ccp
-ftree-ccp
-ftree-ch
-ftree-coalesce-vars
-ftree-copy-prop
-ftree-dce
-ftree-dominator-opts
-ftree-dse
-ftree-forwprop
-ftree-fre
-ftree-phiprop
-ftree-pta
-ftree-scev-cprop
-ftree-sink
-ftree-slsr
-ftree-sra
-ftree-ter
-funit-at-a-time
```

## O2优化项
O1基础上叠加如下项。
```Cpp
-falign-functions  -falign-jumps  ---- 对齐函数地址，加快CPU访问速度，增加代码大小
-falign-labels  -falign-loops  ---- 同上
-fcaller-saves
-fcode-hoisting
-fcrossjumping ---- 生成一致代码，减少代码大小
-fcse-follow-jumps   ----
-fdelete-null-pointer-checks  ---- 去除无用空指针检查(解引用后的检查)
-fdevirtualize  -fdevirtualize-speculatively ---- 尝试优化虚函数调用为实际函数调用，猜测实际调用的对象
-fexpensive-optimizations  ---- 一些很耗费CPU的次要优化
-fgcse  -fgcse-lm  ---- 尝试进行公同子串表达式替换 common subexpress elimination
-fhoist-adjacent-loads ---- likely和unlikely 猜测if/else的情况
-finline-small-functions ---- 小函数inline化
-findirect-inlining ---- 对不是直接调用的小函数也尝试做inline，基于上面的推测
-fipa-bit-cp  -fipa-cp  -fipa-icf
-fipa-ra  -fipa-sra  -fipa-vrp
-fisolate-erroneous-paths-dereference ---- 检测可能导致空指针解引用的异常逻辑，并和主逻辑隔离
-flra-remat
-foptimize-sibling-calls ---- 优化重复调用和递归调用
-foptimize-strlen ---- 优化标准C strlen函数实现
-fpartial-inlining ---- 内联函数的某些部分
-fpeephole2 ---- 特定机型的优化
-freorder-blocks-algorithm=stc ---- 代码块重新排序，software trace cache算法，把常调用的部分流程聚合在一起
-freorder-blocks-and-partition  -freorder-functions ---- 根据局部性原理，将常调用的代码块放在同一个.o文件中
-frerun-cse-after-loop ---- loop优化后重新执行csc
-fschedule-insns  -fschedule-insns2 ---- 优化代码执行次序(减少CPU等待获取数据的损耗)
-fsched-interblock  -fsched-spec
-fstore-merging  ---- 及时数对齐，不足1个Word的填充为1个Word
-fstrict-aliasing
-fthread-jumps ---- 对代码跳转路径的优化
-ftree-builtin-call-dce ---- 去除不可达代码
-ftree-pre ---- 执行partial redundancy elimination
-ftree-switch-conversion  -ftree-tail-merge ---- 对switch的优化
-ftree-vrp ---- 剔除无用变量范围检查
```

## O3优化项
O2基础上叠加如下项。
```Cpp
-fgcse-after-reload
-finline-functions
-fipa-cp-clone
-floop-interchange
-floop-unroll-and-jam
-fpeel-loops
-fpredictive-commoning
-fsplit-paths
-ftree-loop-distribute-patterns
-ftree-loop-distribution
-ftree-loop-vectorize
-ftree-partial-pre
-ftree-slp-vectorize
-funswitch-loops
-fvect-cost-model
-fversion-loops-for-strides
```

***

## 大家对O3普遍比较谨慎，主要因为:
* 前期 Gcc 版本中，O3 is buggy
* O3 往往优化后比 O2 还要慢一些

第 2 点，主要是因为，O3 在做优化时，采用了很激进的策略，例如激进的循环展开、函数内联等，导致生成的代码比较大，可能超出了 CPU 的指令 Cache `instruction cache` ，破坏了局部性和完整性。

对C++开发而言，性能提升最明显的大概还是:

* 对齐 allignment ，变量内存和指令 `-falign`

* stl的优化

  * 小函数内联优化 `-finline-small-functions`

  * template 未调用的函数不会被编译

  * template 独立编译每个类型

template 增加编译时间，但往往性能会有更好的提升。主要在于，template 的机制使得更多的小函数内联成为可能。以 std::sort 为例，和 C 的 qsort 对比，参考 stack flow 上这个问题 [C++ templates for performance](https://stackoverflow.com/questions/8925177/c-templates-for-performance/8925657)? 。


***

debug模式编译器参数为 `-O0 -g –Wall`

release模式编译器参数为 `-O2 -g –Wall`

## -O0
这里只介绍优化编译的参数
-O 用来开启优化编译选项。
-O0：默认模式，不做任何优化。

## -O1



## Gcc 编译优化简介


查查gcc手册就知道了，每个编译选项都控制着不同的优化选项 下面从网络上copy过来的，真要用到这些还是推荐查阅手册
-O设置一共有五种：-O0、-O1、-O2、-O3和-Os。

除了-O0以外，每一个-O设置都会多启用几个选项，请查阅gcc手册的优化选项章节，以便了解每个-O等级启用了哪些选项及它们有何作用。

让我们来逐一考察各个优化等级：

-O0：这个等级（字母“O”后面跟个零）关闭所有优化选项，也是CFLAGS或CXXFLAGS中没有设置-O等级时的默认等级。这样就不会优化代码，这通常不是我们想要的。
-O1：这是最基本的优化等级。编译器会在不花费太多编译时间的同时试图生成更快更小的代码。这些优化是非常基础的，但一般这些任务肯定能顺利完成。
-O2：-O1的进阶。这是推荐的优化等级，除非你有特殊的需求。-O2会比-O1启用多一些标记。设置了-O2后，编译器会试图提高代码性能而不会增大体积和大量占用的编译时间。
-O3：这是最高最危险的优化等级。用这个选项会延长编译代码的时间，并且在使用gcc4.x的系统里不应全局启用。自从3.x版本以来gcc的行为已经有了极大地改变。在3.x，-O3生成的代码也只是比-O2快一点点而已，而gcc4.x中还未必更快。用-O3来编译所有的软件包将产生更大体积更耗内存的二进制文件，大大增加编译失败的机会或不可预知的程序行为（包括错误）。这样做将得不偿失，记住过犹不及。在gcc 4.x.中使用-O3是不推荐的。
-Os：这个等级用来优化代码尺寸。其中启用了-O2中不会增加磁盘空间占用的代码生成选项。这对于磁盘空间极其紧张或者CPU缓存较小的机器非常有用。但也可能产生些许问题，因此软件树中的大部分ebuild都过滤掉这个等级的优化。使用-Os是不推荐的。
