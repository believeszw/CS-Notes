# 全局变量

```shell
$ vim global.c
#include <stdio.h>

int i = 1;

int main()
{
    ++i;

    printf("%d\n",i);
    return 0;
}
$ gcc -S -o global.s global.c
$ cat global.s
movl	i, %eax
addl	$1, %eax
movl	%eax, i
```

## 反汇编

### 反汇编执行文件

```Shell
$ gcc -o global global.c
$ objdump -s -d global > global.txt
# -s 参数可以将所有段的内容以十六进制的方式打印出来
# -d 参数可以将所有包含指令的段反汇编
$ cat global.txt # 文件较长，只取部分
080483c4 <main>:
 80483c4:	55                   	push   %ebp
 80483c5:	89 e5                	mov    %esp,%ebp
 80483c7:	83 e4 f0             	and    $0xfffffff0,%esp
 80483ca:	83 ec 10             	sub    $0x10,%esp
 80483cd:	a1 64 96 04 08       	mov    0x8049664,%eax
 80483d2:	83 c0 01             	add    $0x1,%eax
 80483d5:	a3 64 96 04 08       	mov    %eax,0x8049664
 80483da:	8b 15 64 96 04 08    	mov    0x8049664,%edx
 80483e0:	b8 c4 84 04 08       	mov    $0x80484c4,%eax
 80483e5:	89 54 24 04          	mov    %edx,0x4(%esp)
 80483e9:	89 04 24             	mov    %eax,(%esp)
 80483ec:	e8 03 ff ff ff       	call   80482f4
 80483f1:	b8 00 00 00 00       	mov    $0x0,%eax
 80483f6:	c9                   	leave
 80483f7:	c3                   	ret
# 全局变量 i 最后变成了绝对地址为 0x8049664 的内存块（大小为4字节）。
# 注意这里的 0x8049664 不是指立即数 0x8049664， 如果要表示值为
# 0x8049664 的立即数，应该写为 $0x8049664。
```

### 反汇编目标文件

```shell
$ gcc -c -o global.o global.c
$ objdump -s -d global.o > global.txt
$ cat global.txt # 取 ++i 的部分
  9:	a1 00 00 00 00       	mov    0x0,%eax
  e:	83 c0 01             	add    $0x1,%eax
 11:	a3 00 00 00 00       	mov    %eax,0x0
```

怎么会这样？怎么不是 0x8049664 呢？ 这就是目标文件 和 可执行文件的区别了： 全局变量在目标文件中只有一个冒牌地址， 在链接后（各目标文件协商（为自己的全局变量争一块地盘） 完毕）才填入最终的绝对地址。

### 目标文件和可执行文件

目标文件是编译后的产物， 已经完成了 C语言 到 机器码 的转变， 但是部分机器码的操作数还需要在链接过程中修正。

可执行文件是由多个目标文件和 C 运行库链接而成的， C 运行库提供了诸如 printf 等函数的实现。

linux 下目标文件（默认扩展名是.o）和可执行文件都是 ELF 格式（文件内容按照一定格式进行组织）的二进制文件； 类似的，Windows 下 VISUAL C++ 编译出来的目标文件 （扩展名是.obj）采用 COFF 格式，而可执行文件 （扩展名是.exe）采用 PE 格式， ELF 和 PE 都是从 COFF 发展而来的。

因为 linux 下目标文件和可执行文件的内容格式是一样的， 所以 objdump 既可以反汇编可执行文件也可以反汇编目标文件。

全局变量也变成无名无姓的内存地址了， 而且还是常量！
