# Chapter 4 运算符重载

> 名为 `operator` + 运算符号 的成员函数  
> 除 `operator()` 外，均不能有默认实参  
> 运算符的第一个运算对象是 *this  
> 内置类型的运算符不能重载  
> 运算符的优先级和结合律不能改变  
> `::`, `.`, `.*`, `?:` 运算符不能重载  
> 可以直接调用 `operator` 函数，非成员函数和成员函数均可  
> 运算符被重载后，其求值顺序和短路求值特性不会保留  
> 一般不重载逗号、取地址、逻辑与和逻辑或运算符  

运算符重载为成员或非成员的选择原则：  

- 赋值、下标、调用、成员访问箭头必须是成员  
- 复合赋值一般是成员  
- 改变运算对象状态或很有可能导致状态改变的运算符一般是成员，如递增递减、解引用
- 具有对称性的运算符一般是非成员，如算术运算、关系比较、位运算

举个例子，如果 `+` 运算符是 `string` 类的成员，则显然左边的对象必须是 `string`  
反之，如果定义为 `operator+(const string&, const string&);`，则左右两边的对象只要能转化成 `string` 即可  
  
本章举例使用的 Sales_data 类声明如下：

```C++
class Sales_item 
{
    friend std::istream& operator>>(std::istream&, Sales_item&);
    friend std::ostream& operator<<(std::ostream&, const Sales_item&);
    friend bool operator<(const Sales_item&, const Sales_item&);
    friend bool operator==(const Sales_item&, const Sales_item&);
public:
    // default constructor needed to initialize members of built-in type
    Sales_item(): units_sold(0), revenue(0.0) { }
    Sales_item(const std::string &book): 
                  bookNo(book), units_sold(0), revenue(0.0) { }
    Sales_item(std::istream &is) { is >> *this; }
public:
    // operations on Sales_item objects
    // member binary operator: left-hand operand bound to implicit this pointer
    Sales_item& operator+=(const Sales_item&);
    
    // operations on Sales_item objects
    std::string isbn() const { return bookNo; }
    double avg_price() const;
// private members as before
private:
    std::string bookNo;      // implicitly initialized to the empty string
    unsigned units_sold;
    double revenue;
};
```

## 1. 拷贝赋值运算符

```C++
class A
{
    A &operator=(const A&);
};
```

### 合成拷贝赋值运算符

未声明赋值运算符重载函数或声明为default时会合成  
和合成拷贝构造函数的行为类似  
若定义了移动构造函数和和移动赋值运算符，则合成拷贝赋值运算符会被删除  

## 2. 移动赋值运算符

```C++
class A
{
    A &operator=(A&& rhs) noexcept
    {
        // 检查自赋值，因为 rhs 可能是 std::move(当前对象)
        if (this != rhs)
        {
            // 此处写释放 this->mem 的资源的代码
            mem = rhs.mem;
            rhs.mem = nullptr; // 防止 rhs.mem 的资源被释放
        }
        return *this;
    }
    int *mem;
}
```

### 合成移动赋值运算符

只有未定义拷贝构造函数、拷贝赋值运算符和析构函数，且每个非 `static` 成员均支持移动操作时才会自动合成  
若显式声明为 `=default`，且不是所有非 `static` 成员均可移动，则移动操作会被删除  
若有成员定义了拷贝赋值运算符且未定义移动赋值运算符，则移动操作会隐式删除  
若有成员未定义拷贝赋值运算符但无法合成移动赋值运算符，则移动操作会隐式删除  
若有成员的移动赋值运算符是删除的或不可访问的，则移动操作会隐式删除  
若有顶层 `const` 或引用成员，则移动操作会隐式删除  

## 3. 输出运算符

必须是非成员函数，返回 `ostream` 引用  
第一个形参是 `ostream` 引用，第二个形参是需要打印的常量引用类型  
尽量减少输出格式化，尤其是换行符，因为这不像输出运算符的行为  
由于通常需要访问非公有成员，故通常设置为友元  

```C++
std::ostream& 
operator<<(std::ostream& out, const Sales_item& s)
{
    out << s.isbn() << " " << s.units_sold << " "
        << s.revenue << " " << s.avg_price();
    return out;
}
```

## 4. 输入运算符

必须是非成员函数，返回 `istream` 引用  
第一个形参是 `istream` 引用，第二个形参是需要输入的变量的非常量引用  
输入运算符必须处理错误输入，如类型不正确，遇到 EOF 或其他流错误，以及自定义的格式错误  
在使用输入结果之前必须检查是否正确输入  
若发生输入错误前已经有一部分成员被改变，则发生错误后需要重置对象状态  
由于通常需要访问非公有成员，故通常设置为友元  

```C++
std::istream& 
operator>>(std::istream& in, Sales_item& s)
{
    double price;
    in >> s.bookNo >> s.units_sold >> price;
    // check that the inputs succeeded
    if (in)
        s.revenue = s.units_sold * price;
    else 
        s = Sales_item();  // input failed: reset object to default state
    return in;
}
```

## 5. 算术和关系运算符

通常定义为非成员函数，参数是常量引用，返回一个新值  
算术运算符通常使用复合赋值运算符实现，故一般定义算术运算符之前都要定义相应的复合赋值运算符  

```C++
Sales_data
operator+(const Sales_data &lhs, const Sales_data &rhs)
{
    Sales_data sum = lhs;  // copy data members from lhs into sum
    sum += rhs;            // add rhs into sum
    return sum;
}
```

### 相等运算符

定义时应满足传递性，即 `a == b && b == c` 可得出 `a == c`  
若定义了 `==`，则应同时定义 `!=`，直接通过调用 `==` 实现即可  

```C++
inline
bool operator==(const Sales_data &lhs, const Sales_data &rhs)
{
    return lhs.isbn() == rhs.isbn() && 
           lhs.units_sold == rhs.units_sold && 
           lhs.revenue == rhs.revenue;
}

inline
bool operator!=(const Sales_data &lhs, const Sales_data &rhs)
{
    return !(lhs == rhs);
}
```

### 小于运算符

许多关联容器和算法需要使用该运算符，定义原则是：  

- 与关联容器对关键字的要求一致  
- 若有 `a != b`，则 `a < b` 或 `b < a` 至少有一个成立，常见的不符合情况是只比较某一个成员而不比较其他成员，但 `==` 运算符却比较了其他成员  

## 6. 花括号赋值运算符

必须是成员函数，返回当前类的引用，参数是 `initializer_list`  
例子见 Chapter 5  

## 7. 复合赋值运算符

通常是成员函数，返回当前类的引用，参数是常量引用  

```C++
// member binary operator: left-hand operand is bound to the implicit this pointer
// assumes that both objects refer to the same book
Sales_data& Sales_data::operator+=(const Sales_data &rhs)
{
    units_sold += rhs.units_sold;
    revenue += rhs.revenue;
    return *this;
}
```

## 8. 下标运算符

必须是成员函数，一般返回元素的引用，一般需要常量和非常量版本  
例子见 Chapter 5  

## 9. 递增和递减运算符  

通常是成员函数
前置版本无形参，后置版本有一个 `int` 形参，但不使用，编译器默认将实参设为 0  
前置版本返回当前对象的引用，后置版本返回一个值  
该类运算符经常用于类似迭代器行为的类中，定义时需要检查递增递减行为是否会导致越界  
后置版本在函数开头需要定义一个变量记录 `*this`，最后返回这个变量，使用前置版本实现递增递减  

```C++
class A
{
private:
    int mem;
public:
    A(int n): mem(n) {}
    A &operator++()
    {
        ++mem;
        return *this;
    }
    // 不使用的形参可以不命名
    A operator++(int)
    {
        A ret = *this;
        ++*this;
        return ret;
    }
}

A a(20);
a.operator++(); // 显式调用前置版本
a.operator++(0); // 显式调用后置版本
```

## 10. 解引用和成员访问箭头运算符

在类似迭代器和智能指针的类中经常使用，箭头必须是成员函数，解引用一般是成员函数  
通常设置为 `const` 成员，无形参，解引用返回指向元素的引用，箭头返回解引用运算符返回值的地址  
解引用运算符需要检查当前指针是否指向对象  

```C++
class intPtr
{
private:
    std::vector<int> &m_vec;
    std::size_t m_curr;

public:
    intPtr(std::vector<int> &vec, std::size_t curr): 
        m_vec(vec), m_curr(curr) {}
    
    int &operator*() const
    {
        if (m_curr >= m_vec.size())
            throw std::out_of_range("Invalid position!");
        else
            return m_vec[m_curr];
    }

    int *operator->() const
    {
        return &this->operator*();
    }
}
```

## 11. 函数调用运算符

必须是成员函数，具有函数调用运算符的对象称为函数对象
