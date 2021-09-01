# Chapter 17 命名空间

> 多个库将名字定义在全局命名空间中可能引发命名空间污染  

## 1. 定义格式  

```C++
namespace my_space
{
    class A;
    void fun();
    int n;
}
```

注意花括号后无须加分号，可以在全局命名空间中定义的都可以在命名空间中定义  
命名空间对全局命名空间做了分割，命名空间也可以定义在其他命名空间中，但不能定义在函数或类的内部  
命名空间可以不连续：  

```C++
// 两个 my_space 为同一个命名空间，这两个 my_space 也可以定义在不同文件中
namespace my_space
{
    class A;
    void foo();
}

class A;

// 打开原有的命名空间
namespace my_space
{
    void foo()
    {
        return;
    }
}
```

非内联函数、静态数据成员和变量等只能在程序中定义一次的，在命名空间中也只能定义一次  
`#include` 指令通常写在命名空间外面，因为如果写在里面容易造成命名空间嵌套  

## 2. 模板特例化与命名空间

模板特例化必须在原始模板的命名空间中声明，可以在命名空间外部定义  

```C++
namespace std
{
    template <> struct hash<A>;
}

template <> struct std::hash<A>
{
    /* 函数体 */
}
```

## 3. 全局命名空间

```C++
// 使用作用域运算符访问全局命名空间成员

int member;

namespace my_space
{
    ::member;
}
```

## 4. 内联命名空间

在第一次定义命名空间时添加 `inline` 关键字，该类命名空间的名字可以被外层命名空间直接使用  
下面的例子说明其应用场景：  

```C++
// ver1.0.h
namespace v1
{
    /* 1.0 版本内容 */
    class A;
}
```

```C++
// ver2.0.h
inline namespace v2
{
    /* 2.0 版本内容 */
    class A;
}
```

```C++
// api.h（用户接口）
namespace api
{
    #include "ver2.0.h"
    #include "ver1.0.h"
}
/* 此时用户调用 api::A 时，默认调用 2.0 版本的 A
 * 如果要使用老版本的 A，则需要显式调用：api::v1::A
 */
```

## 5. 未命名命名空间

可以在文件内不连续，但不能跨文件  
两个文件的未命名命名空间相互无关  
如果一个头文件的未命名命名空间被包含多次，则会生成多个实例  
未命名命名空间的作用域与其所在的作用域相同，故在该命名空间中定义名字时要与外部有所区分  

```C++
namespace my_space
{
    namespace
    {
        int i;
    }
}
my_space::i; // 可以访问
```

## 6. 命名空间别名

```C++
namespace ms = my_space;
namespace ma = my_space::A;
```

## 7. using 声明

一次只引入命名空间的一个成员，生命周期为从 `using` 声明开始到作用域结束，会覆盖外层作用域中的相同名字  
`using` 声明可以出现在全局作用域、局部作用域、命名空间作用域和类作用域中，在类作用域中只能指向直接或间接基类成员  

```C++
struct test1
{
    void foo();
};

struct test2 : test1
{
    void foo();
};

struct test3 : test2
{
    using test1::foo;
    void bar()
    {
        // 如果不使用 using 声明则调用 test2::foo
        foo();
    }
};
```

## 8. using 指示

可以出现在全局作用域、局部作用域和命名空间作用域中，使命名空间中所有名字都可见  
`using` 指示的命名空间中的名字，对当前作用域来说，相当于定义在最近的外层作用域中的名字  

```C++
namespace sp
{
    int i, j, k;
}

int j;

void f()
{
    using namespace sp;
    ++j; // 错误，具有二义性
    ++::j; // 正确
    ++sp::j; // 正确
    int k = 0;
    ++k; // 正确，但是使用的是内部定义的 k
}
```

头文件如果在顶层作用域使用 `using` 指示或声明，则会导致该 `using` 指示或声明被加载到包含它的文件中  
因此，头文件通常只在其函数或非顶层命名空间中使用 `using` 指示或声明  

## 10. 名字查找的特例

```C++
std::string s;
std::cin << s;
```

由于未声明 `using namespace std;`，故理论上，应该声明 `using std::operator<<` 才能使用输入运算符  
但实际上有例外，即函数实参为类对象时，除了常规的名字查找外还会查找实参类所属的命名空间，本例中即为 `std`  

```C++
namespace sp
{
    class A
    {
        friend void f1(const A&);
        friend void f2();
    };
}

int main()
{
    sp::A obj;
    f1(obj); // 正确
    f2(); // 错误
}
```

一个未声明的类或函数如果第一次出现在友元声明中，则会被认为是最近的外层命名空间的成员  
因此，f1 通过函数实参的查找特例被找到，而 f2 则找不到  

## 11. std::move 和 std::forward

由于这两个函数模板能接受任何类型的输入，因此极易和自定义的函数产生冲突，故调用时通常不省略 `std::`  

## 12. 重载与实参查找特例

## 13. 重载与 using 声明或指示
