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
