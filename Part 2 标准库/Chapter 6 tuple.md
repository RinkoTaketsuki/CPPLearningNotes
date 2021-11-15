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

## 2. tuple 的赋值操作

按照从左到右的顺序依次赋值  

```C++
// 原理说明（伪代码）
template<typename T1, T2, ..., Tn>
tuple<T1, T2, ..., Tn>& tuple<T1, T2, ..., Tn>::operator=(const tuple<T1, T2, ..., Tn>& rhs)
{
    for (size_t i = 0; i < n; ++i)
        get<i>(*this) = get<i>(rhs);
    return *this
}
```

```C++
int a = 1, b = 2, c = 3;
// 此处的 tie 函数返回 tuple<int&, int&, int&> 类型
// 而右边的 tuple 存储的均为右值，故赋值结果为 a == 2, b == 3, c == 1
tie(a, b, c) = make_tuple(b, c, a);

class ListNode
{
    int m;
    ListNode *next;
};

// 链表 1-->--2-->--3-->--4
ListNode head = new ListNode{1, new ListNode{2, new ListNode{3, new ListNode{4, nullptr}}}};
// 执行赋值操作后：1-->--3-->--4-->--2
// 赋值运算符右边部分假设为 {0x1000300, 0x1000400, 0x1000200} （分别指向结点 3, 4, 2）
// 则 head->next 最后指向 3，head->next->next 最后指向 4，head->next->next->next 最后指向 2
tie(head->next, head->next->next, head->next->next->next) = make_tuple(head->next->next, head->next->next->next, head->next);
```
