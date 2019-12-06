# typeof 在 kernel 中的使用 —— C 语言的“编译时多态”

C 语言本身没有多态的概念，函数没有重载的概念。然而随着 C 语言编写的软件逐渐庞大，越来越多地需要引入一些其他语言中的特性，来帮助更高效地进行开发，Linux kernel 是一个典型例子。

在动态类型的语言里面，往往有 typeof 这种语法，来获取变量的数据类型，比如 JavaScript 当中，typeof 以字符串型式返回了这个变量的数据类型，借由这种特性，往往可以根据传入参数的类型不同，产生不同的行为。

GCC 提供的 typeof，实际上是在预编译时处理的，最后实际转化为数据类型被编译器处理。用法上也和上述语言不太一样。

基本用法是这样的：

```Cpp
int a;
typeof(a) b; //这等同于int b;
typeof(&a) c; //这等同于int* c;
```

那么在内核中这种特性是怎样使用的呢？

```Cpp
/*
 * Check at compile time that something is of a particular type.
 * Always evaluates to 1 so you may use it easily in comparisons.
 */
#define typecheck(type,x) \
({  type __dummy; \
    typeof(x) __dummy2; \
    (void)(&__dummy == &__dummy2); \
    1; \
})

/*
 * Check at compile time that 'function' is a certain type, or is a pointer
 * to that type (needs to use typedef for the function type.)
 */
#define typecheck_fn(type,function) \
({  typeof(type) __tmp = function; \
    (void)__tmp; \
})
```
___

这两段代码来自于 `include/linux/typecheck.h`，用于数据类型检查。

宏 typecheck 用于检查 x 是否是 type 类型，如果不是，那么编译器会抛出一个 `warning(warning: comparison of distinct pointer types lacks a cast);` 而 typecheck_fn 则用于检查函数 function 是否是 type 类型，不一致则抛出 `warning（warning: initialization from incompatible pointer type）`。

原理很简单，对于 typecheck ，只有当 x 的类型与 value 一致，`&__dummy == &__dummy2` 的比较才不会因为类型不匹配而抛出 warning ，详情可以参考 C 语言对于指针操作的标准规定。对于 typecheck_fn ，当然也只有 function 的返回值和参数表与 type 描述一致，才不会因为类型不匹配而抛出 warning 。

到这里有人可能会有一个疑问，内核代码里执行类型检查会不会降低效率？答案是不会的，因为实际上，这些为类型检查而声明的临时变量，实际上在上下文中都没有使用，并且还特别地强制类型转换为 void 防止任何由这些临时变量产生的结果被使用的情况，因此在编译器优化时，就将这些无用的代码删除了。

然后 kernel 中还定义了使用另一种类型检查策略的获取最大最小值的宏。

```Cpp
/*
 * ..and if you can't take the strict
 * types, you can specify one yourself.
 *
 * Or not use min/max/clamp at all, of course.
 */
#define min_t(type, x, y) ({            \
    type __min1 = (x);          \
    type __min2 = (y);          \
    __min1 < __min2 ? __min1: __min2; })

#define max_t(type, x, y) ({            \
    type __max1 = (x);          \
    type __max2 = (y);          \
    __max1 > __max2 ? __max1: __max2; })
```
___

这个例子里面不要求 x 和 y 是严格等于 type 类型，只要 x 和 y 能够安全地完成隐式类型转换为 type 就可以安全通过编译，否则会抛出 warning。

另外一个非常经典的例子就是交换变量。

```Cpp
/*
 * swap - swap value of @a and @b
 */
#define swap(a, b) \
    do { typeof(a) __tmp = (a); (a) = (b); (b) = __tmp; } while (0)
```
___

试想如果没有 typeof，要怎么在 C 语言中实现这种类似 C++ 模板的特性呢？

# 这里还有一个功能和 typeof 类似的运算符： typeid

typeid 是为 RTTI（运行时类型检查） 提供的第二个运算符。
* typeid 运算符允许在运行时确定对象的类型

* 返回的结果是 const type_info&

* typeid 运算符在应用于多态类类型的左值时执行运行时检查，其中对象的实际类型不能由提供的静态信息确定；

* **typeid 也可以在模板中使用以确定模板参数的类型**

* typeid 是操作符，不是函数，运行时获知变量类型名称；

>和 typeof 的主要区别有二：
* typeof(编译器提供) 是一个编译时结构，并返回编译时定义的类型,和 C++11 提供的关键字 decltype 类似
* typeid( C++ 提供) 是一个运行时结构，因此提供了有关该值的运行时类型的信息

typeid 表达式的形式是 typeid(e), 其中 e 可以是**任意表达式**或者**类型的名字**。 typeid 操作的结果是一个**常量对象的引用**，该对象的类型是标准库类型 type_info 或者 type_info 的**公有派生类型**。如果表达式是一个引用，则 typeid 返回该引用所引对象的类型。不过当 typeid 作用于**数组或函数**时，并不会执行向指针的标准类型转换。也就是说，**如果我们对数组 a 执行 typeid(a) ，则所得的结果是数组类型而非指针类型。**

>当运算对象不属于类类型或者是一个不包含任何虚函数的类时， typeid 运算符指示的运算对象的**静态类型**。而当运算对象是定义了至少一个虚函数的类的**左值**时， typeid 的结果直到运行时才会求得。

>当 typeid 作用于指针时（而非指针所指的对象），返回的结果是该指针的**静态编译时类型**

typeid 是否需要运行时检查决定了表达式是否会被求值。只有当类型含有虚函数时，编译器才会对表达式求值。反之，如果类型不含有虚函数，则 typeid 返回表达式的静态类型；编译器无须对表达式求值也能知道表达式的静态类型。

### RTTI 栗子：
```Cpp
class Base {
  friend bool operator==(const Base&, const Base&);
public:
  // Base 的接口成员
protected:
  virtual bool equal(const Base&) const;
  // Base 的数据成员和其他用于实现的成员
};

class Derived : public Base {
public:
  // Derived 的其他接口成员
protected:
  bool equal(const Base&) const;
  // Derived 的数据成员和其他用于实现的成员
};

// 类型敏感的相等运算符
// 接下来介绍我们是如何定义整体的相等运算符的：
bool operator==(const Base &lhs, const Base &rhs) {
  // 如果 typeid 不相同，返回 false；否则虚调用 equal
  return typeid(lhs) == typeid(rhs) && lhs.equal(rhs);
}
```
在这个运算符中，如果运算对象的类型不同则返回 false。否则，如果运算对象的类型相同，则运算符将其工作委托给虚函数 equal 。当运算对象是 Base 的对象时，调用 Base::equal ；当运算对象是 Derived 的对象时，调用 Derived::equal 。

#### 虚 equal 函数
继承体系中的每个类必须定义自己的 equal 函数。派生类的所有函数要做的第一件事都是相同的，那就是将实参的类型转换为派生类类型：
```Cpp
bool Derived::equal(const Base &rhs) const {
  // 我们清楚这两个类型是相等的，所以转换过程不会抛出异常
  auto r = dynamic_cast<const Derived&>(rhs);
  // 执行比较两个 Derived 对象的操作并返回结果
}
```
上面的类型转换永远不会失败，因为毕竟我们只有在验证了运算对象的类型相同之后才会调用该函数。**然而这样的类型转换必不可少，执行了类型转换后，当前的函数才能访问右侧运算对象的派生类成员**

## type_info 类
type_info 的操作
* t1 == t2 如果 type_info 对象 t1 和 t2 表示同一种类型，返回 true， 否则返回 false

* t1 != t2 与上一条相反

* t.name() 返回一个 C 风格的字符串，表示类型名字的可打印形式。类型的名字生成方式因系统而异

* t1.before(t2) 返回一个 boo 值，表示 t1 是否位于 t2 之前。 before 所采用的顺序关系是依赖于编译器的。

一般 type_info 是作为一个基类出现，所以应该提供一个公有的虚析构函数。当编译器希望提供额外的类型信息时，通常在 type_info 的派生类中完成。

type_info 类**没有默认构造函数**，而且他的**拷贝和移动构造函数以及赋值运算符都被定义成删除的。因此我们无法定义或拷贝 type_info 类型的对象，也不能为 type_info 类型的对象赋值。创建 type_info 对象的唯一途径是使用 typeid 运算符**。

栗子：
```Cpp
int arr[10];
Derived d;
Base *p = &d;

std::cout << typeid(42).id() << ", "
          << typeid(arr).name() << ", "
          << typeid(Sales_data).name() << ", "
          << typeid(std::string).name() << ", "
          << typeid(p).name() << ", "
          << typeid(* p).name() << std::endl;
```
在作者的计算机上运行该程序，结果如下
```Cpp
i, A10_i, 10Sales_data, Ss, P4Base, 7Derived
```
