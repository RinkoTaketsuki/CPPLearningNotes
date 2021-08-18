# Chapter 5 动态内存

> 静态内存：存储局部静态对象，类静态成员，未定义在函数内部的变量，局部静态对象在使用前分配，由编译器自动创建和销毁  
> 栈内存：局部非静态对象，生命周期为程序块，由编译器自动创建和销毁  
> 堆内存：生命周期由程序控制  

`new` 和 `delete` 在使用时容易出现问题，未释放内存会导致内存泄漏，过早释放内存可能导致指针指向无效位置  

## 1. 智能指针

```C++
shared_ptr<T> sp; // 空指针，允许多个指针指向同一对象
unique_ptr<T> up; // 空指针，独占对象

(bool)p; // 空指针为 false，否则为 true
*p; // 解引用
p->mem; // 访问成员
p.get(); // 返回 p 中保存的指针（T*）
swap(p, q); p.swap(q); // 交换指针

// shared_ptr 特有操作
make_shared<T>(args); // 返回指向用 args 初始化的动态分配的 T 对象的 shared_ptr
// 若 T 为内置类型，则当 args 为空时进行值初始化
shared_ptr<T> p(q); // 用 q 拷贝初始化 p，q 中的指针必须能转化为 T*，此操作会导致 q 中计数器递增
p = q; // p q 都是 shared_ptr，二者中的指针必须能相互转化，此操作会导致 p 中计数器递减，q 中计数器递增
// shared_ptr 的计数器为 0 时会释放其管理的内存
p.use_count(); // 返回与 p 共享对象的智能指针数量，速度较慢
p.unique(); // 若与 p 共享对象的智能指针数量为 1，则返回 true，否则返回 false
```

### 引用计数

任何对 `shared_ptr` 的拷贝都会导致其指向对象的引用计数递增，如被用作拷贝初始化，函数传参和返回  
任何对 `shared_ptr` 的赋值或销毁都会导致其指向对象的计数递减，如作为赋值运算符的左值，执行析构函数  
引用计数为 0 时，会释放资源，由 `shared_ptr` 的析构函数执行  

## 2. new 和 delete

`new` 在默认情况下对内置类型进行默认初始化，对类类型执行默认构造函数  
可以使用圆括号和花括号进行初始化，若括号中只有一个值，则可以使用 `auto`  

```C++
auto p = new auto(42); // p 为 int*
```

若使用空的圆括号，则进行值初始化（对内置类型，初始化为 0，对类类型，执行默认构造函数）  
可以分配 `const` 对象，但必须初始化，定义了默认构造函数的类类型可以不显式初始化，其他类型必须显式初始化  
分配 `const` 对象返回的是底层 `const` 指针  
一般情况下，若内存分配失败，会抛出 `bad_alloc` 异常，可使用定位 `new` 传递 `nothrow` 使其不抛异常而是返回空指针  

```C++
int *p = new (nothrow) int(42);
```

`delete` 同一个指针两次会破坏堆空间  
如果两个指针指向同一个位置，则 `delete` 其中一个指针并将其置为 `nullptr` 也不会使另一个指针安全  

## 3. shared_ptr 和 new 结合使用

`shared_ptr` 有一个接受指针的构造函数，是 `explicit` 的  

```C++
shared_ptr<int> sp(new int(20)); // 注意不可以用等号初始化
```

默认使用 `delete` 释放对象，但如果传入的指针不是指向堆空间，则需要指定释放操作  

```C++
shared_ptr<T> p(q); // 内置指针 q 必须指向堆空间，且能转换成 T 类型
shared_ptr<T> p(u); // u 是 unique_ptr，p 接管其所指空间，并将 u 置为空
shared_ptr<T> p(q, d); // 内置指针 q 需能转换成 T 类型，使用可调用对象 d 代替 delete
shared_ptr<T> p(p2, d); // p2 是 shared_ptr，使用可调用对象 d 代替 delete
p.reset(); // 若 p 是唯一一个指向对象的 shared_ptr，则释放对象并将 p 置为空
p.reset(q); // q 是内置指针，与上面的区别是将 p 置为 q
p.reset(q, d); // d 是可调用对象，与上面的区别是使用可调用对象 d 代替 delete
// 后面两个版本的 reset，若 p 不是唯一的 shared_ptr，则会将 q 赋给 p，并更新引用计数
// 此功能经常与 unique 方法联用
```

不要使用内置指针访问 `shared_ptr` 指向的对象，容易访问空悬指针  
不要使用 `get()` 返回的内置指针初始化别的 `shared_ptr`  

```C++
shared_ptr<int> p(new int(20));
int *q = p.get();
void foo()
{
    shared_ptr<int> p2(q); // 引用计数为 1 而不是 2
}
// 此时 p2 已被销毁，其指向的内存已被释放
int var = *p; // 访问空悬指针
// p 被销毁，q 指向的空间被 delete 了两次
```

## 4. 抛出异常时的内存管理

```C++
void foo()
{
    shared_ptr<int> sp(new int(20));
    int *p = new int(30);
    // 此处抛出异常，且不被捕获
    delete p;
}
// 此时 sp 会被释放，而 p 不会被释放
```

## 5. 删除器的使用

有些类的析构函数不对分配的空间进行回收，需要手动定义删除器  
智能指针管理非堆空间资源也需手动定义删除器  

```C++
struct A
{
    int *m;
    A(): m(new int(0)) {}
    ~A() {}
};

void deleter(A *p) // 参数必须是 A* 类型
{
    delete p->m;
    delete p;
}

shared_ptr<A> sp(new A(), deleter);
// 此处 sp 被销毁，销毁前先执行 deleter 销毁 m，再销毁 A 类型的对象
// 注意如果没有 delete p; 这条语句，A 的析构函数不会被调用
// 即 deleter 替换了原本的 delete 操作，而不会再执行原本的 delete 操作
```

## 6. unique_ptr

同时只能有一个 `unique_ptr` 指向指定对象，`unique_ptr` 被销毁时，其指向的对象也被销毁  
`unique_ptr` 不支持拷贝或赋值  

```C++
unique_ptr<T> u; // 空指针，使用 delete 删除器
unique_ptr<T> u(q); // 使用内置指针 q 初始化
unique_ptr<T, D> u; // 空指针，使用类型为 D 的删除器
unique_ptr<T, D> u(d); // 空指针，使用删除器 d 替代 delete
u = nullptr; // 释放指向对象并将 u 置为空
u.release(); // u 放弃管理并返回内部的指针，将 u 置为空
u.reset(); // 释放指向对象
u.reset(q); // 释放指向对象并令 u 管理内置指针 q
u.reset(nullptr); // 效果同 u = nullptr;
```

利用 `release`，`reset` 和构造函数可以实现转移所有权  

```C++
unique_ptr<string> p1(new string("hello"));
unique_ptr<string> p2(p1.release());
unique_ptr<string> p3(new string("aloha"));
p2.reset(p3.release());
```

可以将 `release` 的返回值保存在内置指针变量中，但必须记得 `delete`  
可以拷贝或赋值将被销毁的 `unique_ptr`，如函数返回类型可以是 `unique_ptr`  
返回时不执行拷贝构造函数和赋值运算符  

```C++
using upi = unique_ptr<int>;
// 以下两种方式均可
upi foo()
{
    return upi(new int(20));
}
upi bar()
{
    upi ret(new int 20);
    return ret;
}
```

## 7. weak_ptr

不控制指向对象的生存期，其即使绑定 `shared_ptr` 也不会改变引用计数  

```C++
weak_ptr<T> w; // 空指针
weak_ptr<T> w(sp); // 用 shared_ptr 初始化，T 必须能转化成 sp 指向的对象类型
w = p; // p 可以是 shared_ptr 或 weak_ptr，赋值后共享对象
w.reset(); // 将 w 置为空
w.use_count(); // 共享对象的 shared_ptr 数量
w.expired(); // use_count == 0
w.lock(); // expired ? 空 shared_ptr : 指向 w 指向的对象的 shared_ptr
```

不能通过 `weak_ptr` 直接访问对象，因为其绑定的 `shared_ptr` 可能未指向对象

```C++
// 通过 weak_ptr 访问对象的安全方法
if (shared_ptr<T> sp == w.lock())
{
    // 处理 sp
}
```

## 8. 动态数组
