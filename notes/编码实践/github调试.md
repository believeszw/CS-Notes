# github调试

## 总览

* [启动调试](#启动调试)
* [断点设置](#断点设置)
* [变量查看](#变量查看)
* [单步调试](#单步调试)
* [源码查看](#源码查看)


### 启动调试

#### 前言

GDB（GNU Debugger）是 UNIX 及 UNIX-like 下的强大调试工具，可以调试 ada, c, c++, asm, minimal, d, fortran, objective-c, go, java,pascal 等语言。本文以 C 程序为例，介绍 GDB 启动调试的多种方式。

**哪类程序可被调试**

对于 C 程序来说，需要在编译时加上 -g 参数，保留调试信息，否则不能使用 GDB 进行调试。
但如果不是自己编译的程序，并不知道是否带有 -g 参数，如何判断一个文件是否带有调试信息呢？

gdb 文件

例如：

```shell
$ gdb helloworld
Reading symbols from helloWorld...(no debugging symbols found)...done.
```

如果没有调试信息，会提示 no debugging symbols found。
如果是下面的提示：

```shell
Reading symbols from helloWorld...done.
```

则可以进行调试。

**readelf查看段信息**

例如：

```shell
$ readelf -S helloWorld|grep debug
  [] .debug_aranges    PROGBITS           d
  [] .debug_info       PROGBITS           d
  [] .debug_abbrev     PROGBITS           b
  [] .debug_line       PROGBITS           b9
  [] .debug_str        PROGBITS           fc
```

helloWorld 为文件名，如果没有任何 debug 信息，则不能被调试。

**file查看strip状况**

下面的情况也是不可调试的：

```shell
$ file helloWorld
helloWorld: (省略前面内容) stripped
```

如果最后是 stripped ，则说明该文件的符号表信息和调试信息已被去除，不能使用 gdb 调试。但是 not stripped 的情况并不能说明能够被调试。

**调试方式运行程序**

程序还未启动时，可有多种方式启动调试。

**调试启动无参程序**

例如：

```shell
$ gdb helloWorld
(gdb)
```

输入 run 命令，即可运行程序

**调试启动带参程序**

假设有以下程序，启动时需要带参数：

```shell
#include<stdio.h>
int main(int argc,char *argv[])
{
    if( >= argc)
    {
        printf("usage:hello name\n");
        return ;
    }
    printf("Hello World %s!\n",argv[]);
    return  ;
}
```

编译：

```shell
$ gcc -g -o hello hello.c
```

这种情况如何启动调试呢？需要设置参数：

```shell
$ gdb hello
(gdb)run 编程珠玑
Starting program: /home/shouwang/workspaces/c/hello 编程珠玑
Hello World 编程珠玑!
[Inferior  (process ) exited normally]
(gdb)
```

只需要 run 的时候带上参数即可。
或者使用 set args ，然后在用 run 启动：

```shell
$ gdb hello
(gdb) set args 编程珠玑
(gdb) run
Starting program: /home/hyb/workspaces/c/hello 编程珠玑
Hello World 编程珠玑!
[Inferior  (process ) exited normally]
(gdb)
```

**调试core文件**

当程序 core dump 时，可能会产生 core 文件，它能够很大程序帮助我们定位问题。但前提是系统没有限制 core 文件的产生。可以使用命令 limit -c 查看：

```shell
$ ulimit -c
0
```

如果结果是 0，那么恭喜你，即便程序 core dump 了也不会有 core 文件留下。我们需要让 core 文件能够产生：

```shell
$ ulimit -c unlimied  #表示不限制core文件大小
$ ulimit -c 10        #设置最大大小，单位为块，一块默认为512字节
```

上面两种方式可选其一。第一种无限制，第二种指定最大产生的大小。
调试 core 文件也很简单：

```shell
$ gdb 程序文件名 core文件名
```

具体可参看《linux常用命令-开发调试篇》 gdb 部分。

**调试已运行程序**

如果程序已经运行了怎么办呢？
首先使用 ps 命令找到进程 id ：

```shell
$ ps -ef|grep 进程名
```

或者：

```shell
$ pidof 进程名
```

**attach方式**

假设获取到进程 id 为 20829 ，则可用下面的方式调试进程：

```shell
$ gdb
(gdb) attach
```

接下来就可以继续你的调试啦。

可能会有下面的错误提示：

```shell
Could not attach to process.  If your uid matches the uid of the target
process, check the setting of /proc/sys/kernel/yama/ptrace_scope, or try
again as the root user.  For more details, see /etc/sysctl.d/-ptrace.conf
ptrace: Operation not permitted.
```

解决方法，切换到 root 用户：
将 /etc/sysctl.d/10-ptrace.conf 中的

```shell
kernel.yama.ptrace_scope = 1
# ->
kernel.yama.ptrace_scope = 0
```

**直接调试相关 id 进程**

还可以是用这样的方式 gdb program pid ，例如:

```shell
gdb hello   
```

或者：

```shell
gdb hello --pid
```

已运行程序没有调试信息
为了节省磁盘空间，已经运行的程序通常没有调试信息。但如果又不能停止当前程序重新启动调试，那怎么办呢？还有办法，那就是同样的代码，再编译出一个带调试信息的版本。然后使用和前面提到的方式操作。对于 attach 方式，在 attach 之前，使用 file 命令即可：

```shell
$ gdb
(gdb) file hello
Reading symbols from hello...done.
(gdb)attach
```

**小结**

本节主要介绍了两种类型的 GDB 启动调试方式，分别是调试未运行的程序和已经运行的程序。对于什么样的程序能够进行调试也进行了简单说明。

### 断点设置
### 变量查看
### 单步调试
### 源码查看
