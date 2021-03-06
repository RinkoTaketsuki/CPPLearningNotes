# Chapter 10 类型转换

## 1. 隐式转换概述

> 原则：尽可能少地损失精度，如整数一般转换成浮点数

### 何时发生隐式转换

1. 大多数表达式会将比int小的整型转化为int
2. 需要条件的地方会转化为布尔型
3. 初始化和赋值语句中右边的类型转化为左边的类型
4. 算术运算和关系运算会把运算对象的类型统一
5. 函数调用

## 2. 算术转换

1. 大致可以认为运算符的运算对象转化成最宽的对象对应的类型
2. 整型提升：通常情况下，bool/char/signed char/unsigned char/short/unsigned short类型会自动转化为int型，若其容纳的值超过了int的范围，则使用unsigned int；wchar_t/char16_t/char32_t提升为int/unsigned/long/unsigned long/long long/unsigned long long中能包含小类型的所有值的最小的一个
3. 执行完整型提升后，若运算符的运算对象类型还不一致，则分如下情况讨论  
    1. 二者均为带符号类型或无符号类型，则小的转化成大的
    2. 二者一个带符号，一个无符号；若无符号的宽度 >= 有符号的宽度，则转化为无符号对应的类型，否则，若无符号对象的所有值都能存在有符号类型对象中，则转化为有符号对应的类型，否则转化为无符号对应的类型
4. 整型和浮点型混用时均转化为浮点型
5. 整型转化为浮点型：2 -> 3 -> 4
6. 浮点型转化为整型：切除小数部分
7. 任意类型转化为布尔类型：看是否为0

## 3. 数组、指针和引用的隐式转换

1. 数组类型变量赋给指针类型变量时自动转化成首元素指针类型，但数组变量作为decltype、取地址符、sizeof、typeid等运算符的运算对象时，其不会发生转换
2. 0和nullptr可以为任意指针类型，指向非常量的指针可以转换为void*，指向任意对象的指针可以转换为const void*
3. 指向非常量的指针可转化为指向常量的指针，反过来不行
4. 指向非常量的引用可转化为指向常量的引用，反过来不行

## 4. 类类型转换举例

1. C字符串转换为string
2. iostream转化为bool

## 5. 强制类型转换概述

> `cast-name<type>(expression)`  
> `(type-name)expression` 旧式  
> `type-name(expression)` 旧式

尽量避免使用

## 6. static_cast

1. 不能删除底层 `const`，会报错
2. 可以用于窄化，不会警告
3. 可以用于将 `void*` 强转为指定的指针类型
4. 不支持隐式转换不支持的指针类型转化

## 7. const_cast

1. 只有这种 cast 能用来删除底层 `const`，但之后再写入值会出现问题
2. 不能处理非常量类型
3. 一般只有函数重载时要用，其他时候不会有使用的必要

    ```C++
    // 正确用法举例
    class A
    {
        char *str = "";
    
    public:
        const char &operator[](size_t i) const
        {
            return str[i];
        }
        char &operator[](size_t i)
        {
            // 不要使用 const_cast 修改 this
            // 也正因此，不能使用非 const 成员函数实现 const 的版本
            return const_cast<char &>(static_cast<const A &>(*this)[i]);
        }
    };
    ```

## 8. reinterpret_cast

直接把变量在内存中的二进制值重新解释

## 9. 旧式转换

首先尝试static_cast和const_cast，如果都非法，则使用reinterpret_cast
