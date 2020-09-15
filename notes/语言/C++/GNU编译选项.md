# GNU 编译选项

## 目录
* [总览(SYNOPSIS)](#总览(SYNOPSIS))
* [预处理器选项(PreprocessorOption)](#预处理器选项(PreprocessorOption))

## 总览(SYNOPSIS)

```shell
gcc[option|filename ]...
g++[option|filename ]...
```

## 警告(WARNING)

本手册页内容摘自 GNU C 编译器的[完整文档](https://gcc.gnu.org/onlinedocs/gcc/),仅限于解释选项的含义.
除非有人自愿维护,否则本手册页不再更新.如果发现手册页和软件之间有所矛盾,请查对 Info 文件, Info 文件是权威文档.

如果我们发觉本手册页的内容由于过时而导致明显的混乱和抱怨时,我们就停止发布它.不可能有其他选择,象更新 Info 文件同时更新 man 手册,因为其他维护 GNU CC 的工作没有留给我们时间做这个. GNU 工程认为 man 手册是过时产物,应该把时间用到别的地方.

如果需要完整和最新的文档,请查阅 Info 文件 gcc 或 Using and Porting GNU CC (for version 2.0) (使用和移植 GNU CC 2.0) 手册.二者均来自 Texinfo 原文件 gcc.texinfo.

## 描述(DESCRIPTION)

C 和 C++ 编译器是集成的.他们都要用四个步骤中的一个或多个处理输入文件: 预处理(preprocessing), 编译(compilation), 汇编(assembly)和连接(linking). 源文件后缀名标识源文件的 语言,但是对编译器来说, 后缀名控制着缺省设定:

* gcc

认为预处理后的文件(.i)是 C 文件,并且设定 C 形式的连接.

* g++

认为预处理后的文件(.i)是 C++ 文件,并且设定 C++ 形式的连接.

* 源文件后缀名指出语言种类以及后期的操作:

```shell
.c      C源程序;预处理,编译,汇编
.C      C++源程序;预处理,编译,汇编
.cc     C++源程序;预处理,编译,汇编
.cxx    C++源程序;预处理,编译,汇编
.m      Objective-C源程序;预处理,编译,汇编
.i     预处理后的C文件;编译,汇编
.ii    预处理后的C++文件;编译,汇编
.s     汇编语言源程序;汇编
.S     汇编语言源程序;预处理,汇编
.h     预处理器文件;通常不出现在命令行上
```

* 其他后缀名的文件被传递给连接器(linker).通常包括:

```shell
.o     目标文件(Object file)
.a     归档库文件(Archive file)
```

除非使用了 -c, -S 或 -E 选项(或者编译错误阻止了完整的过程),否则连接总是 最后的步骤.在连接阶段中,所有对应于源程序的 .o 文件, -l 库文件,无法识别的文件名(包括指定的 .o 目标文件和 .a 库文件)按命令行中的顺序传递给连接器.

选项(OPTIONS)

选项必须分立给出: -dr 完全不同于 -d -r .

大多数 -f 和 -W 选项有两个相反的格式: -fname 和 -fno-name (或 -Wname 和 -Wno-name ).这里 只列举不是默认选项的格式.

下面是所有选项的摘要,按类型分组,解释放在后面的章节中.

## 总体选项(Overall Option)

```shell
-c -S -E -o file -pipe -v -x language
```

## 语言选项(Language Option)

```shell
-ansi -fall-virtual -fcond-mismatch -fdollars-in-identifiers -fenum-int-equiv -fexternal-templates -fno-asm -fno-builtin -fhosted -fno-hosted -ffreestanding -fno-freestanding -fno-strict-prototype -fsigned-bitfields -fsigned-char -fthis-is-variable -funsigned-bitfields -funsigned-char -fwritable-strings -traditional -traditional-cpp -trigraphs
```

## 警告选项(Warning Option)

```shell
-fsyntax-only -pedantic -pedantic-errors -w -W -Wall -Waggregate-return -Wcast-align -Wcast-qual -Wchar-subscript -Wcomment -Wconversion -Wenum-clash -Werror -Wformat -Wid-clash-len -Wimplicit -Wimplicit-int -Wimplicit-function-declaration -Winline -Wlong-long -Wmain -Wmissing-prototypes -Wmissing-declarations -Wnested-externs -Wno-import -Wparentheses -Wpointer-arith -Wredundant-decls -Wreturn-type -Wshadow -Wstrict-prototypes -Wswitch -Wtemplate-debugging -Wtraditional -Wtrigraphs -Wuninitialized -Wunused -Wwrite-strings
```

## 调试选项(Debugging Option)

```shell
-a -dletters -fpretend-float -g -glevel -gcoff -gxcoff -gxcoff+ -gdwarf -gdwarf+ -gstabs -gstabs+ -ggdb -p -pg -save-temps -print-file-name=library -print-libgcc-file-name -print-prog-name=program
```

## 优化选项(Optimization Option)

```shell
-fcaller-saves -fcse-follow-jumps -fcse-skip-blocks -fdelayed-branch -felide-constructors -fexpensive-optimizations -ffast-math -ffloat-store -fforce-addr -fforce-mem -finline-functions -fkeep-inline-functions -fmemoize-lookups -fno-default-inline -fno-defer-pop -fno-function-cse -fno-inline -fno-peephole -fomit-frame-pointer -frerun-cse-after-loop -fschedule-insns -fschedule-insns2 -fstrength-reduce -fthread-jumps -funroll-all-loops -funroll-loops -O -O2 -O3
```

## 预处理器选项(Preprocessor Option)

```shell
-Aassertion -C -dD -dM -dN -Dmacro[=defn] -E -H -idirafter dir -include file -imacros file -iprefix file -iwithprefix dir -M -MD -MM -MMD -nostdinc -P -Umacro -undef
```

## 汇编器选项(Assembler Option)

```shell
-Wa,option
```

## 连接器选项(Linker Option)

```shell
-llibrary -nostartfiles -nostdlib -static -shared -symbolic -Xlinker option -Wl,option -u symbol
```

## 目录选项(Directory Option)

```shell
-Bprefix -Idir -I- -Ldir
```

## 目标机选项(Target Option)

```shell
-b machine -V version
```

## 配置相关选项(Configuration Dependent Option)

```shell
M680x0 选项
-m68000 -m68020 -m68020-40 -m68030 -m68040 -m68881 -mbitfield -mc68000 -mc68020 -mfpa -mnobitfield -mrtd -mshort -msoft-float
VAX选项
-mg -mgnu -munix

SPARC选项
-mepilogue -mfpu -mhard-float -mno-fpu -mno-epilogue -msoft-float -msparclite -mv8 -msupersparc -mcypress

Convex选项
-margcount -mc1 -mc2 -mnoargcount

AMD29K选项
-m29000 -m29050 -mbw -mdw -mkernel-registers -mlarge -mnbw -mnodw -msmall -mstack-check -muser-registers

M88K选项
-m88000 -m88100 -m88110 -mbig-pic -mcheck-zero-division -mhandle-large-shift -midentify-revision -mno-check-zero-division -mno-ocs-debug-info -mno-ocs-frame-position -mno-optimize-arg-area -mno-serialize-volatile -mno-underscores -mocs-debug-info -mocs-frame-position -moptimize-arg-area -mserialize-volatile -mshort-data-num -msvr3 -msvr4 -mtrap-large-shift -muse-div-instruction -mversion-03.00 -mwarn-passed-structs

RS6000选项
-mfp-in-toc -mno-fop-in-toc

RT选项
-mcall-lib-mul -mfp-arg-in-fpregs -mfp-arg-in-gregs -mfull-fp-blocks -mhc-struct-return -min-line-mul -mminimum-fp-blocks -mnohc-struct-return

MIPS选项
-mcpu=cpu type -mips2 -mips3 -mint64 -mlong64 -mmips-as -mgas -mrnames -mno-rnames -mgpopt -mno-gpopt -mstats -mno-stats -mmemcpy -mno-memcpy -mno-mips-tfile -mmips-tfile -msoft-float -mhard-float -mabicalls -mno-abicalls -mhalf-pic -mno-half-pic -G num -nocpp

i386选项
-m486 -mno-486 -msoft-float -mno-fp-ret-in-387

HPPA选项
-mpa-risc-1-0 -mpa-risc-1-1 -mkernel -mshared-libs -mno-shared-libs -mlong-calls -mdisable-fpregs -mdisable-indexing -mtrailing-colon

i960选项
-mcpu-type -mnumerics -msoft-float -mleaf-procedures -mno-leaf-procedures -mtail-call -mno-tail-call -mcomplex-addr -mno-complex-addr -mcode-align -mno-code-align -mic-compat -mic2.0-compat -mic3.0-compat -masm-compat -mintel-asm -mstrict-align -mno-strict-align -mold-align -mno-old-align

DEC Alpha选项
-mfp-regs -mno-fp-regs -mno-soft-float -msoft-float

System V选项
-G -Qy -Qn -YP,paths -Ym,dir
```

## 代码生成选项(Code Generation Option)

```shell
-fcall-saved-reg -fcall-used-reg -ffixed-reg -finhibit-size-directive -fnonnull-objects -fno-common -fno-ident -fno-gnu-linker -fpcc-struct-return -fpic -fPIC -freg-struct-return -fshared-data -fshort-enums -fshort-double -fvolatile -fvolatile-global -fverbose-asm
```

## 总体选项(Overall Option)


**-x language**

明确指出后面输入文件的语言为 language (而不是从文件名后缀得到的默认选择).这个选项应用于后面 所有的输入文件,直到遇着下一个 -x 选项. language 的可选值有 c, objective-c, c-header, c++, cpp-output, assembler,和 assembler-with-cpp.
-x none
关闭任何对语种的明确说明,因此依据文件名后缀处理后面的文件(就象是从未使用过 -x 选项).
如果只操作四个阶段(预处理,编译,汇编,连接)中的一部分,可以使用 -x 选项(或文件名后缀)告诉 gcc 从哪里开始,用 -c, -S,或 -E 选项告诉 gcc 到哪里结束.注意,某些选项组合(例如, -x cpp-output -E)使gcc不作任何事情.

**-c**

编译或汇编源文件,但是不作连接.编译器输出对应于源文件的目标文件.
缺省情况下, GCC 通过用 .o 替换源文件名后缀 .c, .i, .s,等等,产生目标文件名.可以使用 -o 选项选择其他名字.

GCC 忽略 -c 选项后面任何无法识别的输入文件(他们不需要编译或汇编).

**-S**

编译后即停止,不进行汇编.对于每个输入的非汇编语言文件,输出文件是汇编语言文件.
缺省情况下, GCC 通过用 .o 替换源文件名后缀 .c, .i,等等,产生 目标文件名.可以使用 -o 选项选择其他名字.

GCC 忽略任何不需要编译的输入文件.

**-E**

预处理后即停止,不进行编译.预处理后的代码送往标准输出.
GCC 忽略任何不需要预处理的输入文件.

**-o file**

指定输出文件为 file.该选项不在乎 GCC 产生什么输出,无论是可执行文件,目标文件,汇编文件还是 预处理后的 C 代码.
由于只能指定一个输出文件,因此编译多个输入文件时,使用 -o 选项没有意义,除非输出一个可执行文件.

如果没有使用 -o 选项,默认的输出结果是:可执行文件为 a.out,  source.suffix 的目标文件是 source.o,汇编文件是 source.s, 而预处理后的 C 源代码送往标准输出.

**-v**

(在标准错误)显示执行编译阶段的命令.同时显示编译器驱动程序,预处理器,编译器的版本号.
-pipe
在编译过程的不同阶段间使用管道而非临时文件进行通信.这个选项在某些系统上无法工作,因为那些系统的 汇编器不能从管道读取数据. GNU 的汇编器没有这个问题.


## 语言选项(LANGUAGE OPTIONS)


下列选项控制编译器能够接受的 C 方言:

**-ansi**

支持符合 ANSI 标准的 C 程序.
这样就会关闭 GNU C 中某些不兼容 ANSI C 的特性,例如 asm, inline 和  typeof 关键字,以及诸如 unix 和 vax 这些表明当前系统类型的预定义宏.同时开启不受欢迎和极少使用的 ANSI trigraph 特性,以及禁止 $ 成为标识符的一部分.

尽管使用了 -ansi 选项,下面这些可选的关键字, `__asm__`, `__extension__`, `__inline__` 和 `__typeof__` 仍然有效.你当然不会把 他们用在 ANSI C 程序中,但可以把他们放在头文件里,因为编译包含这些头文件的程序时,可能会指定 -ansi 选项.另外一些预定义宏,如 `__unix__` 和 `__vax__`,无论有没有使用 -ansi 选项,始终有效.

使用 -ansi 选项不会自动拒绝编译非ANSI程序,除非增加 -pedantic 选项作为 -ansi 选项的补充.

使用 -ansi 选项的时候,预处理器会预定义一个 `__STRICT_ANSI__` 宏.有些头文件 关注此宏,以避免声明某些函数,或者避免定义某些宏,这些函数和宏不被 ANSI 标准调用;这样就不会干扰在其他地方 使用这些名字的程序了.

**-fno-asm**

不把 asm, inline 或 typeof 当作关键字,因此这些词可以用做标识符.用 `__asm__`, `__inline__` 和 `__typeof__` 能够替代他们. -ansi 隐含声明了 -fno-asm.

**-fno-builtin**

不接受不是两个下划线开头的内建函数(built-in function).目前受影响的函数有 `_exit, abort, abs, alloca, cos, exit, fabs, labs, memcmp, memcpy, sin, sqrt, strcmp, strcpy`, 和 `strlen`.
-ansi 选项能够阻止 `alloca` 和 `_exit` 成为内建函数.

**-fhosted**

按宿主环境编译;他隐含声明了 -fbuiltin 选项,而且警告不正确的 main 函数声明.

**-ffreestanding**

按独立环境编译;他隐含声明了 -fno-builtin 选项,而且对 main 函数没有特别要求.
(译注:宿主环境(hosted environment)下所有的标准库可用, main 函数返回一个 int 值,典型例子是除了 内核以外几乎所有的程序.对应的独立环境(freestanding environment)不存在标准库,程序入口也不一定是 main,最明显的例子就是操作系统内核.详情参考 gcc 网站最近的资料)

**-fno-strict-prototype**

对于没有参数的函数声明,例如 int foo ();,按 C 风格处理---即不说明参数个数或类型. (仅针对 C++ ).正常情况下,这样的函数 foo 在 C++ 中意味着参数为空.

**-trigraphs**

支持 ANSI C trigraphs. -ansi 选项隐含声明了 -trigraphs.

**-traditional**

试图支持传统 C 编译器的某些方面.详见 GNU C 手册,我们已经把细节清单从这里删除,这样当内容过时后,人们也不会埋怨我们.
除了一件事:对于 C++ 程序(不是 C ), -traditional 选项带来一个附加效应,允许对 this 赋值.他和 -fthis-is-variable 选项的效果一样.

**-traditional-cpp**

试图支持传统 C 预处理器的某些方面.特别是上面提到有关预处理器的内容,但是不包括 -traditional 选项的其他效应.

**-fdollars-in-identifiers**

允许在标识符(identifier)中使用 $ 字符(仅针对 C++).你可以指定 -fno-dollars-in-identifiers 选项显明禁止使用 $ 符. (GNU C++ 在某些 目标系统缺省允许 $ 符,但不是所有系统.)

**-fenum-int-equiv**

允许 int 类型到枚举类型(enumeration)的隐式转换(仅限于 C++).正常情况下 GNU C++ 允许从 enum 到 int 的转换,反之则不行.

**-fexternal-templates**

为模板声明(template declaration)产生较小的代码(仅限于 C++),方法是对于每个模板函数 (template function),只在定义他们的地方生成一个副本.想要成功使用这个选项,你必须在所有使用模板的 文件中,标记 #pragma implementation (定义)或 #pragma interface (声明).
当程序用 -fexternal-templates 编译时,模板实例(template instantiation) 全部是外部类型.你必须让需要的实例在实现文件中出现.可以通过 typedef 实现这一点,他引用所需的每个 实例.相对应的,如果编译时使用缺省选项 -fno-external-templates ,所有模板实例明确的设为内置.

**-fall-virtual**

所有可能的成员函数默认为虚函数.所有的成员函数(除了构造子函数和 new 或 delete 成员操作符)视为所在类的虚函数.
这不表明每次调用成员函数都将通过内部虚函数表.有些情况下,编译器能够判断出可以直接调用某个虚函数;这时就 直接调用.

**-fcond-mismatch**

允许条件表达式的第二和第三个参数的类型不匹配.这种表达式的值是 void.

**-fthis-is-variable**

允许对 this 赋值(仅对 C++).合并用户自定义的自由存储管理机制到 C++ 后,使可赋值的 this 显得不合时宜.因此,默认情况下,类成员函数内部对 this 赋值是无效操作.然而为了 向后兼容,你可以通过 -fthis-is-variable 选项使这种操作有效.

**-funsigned-char**

把 char 定义为无符号类型,如同 unsigned char.
各种机器都有自己缺省的 char 类型.既可能是 unsigned char 也可能是 signed char .

理想情况下,当依赖于数据的符号性时,一个可移植程序总是应该使用 signed char 或 unsigned char.但是许多程序已经写成只用简单的 char,并且期待这是有符号数(或者无符号数,具体情况取决于 编写程序的目标机器).这个选项,和它的反义选项,使那样的程序工作在对应的默认值上.

char 的类型始终应该明确定义为 signed char 或 unsigned char, 即使 它表现的和其中之一完全一样.

**-fsigned-char**

把 char 定义为有符号类型,如同 signed char.
这个选项等同于 -fno-unsigned-char,他是 the negative form of -funsigned-char 的相反选项.同样,  -fno-signed-char 等价于 -funsigned-char.

**-fsigned-bitfields**

**-funsigned-bitfields**

**-fno-signed-bitfields**

**-fno-unsigned-bitfields**

如果没有明确声明 signed 或 unsigned 修饰符,这些选项用来定义有符号位域 (bitfield)或无符号位域.缺省情况下,位域是有符号的,因为他们继承的基本整数类型,如 int,是 有符号数.
然而,如果指定了 -traditional 选项,位域永远是无符号数.

**-fwritable-strings**

把字符串常量存储到可写数据段,而且不做特别对待.这是为了兼容一些老程序,他们假设字符串常量是可写的. -traditional 选项也有相同效果.
篡改字符串常量是一个非常糟糕的想法; 常量就应该是常量.


### 预处理器选项(PreprocessorOption)

下列选项针对C预处理器,预处理器用在正式编译以前,对C 源文件进行某种处理.
如果指定了 `-E'选项, GCC只进行预处理工作.下面的某些选项必须和`-E'选项一起才 有意义,因为他们的输出结果不能用于编译.

-include file
在处理常规输入文件之前,首先处理文件file,其结果是,文件file的内容先得到编译. 命令行上任何`-D'和`-U'选项永远在`-include file'之前处理, 无论他们在命令行上的顺序如何.然而`-include'和`-imacros'选项按书写顺序处理.
-imacros file
在处理常规输入文件之前,首先处理文件file,但是忽略输出结果.由于丢弃了文件file的 输出内容, `-imacros file'选项的唯一效果就是使文件file中的宏定义生效, 可以用于其他输入文件.在处理`-imacrosfile'选项之前,预处理器首先处理`-D' 和`-U'选项,并不在乎他们在命令行上的顺序.然而`-include'和 `-imacros'选项按书写顺序处理.
-idirafter dir
把目录dir添加到第二包含路径中.如果某个头文件在主包含路径(用`-I'添加的路径)中没有 找到,预处理器就搜索第二包含路径.
-iprefix prefix
指定prefix作为后续`-iwithprefix'选项的前缀.
-iwithprefix dir
把目录添加到第二包含路径中.目录名由prefix和dir合并而成,这里 prefix被先前的`-iprefix'选项指定.
-nostdinc
不要在标准系统目录中寻找头文件.只搜索`-I'选项指定的目录(以及当前目录,如果合适).
结合使用`-nostdinc'和`-I-'选项,你可以把包含文件搜索限制在显式指定的目录.

-nostdinc++
不要在C++专用标准目录中寻找头文件,但是仍然搜索其他标准目录. (当建立`libg++'时使用 这个选项.)
-undef
不要预定义任何非标准宏. (包括系统结构标志).
-E
仅运行C预处理器.预处理所有指定的C源文件,结果送往标准输出或指定的输出文件.
-C
告诉预处理器不要丢弃注释.配合`-E'选项使用.
-P
告诉预处理器不要产生`#line'命令.配合`-E'选项使用.
-M  [ -MG ]
告诉预处理器输出一个适合make的规则,用于描述各目标文件的依赖关系.对于每个源文件,预处理器输出 一个make规则,该规则的目标项(target)是源文件对应的目标文件名,依赖项(dependency)是源文件中 `#include引用的所有文件.生成的规则可以是单行,但如果太长,就用`\'-换行符续成多行.规则 显示在标准输出,不产生预处理过的C程序.
`-M'隐含了`-E'选项.

`-MG'要求把缺失的头文件按存在对待,并且假定他们和源程序文件在同一目录下.必须和 `-M'选项一起用.

-MM  [ -MG ]
和`-M'选项类似,但是输出结果仅涉及用户头文件,象这样`#include file"'.忽略系统头文件如`#include <file>'.
-MD
和`-M'选项类似,但是把依赖信息输出在文件中,文件名通过把输出文件名末尾的`.o'替换为 `.d'产生.同时继续指定的编译工作---`-MD'不象`-M'那样阻止正常的编译任务.
Mach的实用工具`md'能够合并`.d'文件,产生适用于`make'命令的单一的 依赖文件.

-MMD
和`-MD'选项类似,但是输出结果仅涉及用户头文件,忽略系统头文件.
-H
除了其他普通的操作, GCC显示引用过的头文件名.
-Aquestion(answer)
如果预处理器做条件测试,如`#if #question(answer)',该选项可以断言(Assert) question的答案是answer. -A-'关闭一般用于描述目标机的标准断言.
-Dmacro
定义宏macro,宏的内容定义为字符串`1'.
-Dmacro=defn
定义宏macro的内容为defn.命令行上所有的`-D'选项在 `-U'选项之前处理.
-Umacro
取消宏macro. `-U'选项在所有的`-D'选项之后处理,但是优先于任何 `-include'或`-imacros'选项.
-dM
告诉预处理器输出有效的宏定义列表(预处理结束时仍然有效的宏定义).该选项需结合`-E'选项使用.
-dD
告诉预处理器把所有的宏定义传递到输出端,按照出现的顺序显示.
-dN
和`-dD'选项类似,但是忽略宏的参量或内容.只在输出端显示`#define name.
