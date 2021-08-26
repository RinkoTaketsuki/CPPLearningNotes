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

用户需要主动给出模板参数才能使用  

```C++
// 类外定义成员函数的写法
template <typename T>
class A
{
    // 类模板的作用域内使用当前类时可以不提供模板实参
    // 即此处可以不写成 A<T> &
    A &operator++(int);
    T &foo(T*);
};

template <typename T>
T &A<T>::foo(T *p)
{
    return *p;
};

// 此处因为没在类作用域内，所以要写 A<T>
template <typename T>
A<T> &A<T>::operator++(int)
{
    // 此处已在作用域内，可以写成 A
    A ret = *this;
    ++*this;
    return ret;
}
```

类实例化之后，成员函数只有被调用时才会实例化，因此即便成员函数中对模板形参的要求不符合实际传递的实参，也能实例化类  

## 3. 模板实例友元

指定类模板或函数模板为友元时，可以指定实参类型，也可以设定全部实例为友元  

```C++
template <typename T>
class B
{
    friend T; // 可以指定 T 为友元
    friend class A<int>; // 指定用 B 实例化的模板为友元
    friend void bar<T>(); // 指定相同类型实例为友元 
    // 这里可以在 typename 后面写非 B 的模板形参列表中的类型，效果相同
    template <typename> friend void foo(); // 指定所有实例为友元
};
```

## 4. 模板类型别名

```C++
// 对类模板实例使用别名
typedef std::vector<char> Str; 
// 对类模板使用别名
template <typename T> using twins = std::pair<T, T>;
// 等价于 std::pair<int, int> t;
twins<int> t;
// 类模板别名的另一种用法
template <typename T> using kv = std::pair<std::string, T>;
// 等价于 std::pair<std::string, double> k;
kv<double> k;
```

## 5. 类模板静态成员

每个实例共享一个静态成员  
静态成员函数也是在被调用时才会实例化  

## 6. 模板参数的作用域

模板参数不能被重用，因此在模板形参列表中不能定义相同的名字  

```C++
template <typename T> void foo()
{
    double T; // 错误，重用了模板参数
}
```

## 7. 模板声明

```C++
// 模板声明的写法
template <typename> class A;
template <typename T> bool operator==(const A<T>&, const A<T>&);
// 具体定义时，模板形参名不必和声明时相同
```

## 8. 模板形参的类型成员

通常情况下，使用作用域运算符时，编译器可根据前面类或命名空间的定义来推断后面的是成员还是类型  
但如果是模板形参，则编译器无法判断其类型，会默认作用域运算符后面的是非类型成员  
因此如果要使用模板形参的类型成员，必须加 `typename` 关键字，注意此处不能使用 `class` 关键字  

```C++
template <typename T>
typename T::value_type at(const T &cont, typename T::size_type index)
{
    if (index >= cont.size())
        return typename T::value_type()
    else
        return cont[index];
}

template <typename T>
class A
{
    // 类型成员定义
    typedef typename T::iterator iter_t;
};
```

## 9. 默认模板实参

函数模板和类模板均可使用  

```C++
// 泛用性更强的 compare
template <typename T, template F = less<T> >
int compare(const T &v1, const T &v2, F f = F())
{
    if (f(v1, v2)) return -1;
    if (f(v2, v1)) return 1;
}
```

## 10. 成员模板

非模板类的成员函数模板和通常函数模板用法相同，使用时推断模板实参  
类模板的成员模板在外部定义时，必须同时提供类模板和成员模板的形参列表  

```C++
template <typename T> class A
{
    // 用迭代器范围初始化
    template<typename It>
    A(It, It);
};

template <typename T>
template <typename It>
A<T>::A(It b, It e): /* 初始化列表 */
{
    /* 函数体 */
}
```

## 11. 控制实例化

假设模板定义在文件 H 中，有 A，B，C 三个文件都使用了 H 中的模板，且模板实参相同，则同一种实例化被重复了三次  
下面的例子为只实例化一次的解决方案  

```C++
// temp.h
template <typename> class A {}

// templateBuild.cpp
// 显式实例化定义
template class A<int>;

// 以下文件使用显式实例化声明
// a.cpp
extern template class A<int>;

// b.cpp
extern template class A<int>;

// c.cpp
extern template class A<int>;
```

显式实例化会实例化所有成员，所以必须保证实参能实例化所有成员函数

## 12. 类型推断

函数模板在实例化时，通常用实参原本的类型来实例化，除了以下情况：  

- 顶层 `const` 在函数形参和实参中都会被忽略  
- `const` 引用或指针的形参接受非 `const` 的引用或指针的实参  
- 函数指针形参接受函数类型的实参  
- 数组实参转化为指向首元素指针的形参  

算术转换、派生类向基类的转换、自定义的转换不会应用于函数模板  

```C++
template <typename T> fobj(T, T);
template <typename T> fref(const T&, const T&);
string s1;
const string s2;
fobj(s1, s2); // T = string，const 可以忽略
fref(s1, s2); // T = string，const string& 可以接受 string&
int a[10], b[20];
fobj(a, b); // T = int*
fref(a, b); // 错误
```

若某函数模板有多个形参使用相同的模板形参，则实例化时必须使用能推断为一致类型的实参  

```C++
template <typename T> fobj(T, T);
long n = 1;
fobj(10, n); // 错误，T 不知道是 int 还是 long
```

至于不涉及模板形参的形参，则执行正常的类型转换  

```C++
template <typename T> fobj(int, T);
long m = 1, n = 2;
fobj(m, n); // 第一个 m 转化为 int，T 实例化为 long
```

## 13. 函数模板显式实参

函数模板的返回类型涉及模板形参时，其可能无法被推断出来，此时必须显式指定模板实参  
