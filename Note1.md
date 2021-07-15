# Chapter 1 C++基础
## 1. 获得main的返回值的方法
Unix: 执行完可执行程序后，运行如下指令
```
echo $?
```
Windows: 执行完可执行程序后，运行如下指令（PowerShell不可用）
```
echo %ERRORLEVEL%
```
## 2. 编译器的使用
GNU编译器：-Wall参数显示所有警告
```
g++ -o <输出文件名> <cpp文件列表>
```
MSVC编译器：/EHsc参数表示打开标准异常处理
```
cl /EHsc <cpp文件列表>
```
## 3. iostream中的对象
### istream: 从键盘读取输入
std::cin 标准输入
### ostream：向Shell输出
std::cout 标准输出<br>
std::cerr 标准错误（不缓冲）<br>
std::clog 标准错误（缓冲）<br>
std::endl 输出换行并将缓冲区内容写入输出流
## 4. 输入输出运算符
### 输出运算符 <<
左侧必须是ostream对象，a << b 的值是 a
### 输入运算符 >>
左侧必须是istream对象，a >> b 的值是 a
## 5. 多行注释不可嵌套
错误样例：
```C++
/* 
 * 多行注释不能嵌套 /* 多行注释 */ 哦
 */
```
## 6. while语句执行顺序
```C++
while (condition) {
    statement;
}
```
1. 检查condition是否为真。若为假，则结束，若为真，则继续
2. 执行statement，跳转回步骤1
## 7. for语句执行顺序
小括号中的部分为循环头，大括号中的部分为循环体。<br>
循环头定义的变量和循环体中定义的变量作用域相同
```C++
for (init-statement; condition; expression) {
    statement;
}
```
1. 先执行init-statement
2. 检查condition是否为真。若为假，则结束，若为真，则继续
3. 执行statement
4. 执行expression，跳转回步骤2
## 8. cin读取不定数量的输入的方法
