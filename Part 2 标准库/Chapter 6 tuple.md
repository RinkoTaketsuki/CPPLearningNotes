# Chapter 6 tuple

> 可以将一些不同类型的数据组合成一个单一的对象，比如用于函数返回多个值  

## 1. 支持的操作

```C++
tuple<T1, T2, ..., Tn> t; // 每个成员进行值初始化
tuple<T1, T2, ..., Tn> t(v1, v2, ..., vn); // 为每个成员提供拷贝初始化值，该构造函数是 explicit 的，小括号可以替换成花括号或等号 + 花括号
make_tuple(v1, v2, ..., vn); // 将实参组成一个 tuple 并返回，自动推断类型，返回 constexpr tuple 类型

// 完全相等时才相等，否则不等
t1 == t2;
t1 != t2;

// 大小比较运算符只有在元素数量相等时才能比较，使用 < 运算符比较

get<i>(t); // 返回 t 的第 i - 1 个成员的引用，可能是左值引用也可能是右值引用

tuple_size<tuple<T1, T2, ..., Tn> >::value; // value 是公有静态 size_t 成员，表示成员数量
tuple_element<i, tuple<T1, T2, ..., Tn> >::type // i 是 size_t 类型，type 表示第 i 个成员的类型
```
