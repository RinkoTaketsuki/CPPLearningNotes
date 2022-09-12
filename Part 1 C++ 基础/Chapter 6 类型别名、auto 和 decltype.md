# Chapter 6 类型别名、auto 和 decltype

## 1. 类型别名

```C++
typedef int* pint;
using rint = int&;
```

## 2. auto 类型说明符

1. 由编译器去分析赋值符号右边表达式的类型，故定义时必须有初始值，不可以只声明

2. 使用 auto 定义多个变量时其基础类型必须一样

    ```C++
    auto i = 0, *p = &i; // 正确
    auto j = 0, d = 3.14; // 错误
    ```

3. auto 会忽略引用和顶层 const，除非明确指出

    ```C++
    int i = 0, &r = i; const int ci = 42; int *const p = &i;
    auto b = ci; // int
    auto c = r; // int
    auto d = &i; // int *
    auto e = &ci; // const int *
    auto f = p; // int *
    const auto g = ci; // const int
    const auto &h = ci; // const int &
    ```

## 3. decltype 类型指示符

```C++
decltype(表达式) 变量;
```

1. 表达式并不会执行
2. *p运算的类型为引用类型
3. ++i，--i的类型为引用类型
4. `(左值)` 的类型为引用类型，但 `(&左值)` 的类型为指针类型
