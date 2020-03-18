# AddressSanitizer使用归纳

Sanitizer 是 google 开源的一个项目，其中包含四个组件：

* [AdressSanitizer(asan)](https://github.com/google/sanitizers/wiki/AddressSanitizer)

* [ThreadSanitizer(tsan)](https://github.com/google/sanitizers/wiki/ThreadSanitizerCppManual)

* [LeakSanitizer](https://github.com/google/sanitizers/wiki/AddressSanitizerLeakSanitizer)

* [MemorySanitizer](https://github.com/google/sanitizers/wiki/MemorySanitizer)

## 和常用内存工具比较

||[AddressSanitizer](https://github.com/google/sanitizers/wiki/AddressSanitizer)|[Valgrind/Memcheck](http://valgrind.org/)|[Dr. Memory](https://dynamorio.org/)|[Mudflap](https://gcc.gnu.org/wiki/Mudflap_Pointer_Debugging)|Guard Page|[gperftools](https://github.com/gperftools/gperftools)|
|----|----|----|----|----|----|----|
| technology | CTI | DBI | DBI | CTI | Library | Library |
| ARCH | x86, ARM, PPC | x86, ARM, PPC, MIPS, S390X, TILEGX | x86 |	all(?) |	all(?) | all(?) |
| OS | Linux, OS X, Windows, FreeBSD, Android, iOS Simulator |	Linux, OS X, Solaris, Android |	Windows, Linux | Linux, Mac(?) | All (1) | Linux, Windows |
| Slowdown | 2x | 20x | 10x | 2x-40x | ? | ? |
| Detects: |
| [Heap OOB](https://github.com/google/sanitizers/wiki/AddressSanitizerExampleHeapOutOfBounds) | yes | yes | yes | yes | some | some |
| [Stack OOB](https://github.com/google/sanitizers/wiki/AddressSanitizerExampleStackOutOfBounds) | yes | no | no | some | no | no |
| [Global OOB](https://github.com/google/sanitizers/wiki/AddressSanitizerExampleGlobalOutOfBounds) | yes | no | no | ? | no | no |
| [UAF](https://github.com/google/sanitizers/wiki/AddressSanitizerExampleUseAfterFree) | yes | yes | yes | yes | yes | yes |
| [UAR](https://github.com/google/sanitizers/wiki/AddressSanitizerExampleUseAfterReturn) | yes (see [AddressSanitizerUseAfterReturn](https://github.com/google/sanitizers/wiki/AddressSanitizerUseAfterReturn)) | no | no | no | no | no |
| UMR | no (see MemorySanitizer) | yes | yes | ? | no | no |
| Leaks | yes (see LeakSanitizer) | yes | yes | ? | no | yes |

* DBI: dynamic binary instrumentation
* CTI: compile-time instrumentation
* UMR: uninitialized memory reads
* UAF: use-after-free (aka dangling pointer)
* UAR: use-after-return
* OOB: out-of-bounds
* x86: includes 32- and 64-bit.
* mudflap was removed in GCC 4.9, as it has been superseded by AddressSanitizer.
* Guard Page: a family of memory error detectors (Electric fence or DUMA on Linux, Page Heap on Windows, libgmalloc on OS X)
* gperftools: various performance tools/error detectors bundled with TCMalloc. Heap checker (leak detector) is only available on Linux. Debug allocator provides both guard pages and canary values for more precise detection of OOB writes, so it's better than guard page-only detectors.


## 支持的检测项
* [Use after free](https://github.com/google/sanitizers/wiki/AddressSanitizerExampleUseAfterFree)

* [Heap buffer overflow](https://github.com/google/sanitizers/wiki/AddressSanitizerExampleHeapOutOfBounds)

* [Stack buffer overflow](https://github.com/google/sanitizers/wiki/AddressSanitizerExampleStackOutOfBounds)

* [Global buffer overflow](https://github.com/google/sanitizers/wiki/AddressSanitizerExampleGlobalOutOfBounds)

* [Use after return](https://github.com/google/sanitizers/wiki/AddressSanitizerExampleUseAfterReturn)

* [Use after scope](https://github.com/google/sanitizers/wiki/AddressSanitizerExampleUseAfterScope)

* [Initialization order bugs](https://github.com/google/sanitizers/wiki/AddressSanitizerInitializationOrderFiasco)

* [Memory leaks](https://github.com/google/sanitizers/wiki/AddressSanitizerLeakSanitizer)


**该工具适用于 x86 ，ARM ，MIPS（所有体系结构的 32 位和 64 位版本）， PowerPC64 。支持的操作系统是 Linux ，Darwin（ OS X 和 iOS Simulator ）， FreeBSD ， Android ：**

| OS | x86 | x86_64 | ARM | ARM64 | MIPS | MIPS64 | PowerPC | PowerPC64 |
|---|---|---|---|---|---|---|---|---|
| Linux |	yes | | | yes | yes | yes |	yes |	yes |
| OS X | yes | yes |						
| iOS Simulator |	yes |	yes |
| FreeBSD |	yes |	yes	|					
| Android |	yes |	yes |	yes |	yes |

## 使用步骤
* 用 -fsanitize=address 选项编译和链接你的程序。
* 用 -fno-omit-frame-pointer 编译，以得到更容易理解 stack trace。
* 用 -fno-optimize-sibling-calls 尾递归消除
* 用 -fsanitize=undefined  未定义行为检测
* 可选择 -O1 或者更高的优化级别编译 ( CPU 会优化掉一部分错误，建议使用 O0)

>注：-Wl,-z,defs 可能会导致链接错误（不要与 AddressSanitizer 同用）

## 如果需要定位到源文件，需要指定以下环境: (否则只会定位到内存地址)

export ASAN_OPTIONS=symbolize=1

export ASAN_SYMBOLIZER_PATH=$(which llvm-symbolizer)

export MSAN_OPTIONS=symbolize=1

export MSAN_SYMBOLIZER_PATH=$(which llvm-symbolizer)

export LSAN_OPTIONS=symbolize=1

export LSAN_SYMBOLIZER_PATH=$(which llvm-symbolizer)

优点： 定位准确， 检查全面

缺点： 需要重新编译可执行文件

## 条件编译
在某些情况下，根据是否启用 AddressSanitizer，可能需要执行不同的代码。 `__has_feature` 可以用于此目的

```Cpp
#if defined(__has_feature)
#if __has_feature(address_sanitizer)
// code that builds only under AddressSanitizer
#  endif
#endif
```
___

## 禁用插桩
某些代码不应由 AddressSanitizer 进行检测。可以使用函数属性 `__attribute__ (no_sanitize ( "address"))` (已弃用同义词 no_sanitize_address 和 no_address_safety_analysis) 来禁用特定函数的检测。其他编译器可能不支持此属性, 因此建议将其与 `__has_feature (address_sanitizer)` 一起使用


## 抑制外部库的报告
运行时干预允许 AddressSanitizer 在未重新编译的代码中查找 bug。如果你在外部库中遇到 bug，我们建议立即将其报告给库维护者，以便得到解决。也可以使用如下方法启用抑制机制来解除这种阻塞关系，以便保持继续测试。此抑制机制只用于抑制外部代码中的问题，它不适用于 AddressSanitizer 重新编译的代码。若要消除外部库的错误，请将 ASAN_OPTIONS 环境变量设置为指向一个抑制文件。可以指定文件的完整路径，或者相对于可执行文件的路径

`ASAN_OPTIONS=suppressions=MyASan.supp`

使用以下格式来指定要抑制的函数或库的名字。你可以在错误报告中看到这些信息。请记住，抑制的范围越小，能捕捉到的 bug 越多
```Cpp
interceptor_via_fun:NameOfCFunctionToSuppress
interceptor_via_fun:-[ClassName objCMethodToSuppress:]
interceptor_via_lib:NameOfTheLibraryToSuppress
```

## 局限
使用更多的物理内存，确切的开销取决于分配的细致程度，分配的越细，开销越大
AddressSanitizer 使用更多的栈内存，大概是三倍的增长
在 64 位平台上 AddressSanitizer 映射但不保留 16TB+ 的虚拟地址空间，**这意味着像 ulimit 这样的工具可能不能像预期一样工作
不支持静态链接**


## 其他
Address sanitizer 的行为可以受一些环境变量控制, 具体参见 google/sanitizers. 在实际使用中, 可能会经常性使用某些设置

如:
* export LSAN_OPTIONS=leak_check_at_exit=false
* export ASAN_OPTIONS="disable_coredump=0:unmap_shadow_on_exit=1:abort_on_error=1:alloc_dealloc_mismatch=0"(To generate a core dump when ASAN detects some error.)
等.

***

在(OS X)上发现了一个问题，在使用 vector 时，会被认为 overflow ，还有以下代码也会在 OS 上报错，但是在 Linux 上却没有任何提示
```Cpp
#include <vector>
#include <iostream>

int main() {
    {
        std::vector<int> tmp_vec{1, 2, 3};
        tmp_vec.resize(1);
        if(tmp_vec[1] == 123) std::cout << "#1" << std::endl;
    }
}
```
使用 clang 6 在 OS X 上输出
```Cpp
==9387==ERROR: AddressSanitizer: container-overflow on address 0x6020000000f4 at pc 0x00010fb5e4aa bp 0x7ffee00a2b90 sp 0x7ffee00a2b88
READ of size 4 at 0x6020000000f4 thread T0
    #0 0x10fb5e4a9 in main (a.out:x86_64+0x1000024a9)
    #1 0x7fff7eaa7014 in start (libdyld.dylib:x86_64+0x1014)
[...]
```
解决方法：
要添加到 mrks 应答，libstdc++ 默认情况下不检测容器溢出(因为它可能导致 Asan 的误报警告，请查看 [wiki](https://github.com/google/sanitizers/wiki/AddressSanitizerClangVsGCC-(6.0-vs-8.1) 以获取更多详细信息)。您需要通过显式启用它 -D_GLIBCXX_SANITIZE_VECTOR (您需要足够的时间 libstdc++ )。
