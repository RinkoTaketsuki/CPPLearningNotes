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

```Shell
g++ -o <输出文件名> <cpp文件列表>
```

MSVC 编译器：/EHsc 参数表示打开标准异常处理

```Shell
cl /EHsc <cpp文件列表>
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

## 6. while 语句执行顺序

```C++
while (condition) 
{
    statement;
}
```

1. 检查 condition 是否为真。若为假，则结束，若为真，则继续
2. 执行 statement，跳转回步骤 1

## 7. for 语句执行顺序

小括号中的部分为循环头，大括号中的部分为循环体  
循环头定义的变量和循环体中定义的变量作用域相同

```C++
for (init-statement; condition; expression) 
{
    statement;
}
```

1. 先执行 init-statement
2. 检查 condition 是否为真。若为假，则结束，若为真，则继续
3. 执行 statement
4. 执行 expression，跳转回步骤 2

## 8. cin 读取不定数量的输入的方法

```C++
while (std::cin >> value) 
{
    std::cout << value << std::endl;
}
```

当 cin 读取到 EOF 或无效输入（如 value 是 int 型但读取的字符不是数字）时，cin 的值会变为假

## 9. 命令行输入 EOF 的方法

Unix： <kbd>Ctrl</kbd>+<kbd>D</kbd>  
Windows： <kbd>Ctrl</kbd>+<kbd>Z</kbd> <kbd>Enter</kbd>

## 10. 文件重定向的方法

```Shell
程序 <输入文件 >输出文件
```

## 11. 防止文件重复包含的方法

```C++
#ifndef HFILE_H
#define HFILE_H

/* 文件内容 */

#endif
```

```C++
#pragma once
```

## 12. using声明

```C++
using std::cin;
using namespace std;
```

> 头文件一般不使用using声明

## 13. 范围for语句

```C++
for (declaration: expression) 
{
    statement;
}
```

declaration：当前变量，expression：迭代序列  
相当于Python中的 `for declaration in expression:`  
如需修改declaration中的值则需声明为引用类型  
