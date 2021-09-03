# Chapter 11 enum

枚举类型将一些整型常量组织在一起形成新的类型，枚举类型本身属于 `constexpr` 类型  
如果枚举类型未命名，则与 `class` 和 `struct` 类似，必须在定义类型时定义对象，在花括号后面，分号前面写对象名列表  

## 1. 枚举成员

限定作用域的枚举类型使用 `enum class` 或 `enum struct` 定义  
不限定作用域的枚举类型使用 `enum` 定义  
限定作用域的枚举类型成员名字只在枚举类型内部可见  
不限定作用域的枚举类型成员名字的作用域与枚举类型所在的作用域相同  

```C++
enum color {red, yellow, green};
enum stoplight {red, yellow, green}; // 错误，不限定作用域的各个 enum 不能有同名成员
enum class peppers {red, yellow, green}; // 正确，不限定作用域的成员名字被隐藏
color eyes = green; // 正确
peppers p = green; // 错误，将 color 类型的值赋给 peppers 类型变量
color hair = color::red; // 正确，显式访问成员
peppers p2 = peppers::red; // 正确
color apple = 2; // 错误，2 不是枚举成员
int i = color::red; // 正确，不限定作用域的枚举成员能隐式转化为 int 或更大的整型
int j = pepper::red; // 错误，限定作用域的枚举类型不会进行隐式转换
```

默认情况下，枚举值从 0 开始，依次加 1  
可以显式指定枚举值，未显式指定枚举值时，其枚举值为上一个枚举值加 1  
枚举成员是 `constexpr`，指定枚举值时必须使用 `constexpr`  
因此，枚举类型的使用场景可以是：  

- 将 `switch` 语句的表达式设为 `enum` 对象，将其枚举值作为 `case` 标签
- 将枚举类型作为类型模板实参，或非类型模板形参的类型
- 将枚举类型作为类的静态数据成员类型并在类内初始化

若有两个重载函数，一个函数只有一个 `enum` 形参，另一个函数只有整型形参，当传入 `enum` 对象时，优先选择 `enum` 形参版本  

## 2. 指定 enum 的成员类型

限定作用域的默认使用 `int`  
不限定作用域的无默认类型，只保证足够大以容纳枚举值  
当枚举类型对象提升为整型时，至少会提升为 `int`，与 `enum` 的成员类型无关  

```C++
enum intValues: unsigned long long
{
    charTyp = 255,
    shortTyp = 65535,
    intTyp = 4294967295UL,
    longTyp = 4294967295UL,
    long_longTyp = 18446744073709551615ULL
};
```

## 3. enum 前置声明

不限定作用域的枚举类型的前置声明必须指定成员类型  
限定作用域的枚举类型的前置声明可以隐式指定为 `int`  
声明和定义必须匹配，包括是否限定作用域一致，成员类型一致  
