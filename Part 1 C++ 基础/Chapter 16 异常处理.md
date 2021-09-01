# Chapter 16 异常处理

## 1. try 语句

```C++
try
{
    /* program-statements */
}
catch(/* exception declaration1 */)
{
    /* handler-statements1 */
}
catch(/* exception declaration2 */)
{
    /* handler-statements2 */
}
catch(/* exception declaration3 */)
{
    /* handler-statements3 */
}
...
```

1. program-statements 部分声明和定义的变量在外部无法访问，在 `catch` 子句内也无法访问
2. 异常处理时从里往外寻找 `catch` 子句，若最终找不到 `catch` 则会调用 `std::terminal()` 函数终止程序运行
3. 栈展开过程中，如果正处于某语句块内，抛出的异常导致程序退出该语句块，则该语句块内的局部对象会被销毁
4. 栈展开过程中，如果处于构造函数内，则已经初始化的对象也会被销毁
5. 栈展开过程中，如果正在初始化数组或容器，则已经初始化的数组或容器中的对象也会被销毁
6. 以上所说的销毁针对类类型指调用析构函数
7. 析构函数必须保证其抛出的异常能被自己处理，否则析构函数在异常发生位置后的释放资源语句无法执行
8. 若析构函数抛出异常导致析构函数无法继续执行，则会直接调用 `std::terminal()`

## 2. 异常对象

编译器使用异常抛出表达式对异常对象进行拷贝初始化，因此：  

- 该表达式必须是完全类型
- 该表达式如果是类类型，则必须有可访问的析构函数和可访问的拷贝或移动构造函数
- 该表达式如果是数组或函数类型，则自动转换为对应的指针类型
- 该表达式不能是指向局部对象的指针，因为指向的对象可能被销毁

异常对象在异常被处理完毕时被销毁  
`throw` 的异常对象的类型与静态类型一致，如果 `throw` 一个基类指针的解引用，则如果对象是派生类对象，其派生部分将被切除  

## 3. catch 子句

声明部分类似函数形参，类型必须是完全类型，可以是左值引用，不能是右值引用，如果不使用形参可以不写形参名  
使用捕获的异常对象初始化形参，如果形参不是引用则拷贝初始化  
如果形参是基类非引用，则用派生类初始化时派生部分被切除，如果是基类引用则派生部分不被切除，但不能访问派生类特有成员  
通常把更专门的 `catch` 子句放在顶端，派生类异常的处理代码应出现在基类异常的处理代码之前  
`catch` 子句不允许大多数类型转换，除了以下几种：  

- 非常量 -> 常量
- 派生类 -> 基类
- 数组或函数 -> 对应指针

### 空 throw

将异常重新抛出给上一级的 `catch` 子句，只能出现在 `catch` 代码块中，若出现在别的地方则直接调用 `terminate`  
只有当前的 `catch` 子句以引用方式接受异常对象时，对异常对象的更改才能被应用到重新抛出中  

### 捕获所有异常

将 `catch` 子句的参数设为 `...` 即可，必须是最后一个 `catch` 子句，否则在其下方的 `catch` 子句不会执行  

## 4. 函数 try 语句块

构造函数初始值列表抛出的异常无法被构造函数体内的 `catch` 子句捕获，必须使用函数 `try` 语句块  

```C++
class A
{
    int mem1;
    string mem2;
public:
    A(int n, const string &s) try : mem1(n), mem2(s)
    {
        /* 构造函数体 */
    }
    catch (/* 异常声明 */)
    {
        /* 异常处理 */
    }
};
```

须注意的是，构造函数的形参初始化过程中抛出的异常属于上一级的调用表达式抛出的异常，由调用者处理  

## 5. noexcept

指定函数不会抛出异常，有助于优化性能  
该声明紧跟在参数列表后面，在后置返回类型之前，函数的声明和定义必须同时声明或不声明 `noexcept`  
如果是成员函数，则其位置在 `const` 和引用限定符之后，在 `= 0`，`final` 和 `override` 之前  
函数指针的声明和定义也可以声明 `noexcept`，但此时该指针只能指向 `noexcept` 函数  
如果 `noexcept` 函数违反约定抛出异常，则会直接调用 `terminate`  
若基类的虚函数声明了 `noexcept`，则派生类派生出来的虚函数也必须声明 `noexcept`  
合成拷贝控制成员调用的某个函数如果不是 `noexcept` 的，则合成不是 `noexcept` 的版本，否则合成版本是 `noexcept` 的  
如果未定义 `noexcept` 或 `noexcept(false)` 的析构函数，则会视情况合成一个 `noexcept` 或 `noexcept(false)` 的析构函数  

### noexcept 实参

接受一个 `bool` 实参，如果实参是 `true` 则表示不会抛出异常，反之表示可能抛出异常（相当于不声明 `noexcept`）  

### noexcept 运算符

运算对象为 `函数(实参)`，返回 `constexpr bool`，表示函数是否会抛出异常  
若函数声明为 `noexcept`，则返回 `true`  
若函数调用的所有函数都是 `noexcept` 的且自身无 `throw` 语句时，返回 `true`  
该运算符不会执行运算对象  

```C++
// 令 f 和 g 的异常说明一致
void f() noexcept(noexcept(g()));
```

## 6. 异常类

> 在此只讨论 `stdexcept` 头文件中定义的异常类

类名|说明
:-|:-
`exception`|异常类
--> `bad_cast`|
--> `bad_alloc`|
--> `runtime_error`|运行时错误类
------> `range_error`|
------> `overflow_error`|
------> `underflow_error`|
--> `logic_error`|逻辑错误类
------> `domain_error`|
------> `invalid_argument`|
------> `length_error`|
------> `out_of_range`|

有默认构造函数：`exception`，`bad_cast`，`bad_alloc`  
其他类必须用 `string` 或 C 字符串初始化  

`exception` 有拷贝构造函数，拷贝赋值运算符，虚析构函数和虚 `what()`，两个虚函数具有多态性  

## 7. 自定义异常类

```C++
// 常见自定义方法举例

class my_error: public std::runtime_error
{
public:
    explicit my_error(const std::string &s):
        std::runtime_error(s);
};
```
