# Chapter 1 IO 库

## 1. IO 类

`<iostream>`

`istream wistream` 从流读取数据  
`ostream wostream` 从流写入数据  
`iostream wiostream` 从流读写数据  

`<fstream>`

`ifstream wifstream` 从文件读取数据  
`ofstream wofstream` 从文件写入数据  
`iofstream wiofstream` 从文件读写数据  

`<sstream>`

`istringstream wistringstream` 从string读取数据  
`ostringstream wostringstream` 从string写入数据  
`iostringstream wiostringstream` 从string读写数据  

> 带 w 前缀的类表示其处理的是宽字符  
> 带 f 和 string 的类都继承自相应的不带 f 和 string 的类  
> 由继承关系可得，相应的不带 f 和 string 类型的 IO 类引用和指针可以指向带 f 和 string 类型的对象  
> `istream &getline(istream&, string&)` 从输入流读取一行数据，返回 string 对象

## 2. IO 对象不能拷贝或赋值

由于不能拷贝 IO 对象，函数的形参和返回类型不能设为 IO 类型  
通常以引用方式定义 IO 类型的形参和返回类型  
但由于读写 IO 对象会导致其发生改变，不能设置为 `const` 引用类型  

## 3. 条件状态

`std::ios_base::iostate` 条件状态类，包含如下的几个条件状态：  
`static const std::ios_base::iostate badbit` 流已崩溃  
`static const std::ios_base::iostate failbit` IO 操作失败  
`static const std::ios_base::iostate eofbit` 流到达文件结束  
`static const std::ios_base::iostate goodbit` 流未处于错误状态，该位为0代表未发生错误
  
设 `s` 为一个 IO 对象：  
`s.eof()` 返回 `bool` 类型，若 `eofbit` 置位，则返回 `true`  
`s.fail()` 返回 `bool` 类型，若 `failbit` 或 `badbit` 置位，则返回 `true`  
`s.bad()` 返回 `bool` 类型，若 `badbit` 置位，则返回 `true`  
`s.good()` 返回 `bool` 类型，若三个错误比特均为0，则返回 `true`  
`s.clear()` 无返回值，将所有状态位复位  
`s.clear(flags)` `flags` 为 `iostate` 类型，将指定的状态位复位  
`s.setstate(flags)` `flags` 为 `iostate` 类型，将指定的状态位置位  
`s.rdstate()` 返回 `iostate` 类型，返回当前的条件状态  
  
实际读取的内容无法存入给定的变量时，`failbit` 会被置位  
到达文件结束位置时，`failbit` 和 `eofbit` 均被置位  
IO 对象转换为 `bool` 类型时，相当于 `!s.fail()`  
  
用法举例：  

```C++
// 使用位运算符更改状态，复位 failbit 和 badbit，其他位不变
cin.clear(cin.rdstate() & ~cin.failbit & ~cin.badbit)

auto old_state = cin.rdstate(); // 保存 cin 当前状态
cin.clear(); // 清除 cin 当前状态
/* 此处操作 cin */
cin.setstate(old_state); // 将 cin 置为原本的状态

char word;
// 读取流直到EOF
while (cin >> word)
{
    /* 此处处理 word */
}
```

## 4. 输出缓冲区

操作系统会将多个输出操作组合为一个单独的设备写操作以提升性能，故通常输出内容会被暂存在缓冲区中  
缓冲刷新，即缓冲区的内容被真正地写入设备或文件  
导致缓冲刷新的原因有：  

- 程序正常结束，即执行 `main` 函数的 `return` 操作
- 缓冲区满
- 使用 `endl` 等操纵符显式刷新缓冲区
- 对流设置 `unitbuf`，使得每次输出操作都会刷新缓冲区，`cerr` 默认此设置
- 流A关联到流B时，读写流A会导致流B的缓冲区被刷新（如 `cin` 和 `cerr` 均关联到 `cout`）

须注意程序崩溃时缓冲区不会被刷新，调试程序时要注意

### endl，flush 和 ends

向流输出这三个操纵符均可刷新缓冲区，`flush` 不输出额外内容，`endl` 额外输出换行符，`ends` 额外输出 `\0`  
先输出字符，再刷新缓冲区  

### unitbuf 和 nounitbuf

向流输出这两个操作符以设置流的刷新属性

### 流关联函数 tie

无参数版本：如果当前对象关联到某输出流，则返回指向这个输出流的指针，否则返回空指针  
带参数版本：接受一个 `ostream` 指针，将当前对象关联到其指向的输出流，若输入空指针则使当前流不再关联其他输出流  
每个流只能关联到一个输出流，但一个输出流可以被多个流关联

## 5. 文件流

### 文件流的特有操作

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

创建文件流对象时，若提供文件名，则构造函数会调用 `open` 函数  
`open` 函数打开文件失败时，会将 `failbit` 置位  
`fstream` 转换为 `bool` 相当于调用 `is_open` 函数  
试图打开已经被打开的文件会导致 `open` 失败，此时需先关闭文件再重新打开文件  
当 `fstream` 对象被销毁时，`close` 函数会自动被调用

```C++
string fileNames[10];
for (size_t i = 0; i < 10; ++i)
{
    fstream f(fileNames[i]);
    if (f)
    {
        /* 处理文件 */
    }
    else
    {
        cerr << "Could not open: " + fileNames[i];
    }
    // 每步循环都会销毁 f，无须显式调用 close 函数
}
```

## 6. 文件模式

`in` 读  
`out` 写  
`app` 每次写前定位到文件末尾
`ate` 打开文件后定位到文件末尾
`trunc` 截断文件
`binary` 二进制 IO

- 只能对 `ofstream` 和 `fstream` 设置 `in`  
- 只能对 `ofstream` 和 `fstream` 设置 `out`  
- 只能对 `out` 模式的流设置 `trunc`  
- 只能对非 `trunc` 模式的流设置 `app`  
- 若设置了 `app`，则 `out` 也隐式地被设置  
- 若只设置 `out`，则 `trunc` 也隐式地被设置，若需阻止文件被截断，则需额外添加 `app` 或 `in` 模式  
- `ate` 和 `binary` 可应用于所有文件流  

类型|默认打开模式
:-:|:-:
`ifstream`|`in`
`ofstream`|`out`
`fstream`|`in\|out`

每次将文件流对象绑定到新文件时都可以重新指定模式  

## 7. string 流

### string 流的特有操作

```C++
stringstream stream; // 未绑定的默认文件流
stringstream stream(s); 
/* 将 string 类型的 s 的拷贝到 stream 中
 * 该构造函数是 explicit 的
 */
stream.str() // 返回 stream 中保存的字符串拷贝
stream.str(s) // 无返回值，将 string 类型的 s 拷贝到 stream 中
```

### istringstream 使用示例

```C++
struct PersonInfo
{
    string name;
    vector<string> phones;
};
string line, phone;
vector<PersonInfo> people;

/* 从标准输入读取如下格式的数据到 people 中：
 * 每行为：名字 电话号码1 电话号码2 ...
 * 不限行数
 */
while (getline(cin, line))
{
    PersonInfo info;
    istringstream record(line);
    record >> info.name;
    while (record >> phone)
        info.phones.push_back(phone);
    people.push_back(info);
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
`setbase(b)`：将整型值设置为 b（`int`） 进制格式，仅支持 8，10，16，否则重置  

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
