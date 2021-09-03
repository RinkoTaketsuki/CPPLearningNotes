# Chapter 10 RTTI

> RTTI：运行时类型识别  

## 1. dynamic_cast 运算符

```C++
// e 必须是有效指针
dynamic_cast<type*>(e);
// e 必须是左值
dynamic_cast<type&>(e);
// e 必须是右值
dynamic_cast<type&&>(e);

/* e 的基本类型只能是如下几种：
 * 1. type
 * 2. type 的 public 派生类
 * 3. type 的 public 直接或间接基类
 * 若不符合，则第一种转换会返回 0
 * 第二种和第三种转换将抛出 bad_cast 异常
 */
```

```C++
// 指针类型的 dynamic_cast 使用例
class B {};
class D: public B {};

B* bp;

// 初始化语句写在条件判断处以防外部使用转换失败的 dp
if (D *dp = dynamic_cast<D*>(bp))
{
    /* 转换成功，处理 dp */
}
else
{
    /* 转换失败，处理 bp */
}
```

```C++
// 引用类型的 dynamic_cast 使用例
class B {};
class D: public B {};

void f(const B& b)
{
    try
    {
        const D &d = dynamic_cast<D&>(b);
        /* 转换成功，使用 d */
    }
    catch(bad_cast)
    {
        /* 转换失败，使用 b */
    }
}
```

## 2. typeid 运算符

运算对象可以是任意对象或类型名，返回 `type_info` 或其公有派生类的常量引用  
会忽略运算对象的顶层 const，如果是引用类型则返回所引对象的类型，但不会进行数组或函数向对应指针的转换  
若运算对象的类定义了虚函数，则返回动态类型，且会对表达式求值，反之则返回静态类型，不对表达式求值  

```C++
// 使用例
D *dp = new D;
B *bp = dp;
if (typeid(*dp) == typeid(*bp))
{
    /* 此条件表示二者指向的对象类型相同 */
}
if (typeid(*bp) == typeid(D))
{
    /* 此条件检查 bp 是否指向派生类对象 */
}

/* 若 bp 为 nullptr，则因为会对表达式求值，抛出 bad_typeid 异常 */
```

```C++
// 使用例2，定义派生类的比较函数
class Base
{
    friend bool operator==(const Base&, const Base&);
protected:
    virtual bool equal(const Base&) const;
};

class Derived: public Base
{
protected:
    virtual bool equal(const Base&) const;
}

bool operator==(const Base &lhs, const Base &rhs)
{
    // 先检查类型是否相同，然后虚调用 equal
    return typeid(lhs) == typeid(rhs) && lhs.equal(rhs);
}

bool Derived::equal(const Base &rhs) const
{
    auto r = dynamic_cast<B&>(rhs);
    // 之后比较 *this 和 r 的各个成员
}

bool Base::equal(const Base &rhs) const
{
    // 比较 *this 和 rhs 的各个成员
}
```

## 3. type_info 类型

```C++
// t1, t2 为 type_info 对象

// 二者表示相同类型时相等
t1 == t2; t1 != t2;

// 返回 C 字符串，表示类型名字，名字格式因系统而异
t.name();

// 返回 bool，表示 t1 是否在 t2 之前，排序因系统而异
t1.before(t2);
```
