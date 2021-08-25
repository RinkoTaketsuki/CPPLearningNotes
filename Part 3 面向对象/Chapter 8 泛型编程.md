# Chapter 8 泛型编程

> 编译器需要掌握模板的定义才能实例化一个模板，因此模板的声明和定义放在同一个头文件中，包括类模板的成员函数的定义  

## 1. 函数模板

```C++
template </*参数列表*/>
```

参数列表包括：  

- 类型参数：用 `typename` 或 `class` 表示，编译时根据实参推断类型，实例化一个具体的函数  
- 非类型参数：类型名 + 参数名，实例化时会被用户提供的值或推断的值代替，因为是在编译时实例化，所以这些值必须是 `constexpr`  

```C++
// 非类型参数使用举例：比较两个字符数组长度
template <unsigned M, unsigned N>
int compare(const char (&p1)[M], const char (&p2)[N])
{
    return strcmp(p1, p2);
}
// 该实例化过程中 M 和 N 分别为 3 和 4
compare("hi", "mom");
```

如果非类型参数的类型是指针，则必须用 `nullptr` 或指向静态生存期对象的指针实例化  
如果非类型参数的类型是左值引用，则必须用静态生存期左值对象实例化  
动态分配的对象不能用来实例化  
函数模板可以声明为 `constexpr` 和 `inline`  
模板应尽量减少对实参类型的要求，举例如下：  

```C++
template <typename T>
// 使用 const T& 避免了 T 不支持拷贝的情况
int compare(const T &v1, const T &v2)
{
    // 不使用大于运算符或 greater<T>，避免不支持大于运算
    // 不使用小于运算符，而使用标准库 less 类模板以支持指针的比较
    if (less<T>(v1, v2)) return -1;
    if (less<T>(v2, v1)) return 1;
}
```

## 2. 类模板
