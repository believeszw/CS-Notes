# 局部变量

有如下代码

```c
int i = 3;
int ans = (++i)+(++i)+(++i);
```

最终 ans 的值是多少？

## 验证

```c
#include <stdio.h>

int main()
{
	int i = 3;
	int ans = (++i)+(++i)+(++i);

	printf("%d\n",ans);
	return 0;
}
```

linux 中编译运行，结果是 16 ，为何 ？

```shell
$ gcc -o inc inc.c
$ ./inc
16
$
```

## 探究

```shell
$ gcc -S -o inc.s inc.c
$ cat inc.s
.file	"inc.c"
.section	.rodata
.LC0:
.string	"%d\n"
.text
.globl main
.type	main, @function
main:
pushl	%ebp
movl	%esp, %ebp
andl	$-16, %esp
subl	$32, %esp
movl	$3, 28(%esp)
addl	$1, 28(%esp)
addl	$1, 28(%esp)
movl	28(%esp), %eax
addl	%eax, %eax
addl	$1, 28(%esp)
addl	28(%esp), %eax
movl	%eax, 24(%esp)
movl	$.LC0, %eax
movl	24(%esp), %edx
movl	%edx, 4(%esp)
movl	%eax, (%esp)
call	printf
movl	$0, %eax
leave
ret
.size	main, .-main
.ident	"GCC: (GNU) 4.5.1 20100924 (Red Hat 4.5.1-4)"
.section	.note.GNU-stack,"",@progbits
```

取出我们需要关注到的部分，并加上注释

```shell
movl	$3, 28(%esp)	# i = 3;
addl	$1, 28(%esp)	# ++i; // 4
addl	$1, 28(%esp)	# ++i; // 5
movl	28(%esp), %eax	# eax = i;
addl	%eax, %eax		# eax = eax + eax; // 10
addl	$1, 28(%esp)	# ++i; // 6
addl	28(%esp), %eax	# eax = eax + i; // 16
movl	%eax, 24(%esp)	# ans = eax;
```

## 思考

同样的程序，在 VC 上编译运行， Debug 模式的运行结果是 16，Release 模式的运行结果是 18； 而在 Visual Studio 2010 中 Debug 和 Release 模式下都是 18。

## 小结

从这个例子中我们应该吸取经验： 被实施递增（递减）操作的变量不应该在表达式中多次出现， 否则结果就不受我们控制了，而是被编译器自由发挥：

>C 标准规定：两个序列点之间， 程序执行的顺序可以是任意的。 这样做给了编译器优化的空间。
