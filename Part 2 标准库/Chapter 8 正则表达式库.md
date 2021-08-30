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
s：处理字符串（`string`）  
c：处理字符数组（`const char*`）  
ws：处理宽字符串（`wstring`）  
wc：处理宽字符数组（`const wchar_t*`）  

## 5. regex 迭代器

类型用四种前缀加上 `regex_iterator` 表示

```C++
// 以 sregex_iterator 类举例
sregex_iterator end; // 默认为尾后迭代器
sregex_iterator it(b, e, r) // 调用 regex_search(b, e, r) 搜索，it 表示第一个匹配位置

*it; it->mem;// 返回 smatch 引用
++it; it++; // 从当前匹配位置开始调用 regex_search

// 二者均为尾后迭代器时相等
// 二者的搜索范围和 regex 对象相同时相等
it1 == it2;
it1 != it2;
```

## 6. 匹配结果类

```C++
// 以下方法适用于四种 match 类和四种 sub_match 类

/* 返回 bool，表示 m 是否被 regex_search 或 regex_match 调用过
 * 若返回 false，则对 m 的其他操作结果为未定义的
 */
m.ready();
// 返回 size_t，若匹配失败则返回 0，否则返回子表达式数目
m.size();
// 返回 bool，表示 m.size() 是否为 0
m.empty();
// 返回 sub_match 的常量引用，表示当前匹配之前的序列
m.prefix();
// 返回 sub_match 的常量引用，表示当前匹配之后的序列
m.suffix();

/* 以下方法中的：
 * n 必须大于等于 0 小于 m.size()
 * n 默认值为 0，n == 0 时表示整个匹配
 */

// 返回 ptrdiff_t，表示第 n 个子匹配的大小
m.length(n);
// 返回 ptrdiff_t，表示第 n 个子匹配的开始位置
m.position(n);
// 返回 string，表示第 n 个子匹配的字符串
m.str(n);
// 返回第 n 个子匹配的 sub_match 常量引用
m[n];

/* 以下迭代器迭代 sub_match
 * 总是返回 smatch::const_iterator
 */
m.begin(); m.end();
m.cbegin(); m.cend();
```

```C++
// 使用例
string toBeSearched;
string pattern;
regex_constants::syntax_option_type flags;
regex r(pattern, flags);

// 只打印匹配字符串
for (sregex_iterator it(toBeSearched.cbegin(), toBeSearched.cend(), r), end_it;
    it != end_it; ++it)
    cout << it->str(); << endl;

// 利用 smatch 的 prefix 和 suffix 成员实现打印匹配的前后内容
// prefix 和 suffix 返回 ssub_match 对象
for (sregex_iterator it(toBeSearched.cbegin(), toBeSearched.cend(), r), end_it;
    it != end_it; ++it)
{
    auto pos = it->prefix().length();
    pos = pos > 40 ? pos - 40 : 0; // 前缀最多取 40 个字符
    cout << it->prefix().str().substr(pos) // 取前缀的后 40 个字符
        << "\n\t\t>>> " << it->str() << " <<<\n" // 格式化输出匹配字符串
        << it->suffix().str().substr(0, 40) << endl // 取后缀的前 40 个字符
}
```

## 7. 子表达式

正则表达式中使用小括号表示子表达式  

```C++
regex r("([[:alnum:]]+)\.(cpp|cxx|cc)", regex::icase);
string s;
smatch results;
regex_search(s, results, r);
// 打印去掉点和缩略名的文件名
cout << results.str(1) << endl;
```

```C++
// 四种 sub_match 类特有成员

// bool 型，指出是否已匹配
m.matched;
// 指向匹配序列首元素和尾后位置的迭代器（string::const_iterator）
// 未匹配时 first == second
m.first; m.second;
// 匹配序列的长度的大小（ptrdiff_t），matched == false 时返回 0
m.length();
// 返回匹配序列
m.str();
// ssub_match 对象可转化为 string 对象，转化非 explicit，即可以直接拷贝或赋值给 string 对象
```

## 8. regex_replace

```C++
// mft 默认值均为 match_default

m.format(dest, fmt, mft); // 向迭代器 dest 格式化插入数据，返回插入序列后面位置的迭代器
m.format(fmt, mft); // 返回 string 保存格式化结果
regex_replace(dest, seq, r, fmt, mft); // 使用 fmt 替换数据，结果写入 dest 迭代器
regex_replace(seq, r, fmt, mft); // 返回 string 保存替换结果
```

`match_flag_type` 值|含义
:-|:-
`match_default`|等价于 `format_default`
`match_not_bol`|首字符不作为行首
`match_not_eol`|尾字符不作为行尾
`match_not_bow`|首字符不作为单词首
`match_not_eow`|尾字符不作为单词尾
`match_any`|返回任意一个匹配
`match_not_null`|不匹配空序列
`match_continuous`|匹配结果必须从首字符开始
`match_prev_avail`|输入序列必须包含第一个匹配之前的内容
`format_default`|使用 ECMAScript 规则替换
`format_sed`|使用 POSIX sed 规则替换
`format_no_copy`|不输出未匹配部分
`format_first_only`|只替换子表达式的第一次出现
