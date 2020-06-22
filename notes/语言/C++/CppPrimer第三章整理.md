# CppPrimer第三章整理

## 目录
* [3.1命名空间的using声明](#命名空间的using声明)
* [3.2标准库类型string](#标准库类型string)
* [3.3标准库类型vector](#标准库类型vector)
* [3.4迭代器介绍](#迭代器介绍)
* [3.5数组](#数组)
* [3.6多维数组](#多维数组)

## 命名空间的using声明

* 头文件不应包含 using 声明

因为头文件的内容会拷贝到所有引用它的文件中去，如果头文件里有某个 using 声明，那么每个使用了该头文件的文件就都会有这个声明。对于某些程序来说，由于不经意间包含了一些名字，反而可能产生始料未及的名字冲突。

## 标准库类型string

* 标准库类型 string 表示可变长的字符序列。

* 定义和初始化 stirng 对象（[更多初始化方式](https://zh.cppreference.com/w/cpp/string/basic_string/basic_string)）
```Cpp
// eg
{
    // string::string()
    std::string s;
    assert(s.empty() && (s.length() == 0) && (s.size() == 0));
  }

  {
    // string::string(size_type count, charT ch)
    std::string s(4, '=');
    std::cout << s << '\n'; // "===="
  }

  {
    std::string const other("Exemplary");
    // string::string(string const& other, size_type pos, size_type count)
    std::string s(other, 0, other.length()-1);
    std::cout << s << '\n'; // "Exemplar"
  }

  {
    // string::string(charT const* s, size_type count)
    std::string s("C-style string", 7);
    std::cout << s << '\n'; // "C-style"
  }

  {
    // string::string(charT const* s)
    std::string s("C-style\0string");
    std::cout << s << '\n'; // "C-style"
  }

  {
    char mutable_c_str[] = "another C-style string";
    // string::string(InputIt first, InputIt last)
    std::string s(std::begin(mutable_c_str)+8, std::end(mutable_c_str)-1);
    std::cout << s << '\n'; // "C-style string"
  }

  {
    std::string const other("Exemplar");
    std::string s(other);
    std::cout << s << '\n'; // "Exemplar"
  }

  {
    // string::string(string&& str)
    std::string s(std::string("C++ by ") + std::string("example"));
    std::cout << s << '\n'; // "C++ by example"
  }

  {
    // string(std::initializer_list<charT> ilist)
    std::string s({ 'C', '-', 's', 't', 'y', 'l', 'e' });
    std::cout << s << '\n'; // "C-style"
  }

  {
    // 重载决议选择 string(InputIt first, InputIt last) [with InputIt = int]
    // 这表现为如同调用 string(size_type count, charT ch)
    std::string s(3, std::toupper('a'));
    std::cout << s << '\n'; // "AAA"
  }
```

* 直接初始化和拷贝初始化

使用等号（=）初始化一个变量，执行的拷贝初始化（copy initialization），编译器把等号右侧的初始值拷贝到新创建的对象中去。不使用等号则是直接初始化(direct initialization)。

对于多个值进行初始化的情况，一般使用直接初始化的方式，除非显示地创建一个（临时）对象用于拷贝:
```cpp
std::string s5 = std::string(10, 'c');
```

### string 对象上的操作

* 读写 string 对象

```cpp
#include <iostream>

int main(){
  std::string s;                // 空字符串
  std::cin >> s;                // 将 string 对象读入 s ，遇到空白为止（自动忽略开头的空白，包括空格符，换行符，制表符等）
  std::cout << s << std::endl;  // 输出 s
  return 0;
}

input : "     Hello World     "
output: "Hello"


```

string 对象的输入输出操作也是返回运算符左侧运算对象作为结果。因此，多个输入或者多个输出可以连写在一起。

```cpp
string s1, s2;
cin >> s1 >> s2;              // 把第一个输入读到 s1 中， 第二个输入读到 s2 中
cout << s1 << s2 << endl;     // 输出两个 string 对象

input : "     Hello World     "
output: "HelloWorld"

```

* 使用 getline 读取一整行

当需要保留输入的 **空白符** 时，用 **getline** ，从给定输入流中读入内容，直到遇到换行符为止**（换行符也被读进来了）** ， getline 只要一遇到换行符就结束读取操作并返回结果，哪怕输入一开始就是换行符，会得到一个空 string

>触发 getline 函数返回的那个换行符实际上被丢弃掉了，得到的 string 对象中并不包含该换行符

* string::size_type 类型

string::size_type 是一个无符号类型的值，而且足够放下任何 string 对象的大小。**如果一条表达式中已经有了 size() 函数就不要再使用 int 了，这样可以避免混用 int 和 unsigned 可能带来的问题。**

* 比较 string 对象

1. 如果两个 string 对象的长度不同，而且较短 string 对象的每个字符都与较长 string 对象对应位置的字符相同，则较短 string 对象小于较长 string 对象。

2. 如果两个 string 对象在某些对应位置上不一致，则 string 对象比较的结果其实是 string 对象中第一对相异字符比较的结果。

```cpp
// eg
string str    = "Hello";
string phrase = "Hello World";
string slang  = "Hiya";

slang > phrase > str


```

* 两个字符串对象相加

```cpp
string s1 = "hello, ", s2 = "world\n";
string s3 = s1 + s2;  // s3 的内容是 hello, world\n
s1 += s2;             // 等价于 s1 = s1 + s2;


```

* 字面值和 string 类型相加

即使一种类型并非所需，我们也可以使用它，不过前提是改种类型可以自动转换成所需类型。**但是在 stirng 对象和字符字面值及字符串字面值混在一条语句中使用时，必须确保每个加法运算符(+)的两侧的运算对象至少有一个是 string :**

```cpp
string s4 = s1 + ", ";      // 正确：把一个 string 对象和一个字面值相加
string s5 = "hello" + ", "; // 错误：两个运算对象都不是 string
string s6 = s1 + ", " + "world";  // 正确：每个加法运算符都有一个运算对象是 string
string s6 = (s1 + ", ") + "world"; // 正确，分组
string s7 = "hello" + ", " + s2;  // 错误：不能把字面值直接相加


```

>为了与 C 兼容，C++ 语言中的字符串字面值并不是标准库类型 string 的对象。切记，字符串字面值与 string 是不同的类型。
