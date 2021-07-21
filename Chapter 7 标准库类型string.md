# Chapter 7 标准库类型string

## 1. 初始化方法

> 带括号的为“直接初始化”，带等号的为“拷贝初始化”

```C++
string s1; // 默认初始化
string s2 = s1; string s2(s1); 
string s3 = "string"; string s3("string");
string s4(5, 's'); string s4 = string(5, 's'); // 初始化为"sssss"
```

## 2. string方法和运算

```C++
is = istream(); os = ostream(); s = string(); s1 = string(); s2 = string()
is >> s;
os << s;
std::istream& getline(is, s); // 从istream中读取一行到string
bool s.empty(); // s为空返回true，否则返回false
size_t s.size(); // 返回字符串长度（不包括'/0'）(以字节为单位)
s[n] // char& （n是size_t类型）
s1 + s2; // string
s1 = s2; // string&
s1 += s2; // string&
s1 == s2; s1 != s2;// bool
s1 < s2; s1 > s2; s1 <= s2; s1 >= s2; // bool
```

## 3. 读写string对象

1. 从cin读取string的注意事项  
自动忽略开头的空白字符，直至下一个空白字符

2. 使用getline读取一行的注意事项
getline从当前位置开始读，若当前字符为换行符则停止，换行符也会被包括进来  
但写入string时换行符会被丢弃

3. size()方法返回size_t类型，其为unsigned long，切勿与带符号类型混用

## 4. string比较大小

设有两个string类型对象s1和s2  

1. 若s1.size() < s2.size()，且s2的前s1.size()个字符与s1均相同，则认为s1 < s2

2. 若从某位置开始s1和s2有不相同的字符，则比较该位置的字符的大小

## 5. string和字符串字面值相加

+号的两边至少要有一个是string类型，这样其中的字符串字面值会自动转化为string类型

## 6. cctype中的字符处理函数

| 函数原型 | 说明 |
|:-:|:-:|
|int isalnum(int)|是否是字母或数字|
|int isalpha(int)|是否是字母|
|int iscntrl(int)|是否是控制字符（\0~\37，\127删除符，不包括空格）|
|int isdigit(int)|是否是数字|
|int isgraph(int)|是否是有图形的字符（字母数字 + 标点符号）|
|int islower(int)|是否是小写字母|
|int isprint(int)|是否是可打印字符（字母数字 + 标点符号 + 空格）|
|int ispunct(int)|是否是标点符号|
|int isspace(int)|是否是空白字符（空格，\t，\v，\r，\n，\f）|
|int isxdigit(int)|是否是16进制数字（数字 + A~F + a~f）|
|int tolower(int)|若是大写字母则返回小写，否则返回原字符|
|int toupper(int)|若是小写字母则返回大写，否则返回原字符|

> 标点符号包括 ! " # $ % & ' ( ) * + , - . / : ; < = > ? @ [ \ ] ^ _ ` { | } ~
