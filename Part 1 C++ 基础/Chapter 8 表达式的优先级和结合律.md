# Chapter 8 表达式的优先级和结合律

## 1. 运算符分类

- 一元运算符
- 二元运算符
- 三元运算符(?:)
- 函数调用运算符（不限运算对象数量）

## 2. 运算符重载

重载运算符时要定义运算对象的类型和返回值的类型，但是运算对象的个数，运算符的优先级和结合律无法改变

## 3. 左值表达式和右值表达式

> C语言中左值和右值可简单地理解为能放在赋值运算符左边的值和右边的值  
> C++中左值是可以使用其地址的值，其他的为右值

1. 需要右值的地方可以用左值，反之不行
2. 赋值运算符 = 左边必须是左值，其结果也是左值
3. 取地址符 & 右边必须是左值，其结果是右值
4. 内置解引用运算符 * 和内置下标运算符 [] ，迭代器解引用运算符，string和vector的下标运算符的结果是左值
5. 内置类型和迭代器的递增运算符 ++ 和递减运算符 -- 的左边必须是左值，结果也是左值
6. decltype()括号中的表达式是左值则会得到引用类型

## 4. 优先级和结合律

复合表达式（有多个运算符的表达式）需要考虑优先级和结合律  
分析一个复合表达式，首先按照优先级从高到低的顺序组合，优先级相同的运算符按照规定的左结合律或右结合律组合  
括号无视优先级和结合律，单独作为一个单元进行组合

## 5. 求值顺序

二元及以上运算符的求值顺序通常不确定（视编译器），下面列出确定求值顺序的运算符：

1. && 先求左边的值，若左边的值为true，则不计算右边的值直接返回true  
2. || 先求左边的值，若左边的值为false，则不计算右边的值直接返回false
3. ?: 先求?左边的值，若为true，则计算左边的值再返回，否则计算右边的值再返回  
4. , 先求左边的值，再求右边的值，返回右边的值

## 6. 运算符优先级表

|优先级/结合律|运算符|
|:-:|:-:|
|1/左|::|
|2/左|.（成员选择）|
|2/左|->（成员选择）|
|2/左|[]|
|2/左|()（函数调用，对象初始化）|
|3/右|++（后置）|
|3/右|--（后置）|
|3/右|typeid|
|3/右|新型强制类型转换|
|4/右|++（前置）|
|4/右|--（前置）|
|4/右|~|
|4/右|!|
|4/右|+（正号）|
|4/右|-（负号）|
|4/右|*（解引用）|
|4/右|&（取地址）|
|4/右|旧强制类型转换|
|4/右|sizeof|
|4/右|sizeof...|
|4/右|new|
|4/右|new[]|
|4/右|delete|
|4/右|delete[]|
|4/右|noexcept|
|5/左|->*|
|5/左|.*|
|6/左|*（乘法）|
|6/左|/（除法）|
|6/左|%|
|7/左|+（加法）|
|7/左|-（减法）|
|8/左|<<|
|8/左|>>|
|9/左|<|
|9/左|<=|
|9/左|>|
|9/左|>=|
|10/左|==|
|10/左|!=|
|11/左|&（位与）|
|12/左|^|
|13/左|\||
|14/左|&&|
|15/左|\|\||
|16/右|?:|
|17/右|=|
|17/右|+=|
|17/右|-=|
|17/右|*=|
|17/右|/=|
|17/右|&=|
|17/右|\|=|
|17/右|^=|
|17/右|<<=|
|17/右|>>=|
|18/右|throw|
|19/左|,|
