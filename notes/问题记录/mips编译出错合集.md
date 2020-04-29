# mips编译出错合集

## gcc - 在mips交叉编译器中，crt1.o 错误
交叉编译时遇到：
```shell
/opt/cross/lib/gcc/mipsel-unknown-linux-gnu/4.8.2/../../../../mipsel-unknown-linux-gnu/bin/ld: cannot find crt1.o: No such file or directory
/opt/cross/lib/gcc/mipsel-unknown-linux-gnu/4.8.2/../../../../mipsel-unknown-linux-gnu/bin/ld: cannot find crti.o: No such file or directory
/opt/cross/lib/gcc/mipsel-unknown-linux-gnu/4.8.2/../../../../mipsel-unknown-linux-gnu/bin/ld: cannot find -lc
/opt/cross/lib/gcc/mipsel-unknown-linux-gnu/4.8.2/../../../../mipsel-unknown-linux-gnu/bin/ld: cannot find crtn.o: No such file or directory
collect2: error: ld returned 1 exit status
```
在编译命令中添加 SYSROOT 。SYSROOT 将自动提供标题和库路径。

你将知道当你有一个 SYSROOT，因为在使用的路径中将有一个 bin/，include/ 和 lib/ 。 例如这里是一个用于 arm-linux-gnueabi ( 。例如，arm-linux-gnueabi-gcc 和 arm-linux-gnueabi-g++ )的SYSROOT:

```shell
$ ls/usr/arm-linux-gnueabi
bin include lib
```

因此，在本例中，你将使用 --sysroot=/usr/arm-linux-gnueabi 。

如果需要查找 SYSROOT的帮助，请执行 find:

```shell
$ find/usr -name crt1.o
/usr/arm-linux-gnueabi/lib/crt1.o
/usr/lib/debug/usr/lib/x86_64-linux-gnu/crt1.o
/usr/lib/x86_64-linux-gnu/crt1.o
```

在你的情况下，你可能会从 /opt/cross 搜索。 显然，你需要目标( arm-linux-gnueabi )的那个，而不是主机( x86_64-linux-gnu )的。

添加--sysroot=将解决这里问题。 在交叉编译时，不应该选择任何具有 crt1.o 或者 crtX.o的文件夹作为sysroot目录。 可能是你的主机文件。 ( 如果你在x86上运行，则为 x86 ) 。 它又有 32bit 位和 64位。

你需要有一个 With，它需要具有适当的sysroot和 crt1.o.，这应该是你的wsdl和目标体系结构。
