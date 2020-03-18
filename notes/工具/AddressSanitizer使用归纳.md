# AddressSanitizer使用归纳










在(OS X)上发现了一个问题，在使用 vector 的 push_back 方法时，会被认为 overflow ，还有以下代码也会在 OS 上报错，但是在 Linux 上却没有任何提示
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
