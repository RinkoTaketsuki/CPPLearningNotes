# Chapter 15 函数重载

> 函数名字相同但是形参列表不同时可以重载，但返回类型不能作为区分条件，main函数不能重载  
> 不同类名的同一个类型不能作为区分条件，顶层const不能作为区分条件  
> 底层const可以作为区分条件，当传入非底层const时优先匹配非底层const版本的函数

## 1. const_cast用于函数重载

```C++
// 利用const_cast构造返回非常量的版本
const string &shorterString(const string &s1, const string &s2)
{
    return s1.size() <= s2.size() ? s1 : s2;
}

string &shorterString(string &s1, string &s2)
{
    const string &r = shorterString(const_cast<const string&>(s1),
        const_cast<const string&>(s2));
    return const_cast<string&>(r);
}
```

## 2. 函数匹配（重载确定）

可能的三种结果：找到最佳匹配，无匹配，二义性调用（有多个可用版本但没有最佳的一个）

> 候选函数：声明可见的同名函数  
> 可行函数：候选函数中形参数量相等，类型兼容的部分  
> 最佳匹配：原则是实参类型与形参类型越接近越好，具体条件有两条  
>
> - 该函数每个实参的匹配都不劣于（<=）任何其他函数对应的实参的匹配
> - 至少有一个实参的匹配优于（>）任何其他函数对应的实参的匹配

### 实参类型转换等级

1. 精确匹配：完全相同、数组或函数类型转化成对应指针、添加或删除实参的顶层const
2. 通过const转换实现的匹配：添加底层const的转换
3. 通过类型提升实现的匹配
4. 通过算术类型转换或指针转换（隐式转成void*或父类指针）实现的匹配
5. 通过类类型转换实现的匹配

```C++
void f(short);
void f(int);
void g(long);
void g(float);
// h函数的形参换成指针类型效果类似
void h(int&);
void h(const int&);

int main()
{
    f('a'); // 调用short属于4，调用int属于3
    g(3.14); // 调用long或float均属于4，具有二义性
    const int a = 0;
    int b = 0;
    h(a); // 只能调用const int，否则底层const被删除
    h(b); // 调用int，非底层const优先（精确匹配）
    return 0;
}
```

## 3. 重载与作用域问题

```C++
void f(double);
void f(const string&);
void g();
void foo()
{
    int g = 0; // 外层的函数g被隐藏
    void f(int); // 上面的两个f函数均被隐藏
    f("hello"); // 报错
    f(3.14) // 不报错，但是调用的是int版本的f函数
}
```
