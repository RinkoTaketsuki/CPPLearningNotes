# Chapter 1 类基础

## 1. this指针

指向当前对象，是顶层 `const` 指针（`Type *const this = &current_object`）  
成员函数必须通过 `this` 指针访问类内其他成员  
成员函数直接使用其他成员时其实是省略了 `this`  
`this` 是每一个成员函数的隐式参数，且为第一个参数  
成员函数返回 `this` 指针指向的对象时，返回类型应使用左值引用类型，否则会返回原对象的拷贝

## 2. 常量成员函数

用法：在函数声明的括号后面加一个const标识符  
由于this指针本身不带底层const，故普通成员函数不能访问常量对象  
常量成员函数的this指针是const class_t *const类型  
也因此，常量成员函数不能修改其他成员  
const对象只能访问const成员函数  
常量成员函数返回this指针指向的对象时，返回类型若是引用类型，则会返回常量引用（const class_t&）  
const声明可作为函数重载的区分条件，非常量对象优先访问非const版本，常量对象优先访问const版本

## 3. 类的作用域

类本身是一个作用域，成员函数内部是嵌套的作用域  
类内部定义的函数是隐式的inline函数
编译器先编译成员声明，再编译成员函数体，故成员函数可以写在其他成员的前面  
可以定义非成员函数来实现类的某些接口，但一般把这些函数声明在和相关类的同一个头文件内

## 4. = default

存在合成版本的成员函数可以用default来声明和定义

## 5. 默认访问权限

class：private  
struct：public

## 6. 友元

在类内部定义或声明函数或类时加friend关键字，说明其不是成员函数或为其他类，但可以访问私有成员  
友元不是类的成员，故可以声明在任何地方  
友元声明仅声明了权限，未声明函数，故须在外部重新声明才能使用，通常把友元和类的声明放在同一个头文件中  
类内定义的友元函数也是隐式内联的  
其他类的私有成员函数不能设为友元  
友元类代表友元类的成员函数可以访问当前类的所有成员，包括私有成员  
友元关系不具有传递性，自己友元类的友元类不是自己的友元类  
需注意友元函数的外部声明导致的友元函数的作用域问题（有些编译器不存在这个问题）

```C++
struct A
{
    friend void fr_f() {}
    void X() {fr_f();} // 错误，fr_f还未声明
    void Y();
    void Z();
}

void A::Y(){fr_f()}; // 错误，fr_f还未声明
void fr_f();
void A::Z(){fr_f()}; // 正确
```

## 7. 类型成员

在public处使用using或typedef规定类型别名，则类外部可以使用这个别名，以实现封装  
须注意类型成员必须先定义后使用，故一般声明在类成员的最前面
当为某命名空间内类型成员指定别名时，应该使用 `typedef typename NameSpace::Type`，以指明 Type 是类型成员而不是其他成员  

## 8. 显式 inline 成员函数

可以在声明时说明，也可以在定义时说明，最好是只在类外定义，且只在定义时说明  
显式inline成员函数一般也定义在头文件中

## 9. 可变数据成员

数据成员声明前面加mutable关键字，用于说明其在const成员函数中也可以修改  
即便是const对象调用const成员函数也可以

## 10. 类内初始化注意事项

必须使用拷贝（等号）初始化或花括号初始化

## 11. 类的声明

```C++
class A;
```

这种只声明的类型A是不完全类型，只能声明指向这种类型的指针和引用、形参和返回类型，不能声明对象  
也因此，类内部可以定义指向其自身类型的引用和指针

## 12. 名字查找

通常的名字查找过程是：

- 在名字所在块中的名字出现位置之前寻找声明
- 若未找到，继续寻找外层的作用域
- 若最终未找到，则报错

类的定义分两步：

- 编译成员声明
- 编译函数体

因此，成员函数中的名字查找方式较为不同：

- 在名字所在块中的名字出现位置之前寻找声明
- 若未找到，在类内继续查找，此时所有成员不分顺序都可以
- 若在类内未找到，则在外层作用域的成员函数定义之前的部分继续寻找
- 如果有成员使用过外部定义的类型名，则在类内不可以再重新定义

下面是一些例子：

```C++
typedef int integer;
double var;
class A
{
public:
    integer f() {return var;}
private:
    integer var;
};
// 编译器从发现f()的声明开始，先找integer的声明
// 首先在类内寻找，未发现声明
// 之后，在类外向前寻找发现声明
// 编译完类声明后，编译成员函数体，此时return语句中的var在类内寻找可以找到
// 故返回的var是类内成员而不是外部的double类型的var
```

```C++
typedef int integer;
class A
{
public:
    integer f() {return var;} // 此处左边的integer返回类型使用外层定义
private:
    typedef int integer; // 已经使用过外层integer，故此处定义错误
    integer var;
};
```

```C++
int var = 0;
class A
{
public:
    typedef int integer;
    void f(integer var)
    {
        foo = var * bar; // 1
    }
    void g()
    {
        foo = var * bar // 2
    }
    void h(integer);

private:
    integer var = 0, foo = 0, bar = 0;
};

A::integer out_f(A::integer);

void A::h(integer var)
{
    foo = out_f(var); // 3
}
// 当编译器处理语句1时，var会先从函数作用域内找，故语句1中的var是指函数形参
// 若需使用类成员var，则需写成 this->var
// 若需使用全局变量var，则需写成 ::var
// 当编译器处理语句2时，此处的var则是类成员var
/* 
 * 当编译器处理语句3时，虽然out_f在类定义之前不可见，但根据在外部作用域的成员函数
 * 前面找的规则，h函数可以发现out_f的声明。若把out_f的声明移到h函数的定义的下方
 * 则会报错
 */
```

## 13. 聚合类

满足如下条件的类称为聚合类：

- 所有成员均为public的
- 未定义任何构造函数
- 无类内初始值
- 无基类，无虚函数

聚合类可以用struct声明也可以用class声明，可以用大括号初始化，但顺序必须对应  
若大括号中数量不足，则剩下的成员被值初始化

## 14. 字面值常量（constexpr）类

字面值常量类的constexpr成员函数并不是隐式const的（教材有误）  
满足如下条件的 __聚合类__ 是字面值常量类：

- 成员变量必须全部是字面值类型（可转化为constexpr的类型）
- 至少含有一个constexpr构造函数
- 类内初始值必须是constexpr，若某成员变量是类类型，则其必须使用其类的constexpr构造函数
- 必须使用析构函数的默认定义

constexpr构造函数可声明为default，一般把constexpr构造函数的函数体设为空  
构造函数的constexpr属性不能作为重载区分条件  
constexpr类的所有成员变量必须都初始化  
constexpr对象只能调用constexpr构造函数
constexpr类的constexpr对象只能调用const成员函数

## 15. static成员

静态成员函数不包含this指针，也不可以声明为const
静态成员可以用成员运算符访问，也可以用作用域运算符访问  
成员函数使用静态成员无须标注作用域  
static关键字只能在类内使用，类外定义静态成员函数时不能加static  
由于静态成员不属于任何对象，故构造函数不能初始化静态成员，通常只能在外部定义和初始化静态成员  
外部定义静态成员变量时可以直接使用类的其他静态成员  
静态成员的定义一般放在定义非内联成员函数的文件中  
在类内定义static成员函数，若其调用其他static成员，则该成员必须是已经初始化过的字面值类型  
在类内定义static成员变量，该变量只能是const或constexpr类型  
静态成员变量可以是不完全类型（指针和引用变量也可以是不完全类型），而一般成员变量不行  
静态成员变量可以作为成员函数的默认实参，而一般成员变量不行  

## 16. = delete

通知编译器不希望定义指定的成员函数，通常用于阻止拷贝和赋值操作  
所有成员函数均可以删除，但如果删除析构函数，则会导致无法创建该类的变量和临时对象，且动态分配的对象无法释放（不能 __delete__）

### 使编译器自动定义删除的成员函数的情形

- 类的某个成员的析构函数、拷贝构造函数、拷贝赋值运算符是 __delete__ 或不可访问（__private__ 或 __protected__ 导致）的，则自动生成 __delete__ 的对应函数
- 类有 __const__ 或引用类型的成员变量，则自动生成 __delete__ 的拷贝赋值运算符
- 类的析构函数是 __delete__ 或不可访问的，或类有未类内初始化的引用类型成员变量，或有未类内初始化且无显式定义的默认构造函数的 __const__ 类型成员变量时，自动生成 __delete__ 的默认构造函数

总之，如果一个类不能默认地构造、拷贝或销毁，则对应的默认生成的成员函数是 __delete__ 的

### 利用 private 实现等效操作

如需删除某个函数，可以声明其为 __private__ 而不定义，这样外部和友元便都无法访问

## 17. 何时合成拷贝控制成员

只有在调用时才会合成对应成员  

```C++
class Empty {};

int main()
{
    Empty e1; // 合成默认构造函数
    Empty e2(e1); // 合成拷贝构造函数
    e2 = e1; // 合成拷贝赋值运算符
    // 合成析构函数
    return 0;
}
```
