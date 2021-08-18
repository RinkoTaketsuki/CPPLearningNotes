# Chapter 4 运算符重载

> 名为 __operator__ + 运算符号 的成员函数

## 1. 赋值运算符

```C++
class A
{
    A& operator=(const A&);
};
```

### 合成拷贝赋值运算符

未声明赋值运算符重载函数或声明为default时会合成  
和合成拷贝构造函数的行为类似
