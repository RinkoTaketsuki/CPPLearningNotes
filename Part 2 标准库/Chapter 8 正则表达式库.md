# Chapter 8 正则表达式库

`regex` 类表示一个正则表达式  

## 1. regex_match 和 regex_search

这两个函数均返回 bool，分别表示字符串匹配正则表达式和存在子串匹配表达式，有两种输入参数  

1. `seq, r, mft`：`seq` 表示字符串或迭代器范围或指向 C 字符串的指针，`r` 是 `regex` 对象，  
    `mft` 是 `regex_constants::match_flag_type` 值，可选  
2. `seq, m, r, mft`：`m` 为各种 match 对象，保存匹配结果

```C++
// 使用例
string pattern("magnet:\?xt=urn:btih:.+");
regex r(pattern);
smatch results;
string test_str("magnet:\?xt=urn:btih:DEADBEEF123");
if (regex_search(test_str, results, r))
    cout << results.str() << endl; // 输出第一个匹配的子串
```

## 2. regex 选项

```C++
// 构造函数、赋值运算符、assign 会抛出 regex_error 异常

/* re 是正则表达式，可以是如下的类型：
 * 1. string
 * 2. 迭代器范围
 * 3. 花括号字符列表
 * 4. 指向 C 字符串的指针
 * 5. 字符指针 + size_t 类型的长度
 * 6. 另一个 regex 对象
 */
regex r(re);
/* f 为标志集（regex_constants::syntax_option_type）
 * 默认为 regex::ECMAScript
 * 每种选项间以或运算符连接
 */
regex r(re, f);
/* 将 regex 对象的正则表达式替换为 re，可以是如下的类型：
 * 1. string
 * 2. 另一个 regex 对象
 * 3. 花括号字符列表
 * 4. 指向 C 字符串的指针
 */ 
r = re;
/* 将 regex 对象的正则表达式替换为 re，标志集替换为 f
 * re 的可选类型与构造函数相同，返回自身引用
 */
r.assign(re, f);
// 返回 r 中子表达式的数目（unsigned int）
r.mark_count();
// 返回 r 的标志集（regex_constants::syntax_option_type）
r.flags();
```

标志|含义
:-|:-
icase|忽略大小写
nosubs|不保存匹配的子表达式
optimize|执行速度优先于构造速度
ECMAScript|使用 ECMA-262 指定的语法
basic|使用 POSIX 基本的正则表达式语法
extend|使用 POSIX 扩展的正则表达式语法
awk|使用 POSIX 的 awk 语言的正则表达式语法
grep|使用 POSIX 的 grep 语法
egrep|使用 POSIX 的 egrep 语法

```C++
// 匹配 C++ 源文件名：至少一个大小写字母或数字加上点加上缩略名，忽略大小写
regex r("[[:alnum:]]+\.(cpp|cxx|cc)$", regex::icase);
```

正则表达式本身是在运行时才编译的，其编译非常慢，应尽量避免在循环内创建正则表达式  

## 3. regex_error

编写的正则表达式存在错误时会抛出此异常  
`e.what()` 返回信息，`e.code()` 返回错误类型编号  
错误类型为编号下表，命名空间为 `regex_constants`，类型为 `regex_constants::error_type`

错误类型编号|含义
:-|:-
error_collate|无效的元素校对请求
error_ctype|无效的字符类
error_escape|无效的转义字符或尾置转义
error_backref|无效的向后引用
error_brack|不匹配的方括号
error_paren|不匹配的小括号
error_brace|不匹配的花括号
error_badbrace|{}中无效的范围
error_range|无效的字符范围（如[z-a]）
error_space|内存不足，无法处理正则表达式
error_badrepeat|重复字符（*、?、+ 或 {）之前没有有效的正则表达式
error_complexity|要求的匹配过于复杂
error_stack|栈空间不足，无法处理匹配

## 4. 正则表达式类前缀

基本的正则表达式类有 `regex`，`match`，`sub_match`，`regex_iterator`  
`wregex`：正则表达式是 `wchar_t` 表示的  
剩下三种正则表达式类前面要加四种前缀：  
s：处理字符串  
c：处理字符数组  
ws：处理宽字符串  
wc：处理宽字符数组  

## 5. regex 迭代器
