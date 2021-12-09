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

> 可调用对象：函数、函数指针、lambda 表达式、`bind` 创建的对象、重载了函数调用运算符的类  
> 可调用对象的类型：lambda 表达式为未命名类，函数和函数指针由返回和实参类型决定  
> 相同返回和实参类型的普通函数、函数指针、lambda 表达式和函数对象类属于不同的类型，但它们的调用形式相同  

### function 模板

```C++
function<T> f; // 空 function，T 是调用形式
function<T> f(nullptr); // 空 function
function<T> f(obj); // obj 可以是任意符合调用形式的可调用对象
function<T> f = obj; // 同上
(bool)f; // 空时为假，否则为真
f(args); // 调用 f
```

类型成员|说明
:-|:-
`result_type`|返回类型
`argument_type`|仅有一个形参时才有的类型，形参类型
`first_argument_type`|仅有两个形参时才有的类型，第一个形参类型
`second_argument_type`|仅有两个形参时才有的类型，第二个形参类型

通过该模板，相同调用形式的不同类型的可调用对象可以统一转化为 `function<T>` 类型  

```C++
struct divide
{
    int operator()(int i, int j) { return i / j; }
};

int main()
{
    int add(int i, int j) { return i + j; }
    auto mod = [] (int i, int j) { return i % j; };
    // 编写计算器需要的函数 map
    map<string, function<int(int, int)> > binops = {
        {"+", add}, // 函数指针
        {"-", std::minus<int>()}, // 标准库函数对象
        {"/", divide()}, // 函数对象
        {"*", [] (int i, int j) { return i * j; }}, // 未命名 lambda 对象
        {"%", mod} // 已命名 lambda 对象
    }
    return 0;
}
```

重载函数的名字不能直接存入 `function` 对象，需要新定义一个函数指针消除二义性  
未命名 lambda 表达式不存在此问题  

```C++
int add(int, int);
double add(double, double);
function<int(int, int)> f(add); // 报错，即便 function 类中有调用方式
int (*fp)(int, int) = add; // 函数指针只会接受正确的版本
function<int(int, int)> f(fp); // 正确
```

## 12. 类型转换运算符

必须是成员函数，不能转化为 `void` 类型和不能作为返回值的类型（如数组和函数类型）  
可以是数组指针或函数指针类型或引用类型  
无显式返回类型，无形参，通常是 `const` 成员  

```C++
class SmallInt
{
public:
    SmallInt(int i = 0): val(i)
    {
        if (i < 0 || i > 255)
            throw std::out_of_range("Bad SmallInt Value");
    }
    operator int() const { return val; }
private:
    std::size_t val;
}

SmallInt si; // 调用默认构造函数
si = 4; // 调用转换构造函数将字面值 4 转化为 SmallInt，然后调用 SmallInt::operator=
si + 3; // 调用 SmallInt::operator int 将 si 转化为 int，然后执行 int 加法

// 用户自定义的类型转换只能隐式执行一次，但其之前或之后可以运行内置的类型转换
SmallInt si = 3.14 // 先执行内置类型转换 double -> int，再执行转换构造函数
si + 3.14; // 先调用 SmallInt::operator int 将 si 转化为 int，再执行内置类型转换 int -> double，然后执行 double 加法
```

不能滥用自定义类型转换运算符  

```C++
// 假设 istream 支持向 bool 类型的隐式转换
int i = 20; cin << i;
// cin 会被转化为 bool 值，然后该值会被转化为 int，最终执行 int 左移
```

### 显式类型转换运算符

在函数名前面加 `explicit` 标识符，表示不会用于隐式类型转换，必须显式调用或使用显式强转  
但以下的例外情况可以隐式转换：  

- `if`，`while`，`do`，`for` 语句的条件部分  
- `!`，`&&`，`||` 运算符的运算对象  
- `?:` 运算符的条件部分  

`operator bool` 经常被定义，且定义成 `explicit` 的  
`istream` 的 `operator bool` 即为 `explicit` 的，返回 `istream::good()`  

### 二义性问题

转换构造函数和类型转换运算符极易产生二义性问题，故有以下的经验规则：  

- 勿定义相同类型转换：即在 A 中定义参数类型为 B 的转换构造函数，同时在 B 中定义转换为 A 的运算符  
- 尽量避免定义转换为内置算术类型的运算符，尤其是已经定义了一个这种运算符时：

    1. 不要再定义接受内置算术类型的运算符，因为此时如果用户试图使用该运算符，则类型转换操作会执行，然后使用内置版本的运算符  
    2. 不要定义多个转换为内置算术类型的运算符，而是让内置类型转换执行相应的任务  

- 总之，除了 `explicit operator bool` 尽量别定义别的类型转换运算符，尽量限制使用支持隐式的转换构造函数  

二义性问题也存在于函数重载问题中  

```C++
struct A { A(int); };
struct B { B(int); };
struct C { C(double); };
foo(const A&);
foo(const B&);
foo(const C&);
foo(10);
/* 如果只定义了 A 和 B，显然调用具有二义性
 * 如果只定义了 A 和 C，也具有二义性，因为都属于类类型转换，不会考虑内置类型转换
 */
```

```C++
class SmallInt
{
    friend operator+(const SmallInt&, const SmallInt&);
public:
    smallInt(int i = 0);
    operator int() const { return val; }
private:
    std::size_t val;
};

SmallInt s1, s2;
s1 + s2; // 使用重载版本的函数
int i = s1 + 0; // 具有二义性，使用重载和内置版本的函数均可
```

## 13. new 和 delete 运算符

### new 和 new[] 的行为

1. 调用 `operator new` 或 `operator new[]` 函数，分配一块足够大的、原始的、未命名的内存空间  
2. 调用相应的构造函数构造对象，若为 new[] 运算符则会调用多次构造函数  
3. 返回指向对象的指针  
4. 第一步可能抛出 `bad_alloc` 异常，此时构造函数不会执行  

### delete 和 delete[] 的行为

1. 调用对象的析构函数，若为 delete[] 运算符则会调用多次析构函数，析构顺序与构造顺序相反  
2. 调用 `operator delete` 或 `operator delete[]` 函数，在该函数中释放空间

### 标准库接口

自定义的四种运算符可以定义为成员函数，也可以是非成员函数，如果未定义则使用全局作用域的标准库版本  
如 `::new` 的表达式可指定使用标准库版本  

```C++
// 可能抛出 bad_alloc 异常的版本
void *operator new(size_t);
void *operator new[](size_t);
void *operator delete(void*) noexcept;
void *operator delete[](void*) noexcept;

// 不会抛出异常的版本
void *operator new(size_t, nothrow_t&) noexcept;
void *operator new[](size_t, nothrow_t&) noexcept;
void *operator delete(void*, nothrow_t&) noexcept;
void *operator delete[](void*, nothrow_t&) noexcept;

/* nothrow_t 定义在头文件 <new> 中，是一个空 struct
 * 该文件中还定义了一个名为 nothrow 的 const nothrow_t 对象
 * 通过该对象调用不抛异常的版本
 */
```

重载这些运算符时，`noexcept` 声明要保持不变，返回类型保持不变，`new` 的第一个形参必须是 `size_t` 且无默认实参  
`delete` 类似，第一个形参必须是 `void*`，第一个形参是指向待释放内存的指针，可以有第二个形参（`size_t`），表示释放的空间大小  
调用 `new` 时，第一个形参会被传入对象或对象数组的大小  
重载运算符只能在全局作用域和类作用域中声明  
在类作用域中声明时，其为隐式 `static` 成员，因为 `new` 在对象构造之前调用，`delete` 在对象销毁之后调用，且不能操作对象成员  

```C++
// 通常的 operator new 和 operator delete 的定义方法
void *operator new(size_t size)
{
    if (void *mem = malloc(size))
        return mem;
    else
        throw bad_alloc();
}

void operator delete(void* mem) noexcept
{
    free(mem);
}
```

### 定位 new 的行为

可以作为显式调用构造函数的一种方法  

1. 调用 `operator new` 或 `operator new[]` 函数（有指针形参的版本），标准库版本（全局作用域）的该函数仅返回输入的指针，别的什么都不做  
2. 调用相应的构造函数构造对象，若为定位 new[] 运算符则会调用多次构造函数  
3. 返回指向对象的指针  

```C++
void *operator new(size_t, void*);
void *operator new[](size_t, void*);

// 使用定位 new 表达式的方法
class A
{
    int mem1;
    double mem2;
public:
    A() = default;
    A(int n): mem1(n), mem2(0) {}
};

char *buf = new char[1000];

A *p1 = new (buf) A;
A *p2 = new (buf + sizeof(A)) A(42);
A *p3 = new (buf + sizeof(A) * 2) A[3];
A *p4 = new (buf + sizeof(A) * 5) A[4] {1, 2, 3, 4};
```

### new 和 delete 表达式的行为模拟

```C++
// A *p = new A(/*args*/);
A *p;
try
{
    p = static_cast<A*>(::operator new(sizeof(A)));
    new(p) A(/*args*/);
}
catch (std::bad_alloc) {}

// delete *p;
p->~A();
::operator delete(p);
```
