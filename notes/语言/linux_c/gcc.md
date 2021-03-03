# gcc

<table border="1">
 <tr>
  <th>编译阶段</th>
  <th>命令</th>
  <th>截断后的产物</th>
 </tr>
 <tr>
  <td></td>
  <td></td>
  <td>C源程序</td>
 </tr>
 <tr>
  <td>预处理</td>
  <td>gcc -E</td>
  <td>替换了宏的C源程序(没有了#define,#include…),
	删除了注释</td>
 </tr>
 <tr>
  <td>编译</td>
  <td>gcc -S</td>
  <td>汇编源程序</td>
 </tr>
 <tr>
  <td>汇编</td>
  <td>gcc -c</td>
  <td>目标文件，二进制文件，
  允许有不在此文件中的外部变量、函数</td>
 </tr>
 <tr>
  <td>链接</td>
  <td>gcc</td>
  <td>可执行程序，一般由多个目标文件或库链接而成，
	二进制文件，所有变量、函数都必须找得到</td>
 </tr>
</table>

## 照妖镜

一个C程序（max.c）：

```C
#define MAX(a,b) ((a)>=(b)?(a):(b))

int main(){
	int c = MAX(1,2); // 注注注注释
	return 0;
}
```

```Shell
$ gcc -E -o max2.c max.c
$ cat max2.c
# 1 "max.c"
# 1 "<built-in>"
# 1 "<命令行>"
# 1 "max.c"


int main(){
 int c=((1)>=(2)?(1):(2));
 return 0;
}
```

## 火眼金睛

想要看到更多细节，就需要有**火眼金睛**了，这里有一个 hello.c

```c
#include <stdio.h>

int main(){
	printf("Hello, World!\n");
	return 0;
}
```

```shell
$ gcc -S -o hello.s hello.c
$ cat hello.s
.file	"hello.c"
.section	.rodata
.LC0:
.string	"Hello, World!"
.text
.globl main
.type	main, @function
main:
pushl	%ebp
movl	%esp, %ebp
andl	$-16, %esp
subl	$16, %esp
movl	$.LC0, (%esp)
call	puts
movl	$0, %eax
leave
ret
.size	main, .-main
.ident	"GCC: (GNU) 4.5.1 20100924 (Red Hat 4.5.1-4)"
.section	.note.GNU-stack,"",@progbits
```

**汇编过程在其他篇章**
