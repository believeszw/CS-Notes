# 函数调用

这里有几个点需要说明一下：所使用机器是 x86-64 架构

* %rbp 是栈帧指针，用于标识当前栈帧的起始位置

* %rsp 是堆栈指针寄存器，通常会指向栈顶位置，堆栈的 pop 和 push 操作就是通过改变 %rsp 的值即移动堆栈指针的位置来实现的。

* %rax 通常用于存储函数调用的返回结果，同时也用于乘法和除法指令中。在 imul 指令中，两个 64 位的乘法最多会产生 128 位的结果，需要 %rax 与 %rdx 共同存储乘法结果，在 div 指令中被除数是 128 位的，同样需要 %rax 与 %rdx 共同存储被除数

* %rdi, %rsi, %rdx, %rcx, %r8, %r9 六个寄存器用于存储函数调用时的 6 个参数（如果有 6 个或 6 个以上参数的话）。

* %rbx，%rbp，%r12，%r13，%14，%15 用作数据存储，遵循被调用者使用规则，简单说就是随便用，调用子函数之前要备份它，以防他被修改

* %r10，%r11 用作数据存储，遵循调用者使用规则，简单说就是使用之前要先保存原值

* 堆栈平衡：函数调完成后，要返还所有使用过的栈空间。

* 参数传递方式：参数总是从右到左压栈，也就是最后一个参数先入栈。


## C 源程序（call.c）

```shell
$ vim call.c
int bar(int c,int d) {
  int e = c + d;
  return e;
}
int foo(int a,int b) {
  return bar(a, b);
}
int main(void) {
  foo(2,3);
  return 0;
}
```

## 反汇编

如果在编译时加上 -g 选项，那么用 objdump 反汇编时可以把 C 代码和汇编代码穿插起来显示，这样 C 代码和汇编代码的对应关系看得更清楚。反汇编的结果很长，以下只列出我们关心的部分。

```shell
$ gcc -g call.c
$ objdump -dS a.out
...
00000000000005fa <bar>:
int bar(int c,int d) {
 5fa:	55                   	push   %rbp
 5fb:	48 89 e5             	mov    %rsp,%rbp
 5fe:	89 7d ec             	mov    %edi,-0x14(%rbp)
 601:	89 75 e8             	mov    %esi,-0x18(%rbp)
int e = c + d;
 604:	8b 55 ec             	mov    -0x14(%rbp),%edx
 607:	8b 45 e8             	mov    -0x18(%rbp),%eax
 60a:	01 d0                	add    %edx,%eax
 60c:	89 45 fc             	mov    %eax,-0x4(%rbp)
  return e;
 60f:	8b 45 fc             	mov    -0x4(%rbp),%eax
}
 612:	5d                   	pop    %rbp
 613:	c3                   	retq

0000000000000614 <foo>:
int foo(int a,int b) {
 614:	55                   	push   %rbp
 615:	48 89 e5             	mov    %rsp,%rbp
 618:	48 83 ec 08          	sub    $0x8,%rsp
 61c:	89 7d fc             	mov    %edi,-0x4(%rbp)
 61f:	89 75 f8             	mov    %esi,-0x8(%rbp)
  return bar(a, b);
 622:	8b 55 f8             	mov    -0x8(%rbp),%edx
 625:	8b 45 fc             	mov    -0x4(%rbp),%eax
 628:	89 d6                	mov    %edx,%esi
 62a:	89 c7                	mov    %eax,%edi
 62c:	e8 c9 ff ff ff       	callq  5fa <bar>
}
 631:	c9                   	leaveq
 632:	c3                   	retq

0000000000000633 <main>:
int main(void) {
 633:	55                   	push   %rbp
 634:	48 89 e5             	mov    %rsp,%rbp
  foo(2,3);
 637:	be 03 00 00 00       	mov    $0x3,%esi
 63c:	bf 02 00 00 00       	mov    $0x2,%edi
 641:	e8 ce ff ff ff       	callq  614 <foo>
  return 0;
 646:	b8 00 00 00 00       	mov    $0x0,%eax
}
 64b:	5d                   	pop    %rbp
 64c:	c3                   	retq
 64d:	0f 1f 00             	nopl   (%rax)
```

要查看编译后的汇编代码，其实还有一种办法是 gcc -S call.c，这样只生成汇编代码 call.s，而不生成二进制的目标文件。

```shell
$ cat call.s # 只取部分，与上面的做对比
bar:
.LFB0:
	.cfi_startproc
	pushq	%rbp
	.cfi_def_cfa_offset 16
	.cfi_offset 6, -16
	movq	%rsp, %rbp
	.cfi_def_cfa_register 6
	movl	%edi, -20(%rbp)
	movl	%esi, -24(%rbp)
	movl	-20(%rbp), %edx
	movl	-24(%rbp), %eax
	addl	%edx, %eax
	movl	%eax, -4(%rbp)
	movl	-4(%rbp), %eax
	popq	%rbp
	.cfi_def_cfa 7, 8
	ret
	.cfi_endproc
.LFE0:
	.size	bar, .-bar
	.globl	foo
	.type	foo, @function
foo:
.LFB1:
	.cfi_startproc
	pushq	%rbp
	.cfi_def_cfa_offset 16
	.cfi_offset 6, -16
	movq	%rsp, %rbp
	.cfi_def_cfa_register 6
	subq	$8, %rsp
	movl	%edi, -4(%rbp)
	movl	%esi, -8(%rbp)
	movl	-8(%rbp), %edx
	movl	-4(%rbp), %eax
	movl	%edx, %esi
	movl	%eax, %edi
	call	bar
	leave
	.cfi_def_cfa 7, 8
	ret
	.cfi_endproc
.LFE1:
	.size	foo, .-foo
	.globl	main
	.type	main, @function
main:
.LFB2:
	.cfi_startproc
	pushq	%rbp
	.cfi_def_cfa_offset 16
	.cfi_offset 6, -16
	movq	%rsp, %rbp
	.cfi_def_cfa_register 6
	movl	$3, %esi
	movl	$2, %edi
	call	foo
	movl	$0, %eax
	popq	%rbp
	.cfi_def_cfa 7, 8
	ret
	.cfi_endproc
```

我们进入 gdb 模式

```shell
(gdb) start
Temporary breakpoint 1 at 0x637: file call.c, line 9.
Starting program: /home/szw/mypro/test/a.out

Temporary breakpoint 1, main () at call.c:9
9	  foo(2,3);
(gdb) s
foo (a=2, b=3) at call.c:6
6	  return bar(a, b);
(gdb) s
bar (c=2, d=3) at call.c:2
2	int e = c + d;
(gdb) disassemble
Dump of assembler code for function bar:
   0x00005555555545fa <+0>:	push   %rbp
   0x00005555555545fb <+1>:	mov    %rsp,%rbp
   0x00005555555545fe <+4>:	mov    %edi,-0x14(%rbp)
   0x0000555555554601 <+7>:	mov    %esi,-0x18(%rbp)
=> 0x0000555555554604 <+10>:	mov    -0x14(%rbp),%edx
   0x0000555555554607 <+13>:	mov    -0x18(%rbp),%eax
   0x000055555555460a <+16>:	add    %edx,%eax
   0x000055555555460c <+18>:	mov    %eax,-0x4(%rbp)
   0x000055555555460f <+21>:	mov    -0x4(%rbp),%eax
   0x0000555555554612 <+24>:	pop    %rbp
   0x0000555555554613 <+25>:	retq
End of assembler dump.
(gdb) si
0x0000555555554607	2	int e = c + d;
(gdb) si
0x000055555555460a	2	int e = c + d;
(gdb) si
0x000055555555460c	2	int e = c + d;
(gdb) si
3	  return e;
(gdb) si
4	}
(gdb) bt
#0  bar (c=2, d=3) at call.c:4
#1  0x0000555555554631 in foo (a=2, b=3) at call.c:6
#2  0x0000555555554646 in main () at call.c:9
(gdb) info registers
rax            0x5	5
rbx            0x0	0
rcx            0x555555554650	93824992233040
rdx            0x2	2
rsi            0x3	3
rdi            0x2	2
rbp            0x7fffffffe178	0x7fffffffe178
rsp            0x7fffffffe178	0x7fffffffe178
r8             0x7ffff7dced80	140737351839104
r9             0x7ffff7dced80	140737351839104
r10            0x1	1
r11            0x0	0
r12            0x5555555544f0	93824992232688
r13            0x7fffffffe280	140737488347776
r14            0x0	0
r15            0x0	0
rip            0x555555554612	0x555555554612 <bar+24>
eflags         0x206	[ PF IF ]
cs             0x33	51
ss             0x2b	43
ds             0x0	0
es             0x0	0
fs             0x0	0
gs             0x0	0
(gdb) x/20 $rsp
0x7fffffffe178:	-7792	32767	1431651889	21845
0x7fffffffe188:	3	2	-7776	32767
0x7fffffffe198:	1431651910	21845	1431651920	21845
0x7fffffffe1a8:	-140493833	32767	0	32
0x7fffffffe1b8:	-7544	32767	0	1
(gdb)
```

* disassemble : 可以反汇编当前函数或者指定的函数，单独用 disassemble 命令是反汇编当前函数，如果 disassemble 命令后面跟函数名或地址则反汇编指定的函数。

* si : 可以一条指令一条指令地单步调试。

* info registers : 可以显示所有寄存器的当前值。

* p $rsp : 可以打印 rsp 寄存器的值，在上例中 rsp 寄存器的值是 0x7fffffffe178，所以 x/20 $rsp 命令查看内存中从 0x7fffffffe178 地址开始的 20 个 32 位数。

我们从 main 函数的这里开始看起：

```shell
0000000000000633 <main>:
int main(void) {
 633:    55                       push   %rbp # 将 rbp 压入栈
 634:    48 89 e5                 mov    %rsp,%rbp # 将 rsp 的值给 rbp
  foo(2,3);
 637:    be 03 00 00 00           mov    $0x3,%esi # 立即数 3 赋值给第二个参数
 63c:    bf 02 00 00 00           mov    $0x2,%edi # 立即数 2 赋值给第一个参数
 641:    e8 ce ff ff ff           callq  614 <foo> # 调用 foo 函数
  return 0;
 646:    b8 00 00 00 00           mov    $0x0,%eax # 返回值 0
}
 64b:    5d                       pop    %rbp
 64c:    c3                       retq
 64d:    0f 1f 00                 nopl   (%rax)
```
**这里我们看到有 32 位寄存器 esi 和 edi 因为 e 开头的寄存器命名依然可以直接运用于相应寄存器的低 32 位**

要调用函数 foo 先要把参数准备好，第二个参数保存在 esi，第一个参数保存在 edi，可见参数是从右向左依次压栈的。然后执行 call 指令，这个指令有两个作用：

* foo 函数调用完之后要返回到 call 的下一条指令继续执行，所以把 call 的下一条指令的地址 0x614 压栈，同时把 rsp 的值减 8(64 位机器指针大小)， rsp 的值现在是 0x7fffffffe198。

* 修改程序计数器 eip，跳转到 foo 函数的开头执行。

现在看 foo 函数的汇编代码：
```shell
0000000000000614 <foo>:
int foo(int a,int b) {
 614:    55                       push   %rbp
 615:    48 89 e5                 mov    %rsp,%rbp
 618:    48 83 ec 08              sub    $0x8,%rsp # 申请 8 字节空间
 61c:    89 7d fc                 mov    %edi,-0x4(%rbp) # 第一个参数放入栈底向下偏移 4 个字节的位置
 61f:    89 75 f8                 mov    %esi,-0x8(%rbp) # 第二个参数放入栈底向下偏移 8 个字节的位置
  return bar(a, b);
 622:    8b 55 f8                 mov    -0x8(%rbp),%edx # 将 b 赋值给第三个参数
 625:    8b 45 fc                 mov    -0x4(%rbp),%eax # 将 a 赋值给第返回值
 628:    89 d6                    mov    %edx,%esi # 将第三个参数赋值给 bar 第二哥参数
 62a:    89 c7                    mov    %eax,%edi # 将第四个参数赋值给第一个参数
 62c:    e8 c9 ff ff ff           callq  5fa <bar>
}
 631:    c9                       leaveq
 632:    c3                       retq
```
push %rbp 指令把 rbp 寄存器的值压栈，同时把 rsp 的值减 8。rsp 的值现在是 0x7fffffffe190，下一条指令把这个值传送给 rbp 寄存器。这两条指令合起来是把原来 rbp 的值保存在栈上，然后又给 rbp 赋了新值。在每个函数的栈帧中，rbp 指向栈底，而 rsp 指向栈顶，在函数执行过程中 rsp 随着压栈和出栈操作随时变化，而 rbp 是不动的，函数的参数和局部变量都是通过 rbp 的值加上一个偏移量来访问，例如 foo 函数的参数 a 和 b 分别通过 ebp+4 和 ebp+8 来访问。所以下面的指令把参数 a 和 b 再次压栈，为调用 bar 函数做准备，然后把返回地址压栈，调用 bar 函数：






















11126491
