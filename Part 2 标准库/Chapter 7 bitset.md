# Chapter 7 bitset

> 低编号为低位，高编号为高位

## 1. 支持的操作

```C++
/* b 有 n（size_t）位，每一位均为 0
 *默认构造函数是 constexpr 的
 */
bitset<n> b;
/* 使用 u（unsigened long long）的低位初始化 b
 * 若 n 比 unsigned long long 还长，则高位补 0
 * 该构造函数是 constexpr 的
 */
bitset<n> b(u);
/* 使用 s（string）从 pos（size_t）开始 m（size_t）个字符的拷贝初始化 b
 * zero（char）指定 0 对应的字符，one（char）指定 1 对应的字符
 * 默认值如下：pos = 0, m = string::npos, zero = '0', one = '1'
 * 只指定一个字符时，指定的是 one
 * s 也可以是字符数组，但如果不指定 m，s 必须是 C 字符串
 * 若 s 包含 zero 和 one 以外的字符，则抛出 invalid_argument 异常
 * 若 s 长度不足 n，则高位补 0
 * 该构造函数是 explicit 的
 */
bitset<n> b(s, pos, m, zero, one);
// 初始化为全 1 的常用方法
bitset<n> b(~0ULL);

// pos 为 size_t 类型，v 为 bool 类型，zero 和 one 为 char 类型
// os 为 ostream 类型，is 为 istream 类型

// 以下函数返回 bool
b.any(); // 是否有 1
b.all(); // 是否全 1
b.none(); // 是否全 0
b.test(pos); // 测试 pos 对应的位是否为 1

b[pos]; // b 为 const 时返回 bool 值，否则返回 bitset<n>::reference 类型

b.count(); // 返回 1 的个数（size_t）
b.size(); // 返回位数（constexpr size_t）

// 以下函数返回自身引用
b.set(); // 将所有位置位
b.set(pos, v); // 将 pos 位置置为 v
b.reset(); // 将所有位复位
b.reset(pos); // 将 pos 位复位
b.flip(); // 将所有位取反
b.flip(pos); // 将 pos 位取反

b.to_ulong(); // 返回 unsigned long 值，若 b 太长，则抛 overflow_error 异常
b.to_ullong(); // 返回 unsigned long long 值，若 b 太长，则抛 overflow_error 异常

b.to_string(zero, one); // 返回 string，zero 和 one 默认为 '0' 和 '1'

os << b; // 用 '0' 和 '1' 打印
is >> b; // 用 '0' 和 '1' 写入，直到下一个字符不是 '0' 或 '1' 或达到 b.size()
```

对 `bitset<n>::reference` 对象可以操作单独的位，可以使用位运算符  
