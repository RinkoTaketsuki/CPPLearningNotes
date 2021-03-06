# Chapter 7 继承和多态

## 1. 简单举例

### 基类

```C++
class Quote
{
public:
    Quote() = default;
    Quote(const std::string &book, double sales_price): bookNo(book), price(sales_price) {}

    /* 基类的析构函数通常是虚的 */
    virtual ~Quote() = default;
    std::string isbn() { return bookNo; }
    virtual double net_price(std::size_t n) const { return n * price; }
private:
    std::string bookNo;

/* 基类和派生类都可以访问的成员 */
protected:
    double price = 0.0;
};
```

设置为虚函数的成员函数会被动态绑定  
除静态成员函数和构造函数外都可以设置为虚函数  
`virtual` 只能出现在类内部的声明语句中  
在基类中是虚函数的在派生类中也是虚函数  

### 派生类

```C++
/* 类派生列表：访问说明符 + 基类，以逗号分隔
 * 访问说明符用于控制从基类继承来的成员对派生类对象的用户是否可见
 * 当访问说明符为 public 时，派生类可以访问基类的所有公有成员
 * 并且基类的指针和引用可以指向派生类对象，当基类指针或引用指向派生类对象时，
 * 指向的是其基类部分，此种转换称为派生类到基类的类型转换
 * 通常使用单继承
 * 不能继承不完整类型
 */
class Bulk_quote : public Quote
{
public:
    Bulk_quote() = default;
    /* 派生类必须使用基类的构造函数来初始化其基类部分
     * 先执行基类构造函数初始化列表，再执行基类构造函数体，
     * 再执行派生类构造函数初始化列表，再执行派生类构造函数体
     * 如果初始化列表前面没有指明调用哪个构造函数，
     * 则使用基类默认构造函数初始化基类部分
     */
    Bulk_quote(const std::string &book, double p, std::size_t qty, double disc):
        Quote(book, p), min_qty(qty), discount(disc) {}
    /* 派生类也可以不覆盖虚函数，直接继承
     * 下面的函数前面可以加 virtual 关键字，也可以不加，都会被派生类的派生类当成虚函数
     * override 关键字用于显式声明该函数是覆盖的基类的虚函数
     */
    double net_price(std::size_t) const override;
private:
    std::size_t min_qty;
    double discount;
};

/* 派生类访问基类成员实例 */
double Bulk_quote::net_price(std::size_t cnt) const
{
    if (cnt >= min_qty)
        return (1 - discount) * cnt * price;
    else
        return cnt * price;
}
```

基类定义的静态成员在所有派生类里也只有一个实例  
派生类只能访问 `public` 和 `protected` 的静态成员  
派生类仅声明不定义时（如 `class Bulk_quote;`），后面不能跟类派生列表  
作为基类的类必须定义过，不能只声明，也因此，类不能继承自己  
`final` 关键字可以加在类名后面，表示其不能被继承  

## 2. 基类和派生类间的类型转换

存放基类指针的智能指针也可以绑定派生类对象  

> 静态类型：编译时已知，变量声明时的类型或表达式生成的类型  
> 动态类型：运行时才可知，变量或表达式在内存中实际的类型  

基类的指针和引用类型的静态类型和动态类型可以不一致，若其实际指向派生类对象，则其动态类型为派生类  
不存在基类指针或引用向派生类指针或引用的隐式转换，即便是如下情况：  

```C++
Bulk_quote b;
Quote *p = &b;
Bulk_quote *q = p;
/* p 为基类指针，指向派生类对象，此时也不能将 p 赋给派生类指针 q
 * 如果一定要转换，可以使用 dynamic_cast
 * 如果确定转换没问题，也可以使用 static_cast
 */ 
```

基类和派生类对象间不能进行转换，但基类的拷贝初始化和拷贝赋值可以用派生类对象，因为其参数是 `const` 引用  
此时基类对象显然只保留了基类成员的信息，派生类成员信息被切掉了  

## 3. 虚函数

对含有虚函数成员的对象，构造函数会创建一个 `vptr` 指针指向虚函数表 `vtbl`  
`vtbl` 的第一个元素是当前对象的 `type_info` 对象  

> 动态绑定：指针和引用类型的基类和派生类对象，当程序运行时，才能确定动态类型，进而确定使用哪一个虚函数  
> 而非指针和引用类型的对象则可以在编译时便确认使用哪个虚函数  
> 非虚函数的调用也是在编译时确定的  

### 派生类覆盖虚函数的注意事项  

形参类型必须与基类完全一致，`const` 和引用限定符也必须相同  
返回参数一般情况下需要完全一致，但返回类本身类型的指针或引用时除外  
派生类可以返回基类的指针或引用  
若从基类到派生类的类型转换是可访问的，则基类可以返回派生类的指针或引用  

### override

如果派生类定义了一个名字和基类的虚函数相同，但形参类型和限定符不一致的成员函数，则编译器不会将其认为是覆盖基类的虚函数  
通过添加 `override` 关键字可以强调此函数是覆盖的基类的虚函数，这样编译器会进行检查  

### final

在基类成员函数后添加 `final` 关键字后，派生类不能覆盖该函数，会报错  
此方法可用于派生类定义的覆盖基类的虚函数，防止其被覆盖  

### 默认实参

当使用指针或引用类型时，使用哪一个默认实参由静态类型决定，与其实际类型无关  
因此，基类和派生类虚函数的默认实参最好一致  

### 强行执行特定版本的虚函数

使用作用域运算符指定即可  

## 4. 抽象基类

含有或继承了纯虚函数的类为抽象基类，不能定义抽象基类的对象  
纯虚函数在声明时后面加 `= 0` 标志，只能在类内声明时加  
可以给纯虚函数添加函数体，但必须写在类外  

## 5. 访问控制与继承

### protected

基类可以将成员设置为 `protected`，此时：

- 外部无法访问，内部和派生类可以访问
- 派生类的成员只能通过派生类对象（静态类型）来访问，如果派生类某个成员或友元函数的形参是基类的指针或引用，则不能访问该形参的 `protected` 成员  

### 派生访问说明符  

通过派生访问说明符，派生类继承基类时，基类的所有的成员的访问权限会缩小到指定的范围  
`public` 不改变访问权限  
`protected` 将基类的所有 `public` 成员都改为 `protected`  
`private` 将基类的所有成员都改为 `private`  
  
只有当派生类公有继承基类时，外部才能使用派生类到基类的转换  
派生类的成员函数和友元总能使用派生类到基类的转换  
只有当派生类公有或受保护继承基类时，派生类的成员函数和友元才能使用基类到派生类的转换  

### 友元与继承

友元关系不能继承，但基类的友元可以访问派生类的基类部分  

### 使用 using 改变可访问性

假如基类 B 有成员 a，派生类声明 `using B::a` 可以无视派生访问说明符，而是使用自定义的访问权限  
这取决于 `using` 语句声明在哪一块，如果声明在 `public:` 下面则访问权限就是 `public`  

### 默认继承保护级别

即不写派生访问说明符时的继承保护级别  
使用 `class` 定义的默认级别是 `private`，使用 `struct` 定义的默认级别是 `public`  

## 6. 作用域与继承

派生类的作用域被嵌套在基类的作用域之内  
查找名字时，如果在当前类的作用域之内找不到名字，还会在其基类中查找  
名字查找是在编译时进行的，即取决于静态类型，这意味着如果基类指针或引用指向派生类时，该指针或引用不能访问派生类独有的成员  
派生类的成员会隐藏同名的基类成员，但可以通过作用域运算符使用被隐藏的成员  
  
举例：`p->mem()` 的查找过程：  

1. 确定静态类型  
2. 在静态类型对应的类里面查找名字  
3. 在继承链上逐个向上查找名字，若最终找不到则报错  
4. 若找到，则进行类型检查，看 `mem` 的调用是否合法  
5. 若调用合法，则看其是否是虚函数  
6. 若是虚函数且调用成员是通过指针或引用对象，则运行时决定使用哪个虚函数，反之则执行常规函数调用  

名字查找优先于类型检查，比如派生类 D 和基类 B 有同名但形参列表不同的成员函数 f，派生类对象调用 f，但形参列表与 B::f 一致，此时会报错  
如果要使用 B::f，必须用作用域运算符显式说明  
  
举例：虚函数与作用域：  

```C++
class Base
{
public:
    virtual int f();
};

class D1 : public Base
{
public:
    int f(int); // 隐藏了 Base::f，但不是虚函数
    virtual void g();
};

class D2 : public D1
{
public:
    int f(int); // 隐藏了 D1::f，不是虚函数
    int f(); // 覆盖了 Base::f，是虚函数
    void g(); // 覆盖了 D1::g，是虚函数
};

Base bobj; D1 d1obj; D2 d2obj;
Base *bp1 = &bobj, *bp2 = &d1obj, *bp3 = d2obj;
bp1->f(); // 调用 Base::f
bp2->f(); // 调用 Base::f
bp3->f(); // 调用 D2::f
bp1->f(0); // 错误
bp2->f(0); // 调用 D1::f
bp3->f(0); // 调用 D2::f

D1 *d1p = &d1obj; D2 *d2p = &d2obj;
bp2->g(); // 错误
d1p->g(); // 调用 D1::g
d2p->g(); // 调用 D2::g
```

```C++
// 基类有重载虚函数的情形
class B
{
public:
    virtual void foo();
    virtual void foo(int);
};

class D : public B
{
public:
    // using B::foo;
    void foo();
};

D obj;
B *pb = &obj;
D *pd = &obj;
pb->foo(10); // 正确
pd->foo(10); // 错误，无法调用 B::foo(int)，必须加上面注释掉的 using 声明
```

## 7. 虚析构函数

由于静态类型与动态类型不一定相同，故 `delete` 某静态类型指针可能出现调用基类析构函数的情况，此时便需要把基类的析构函数设置为虚函数  
由于虚函数特性可以继承，故只需把最上层的基类的析构函数设置为虚函数即可，此时 `delete` 总能调用正确的析构函数  

## 8. 拷贝控制与继承

1. 若基类的默认构造函数、拷贝构造函数、拷贝赋值运算符或析构函数是删除的或不可访问的，则派生类中对应的成员是删除的，因为基类部分的相应操作无法执行  
2. 若基类的析构函数是删除的或不可访问的，则派生类中合成的默认构造函数、拷贝构造函数和移动构造函数是删除的，因为基类部分无法销毁  
3. 若基类的移动操作是被删除的或不可访问的，则派生类中使用 `= default` 请求移动操作时，对应的合成的移动操作是删除的，因为基类部分无法移动  

基类和派生类的拷贝控制成员定义示例：  

```C++
class B
{
public:
    B() = default;
    B(const B&) = default;
    B(B&&) = default;
    B &operator=(const B&) = default;
    B &operator=(B&&) = default;
    virtual ~B() = default;
    int memb;
};

class D : public B
{
public:
    // D 的默认构造函数调用 B 的默认构造函数。
    // D 的其他构造函数要想使用 B 对应类型的构造函数，
    // 则必须显式调用，否则会调用 B 的默认构造函数。
    // 此处利用了动态类型特性
    D(const D &d): B(d), memd(d.memd) {}
    D(D &&d): B(std::move(d)), memd(d.memd) {}
    // B::operator= 不会被自动调用，必须显式调用
    D &operator=(const D &rhs)
    {
        B::operator=(rhs);
        memd = rhs.memd;
        return *this;
    }
    D &operator=(D&& rhs)
    {
        B::operator=(std::move(rhs));
        memd = rhs.memd;
        return *this;
    }
    // 派生类析构函数只负责销毁自己分配的资源
    // 之后会执行基类的析构函数销毁基类部分的资源
    ~D() = default;
    int memd;
};
```

## 9. 在构造函数和析构函数中使用虚函数的注意事项  

建立派生类对象时，先执行基类的构造函数，此时该对象处于未完成状态，如果基类析构函数调用虚函数的派生类版本则会产生错误  
销毁对象时先执行派生类的析构函数，再执行基类的析构函数，如果基类析构函数调用虚函数的派生类版本则会产生错误  

## 10. 继承的构造函数

在派生类中声明 `using B::B` 即可继承基类的构造函数（默认、拷贝、移动构造函数不能继承）  
继承来的构造函数的统一形式是（假设基类是 B，派生类是 D）`D(args): B(args) {}`，D 的剩余成员将被默认初始化  
该 `using` 声明的所在位置不会影响原本构造函数的访问级别，继承来的构造函数的访问级别始终与基类相同  
如果要被继承的某个构造函数具有 n 个默认实参，则会继承 n + 1 个构造函数，其中一个无默认实参，剩下 n 个函数每个分别忽略掉一个形参  
如果要被继承的某个构造函数在派生类里已经有对应的相同形参列表的构造函数，则该构造函数不会被继承，而是使用派生类中定义的版本  

## 11. 容器与继承

派生类对象拷贝或赋值给基类对象时，其派生类部分会被删掉  
因此，基类容器存储派生类对象时，无法访问其派生类的成员，也无法调用派生类版本的虚函数  
解决方法是令容器存储基类指针（最好是智能指针），这样便能存储派生类对象  
