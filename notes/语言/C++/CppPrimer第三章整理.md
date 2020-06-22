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

* string 对象上的操作
