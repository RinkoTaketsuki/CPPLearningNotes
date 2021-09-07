# Chapter 18 不可移植特性

> 此章节中的特性在使用时需注意其不可移植特性，即如果在多种机器上编译和运行时需要重构代码  

## 1. 位域

类可将其非静态数据成员设置为位域  
位域必须是整型或枚举类型，通常使用无符号类型，不同机器的有符号类型位域的行为不同  

```C++
// Example

typedef unsigned int Bit;
class File
{
    // The number following the colon represents the numbers of bits
    // occupied by such bit-field.
    Bit mode: 2;
    Bit modified: 1;
    Bit prot_owner: 3;
    Bit prot_group: 3;
    Bit prot_world: 3;
public:
    enum modes { READ = 01, WRITE = 02, EXECUTE = 03 };
    File &open(modes);
    void close();
    void write();
    bool isRead() const;
    void setWrite();
};

void File::write()
{
    modified = 1;
    /* other statements */
}

void File::close()
{
    if (modified)
        /* save modified contents */
}

File &File::open(File::modes m)
{
    mode |= READ; // set default open mode: READ

    /* process other open modes */

    if (m & WRITE)
        /* open current file as r/w mode */
    return *this;
}

inline bool File::isRead() const { return mode & READ; }
inline void File::setWrite() { mode |= WRITE; }
```

如果可能的话，类内部连续定义的位域会被压缩在同一个整数的相邻位，具体能否压缩以及如何压缩与机器相关  
取地址运算符不能作用于位域，任何指针也无法指向位域  

## 2. volatile 限定符

当对象的值可能在程序控制或检测之外被改变时，应将对象声明为 `volatile`，告诉编译器不对该对象进行优化  
类似 `const` 限定符：  

- 成员函数也可以是 `volatile` 的，`volatile` 对象只能调用 `volatile` 的成员函数
- `volatile` 指针分为顶层和底层 `volatile`，只有底层 `volatile` 指针和 `volatile` 引用才能指向 `volatile` 对象

合成的拷贝/移动构造函数和赋值运算符对 `volatile` 对象无效，若要使用必须自定义  

```C++
class Foo
{
public:
    Foo(const volatile Foo&);
    // 赋值给非 volatile 对象
    Foo &operator=(const volatile Foo&);
    // 赋值给 volatile 对象
    Foo &operator=(const volatile Foo&) volatile;
};
```

## 3. 链接指示 extern "C"

> 链接指示用于指出非 C++ 语言函数
> 把 C++ 代码和其他语言代码放在一起使用时，要求有权访问对应语言的编译器，且该编译器与当前 C++ 编译器兼容  

### 注意事项

不能出现在类或函数定义的内部，相同的链接指示必须在每次声明中都出现  

```C++
// 可能出现在 <cstring> 中的链接指示
// 单语句链接指示
extern "C" size_t strlen(const char*);
// 复合语句链接指示
extern "C"
{
    int strcmp(const char*, const char*);
    char *strcat(char*, const char*);
}
```

```C++
// 链接指示可应用于整个头文件，且链接指示可以嵌套
extern "C"
{
    #include <string.h>
}
```

编写函数所用的语言是函数类型的一部分，因此函数指针也要指定对应的链接指示  

```C++
// pf1 和 pf2 的类型不同
void (*pf1)(int);
extern "C" void (*pf2)(int);
pf1 = pf2; // 视编译器而定，严格来说是非法的
```

链接指示对整个声明都有效，包括作为返回类型和形参类型的函数指针  

```C++
extern "C" void f1(void (*)(int)); // 调用 f1 时必须传入 C 函数或其指针

// 给 C++ 函数传入 C 函数的方法：必须使用类型别名
extern "C" typedef void FC(int);
void f2(FC*);
```

声明了 `extern "C"` 的函数可以被 C 语言代码调用，但要求函数内不能有 C++ 特有的东西  

### 使用预处理器使一个文件可以同时编译为 C 和 C++ 语言

```C++
#ifdef __cplusplus // 若此宏被定义，则说明要被编译为 C++
extern "C"
#endif
int strcmp(const char*, const char*);
```

### 链接提示与函数重载

C 语言不支持函数重载，链接提示只能用于一组重载函数中的某一个，且一组重载函数中如果有一个是 C 函数，则其他的必须是 C++ 函数  
使用类形参和返回类型的函数原则上只能是 C++ 函数，不过有些编译器仍支持 C 函数使用类形参和返回类型  

```C++
extern "C" void print(const char*);
extern "C" void print(int); // 错误
```
