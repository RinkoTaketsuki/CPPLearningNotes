# Chapter 1 入门 & 杂项说明

## 1. 使用命令行获取进程退出代码的方法

Unix: 执行完可执行程序后，运行如下指令

```Sh
echo $?
```

Windows CMD: 执行完可执行程序后，运行如下指令

```cmd
echo %ERRORLEVEL%
```

Windows Powershell: 执行完可执行程序后，运行如下指令

```powershell
echo $LASTEXITCODE
```

## 2. 编译器的使用

GNU 编译器：-Wall 参数显示所有警告  
默认输出文件名：a.out

```Sh
# 用法举例
g++ -o <输出文件名> <C++ 源文件或目标文件列表>
g++ -c <C++ 源文件> # 生成 .o 目标文件
```

MSVC 编译器：/EHsc 参数表示打开标准异常处理  
默认输出文件名：把 main 函数所在文件的文件名后缀替换为 .exe 所得到的文件名

```sh
# 用法举例
cl /EHsc <C++ 源文件列表>
cl -c <C++ 源文件列表> # 生成 .obj 目标文件
cl -o <输出文件名> <目标文件列表>
```

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

## 10. using 声明

```C++
using std::cin;
using namespace std;
```

> 头文件一般不使用 `using` 声明

## 11. assert 宏和 NDEBUG

`assert(expr)` 这个宏当expr为0时，输出expr名字并终止程序执行，需包含 `cassert`  
`assert` 宏的生效条件是 `NDEBUG` 宏未定义，若在文件开头添加 `#define NDEBUG` 则 `assert` 语句将被预处理器无视  
可以使用编译器选项 `-D NDEBUG`（MSVC使用 `\D`）  
可以用 `NDEBUG` 编写自己定义的调试代码

```C++
#ifndef NDEBUG
/* 调试代码 */
#endif
```

## 12. 编译器定义的变量

```C++
__func__; // const char 数组，局部静态变量，存储函数名
__FILE__; // 字符串宏，存储当前文件名（绝对路径名）
__LINE__; // 字符串宏，存储当前行号
__TIME__; // 字符串宏，存储文件编译时间（不包含日期）
__DATE__; // 字符串宏，存储文件编译日期
```
