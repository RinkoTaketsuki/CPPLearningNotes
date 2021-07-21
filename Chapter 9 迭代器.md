# Chapter 9 迭代器

## 1. 有迭代器类型的成员函数begin()和end()

```C++
// v是某有迭代器的类型的对象
auto b = v.begin(), e = v.end();
// b和e是迭代器类型，具体类型定义与v有关
// b指向第一个元素，e指向最后一个元素的后面位置
// 当v为空时，b==e
```

## 2. （通用）迭代器类型的运算

```C++
iterator_t::iterator iter, iter1, iter2;
*iter; // 返回iter所指元素的引用
iter->member; // 等价于(*iter).member；须注意成员运算优先级高于解引用运算
++iter; // 令iter指向下一个元素
--iter; // 令iter指向上一个元素
iter1 == iter2; // bool
iter1 != iter2; // bool
```

## 3. const_iterator类型

该类迭代器只能读不能写，如果要迭代的对象是常量，则必须使用const_iterator  
常量的begin和end方法返回的也是const_iterator类型  
非常量可以使用cbegin和cend方法返回const_iterator类型迭代器

## 4. string和vector迭代器额外支持的运算

```C++
string::iterator iter, iter1, iter2;
iter + n; // n为ptrdiff_t （可正可负）（g++是long）
iter - n; // n为ptrdiff_t
iter += n;
iter -= n;
iter1 - iter2; // ptrdiff_t
// 还有大小比较运算符
```
