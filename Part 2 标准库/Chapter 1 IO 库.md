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
