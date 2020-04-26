# The LD_DEBUG environment variable
LD_DEBUG 是 glibc 中的 loader 为了方便自身调试而设置的一个环境变量。通过设置这个环境变量，可以方便的看到 loader 的加载过程,使用ldd命令可以知道程序依赖于哪些库，在找不到这些库的时候，使用LD_DEBUG可以知道系统在哪些路径下进行了尝试。

LD_DEBUG 可以用来查看程序搜索库的路径，使用方法如下:

LD_DEBUG=libs ./your_program

LD_DEBUG=help cat 查看命令使用方法

```shell
LD_DEBUG=help cat
Valid options for the LD_DEBUG environment variable are:

  libs        display library search paths
  reloc       display relocation processing
  files       display progress for input file
  symbols     display symbol table processing
  bindings    display information about symbol binding
  versions    display version dependencies
  all         all previous options combined
  statistics  display relocation statistics
  unused      determined unused DSOs
  help        display this help message and exit

To direct the debugging output into a file instead of standard output
a filename can be specified using the LD_DEBUG_OUTPUT environment
variable.
```
