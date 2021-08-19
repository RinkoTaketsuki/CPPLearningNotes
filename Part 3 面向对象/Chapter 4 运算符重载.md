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
