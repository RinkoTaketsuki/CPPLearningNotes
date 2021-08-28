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
成员模板不能是虚函数  

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
template <typename T> void fobj(T, T);
template <typename T> void fref(const T&, const T&);
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
template <typename T> void fobj(T, T);
long n = 1;
fobj(10, n); // 错误，T 不知道是 int 还是 long
```

至于不涉及模板形参的形参，则执行正常的类型转换  

```C++
template <typename T> void fobj(int, T);
long m = 1, n = 2;
fobj(m, n); // 第一个 m 转化为 int，T 实例化为 long
```

## 13. 函数模板显式实参

函数模板的返回类型涉及模板形参时，其可能无法被推断出来，此时必须显式指定模板实参  
显式指定实参的顺序是从左到右，若指定的实参数量不够则右边部分由编译器推断  
已经指定了模板实参的函数实参执行正常的类型转换  

```C++
template <typename T> void fobj(T, T);
long n = 1;
fobj<int>(n, 20); // T = int，n 转化为 int
fobj<long>(n, 20); // T = long，20 转化为 long
```

## 14. 使用尾置返回类型指定返回类型  

显式实参无法解决所有的返回类型问题，如下例，函数形参为迭代器类型，需要返回迭代器指向的元素类型  
此时只能用尾置返回类型和 `decltype` 解决  

```C++
// 如果强行使用前置返回类型，则会导致 b 未定义，无法使用 decltype
template<typename It>
auto f(It b, It e) -> decltype(*b)
{
    return *b;
}

// 利用标准库的类型转换模板，实现返回拷贝
// remove_reference<Class&> = Class
// *b 是元素的引用类型
template<typename It>
auto f(It b, It e) -> remove_reference<decltype(*b)>::type
{
    return *b;
}
```

## 15. 标准类型转换模板

`Mod`|`T`|`Mod<T>::type`
:-|:-|:-
`remove_reference`|X& 或 X&&|X
-|其他 T|不变
`add_const`|X& 或 const X 或函数 X|不变
-|其他 T|const T
`add_lvalue_reference`|X&|不变
-|X&&|X&
-|其他 T|T&
`add_rvalue_reference`|X& 或 X&&|不变
-|其他 T|T&&
`remove_pointer`|X*|X
-|其他 T|不变
`add_pointer`|X& 或 X&&|X*
-|其他 T|T*
`make_signed`|unsigned X|X
-|其他 T|不变
`make_unsigned`|带符号类型 X|unsigned X
-|其他 T|不变
`remove_extent`|X[n]|X
-|其他 T|不变
`remove_all_extents`|X[n1][n2]...|X
-|其他 T|不变

## 16. 函数指针指向函数模板实例

给函数指针赋给函数模板时，根据函数指针的返回和形参类型实例化  
但如下情况会出问题  

```C++
template <typename T> int fref(const T&, const T&);
void func(int(*)(const int&, const int&));
void func(int(*)(const string&, const string&));
func(fref); // 错误，不知道 T 实例化为 int 还是 string
func(fref<int>); // 使用显式模板实参可以消除歧义
```

## 17. 实参是引用类型时的类型推断

> 引用折叠：当模板实参类型或类型别名的原类型为引用类型时，在其后面加引用符号会进行引用折叠  

`X& &`，`X& &&`，`X&& &` 都将被折叠为 `X&`
`X&& &&` 将被折叠为 `X&&`

```C++
int i; const int ci;
// f1 函数实参必须是左值
template <typename T> void f1(T&);
f1(i); // T = int
f1(ci); // T = const int
f1(10); // 错误

template <typename T> void f2(const T&);
f2(i); // T = int
f2(ci); // T = int
f2(10); // T = int

template <typename T> void f3(T&&);
f3(10); // T = int
f3(i); // T = int&，折叠后函数实参为 int&
f3(ci); // T = const int&，折叠后函数实参为 const int&
```

```C++
// 右值引用函数形参可能引发的错误
// 若 val 为 int 右值，则 T = int；若 val 为 int 左值，则 T = int&
template <typename T> void f(T&& val)
{
    // 该语句可能是拷贝初始化，也有可能是绑定引用
    T t = val;
    // 该语句可能影响 val
    t = foo(t);
    // 如果 t 是引用类型，则判断条件永远为 true
    if (val == t) {}
}
```

## 18. std::move 的原理

```C++
template <typename T>
typename remove_reference<T>::type&&
move(T &&t)
{
    return static_cast<typename remove_reference<T>::type&&>(t);
}
```

当传入右值时，T 实例化为对应类型，`static_cast` 什么也不做  
当传入左值时，函数形参类型为左值引用，`static_cast` 将其转化为右值引用  

## 19. 使用 std::forward 保持类型信息  

`std::forward` 是函数模板，必须显式实例化，其返回类型为在模板实参的基础上添加 `&&`，可能进行引用折叠  
如此，`std::forward` 可用于保持函数实参的引用和 `const` 属性  
调用时通常不省略 `std::` 以避免歧义  

```C++
int i;
void g(int &&rv, int &lv);
// 翻转函数
template <typename F, T1, T2>
void flip(F f, T1 &&t1, T2 &&t2)
{
    f(std::forward<T2>(t2), std::forward<T1>(t1));
}
/* 首先，F = void(*)(int&&, int&)，T1 = int&，T2 = int
 * 得到实例化后的结果 void flip(void(*)(int&&, int&) f, int &t1, int &&t2)
 * flip 传递实参：
 * void(*)(int&&, int&) f = g;
 * int &t1 = i;
 * int &&t2 = 42;
 * std::forward<T2>(t2) 返回 int&& 类型，是右值，而 t2 是左值
 * std::forward<T1>(t1) 返回 int& 类型，是左值
 * g 传递实参：
 * int &&rv = std::forward<T2>(t2)
 * int &lv = std::forward<T1>(t1)
 */
flip(g, i, 42);
```

## 20. 重载函数模板  

函数模板可以被函数模板或非模板函数重载  
所有可行的模板实例都会参与原本的匹配规则，此外当有多个函数提供同样好的匹配时还有一些规则：  

- 如果同样好的函数中只有一个是非模板函数，则选择此函数  
- 如果同样好的函数中没有非模板函数，且有一个模板比其他的更特例化，则选择该模板  
- 否则有歧义  

```C++
template <typename T> void fcn(const T& cr);
template <typename T> void fcn(T* p);
string s;
using strp = string*;
using cstrpr = const strp&;
/* 注意在指针类型前面加 const 的情况与一般类型不同
 * const strp 展开后是 string *const，而不是 const string*
 */
const strp pc = &s;
/* 注意课本上写的有问题
 * 如果选择第一个模板，&s 是 strp 类型，T 被实例化为 strp，
 * 函数实例为 void fcn(cstrpr cr)
 * 由于 cstrpr 中的底层 const 可以看作是加到 strp 上的顶层 const
 * 故函数实例展开为 void fcn(string *const &cr)
 * 即相当于 string *const 类型的引用
 * 如果选择第二个模板，&s 是 strp 类型，T 被实例化为 string
 * 函数实例为 void fcn(string *p)
 * 因此，当 &s 被传入这两个实例时，第一个会进行 string* -> string *const 的转换，而第二个不用
 * 故选择第二个模板
 */
fcn(&s);
/* 如果选择第一个模板，与上面同理，函数实例为
 * void fcn(const string *const &cr)
 * 如果选择第二个模板，与上面同理，函数实例为
 * void fcn(const string *cr)
 * 因此，当 cp 被传入这两个实例时，第一个会进行 const string* -> const string *const 的转换，而第二个不用
 * 故选择第二个模板
 */
const string *cp = &s;
fcn(cp);
/* 在类型别名中，套娃的 const 将被忽略
 * const cchar6& 展开为 const char(&)[6]，是对 char[6] 的常量引用
 */
using cchar6 = const char[6];
const cchar6 &r = "hello";
/* 如果选择第一个模板，则 T = char[6]（忽略套娃 const），函数实例化为 void fcn(const char (&cr)[6] cr)
 * 如果选择第二个模板，则 T = const char（数组到指针的转换），函数实例化为 void fcn(const char* p)
 * 因此，当 "hello" 被传入这两个实例时，第一个会进行 const char[6] -> char[6] 的转换，第二个会进行数组到指针的转换
 * 故选择第二个模板
 */
fcn("hello");
// 如果希望 fcn 把 C 字符串当成 string 处理，则可以加上这三个非模板函数，且第一个函数必须最先声明
// 后两个函数调用第一个函数，而不调用函数模板
void fcn(const string &cr);
void fcn(char *p)
{
    return fcn(string(p));
}
void fcn(const char *p)
{
    return fcn(string(p));
}
```

## 21. 可变参数模板

```C++
// Args：模板参数包，rest：函数参数包
template <typename T, typename... Args>
void foo(const T &t, const Args &...rest)
{
    // sizeof... 运算符返回包大小（unsigned long）
    // sizeof... 不会对运算对象求值
    sizeof...(Args);
    sizeof...(rest);
}
string s;
foo(0, s, 0, .0); // T = int, Args = {string, int, double}
foo(s, 0, "hello"); // T = string, Args = {int, char[6]}
foo("hi"); // T = char[3]，Args = {}
```

可变参数函数模板通常是递归的，下面为一个例子：  

```C++
// 终止递归并打印最后一个参数
template <typename T>
ostream &print(ostream &os, const T &t)
{
    return os << t;
}
// 打印除第一个参数外剩下的所有参数
template <typename T, typename... Args>
ostream &print(ostream &os, const T &t, const Args &...rest)
{
    os << t << ", ";
    return print(os, rest...)
}
```

```C++
// 更复杂的解包
template <typename T>
T& plus1(T& t) { return ++t; }
template <typename T, typename... Args>
ostream &print(ostream &os, const T &t, const Args &...rest)
{
    os << t << ", ";
    // 对 rest 的每一个元素执行 plus1 操作
    // ... 将 plus1(rest) 解包
    // 然后作为参数赋给 print
    // 注意不能写成 plus1(rest...)，否则就是调用一次 plus1，参数是 rest 中全部元素  
    return print(os, plus1(rest)...))
}
```

参数包可以转发，见 Chapter 5  

## 22. 模板特例化

有时定义的模板不适合所有类型，针对一些特殊类型做专门的定义  
用 `template <>` 表示特例化一个模板，此时函数形参要与原模板吻合  
函数模板特例化以后是原模板一个实例，而不是一个模板，不会影响原模板的函数重载  
但如果定义成非模板函数则会影响重载  
类模板特例化例子如下：  

```C++
// 特例化 hash，假设自定义类为 X，支持 == 运算符
// X 的成员为 A 类型的 a，B 类型的 b，C 类型的 c，都是标准库或内置类型
// 这样当无序容器存储 X 对象时就有默认哈希函数了
// 在 X 类中要把这个 hash<X> 声明为友元
namespace std
{
template <>
struct hash<X>
{
    typedef size_t result_type;
    typedef X argument_type;
    size_t operator()(const X&) const;
};

size_t hash<X>::operator()(const X &x) const
{
    return hash<A>()(a) ^ hash<B>()(b) ^ hash<C>()(c);
}
}
```

类模板的部分特例化，即指定一部分实例的特例：  

```C++
// remove_reference 的实现
template <class T>
struct remove_reference { typedef T type; };
template <class T>
struct remove_reference<T&> { typedef T type; };
template <class T>
struct remove_reference<T&&> { typedef T type; };
```

可以单独特例化类模板的成员函数：  

```C++
template <typename T>
struct Foo
{
    T mem;
public:
    Foo(const T &t): mem(t) {}
    void Bar()
    {
        /* 原模板的函数体内容 */
    }
};

template<>
void Foo::Bar<int>()
{
    /* Foo<int> 的函数体内容 */
}
```
