# ldconfig, ldd 与 LD_LIBRARY_PATH 之间的关系

## 注意事项

* 64 位的 linux 机器上的默共享库的查找路径为：/lib64 /usr/lib64。实测发现不会搜索 /lib /usr/lib。而且以上的两个目录没有什么 so 文件。/usr/local/lib64、/usr/local/lib 这两个目录也不会搜索的。

* 动态库的搜索路径搜索的先后顺序是： (**注:居然没有当前路径**) 　　

  1. 编译目标代码时指定的动态库搜索路径; //LDIRNAME 　　

  2. 环境变量 LD_LIBRARY_PATH 指定的动态库搜索路径; 　　

  3. 配置文件 /etc/ld.so.conf 中指定的动态库搜索路径; 只需在在该文件中追加一行库所在的完整路径如 "/root/test/conf/lib" 即可,然后 ldconfig 使修改生效。（实际上是根据缓存文件 /etc/ld.so.cache 来确定路径） 　　

  4. 默认的动态库搜索路径 /lib;（64 位机器为 /lib64） 　　

  5. 默认的动态库搜索路径 /usr/lib。（64 位机器为 /usr/lib64）

* 关于 ldconfig

  1. ldconfig 主要的作用是根据 /etc/ld.so.conf 的内容，查找内容中所包含目录下实际的动态库文件，生成搜索共享库的缓存文件 /etc/ld.so.cache 。

  2. 缓存必须与实际路径的文件相一致。机器比较傻，只认缓存，然后按照缓存的路径去实际路径查找文件。增加或删除了实际的共享库路径下的文件，而没有更新缓存，执行被其依赖的可执行文件时会出错。

  3. 查看共享库的缓存内容。ldconfig -p。

* /lib 或 /usr/lib（64 位系统下为 /lib64 /usr/lib64）路径下的共享库比较特殊。

  1. 它是默认的共享库的搜索路径。

  2. 它没有放到 /etc/ld.so.conf 文件中。但是在 /etc/ld.so.cache 的缓存中有它。

  3. 其路径下的共享库的变动即时生效，不用执行 ldconfig。就算缓存 ldconfig -p 中没有，新加入的动态库也可以执行。

## linux 中搜索动态库的顺序

Linux 运行的时候，是如何管理共享库(*.so)的？ 在 Linux 下面，共享库的寻找和加载是由 /lib/ld.so 实现的。 ld.so 在标准路经(/lib, /usr/lib) 中寻找应用程序用到的共享库。 但是，如果需要用到的共享库在非标准路经，ld.so 怎么找到它呢？ 目前，Linux 通用的做法是将非标准路经加入 /etc/ld.so.conf，然后运行 ldconfig 生成 /etc/ld.so.cache。 ld.so 加载共享库的时候，会从 ld.so.cache 查找。 传统上，Linux 的先辈 Unix 还有一个环境变量：LD_LIBRARY_PATH 来处理非标准路经的共享库。ld.so 加载共享库的时候，也会查找这个变量所设置的路经。 LD_LIBRARY_PATH=$LD_LIBRARY_PATH:./lib export LD_LIBRARY_PATH 但是，有不少声音主张要避免使用 LD_LIBRARY_PATH 变量，尤其是作为全局变量

## ldd 的作用

ldd 命令的作用是 打印共享库的依赖关系

首先 ldd 不是一个可执行程序，而只是一个 shell 脚本

ldd 能够显示可执行模块的 dependency，其原理是通过设置一系列的环境变量，如下：LD_TRACE_LOADED_OBJECTS、LD_WARN、LD_BIND_NOW、LD_LIBRARY_VERSION、 LD_VERBOSE 等。当 LD_TRACE_LOADED_OBJECTS 环境变量不为空时，任何可执行程序在运行时，它都会只显示模块的 dependency，而程序并不真正执行。要不你可以在 shell 终端测试一下，如下： 　　

1. export LD_TRACE_LOADED_OBJECTS=1

2. 再执行任何的程序，如 ls 等，看看程序的运行结果

ldd 显示可执行模块的 dependency 的工作原理，其实质是通过 ld-linux.so（elf 动态库的装载器）来实现的。我们知道，ld-linux.so 模块会先于 executable 模块程序工作，并获得控制权，因此当上述的那些环境变量被设置时，ld-linux.so 选择了显示可执行模块的 dependency。

实际上可以直接执行 ld-linux.so 模块，如：/lib/ld-linux.so.2 --list program（这相当于 ldd program）

语法：ldd(选项)(参数)

选项：

--version：打印指令版本号； -v：详细信息模式，打印所有相关信息； -u：打印未使用的直接依赖； -d：执行重定位和报告任何丢失的对象； -r：执行数据对象和函数的重定位，并且报告任何丢失的对象和函数； --help：显示帮助信息。
参数：

文件：指定可执行程序或者文库。

示例：查看 passwd 程序运行所依赖的库:

```shell
NTP-slave:/usr/local/openssl/lib # ldd /usr/bin/passwd

linux-vdso.so.1 =>  (0x00007fff15dff000)

libpam_misc.so.0 => /lib64/libpam_misc.so.0 (0x00007fce5ed59000)

libpam.so.0 => /lib64/libpam.so.0 (0x00007fce5eb4b000)

libldap-2.4.so.2 => /usr/lib64/libldap-2.4.so.2 (0x00007fce5e901000)

        ...省略...

libcrypto.so.0.9.8 => /usr/lib64/libcrypto.so.0.9.8 (0x00007fce5cefc000)

/lib64/ld-linux-x86-64.so.2 (0x00007fce5f1a3000)

libz.so.1 => /lib64/libz.so.1 (0x00007fce5cce5000)
```

第一列：程序需要依赖什么库

第二列: 系统提供的与程序需要的库所对应的库

第三列：库加载的开始地址

通过上面的信息，我们可以得到以下几个信息：

1. 通过对比第一列和第二列，我们可以分析程序需要依赖的库和系统实际提供的，是否相匹配

2. 通过观察第三列，我们可以知道在当前的库中的符号在对应的进程的地址空间中的开始位置
如果依赖的某个库找不到，通过这个命令可以迅速定位问题所在
