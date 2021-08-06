# Chapter 1 入门 & 杂项说明

## 1. 获得 main 的返回值的方法

Unix: 执行完可执行程序后，运行如下指令

```Shell
echo $?
```

Windows: 执行完可执行程序后，运行如下指令（PowerShell 不可用）

```Shell
echo %ERRORLEVEL%
```

## 2. 编译器的使用

GNU 编译器：-Wall 参数显示所有警告  
默认输出文件名：a.out

```Shell
g++ -o <输出文件名> <cpp文件列表>
g++ -c <cpp文件> # 生成.o文件
g++ -o <输出文件名> <o文件列表>
```

MSVC 编译器：/EHsc 参数表示打开标准异常处理  
默认输出文件名：main函数所在文件去掉后缀名.exe

```Shell
cl /EHsc <cpp文件列表>
cl -c <cpp文件> # 生成.obj文件
cl -o <输出文件名> <obj文件列表>
```

## 3. iostream 中的对象

### istream: 从键盘读取输入

|          |          |
| :------- | :------- |
| std::cin | 标准输入 |

### ostream：向 Shell 输出

|           |                                |
| :-------- | :------------------------------|
| std::cout | 标准输出                        |
| std::cerr | 标准错误（不缓冲）              |
| std::clog | 标准错误（缓冲）                |
| std::endl | 输出换行并将缓冲区内容写入输出流 |

## 4. 输入输出运算符

### 输出运算符 <<

左侧必须是 ostream 对象，a << b 的值是 a

### 输入运算符 >>

左侧必须是 istream 对象，a >> b 的值是 a

## 5. 多行注释不可嵌套

错误样例：

```C++
/*
 * 多行注释不能嵌套 /* 多行注释 */ 哦
 */
```

## 6. cin 读取不定数量的输入的方法

```C++
while (std::cin >> value) 
{
    std::cout << value << std::endl;
}
```

当 cin 读取到 EOF 或无效输入（如 value 是 int 型但读取的字符不是数字）时，cin 的值会变为假

## 7. 命令行输入 EOF 的方法

Unix： <kbd>Ctrl</kbd>+<kbd>D</kbd>  
Windows： <kbd>Ctrl</kbd>+<kbd>Z</kbd> <kbd>Enter</kbd>

## 8. 文件重定向的方法

```Shell
程序 <输入文件 >输出文件
```

## 9. 防止文件重复包含的方法

```C++
#ifndef HFILE_H
#define HFILE_H

/* 文件内容 */

#endif
```

```C++
#pragma once
```

## 10. using声明

```C++
using std::cin;
using namespace std;
```

> 头文件一般不使用using声明

## 11. assert宏和NDEBUG

`assert(expr)` 这个宏当expr为0时，输出expr名字并终止程序执行，需包含cassert  
assert宏的生效条件是NDEBUG宏未定义，若在文件开头添加 `#define NDEBUG` 则assert语句将被预处理器无视  
也可以使用编译器选项 `-D NDEBUG` 添加宏（MSVC使用 `\D`）  
可以用NDEBUG编写自己定义的调试代码

```C++
#ifndef NDEBUG
/* 调试代码 */
#endif
```

## 12. 编译器定义的变量

```C++
__func__; // const char数组，局部静态变量，存储函数名
__FILE__; // 字符串宏，存储当前文件名（绝对路径名）
__LINE__; // 字符串宏，存储当前行号
__TIME__; // 字符串宏，存储文件编译时间（不包含日期）
__DATE__; // 字符串宏，存储文件编译日期
```
