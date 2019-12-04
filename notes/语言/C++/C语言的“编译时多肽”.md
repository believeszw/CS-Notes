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

希望大家对 typeof 的使用有了一个更好的理解，欢迎评论！
