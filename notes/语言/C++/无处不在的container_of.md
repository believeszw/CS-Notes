# 无处不在的container_of

最后不得不提的就是 container_of 宏，在 kernel 中也被广泛使用, container_of 宏用来根据成员的地址来获取结构体的地址。

```Cpp
// 巧妙之处在于将地址0强制转换为type类型的指针，从而定位到member在结构体中偏移位置。编译器认为0是一个有效的地址，从而认为0是type指针的起始地址。
#define offsetof(type, member) (size_t)&( ( (type*)0 )->member )

/**
 * container_of - cast a member of a structure out to the containing structure
 * @ptr:    the pointer to the member.
 * @type:   the type of the container struct this is embedded in.
 * @member: the name of the member within the struct.
 *
 */
#define container_of(ptr, type, member) ({          \
    const typeof( ((type *)0)->member ) * __mptr = (ptr); \
    (type *)( (char *)__mptr - offsetof(type,member) );})
```
___

container_of 宏分为两部分，

第一部分：`const typeof( ((type *)0)->member ) *__mptr = (ptr);`

通过 typeof 定义一个member指针类型的指针变量 \_\_mptr，（即 \_\_mptr 是指向 member 类型的指针），并将 \_\_mptr 赋值为 ptr。

第二部分： `(type * )( (char * )__mptr - offsetof(type,member) )` ，通过 offsetof 宏计算出 member 在 type 中的偏移，然后用 member 的实际地址 \_\_mptr 减去偏移，得到 type 的起始地址，即指向 type 类型的指针。

第一部分的目的是为了将统一转换为 member 类型指针。
