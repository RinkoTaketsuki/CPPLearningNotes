# Chapter 1 IO 库

## 1. 源码速览

下面给出 IO 库一些重要的类、函数和对象的相关源码，编译器为 g++ in MinGW-w64 9.0

### 1.1 `<bits/basic_ios.h>`

```cpp
template<typename _CharT, typename _Traits>
class basic_ios : public ios_base {
    // defination
};
```

### 1.2 `<ostream>`

```cpp
template <typename _CharT, typename _Traits>
class basic_ostream : virtual public basic_ios<_CharT, _Traits> {
    // defination
};

// Write a newline and flush the stream.
template <typename _CharT, typename _Traits>
inline basic_ostream<_CharT, _Traits> &endl(
    basic_ostream<_CharT, _Traits>& __os) {
    return flush(__os.put(__os.widen('\n')));
}

// Write a null character into the output sequence.
template <typename _CharT, typename _Traits>
inline basic_ostream<_CharT, _Traits> &ends(
    basic_ostream<_CharT, _Traits>& __os) { 
    return __os.put(_CharT());
}

// Flushes the output stream.
template <typename _CharT, typename _Traits>
inline basic_ostream<_CharT, _Traits> &flush(
    basic_ostream<_CharT, _Traits>& __os) {
    return __os.flush();
}

/**
 * 本文件中还有关于成员及非成员函数模板 operator<<() 的相关定义，使 basic_ostream
 * 支持各种内置类型和各种字符串类型的输出，以及上面三个操作符（manipulator）。
 * 这些函数均返回输入的流对象的左值引用。
 */
```

### 1.3 `<istream>`

```cpp
template <typename _CharT, typename _Traits>
class basic_istream : virtual public basic_ios<_CharT, _Traits> {
    // defination
};

template <typename _CharT, typename _Traits>
class basic_iostream : 
    public basic_istream<_CharT, _Traits>,
    public basic_ostream<_CharT, _Traits> {
    // defination
};

// Skip leading whitespace.
// The defination is in "bits/istream.tcc"
template <typename _CharT, typename _Traits>
basic_istream<_CharT, _Traits> &ws(basic_istream<_CharT, _Traits> &__is);

/**
 * 本文件中还有关于成员及非成员函数模板 operator>>() 的相关定义，使 basic_istream
 * 支持各种内置类型和各种字符串类型的输入，以及上面的操作符（manipulator）。
 * 这些函数均返回输入的流对象的左值引用。
 */
```

### 1.4 `<sstream>`

```cpp
template <typename _CharT, typename _Traits, typename _Alloc>
class basic_istringstream : public basic_istream<_CharT, _Traits> {
    // defination
};

template <typename _CharT, typename _Traits, typename _Alloc>
class basic_ostringstream : public basic_ostream<_CharT, _Traits> {
    // defination
};

template <typename _CharT, typename _Traits, typename _Alloc>
class basic_stringstream : public basic_iostream<_CharT, _Traits> {
    // defination
};
```

### 1.5 `<fstream>`

```cpp
template<typename _CharT, typename _Traits>
class basic_ifstream : public basic_istream<_CharT, _Traits> {
    // defination
};

template<typename _CharT, typename _Traits>
class basic_ofstream : public basic_ostream<_CharT,_Traits> {
    // defination
};

template<typename _CharT, typename _Traits>
class basic_fstream : public basic_iostream<_CharT, _Traits> {
    // defination
};
```

### 1.6 `<iosfwd>`

```cpp
class ios_base;

template <typename _CharT, typename _Traits = char_traits<_CharT> >
class basic_ios;

template <typename _CharT, typename _Traits = char_traits<_CharT> >
class basic_istream;

template <typename _CharT, typename _Traits = char_traits<_CharT> >
class basic_ostream;

template <typename _CharT, typename _Traits = char_traits<_CharT> >
class basic_iostream;

template <typename _CharT, typename _Traits = char_traits<_CharT>,
    typename _Alloc = allocator<_CharT> >
class basic_istringstream;

template <typename _CharT, typename _Traits = char_traits<_CharT>,
    typename _Alloc = allocator<_CharT> >
class basic_ostringstream;

template <typename _CharT, typename _Traits = char_traits<_CharT>,
    typename _Alloc = allocator<_CharT> >
class basic_stringstream;

template <typename _CharT, typename _Traits = char_traits<_CharT> >
class basic_ifstream;

template <typename _CharT, typename _Traits = char_traits<_CharT> >
class basic_ofstream;

template <typename _CharT, typename _Traits = char_traits<_CharT> >
class basic_fstream;

typedef basic_istream<char> istream;
typedef basic_ostream<char> ostream;
typedef basic_iostream<char> iostream;

typedef basic_ifstream<char> ifstream;
typedef basic_ofstream<char> ofstream;
typedef basic_fstream<char> fstream;

typedef basic_istringstream<char> istringstream;
typedef basic_ostringstream<char> ostringstream;
typedef basic_stringstream<char> stringstream;

typedef basic_istream<wchar_t> wistream;
typedef basic_ostream<wchar_t> wostream;
typedef basic_iostream<wchar_t> wiostream;

typedef basic_istringstream<wchar_t> wistringstream;
typedef basic_ostringstream<wchar_t> wostringstream;
typedef basic_stringstream<wchar_t> wstringstream;

typedef basic_ifstream<wchar_t> wifstream;
typedef basic_ofstream<wchar_t> wofstream;
typedef basic_fstream<wchar_t> wfstream;
```

### 1.7 `<iostream>`

```cpp
extern istream cin;     // Linked to standard input
extern ostream cout;    // Linked to standard output
extern ostream cerr;    // Linked to standard error (unbuffered)
extern ostream clog;    // Linked to standard error (buffered)

extern wistream wcin;   // Linked to standard input
extern wostream wcout;  // Linked to standard output
extern wostream wcerr;  // Linked to standard error (unbuffered)
extern wostream wclog;  // Linked to standard error (buffered)
```

## 2. 注意事项

> 约定：IO 类型为上述后缀为 `stream` 的诸类型（包括类模板），IO 对象为 IO 类型的对象。

IO 类型没有拷贝构造函数和拷贝赋值函数，因此函数的形参和返回类型不能设为 IO 类型。  
通常以左值或右值引用方式定义 IO 类型的形参和返回类型。  
由于读写 IO 对象会导致其发生改变，不能使用 `const` 修饰 IO 类型及其引用。  

## 3. 条件状态

`std::ios_base` 中有若干常静态成员表示条件状态的状态位，这些状态的类型均为枚举类型 `std::ios_base::iostate`  

|成员名称|含义|初始值|
|:-|:-|:-|
|`std::ios_base::badbit`|流已崩溃|`1 << 0`
|`std::ios_base::eofbit`|流到达 EOF|`1 << 1`
|`std::ios_base::failbit`|IO 操作失败|`1 << 2`
|`std::ios_base::goodbit`|流未处于错误状态，该位为 0 代表未发生错误|`0`

设 `s` 为一个 IO 对象，`stat` 表示 `s.rdstate()`，该函数返回 `s` 的当前状态（`std::ios_base::iostate` 类型）。下面的表格省略了命名空间限定符：  

|成员函数|功能|
|:-|:-|
|`s.eof()`|返回 `bool` 类型，表示 `stat & eofbit`  
|`s.fail()`|返回 `bool` 类型，表示 `stat & (badbit \| failbit)`  
|`s.bad()`|返回 `bool` 类型，表示 `stat & badbit`  
|`s.good()`|返回 `bool` 类型，表示 `0 == stat`  
|`s.clear()`|无返回值，设置当前状态为 0  
|`s.clear(flags)`|`flags` 为 `iostate` 类型，将当前状态直接设定为 `flags`  
|`s.setstate(flags)`|`flags` 为 `iostate` 类型，令当前状态 `\|= flags`  

通常来讲，实际读取的内容无法存入给定的变量时，`failbit` 会被置位，到达文件结束位置时，`failbit` 和 `eofbit` 均被置位。  

IO 对象转换为 `bool` 类型相当于 `!s.fail()`  
  
用法举例：  

```C++
// 使用位运算符更改状态，复位 failbit 和 badbit，其他位不变
cin.clear(cin.rdstate() & ~cin.failbit & ~cin.badbit)

auto old_state = cin.rdstate(); // 保存 cin 当前状态
cin.clear();                    // 清除 cin 当前状态

/* 此处操作 cin */

cin.setstate(old_state); // 将 cin 置为原本的状态

char word;
// 读取流直到 EOF
while (cin >> word) {
    /* 此处处理 word */
}
```

## 4. 输出缓冲区

操作系统会将多个输出操作组合为一个单独的设备写操作以提升性能，故通常输出内容会被暂存在缓冲区中。缓冲刷新即缓冲区的内容被真正地写入设备或文件。  

### 4.1 缓冲刷新的原因

- 程序正常结束
- 缓冲区满
- 使用 `endl` 等操纵符显式刷新缓冲区
- 对流设置 `unitbuf`，使得每次输出操作都会刷新缓冲区，`cerr` 默认有此设置
- 流 A 关联到流 B 时，读写流 A 会导致流 B 的缓冲区被刷新（如 `cin` 和 `cerr` 均关联到 `cout`）

须注意程序崩溃时缓冲区不会被刷新，调试程序时要注意。

### 4.2 unitbuf 和 nounitbuf 操作符

这两个操作符为 `std::ios_base::fmtflags` 枚举类型，向流输出这两个操作符以设置流的刷新属性

### 4.3 流关联成员函数 `std::basic_ios<_CharT, _Traits>::tie()`

无参数版本：如果 `this` 关联到某输出流，则返回指向这个输出流的指针，否则返回空指针。  

带参数版本：接受一个 `std::basic_ostream<_CharT, _Traits>` 指针，将当前对象关联到其指向的输出流，若输入空指针则使当前流不再关联其他输出流。返回指向原来所绑定的输出流的指针（或为空）。  

每个流只能关联到一个输出流，但一个输出流可以被多个流关联。

## 5. 文件流

### 5.1 文件流的特有操作举例

```C++
fstream f; // 未绑定的默认文件流
fstream f(s, mode); 
/* 按 mode 打开文件 s（string 或 C 字符串），mode 的类型是
 * std::ios::openmode，有默认值，默认打开模式与 f 类型相关
 * 该构造函数是 explicit 的
 */
f.open(s, mode); // 按 mode 打开名为 s 的文件，无返回值
f.close(); // 关闭文件，无返回值
f.is_open(); // 返回 bool 类型，确定 f 绑定的文件是否成功打开且未关闭
```

创建文件流对象时，若提供文件名，则构造函数会调用 `open` 函数。`open` 函数打开文件失败时（通常是因为没有 `close` 就 `open` 新的文件），会将 `failbit` 置位。  

文件流对象转换为 `bool` 相当于调用 `is_open` 函数。  

当文件流对象被销毁时，`close` 函数会自动被调用，如下方代码所示：

```C++
string fileNames[10];
for (size_t i = 0; i < 10; ++i) {
    fstream f(fileNames[i]);
    if (f) {
        /* 处理文件 */
    }
    else {
        cerr << "Could not open: " + fileNames[i] + '\n';
    }
    // 每步循环都会销毁 f，无须显式调用 close 函数
}
```

### 5.2 打开模式

文件流的打开模式均为 `std::ios_base::openmode` 枚举类型，均为 `std::ios_base` 的常静态成员。

|名称|值|含义
|:-|:-|:-|
|`std::ios_base::app`|`1 << 0`|每次写前定位到文件末尾
|`std::ios_base::ate`|`1 << 1`|打开文件后定位到文件末尾
|`std::ios_base::binary`|`1 << 2`|二进制文件模式，若不设置该模式则表示文本文件模式
|`std::ios_base::in`|`1 << 3`|读
|`std::ios_base::out`|`1 << 4`|写
|`std::ios_base::trunc`|`1 << 5`|截断文件

- 只能对支持输入的文件流设置 `in`  
- 只能对支持输出的文件流设置 `out`  
- 只能对 `out` 模式的文件流设置 `trunc`  
- 只能对非 `trunc` 模式的文件流设置 `app`  
- 若设置了 `app`，则 `out` 也隐式地被设置  
- 若只设置 `out`，则 `trunc` 也隐式地被设置，若需阻止文件被截断，则需额外添加 `app` 或 `in` 模式  
- `ate` 和 `binary` 可应用于所有文件流  

类型|默认打开模式
:-:|:-:
`ifstream`|`in`
`ofstream`|`out`
`fstream`|`in\|out`

每次将文件流对象绑定到新文件时都可以重新指定模式。

## 6. string 流

### string 流的特有操作

```C++
stringstream stream; // 未绑定的默认 string 流

stringstream stream(s); // 使用 string 类型的 s 初始化，explicit

stream.str() // 返回 stream 中保存的字符串拷贝
stream.str(s) // 无返回值，将 string 类型的 s 拷贝到 stream 中
```

### istringstream 使用示例

```C++
struct PersonInfo {
    string name;
    vector<string> phones;
};

string line, phone;
vector<PersonInfo> people;

 // 从标准输入读取如下格式的数据到 people 中：
 // 每行为：名字 电话号码1 电话号码2 ...
while (getline(cin, line)) {
    people.emplace_back();
    PersonInfo &info = people.back();
    istringstream record(line);
    record >> info.name;
    while (record >> phone)
        info.phones.push_back(phone);
}
```

### ostringstream 使用示例

```C++
/* 检查上个示例中的号码，将其进行格式转换
 * 若某人的某个号码无法转换，则输出错误信息
 * invalid: 检查是否不合法
 * format: 转换函数
 */
 for (const auto &entry : people)
 {
    ostringstream formatted, badNums;
    for (const auto &nums : entry.phones)
    {
        if (invalid(nums))
        {
           badNums << " " << nums;
        }
        else
        {
           formatted << " " << format(nums);
        }
    }
    if (badNums.str().empty())
    {
        cout << entry.name << formatted.str() << endl;
    }
    else
    {
        cerr << "Input error: " << entry.name << "\nInvalid number(s):"
            << badNums.str() << '\n';
    }
 }
 ```

## 8. 操纵符详解

> 操纵符为一个函数或一个对象，能改变流的状态，可作为输入输出运算符的运算对象  
> set 开头的操作符在 `<iomanip>` 头文件中  

### 整数的格式

通常是成对的，一个设置操纵符和一个复原操作符  

`boolalpha`：使布尔值打印为 `true` 或 `false`  
`noboolalpha`：使布尔值打印为 `0` 或 `1`（默认）  
`oct`：使整型值打印为八进制格式（没有前面的 0）  
`hex`：使整型值打印为十六进制格式（没有前面的 0x）  
`dec`：使整型值打印为十进制格式（默认）  
`default`：使整型值打印为十进制格式  
`setbase(b)`：将整型值设置为 b（`int`） 进制格式，仅支持 8，10，16，否则重置为十进制  

### 浮点数的格式

默认情况下小数点后有六位，若无小数部分不打印小数点  
非常大和非常小的值会打印为科学计数法形式  

```C++
// 设置精度
ostream os;

os.precision(); // 返回当前精度（streamsize）
os.precision(n); // 将当前精度改为 n（streamsize），返回修改前的精度

setprecision(n); // 操纵符，将当前精度改为 n（int）
```

`showpoint`：浮点值总是显示小数点  
`noshowpoint`：浮点值仅当含小数部分时才显示小数点（默认）  
`fixed`：浮点值显示为定点十进制  
`scientific`：浮点值显示为科学计数法  
`hexfloat`：浮点值显示为十六进制  
`defaultfloat`：重置浮点值显示为十进制，是否科学计数法看情况（默认）  

### 其他格式

`uppercase`：使十六进制的 X 和 A~F 大写，使科学计数法的 E 大写  
`nouppercase`：使十六进制的 x 和 a~f 小写，使科学计数法的 e 小写（默认）  
`showbase`：显示八进制和十六进制的前缀  
`noshowbase`：不显示八进制和十六进制的前缀（默认）  
`showpos`：非负数显示正号  
`noshowpos`：非负数不显示正号（默认）  
`left`：左对齐输出  
`right`：右对齐输出（默认）  
`internal`：两端对齐输出，左对齐符号，右对齐值  
`skipws`：输入运算符跳过空白符（默认）  
`noskipws`：输入运算符不跳过空白符  
`setw(n)`：指定下一次输出的最小空间 n（`int`）  
`setfill<char_type>(ch)`：指定使用 ch（`char_type`）代替空格作为补白字符  

## 9. 未格式化 IO

> 将 IO 流当作单纯的字节序列来处理  

```C++
int ch; // 注意不能定义成 char 类型，否则无法保存 EOF 宏
// 此外 如果 char 是 signed char，则问题更多

// 从 is 读取一个字节到 ch，返回 istream 引用
is.get(ch);
// 向 os 写入一个字节，返回 ostream 引用
is.get(ch);
// 从 is 读取一个字节作为 int 返回
is.get();
// 将 ch 放回 is 的读取位置，返回 istream 引用
is.putback(ch);
// 反向移动 is 的读取位置，相当于撤销 is.get() 的操作，返回 istream 引用
is.unget();
// 将下一个要读取的字节作为 int 返回，但不移动读取位置
is.peek();
```

`putback()` 和 `unget()` 不能连续调用，标准库只能保存上一次的读取内容  
在返回 `int` 的函数中，如果读取的是真实字符，则将 `char` 转化为 `unsigned char`，再提升到 `int`  
故真实字符的返回值总是正的，若读取到 EOF，则返回标准库规定的负值（`cstdio` 中的 `EOF` 宏）  

```C++
// 多字节操作
char s[256];
streamsize size;
char delim;

/* 从 is 中读取最多 size 个字节并保存在 sink（char*） 中
 * 遇到 delim 或分隔符或 EOF 时停止，遇到的 delim 或分隔符会保存在输入流中不读取
 * 返回 istream 引用
 */
is.get(s, size, delim);
// 与上一个函数相似，但会读取并丢弃 delim 或分隔符
is.getline(s, size, delim);
// 读取最多 size 个字节到 s 中，返回 istream 引用
is.read(s, size)
/* 返回上一次未格式化读取操作的字节数（streamsize）
 * peek，unget，putback 会使其返回 0
 */
is.gcount();
// 将 s 中的 size 个字节写入到 os 中，返回 ostream 引用
os.write(s, size);
// 读取并忽略最多 size 个字符，包括 delim
// size 默认为 1，delim 默认为 EOF
is.ignore(size, delim);
```

## 10. 流随机访问

`istream` 和 `ostream` 类型通常不支持随机访问  

```C++
// 后缀 g 表示输入流，后缀 p 表示输出流

/* 返回当前位置（流类型::pos_type）
 * 位置标记只有一个，即 fstream 和 stringstream 的
 * g 和 p 版本返回的标记相同
 */
tellg(); tellp();

seekg(pos); seekp(pos); // 跳转到指定位置，输入值通常为 tell 的返回值

/* 跳转到 form 之前或之后 off（流类型::off_type）个字符
 * form 的可选值如下：（命名空间为当前流类型）
 * beg：表示开始位置
 * cur：表示当前位置
 * end：表示结束位置
 */
seekg(off, form); seekp(off, form)
```

```C++
// 使用例：在文件的最后一行写入文件每一行的起始 offset

int main()
{
    // 开始的位置便是文件末尾
    fstream inOut("filename", fstream::ate | fstream::in | fstream::out);
    if (!inOut)
    {
        cerr << "Unable to open file!\n";
        return EXIT_FAILURE; // 定义于 cstdlib
    }
    auto endMark = inOut.tellg(); // 保存当前的文件末尾
    inOut.seekg(0, fstream::beg); // 重定位到文件开始
    size_t cnt = 0; // 记录字节数
    string line; // 保存文件行
    while (!inOut && inOut.tellg() != endMark && getline(inOut, line))
    {
        cnt += line.size() + 1; // + 1 表示换行符
        auto mark = inOut.tellg(); // 记录当前读取位置
        inOut.seekp(0, fstream::end); // 重定位到文件结束
        inOut << cnt; // 输出数据
        if (mark != endMark) inOut << "　"; // 若未读取到原来的最后一行，输出空格
        inOut.seekg(mark); // 跳转回原来的读取位置
    }
    inOut.seekp(0, fstream::end);
    inOut << '\n'; // 最后输出换行符
    return 0;
}
```
