# Chapter 14 union

可以有多个数据成员，但同时只能有一个成员有值  
因此一个 `union` 对象占的存储空间至少能容纳其最大成员即可  
`union` 不能含有引用类型成员  
`union` 可以设置成员访问权限，默认权限是 `public`  
`union` 可以定义成员函数，但不能继承和被继承，也不能定义虚函数  
`union` 可以使用花括号赋值或初始化，花括号内只能有一个成员，会对其第一个成员初始化和赋值  
对 `union` 的成员赋值会导致其他成员变为未定义状态  

## 1. 匿名 union

会直接定义一个对象，在程序中直接使用成员名字而不使用成员访问运算符  
匿名 `union` 只能定义 `public` 成员，不能定义成员函数  

```C++
union
{
    char cv;
    int iv;
    double dv;
}

cv = 'a';
dv = 3.14;
```

## 2. 含有类类型成员的 union

当给类类型成员赋值时，需要调用对应类的构造函数  
当给其他成员赋值时，需要调用对应类的析构函数  
若 `union` 有类类型成员定义了默认构造函数或拷贝控制成员，则 `union` 对应的合成版本将被删除  

## 3. 使用类管理 union

```C++
class Token
{
public:
    Token(): tok(INT), ival{0};
    Token(const Token &t): tok(t.tok)
    {
        copyUnion(t);
    }
    Token &operator=(const Token&);
    ~Token()
    {
        if (tok == STR)
            sval.~string();
    }
    Token &operator=(const std::string&);
    Token &operator=(char);
    Token &operator=(int);
    Token &operator=(double);
private:
    enum {INT, CHAR, DBL, STR} tok; // 判别式
    // 每个 Token 对象会有一个匿名 union 成员
    union
    {
        char cval;
        int ival;
        double dval;
        std::string sval;
    };
    // 检查判别式并拷贝成员
    void copyUnion(const Token&);
};

// 赋非类类型的值
Token &Token::operator=(int i)
{
    if (tok == STR)
        sval.~string();
    ival = i;
    tok = INT;
    return *this;
}

// 赋类类型的值
Token &Token::operator=(const std::string &s)
{
    if (tok == STR)
        sval = s;
    else
        new(&sval) string(s); // 定位 new 表达式
    tok = STR;
    return *this;
}

void Token::copyUnion(const Token &t)
{
    switch (t.tok)
    {
        case Token::INT:
            ival = t.ival; break;
        case Token::CHAR:
            cval = t.cval; break;
        case Token::DBL:
            dval = t.dval; break;
        case Token::STR:
            new(&sval) string(t.sval); break;
    }
}

Token &Token::operator=(const Token &t)
{
    if (tok == STR && t.tok != STR)
        sval.~string();
    if (tok == STR && t.tok == STR)
        sval = t.sval;
    else
        copyUnion(t);
    tok = t.tok;
    return *this;
}
```
