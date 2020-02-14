# gtest单元测试工具和lcov覆盖率统计工具的结合使用

## 环境配置
* ubuntu 18.04 server
* 从 github 克隆最新版 GoogleTest；
```cpp
git clone https://github.com/google/googletest.git
// 修改 CMakeList.txt 添加 add_compile_options(-fPIC) 提供跨平台支持
cd googletest
cmake -Dgtest_build_samples=ON .
make
```
* 安装 lcov
```shell
sudo apt install lcov
```
* 修改 CMake\
// 添加--coverage到编译器flag中，这个参数是很重要的，因为这是生成代码覆盖率所必须的，关于该编译参数的说明见这里[Program Instrumentation Options](https://gcc.gnu.org/onlinedocs/gcc/Instrumentation-Options.html#Instrumentation-Options)
```cpp
SET(CMAKE_CXX_FLAGS  "${CMAKE_CXX_FLAGS} --coverage")
```
>--coverage 等同于编译参数 -fprofile-arcs -ftest-coverage 以及在链接时增加 -lgcov。

此处的编译结果除了得到可执行文件 a.out，还会得到一个 array_test.gcno 文件。该文件包含了代码与行号的信息，在生成覆盖率时会需要这个文件。

>很显然，带 --coverage 编译参数得到的编译产物会比不带这个参数要包含更多的信息，因此编译产物会更大。所以这个参数只适合在需要生成代码覆盖率的时候才加上。对于正式发布的编译产物，不应该添加这个编译参数。

## 编译
当我们执行上面编译出来的可执行文件 a.out 时，我们还会得到每个源码文件对应的 gcda 后缀的文件。由 array_test.gcno 和 array_test.gcda 这两个文件，便可以得到代码的覆盖率结果了。文件在 `/cmake-build-debug/02-Arrays/test/CMakeFiles/ArraysTest.dir/` 下，可以使用 `find -name ".gcno"` 查找。

>只需要通过 gcov 指定源文件的名称（不需要带后缀）：gcov array_test，便可以得到包含覆盖率的结果文件 array_test.c.gcov了。不过是文本形式

## 使用LCOV统计覆盖率
```shell
lcov -d . -t 'ArraysTest' -o 'ArraysTest.info' -b . -c
genhtml -o result ArraysTest.info
```

查看index.html文件，结果如下 可以点击链接查看每个目录，每个文件的覆盖率结果，包括行覆盖率（Line）和方法覆盖率（Function）
