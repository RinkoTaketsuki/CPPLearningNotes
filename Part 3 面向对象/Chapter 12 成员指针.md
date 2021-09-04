# Chapter 12 成员指针

```C++
// 教材使用的 Screen 类源码

class Screen {
public:
    typedef std::string::size_type pos;

    // Action is a type that can point to a member function of Screen
    // that returns a reference to a Screen and takes no arguments
    typedef Screen&(Screen::*Action)();

    // constructor: build screen of given size containing all blanks
    Screen(pos ht = 0, pos wd = 0): contents(ht * wd, ' '), cursor(0), 
                                    height(ht), width(wd) { }
    friend int main();
    // data is a static member that returns a pointer to member
    static const std::string Screen::*data() 
        { return &Screen::contents; }

    char get_cursor() const { return contents[cursor]; }
    char get() const { return contents[cursor]; }
    inline char get(pos ht, pos wd) const;
private:
    std::string contents;
    pos cursor;
    pos height, width;

public:
    // cursor movement functions 
    // beware: these functions don't check whether the operation is valid
    Screen& home() { cursor = 0; return *this; }
    Screen& forward() { ++cursor; return *this; }
    Screen& back() { --cursor; return *this; }
    Screen& up() { cursor += height; return *this; }
    Screen& down() {cursor -= height; return *this; }

    // specify which direction to move; enum see XREF(enums)
    enum Directions { HOME, FORWARD, BACK, UP, DOWN };
    Screen& move(Directions);
private:
    static Action Menu[];      // function table
};

char Screen::get(pos r, pos c) const // declared as inline in the class
{
    pos row = r * width;      // compute row location
    return contents[row + c]; // return character at the given column
}

inline
Screen& Screen::move(Directions cm)
{
    // run the element indexed by cm on this object
    return (this->*Menu[cm])(); // Menu[cm] points to a member function
}

Screen::Action Screen::Menu[] = { &Screen::home,
                                  &Screen::forward,
                                  &Screen::back,
                                  &Screen::up,
                                  &Screen::down,
                                };
```

## 1. 数据成员指针

```C++
Screen myScreen, *pScreen = &myScreen;

// 声明一个指向 Screen 类成员的 const string 指针
// 与通常的底层 const 指针类似，可以指向常量或非常量成员，但不能通过该指针修改指向的成员
const string Screen:: *pdata;

// 数据成员指针可以不指向某个特定对象的成员，因为初始化和赋值不会使指针指向任何对象
pdata = &Screen::contents;

// 使用 auto 或 decltype 声明成员指针更容易
// auto pdata = &Screen::contents;

// .* 或 ->* 运算符解引用 pdata，获得 myScreen.contents
// 可以理解为先解引用 pdata 以得知获取哪个成员，再使用 . 或 -> 运算符获取成员
auto s1 = myScreen.*pdata;
auto s2 = pScreen->.*pdata;
```

常规的访问控制规则对成员指针也适用，由于数据成员通常为私有成员，故数据成员指针通常使用专门的函数返回  
具体的例子见 `Screen::data()`，是静态成员函数  
上面例子中的赋值语句在类外部或在友元外部无法使用，通常是写成 `auto pdata = Screen::data();`

## 2. 成员函数指针

```C++
Screen myScreen, *pScreen = &myScreen;

// 简易定义方法
// 成员函数不会转化成对应的成员函数指针，因此不能忽略取地址符
auto pmf = &Screen::get_cursor;

// 指向成员函数的指针的类型需要指定形参和返回类型，const 限定符和引用限定符
char (Screen::*pmf2)(Screen::pos, Screen::pos) const = &Screen::get;

// 解引用，须注意调用运算符优先级高于 .* 和 ->* 运算符
char c1 = (pScreen->*pmf)();
char c2 = (myScreen.*pmf2)(0, 0);
```

应用例子见 `Screen::move`

## 3. 成员指针类型别名

```C++
// 两种等价写法
using Action = char (Screen::*)(Screen::pos, Screen::pos);
typedef char(Screen::*Action)(Screen::pos, Screen::pos);

// 使用例 1：赋值
Action get = &Screen::get;

// 使用例 2：作为形参类型或返回类型，可指定默认实参
Screen &action(Screen&, Action = &Screen::get);

// 等价的三种调用
Screen myScreen;
action(myScreen);
action(myScreen, get);
action(myScreen, &Screen::get);
```

## 4. 将成员函数作为可调用对象

`function` 模板可用于将成员函数指针转化为可调用对象，需提供接受 `this` 的形参类型  
`mem_fn` 也可用于将成员函数指针转化为可调用对象，会提供多个重载版本的函数以接受不同的 `this` 的形参类型  
`bind` 与 `mem_fn` 类似，但需要注意第一个参数为 `this`，可以是指针或引用或值  

```C++
vector<string> svec;
vector<string*> pvec;
auto p = &string.empty;

// 查找空字符串

// fcn 在 find_if 内部是以 ((*iter).*p)() 形式调用的
function<bool(const string&)> fcn = p;
find_if(svec.begin(), svec.end(), fcn);

// fcn2 在 find_if 内部是以 ((*iter)->*p)() 形式调用的
function<bool(const string*)> fcn2 = p;
find_if(pvec.begin(), pvec.end(), fcn2);

// f 无须关心当前对象是以引用还是指针还是拷贝赋值还是其他形式传入
auto f = mem_fn(p);
find_if(svec.begin(), svec.end(), f);
find_if(pvec.begin(), pvec.end(), f);
f(svec[0]);
f(&svec[0]);
f(pvec[0]);
```

```C++
// bind 举例

std::string s("hello");
char& (std::string::*p)(std::size_t) = &std::string::at;
auto f = bind(p, std::placeholders::_1, std::placeholders::_2);
auto ret3rd = bind(p, std::placeholders::_1, 2);
auto hello = bind(p, s, std::placeholders::_1);
std::cout << f(s, 2) << std::endl;
std::cout << ret3rd(s) << std::endl;
std::cout << hello(2) << std::endl;
```
