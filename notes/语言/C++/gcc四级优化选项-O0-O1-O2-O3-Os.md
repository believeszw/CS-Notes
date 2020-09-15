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
-fcprop-registers ---- 使用寄存器之间 copy-propagation 传值;因为在函数中把寄存器分配给变量, 所以编译器执行第二次检查以便减少调度依赖性(两个段要求使用相同的寄存器)并且删除不必要的寄存器复制操作
-fdce
-fdefer-pop ---- 推迟退出函数调用的参数，对于那些需要在函数调用后必须取出（pop）函数参数的机器而言，打开该项编译器将把函数调用的参数压入栈，等必要时几个函数调用参数一起取出（pop）。这将节省处理时间。
-fdelayed-branch ---- 如果对目标机支持这个功能,它试图重新排列指令,以便利用延迟分支 (delayed branch) 指令后面的指令空隙.
-fdse
-fforward-propagate
-fguess-branch-probability ---- 使用启发式算法预测分之指令，增加指令的命中率，提升运行效果。
-fif-conversion ---- if-then 语句应该是应用程序中仅次于循环的最消耗时间的部分，简单的 if-then 语句可能在最终的汇编语言代码中产生众多的条件分支通过减少或者删除条件分支, 以及使用条件传送 设置标志和使用运算技巧来替换他们, 编译器可以减少 if-then 语句中花费的时间量。
-fif-conversion2 ---- 这种技术结合更加高级的数学特性， 减少实现 if-then 语句所需的条件分支。
-finline-functions-called-once
-fipa-profile
-fipa-pure-const
-fipa-reference
-fipa-reference-addressable
-floop-optimize ---- 通过优化如何生成汇编语言中的循环， 编译器可以在很大程序上提高应用程序的性能。 通常, 程序由很多大型且复杂的循环构成。 通过删除在循环内没有改变值的变量赋值操作, 可以减少循环内执行指令的数量, 在很大程度上提高性能。 此外优化那些确定何时离开循环的条件分支， 以便减少分支的影响
-fmerge-constants ---- 该模式下在不影响调试的状况下还会打开 -fomit-frame-pointer 优化项。同时该模式不会为Ada编译器打开 -ftree-sra 优化项，如需要则请使用命令参数输入 -ftree-sra 进行优化。
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
-funit-at-a-time  ----  在代码生成前，先分析整个的汇编语言代码。这将使一些额外的优化得以执行，但是在编译器间需要消耗大量的内存。（有资料介绍说：这使编译器可以重新安排不消耗大量时间的代码以便优化指令缓存。）
```

## O2优化项
O1基础上叠加如下项。
```Cpp
-falign-functions  ---- 对齐函数地址，加快CPU访问速度，增加代码大小
-falign-jumps  ----  对齐分支代码到2的n次方边界。在这种情况下，无需执行傀儡指令（dummy operations）
-falign-labels  ----  对齐分支到2的n次幂边界。这种选项容易使代码速度变慢，原因是需要插入一些dummy operations当分支抵达usual flow of the code.
-falign-loops  ---- 对齐循环到2的n次幂边界。期望可以对循环执行多次，用以补偿运行dummy operations所花费的时间。
-fcaller-saves  ----  通过存储和恢复call调用周围寄存器的方式，使被call调用的value可以被分配给寄存器，这种只会在看上去能产生更好的代码的时候才被使用。（如果调用多个函数, 这样能够节省时间, 因为只进行一次寄存器的保存和恢复操作, 而不是在每个函数调用中都进行。）
-fcode-hoisting
-fcrossjumping ---- 生成一致代码，减少代码大小，这是对跨越跳转的转换代码处理， 以便组合分散在程序各处的相同代码。 这样可以减少代码的长度， 但是也许不会对程序性能有直接影响。  
-fcse-skip-blocks ---- 与-fcse-follow-jumps类似，不同的是，根据特定条件，跟随着cse跳转的会是整个的blocks
-fcse-follow-jumps   ---- 在公用子表达式消元时，当目标跳转不会被其他路径可达，则扫描整个的跳转表达式。例如，当公用子表达式消元时遇到if...else...语句时，当条为false时，那么公用子表达式消元会跟随着跳转。   
-fdelete-null-pointer-checks  ---- 去除无用空指针检查(解引用后的检查)
-fdevirtualize  -fdevirtualize-speculatively ---- 尝试优化虚函数调用为实际函数调用，猜测实际调用的对象
-fexpensive-optimizations  ---- 一些很耗费CPU的次要优化
-fforce-mem  ---- 在做算术操作前，强制将内存数据copy到寄存器中以后再执行。这会使所有的内存引用潜在的共同表达式，进而产出更高效的代码，当没有共同的子表达式时，指令合并将排出个别的寄存器载入。这种优化对于只涉及单一指令的变量, 这样也许不会有很大的优化效果. 但是对于再很多指令(必须数学操作)中都涉及到的变量来说, 这会时很显著的优化, 因为和访问内存中的值相比 ,处理器访问寄存器中的值要快的多。 
-fgcse  -fgcse-lm  ---- 尝试进行公同子串表达式替换 common  subexpress elimination , 全局公用子表达式消除将试图移动那些仅仅被自身存储kill的装载操作的位置。这将允许将循环内的load/store操作序列中的load转移到循环的外面（只需要装载一次），而在循环内改变成copy/store序列。在选中-fgcse后，默认打开。
-fgcse-las ---- 全局公用子表达式消除pass将消除在store后面的不必要的load操作，这些load与store通常是同一块存储单元（全部或局部）
-fgcse-sm ---- 当一个存储操作pass在一个全局公用子表达式消除的后面，这个pass将试图将store操作转移到循环外面去。如果与-fgcse-lm配合使用，那么load/store操作将会转变为在循环前load，在循环后store，从而提高运行效率，减少不必要的操作。
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
-fpeephole2 ---- 允许计算机进行特定的观察孔优化(这个不晓得是什么意思)，-fpeephole与-fpeephole2的差别在于不同的编译器采用不同的方式，由的采用-fpeephole，有的采用-fpeephole2，也有两种都采用的。
-fregmove ---- 编译器试图重新分配move指令或者其他类似操作数等简单指令的寄存器数目，以便最大化的捆绑寄存器的数目。这种优化尤其对双操作数指令的机器帮助较大。
-freorder-blocks-algorithm=stc ---- 代码块重新排序，software trace cache算法，把常调用的部分流程聚合在一起
-freorder-blocks-and-partition  -freorder-functions ---- 根据局部性原理，将常调用的代码块放在同一个.o文件中
-frerun-cse-after-loop ---- loop优化后重新执行 csc
-frerun-loop-opt ---- 两次运行循环优化 l -fgcse：执行全局公用子表达式消除pass。这个pass还执行全局常量和copy propagation。这些优化操作试图分析生成的汇编语言代码并且结合通用片段， 消除冗余的代码段。如果代码使用计算性的goto, gcc指令推荐使用-fno-gcse选项。 
-fschedule-insns ---- 编译器尝试重新排列指令，用以消除由于等待未准备好的数据而产生的延迟。这种优化将对慢浮点运算的机器以及需要load memory的指令的执行有所帮助，因为此时允许其他指令执行，直到load memory的指令完成，或浮点运算的指令再次需要cpu。
-fschedule-insns2 ---- 与-fschedule-insns相似。但是当寄存器分配完成后，会请求一个附加的指令计划pass。这种优化对寄存器较小，并且load memory操作时间大于一个时钟周期的机器有非常好的效果。
-fsched-interblock  -fsched-spec  ----  这种技术使编译器能够跨越指令块调度指令。 这可以非常灵活地移动指令以便等待期间完成的工作最大化。

-fsched-spec-load  ----  允许一些load指令进行一些投机性的动作。（具体不详）相同功能的还有-fsched-spec-load-dangerous，允许更多的load指令进行投机性操作。这两个选项在选中-fschedule-insns时默认打开。
-fstore-merging  ---- 及时数对齐，不足1个Word的填充为1个Word
-fstrength-reduce ---- 这种优化技术对循环执行优化并且删除迭代变量。 迭代变量是捆绑到循环计数器的变量, 比如使用变量, 然后使用循环计数器变量执行数学操作的for-next循环。
-fstrict-aliasing  ----  这种技术强制实行高级语言的严格变量规则。 对于c和c++程序来说, 它确保不在数据类型之间共享变量. 例如, 整数变量不和单精度浮点变量使用相同的内存位置。
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
-finline-functions  ----  内联简单的函数到被调用函数中。由编译器启发式的决定哪些函数足够简单可以做这种内联优化。默认情况下，编译器限制内联的尺寸，3.4.6中限制为600（具体含义不详，指令条数或代码size？）可以通过-finline-limit=n改变这个长度。这种优化技术不为函数创建单独的汇编语言代码， 而是把函数代码包含在调度程序的代码中。 对于多次被调用的函数来说, 为每次函数调用复制函数代码。 虽然这样对于减少代码长度不利, 但是通过最充分的利用指令缓存代码, 而不是在每次函数调用时进行分支操作, 可以提高性能。 
-fipa-cp-clone
-floop-interchange
-floop-unroll-and-jam
-fpeel-loops
-fpredictive-commoning
-frename-registers  ----  在寄存器分配后，通过使用registers left over来避免预定代码中的虚假依赖。这会使调试变得非常困难，因为变量不再存放于原本的寄存器中了。
-fsplit-paths
-ftree-loop-distribute-patterns
-ftree-loop-distribution
-ftree-loop-vectorize
-ftree-partial-pre
-ftree-slp-vectorize
-funswitch-loops  ----  将无变化的条件分支移出循环，取而代之的将结果副本放入循环中。
-fvect-cost-model
-fversion-loops-for-strides
-fweb  ----  构建用于保存变量的伪寄存器网络。 伪寄存器包含数据, 就像他们是寄存器一样, 但是可以使用各种其他优化技术进行优化, 比如cse和loop优化技术。这种优化会使得调试变得更加的不可能，因为变量不再存放于原本的寄存器中。
```

## Os

要是对程序的尺寸进行优化。打开了大部分 O2 优化中不会增加程序大小的优化选项，并对程序代码的大小做更深层的优化。（通常我们不需要这种优化）Os 会关闭如下选项： `-falign-functions -falign-jumps -falign-loops  -falign-labels   -freorder-blocks   -fprefetch-loop-arrays `

## 优化介绍小结
O0 选项不进行任何优化，在这种情况下，编译器尽量的缩短编译消耗（时间，空间），此时，debug 会产出和程序预期的结果。当程序运行被断点打断，此时程序内的各种声明是独立的，我们可以任意的给变量赋值，或者在函数体内把程序计数器指到其他语句,以及从源程序中 精确地获取你期待的结果. 

O1 优化会消耗少多的编译时间，它主要对代码的分支，常量以及表达式等进行优化。 

O2 会尝试更多的寄存器级的优化以及指令级的优化，它会在编译期间占用更多的内存和编译时间。 

O3 在 O2 的基础上进行更多的优化，例如使用伪寄存器网络，普通函数的内联，以及针对循环的更多优化。 

Os 主要是对代码大小的优化，我们基本不用做更多的关心。 通常各种优化都会打乱程序的结构，让调试工作变得无从着手。并且会打乱执行顺序，依赖内存操作顺序的程序需要做相关处理才能确保程序的正确性。  


## 优化代码有可能带来的问题 

1．调试问题：正如上面所提到的，任何级别的优化都将带来代码结构的改变。例如：对分支的合并和消除，对公用子表达式的消除，对循环内 load/store 操作的替换和更改等，都将会使目标代码的执行顺序变得面目全非，导致调试信息严重不足。 

2．内存操作顺序改变所带来的问题：在 O2 优化后，编译器会对影响内存操作的执行顺序。例如：-fschedule-insns 允许数据处理时先完成其他的指令；-fforce-mem 有可能导致内存与寄存器之间的数据产生类似脏数据的不一致等。对于某些依赖内存操作顺序而进行的逻辑，需要做严格的处理后才能进行优化。例如，采用 volatile 关键字限制变量的操作方式，或者利用 barrier 迫使 cpu 严格按照指令序执行的。



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

> debug模式编译器参数为 `-O0 -g –Wall` \
> release模式编译器参数为 `-O2 -g –Wall`
