# Chapter 2 容器和顺序容器

> 容器：特定类型对象的集合  
> 顺序容器：元素的存储和访问顺序不依赖元素的值的容器  

## 1. 顺序容器的类型和特性

- `vector` 可变大小数组，可随机访问，尾部外插入和删除元素较快
- `array` 固定大小数组，可随机访问，不能添加或删除元素
- `deque` 双端队列，可随机访问，头尾部插入和删除元素较快
- `list` 双向链表，可双向顺序访问，在任意位置插入和删除元素都较快
- `forword_list` 单向链表，可单向顺序访问，在任意位置插入和删除元素都较快
- `string` 和 `vector` 类似，但专门用于存储字符

`forword_list` 不支持 `size` 操作，其他顺序容器的 `size` 操作均为常数时间操作  

## 2. 选择容器的基本原则

- 通常使用 `vector`
- 元素数量较多但是大小较小，且空间开销必须很低的场合，不要使用链表
- 要求随机访问，应使用 `vector` 和 `deque`
- 要求在中间位置插入和删除元素，应使用链表
- 要求在头尾位置插入和删除元素，应使用 `deque`
- 要求在读取输入时在中间位置插入元素，并且需要随机访问，则有下面两种方法
    1. 使用 `vector`，在尾部插入元素，再使用 `sort`
    2. 在输入阶段使用链表，在中间插入元素，之后再将链表中的内容拷贝到 `vector` 中
- 尽量使用迭代器，避免随机访问，这样容器选择更灵活

## 3. 通用的容器操作（不止顺序容器）

类型别名|说明
:-:|:-:
`iterator`|迭代器类型
`const_iterator`|不可修改元素的迭代器类型
`size_type`|容器大小类型
`difference_type`|迭代器距离类型
`value_type`|元素类型
`reference`|相当于 `value_type&`
`const_reference`|相当于 `const value_type&`
`reverse_iterator`|逆序迭代器类型
`const_reverse_iterator`|不可修改元素的逆序迭代器类型

```C++
// 设容器类型为 C

// 构造函数
C c; // 构造空容器，array 类型较特殊
C c1(c2); // 构造 c2 的拷贝 c1，若为 array 类型则必须大小相等
C c1 = c2; // 同上
C c(b, e); // 将迭代器 b 和 e 指定的范围内的元素拷贝给 c（array 不支持）
C c{ele1, ele2, ...}; // 列表初始化，若为 array 类型则必须大小相等

// 除 array 以外的顺序容器支持的构造函数
C c(n); 
/* 使容器包含 n （size_type）个元素，每个元素都使用默认构造函数初始化
 * 或值初始化，是 explicit 的，string 不支持，无默认构造函数的元素类型
 * 也不支持
 */
C c(n, t); // 使容器包含 n 个值为 t 的元素（使用拷贝初始化）

// 赋值（元素使用拷贝初始化）
c1 = c2; // 用 c2 中的元素替换 c1 中的元素
c1 = {ele1, ele2, ...}; // 用列表中的元素替换 c1 中的元素（array 不支持）

// 所有容器都支持相等运算符（==，!=）
// 无序关联容器不支持大小关系运算符（>，<，>=，<=）
// 这两类运算符两边的容器和元素类型必须相同
// == 和 != 运算符是逐个调用元素的 == 运算符实现的，其他的是逐个调用
// 元素的 < 运算符实现的，若元素类未定义相应的运算符（const 成员），则无法比较

// 方法
c1.swap(c2); // 无返回值，交换 c1 和 c2 的元素，效率比 c1 = c2 高
swap(c1, c2); // 同上
c.size(); // 返回元素数量（单向链表不支持）（size_type）
c.max_size(); // 返回 c 可存储的最大元素数量（size_type）
c.empty(); // 返回 bool 类型，检查是否为空
c.insert(arguments); // 插入元素（不同容器类型接口不同）
c.emplace(arguments); // 构造元素（不同容器类型接口不同）
c.erase(arguments); // 删除元素（不同容器类型接口不同）
c.clear(); // 无返回值，清空元素

// 迭代器
c.begin(); c.end(); // 返回首尾迭代器，注意 end 指向尾元素的后一个位置
c.cbegin(); c.cend(); // 返回 const_iterator 版本的首尾迭代器

// 反向容器的迭代器（单向链表不支持）
c.rbegin(); c.rend();
c.crbegin(); c.crend();

// assign 操作（不适用于关联容器和 array）（拷贝初始化）
c.assign(b, e); 
/* 将 c 中的元素替换为 b 和 e 表示的迭代器范围中的元素
 * b 和 e 不能指向 c 中的元素
 */
c.assign(n, t); // 将 c 中元素替换为 n 个 t 元素
c.assign(ele1, ele2, ...) // 将 c 中元素替换为初始化列表中的元素
```

## 4. 迭代器

### （通用）迭代器类型的运算

```C++
*iter; // 返回iter所指元素的引用
iter->member; // 等价于(*iter).member；须注意成员运算优先级高于解引用运算
++iter; // 令iter指向下一个元素
--iter; // 令iter指向上一个元素（单向链表不支持）
iter1 == iter2; // bool
iter1 != iter2; // bool
```

### const_iterator 类型

该类迭代器只能读不能写，如果要迭代的对象是常量，则必须使用 `const_iterator`  
常量的 `begin` 和 `end` 方法调用的是重载版本（`const` 成员函数）返回 `const_iterator` 类型  
非常量可以使用 `cbegin` 和 `cend` 方法返回 `const_iterator` 类型迭代器  
可以将 `iterator` 转化为 `const_iterator`，反之则不行（类似指向 `const` 类型的指针和引用）  
不需要写内容时尽量使用 `const_iterator`

### string，vector，deque，array 迭代器额外支持的运算

```C++
iter + n; // n为ptrdiff_t （可正可负）（g++是long）
iter - n; // n为ptrdiff_t
iter += n;
iter -= n;
iter1 - iter2; // 值为 ptrdiff_t
// 还有大小比较运算符
```

### 迭代器范围

由指向同一个容器的 `begin` 和 `end` 两个迭代器对象表示，左闭右开  
`begin == end`，则范围为空，否则至少包含一个元素  
能通过反复递增 `begin` 来到达 `end`  

### 迭代器、指针和引用的失效

改变容器后最好重新定位迭代器，即尽量使用插入和删除函数的返回值  
不要使用变量保存 `end()` 返回的迭代器

#### 添加元素

`vector` 和 `string` 的存储空间重新分配时三者均失效  
未重新分配时，`vector` 和 `string` 的插入位置之后的三者均失效  
`deque` 在首尾位置以外插入时三者均失效  
`deque` 在首尾位置插入时只有迭代器失效

#### 删除元素

`vector`、`string` 和 `deque` 的尾后迭代器总会失效  
`deque` 在首尾之外删除时，指向剩下元素的三者均失效  
`deque` 在首尾位置删除时，除尾后迭代器以外三者均不会失效

## 5. 类型成员

使用时需显式指定类名  
在不了解元素类型的情况下很有用  

## 6. 初始化和赋值的注意事项

拷贝初始化和赋值必须保证二者元素类型匹配（可转换到目标容器元素类型），容器类型也需相同（教材描述好像有问题）  
使用 `assign` 可以实现不同容器类型相容元素类型的赋值操作  
使用迭代器初始化注意左闭右开  
`array` 初始化时元素类型和大小（`size_t`）同时作为其元素类型的一部分  
`array` 在默认初始化和大括号元素数量不足的初始化时，会将剩余元素全部值初始化  
`array` 可以进行拷贝初始化和赋值操作，而数组则不行  
`array` 不支持花括号赋值操作

```C++
// 有趣的例子
class A
{
public:
    A(int n): m(n) {}
    A(const A &obj)
    {
        m = 100;
    }
    int m;
};

int main()
{
    vector<A> c1(10, 2);
    // 初始化元素时，先调用转换构造函数，把 (int)2 转化为 A(2)
    // 再调用拷贝构造函数，使 m 等于 100
    cout << c1.at(3).m << endl; // 输出 100
    return 0;
}
```

## 7. swap 的注意事项

`swap` 只能交换两个完全相同的容器类型的内容  
`swap` 操作（除 `array` 外）不会移动元素，而是改变两个容器的数据组织形式，可以在常数时间内完成  
`swap` 之后（除 `array` 外），指向容器元素的迭代器、指针和引用指向的位置不会改变，如 `iter` 原来指向 `c1[n]`, `swap(c1, c2)` 之后会指向 `c2[n]`  
对 `string` 容器调用 `swap` 会导致原来的迭代器、指针和引用均失效  
尽量使用非成员版本的 `swap`  

## 8. 添加元素

### 添加元素的注意事项

`array` 不支持添加元素的操作  
添加元素可能导致重新分配存储空间，会导致元素从旧空间移到新空间中  
添加到容器中的元素是拷贝，与原本添加的对象无关  
向 `vector`、`string`、`deque` 添加元素会使原本的迭代器、指针和引用失效  
`emplace` 版本的方法是不拷贝，直接构造

### push_back，emplace_back

将一个元素放到容器末尾，无返回值，单向链表不支持  

### push_front，emplace_front

将一个元素放到容器头部，无返回值，单向链表、`vector`、`string` 不支持

### insert，emplace（非单向链表版本）

```C++
c.insert(p, t); // 在 p 之前插入 t，返回指向新元素的迭代器
c.emplace(p, args); // 在 p 之前构造新元素，返回指向新元素的迭代器
c.insert(p, n, t);
/* 在 p 之前插入 n 个 t 拷贝初始化，返回第一个添加元素的迭代器
 * 若 n 为 0，则返回 p
 */ 
c.insert(p, b, e);
/* 在 p 之前插入 b 和 e 构成的迭代器范围内的元素（拷贝初始化），
 * 返回第一个添加元素的迭代器
 * 若 b == e，则返回 p，b 和 e 不能指向 c 中的元素
 */
c.insert(p, {ele1, ele2, ...});
/* 在 p 之前插入花括号列表内的元素（拷贝初始化），
 * 返回第一个添加元素的迭代器
 * 若列表为空，则返回 p
 */
```

```C++
// 利用 insert 的返回值不断向开头插值
list<string> lst;
auto iter = lst.begin();
while (cin << word)
    iter = lst.insert(iter, word);
    // 等价于 push_front(word);
```

## 9. 访问元素

```C++
c.back(); // 返回指向尾元素的引用，若 c 为空则行为未知
c.front(); // 返回指向首元素的引用，若 c 为空则行为未知
c[n]; // 返回下标为 n（size_type） 的元素的引用，若 n >= c.size() 则行为未知
c.at(n); // 同上，但下标越界时会抛出 out_of_range 异常
```

若容器是 `const` 的，则返回 `const` 引用，反之返回普通引用  
须注意若要藉此改变容器中元素的值，必须用引用类型变量接收返回值  
下标操作必须是支持随机访问的容器才支持  

## 10. 删除元素

### 删除元素的注意事项

单向链表不支持`popback`  
`vector` 和 `string` 不支持 `popfront`  

### popback 和 popfront

分别是删除尾元素和首元素，若容器为空则行为未定义，无返回值

### erase（非单向链表版本）和 clear

可以删除一个迭代器指向的元素，也可以删除迭代器范围内的元素，返回删除的最后一个元素的后一个位置的迭代器，即 `erase(b, e)` 返回 `e`，调用函数后 `b == e`  
若需删除所有元素可以直接调用 `clear`

## 11. 单向链表的特殊操作

> 首前迭代器：头元素前面一个位置的迭代器，不可解引用

```C++
forword_list<T> lst;
lst.before_begin(); // 返回首前迭代器
lst.cbefore_begin(); // 固定返回 const_iterator 版本的首前迭代器

// 以下五个插入函数与通常顺序容器类似，但是是在 p 后面的位置插入
// 若 p 是尾后迭代器，则函数行为未定义
lst.insert_after(p, t);
lst.emplace_after(p, t);
lst.insert_after(p, n, t);
lst.insert_after(p, b, e);
lst.insert_after(p, {ele1, ele2, ...});

// 以下两个删除函数与通常顺序容器类似，但是是删除 p 后面的位置的元素
// 或 [b, e) 内的元素，返回最后一个被删除元素后面位置的迭代器（可能是尾后迭代器）
// 若 p 指向尾元素或是尾后迭代器则函数行为未定义
lst.erase_after(p);
lst.erase_after(b, e);

// 单向链表无 insert、emplace、erase 方法
```

单向链表添加和删除元素同时处理当前迭代器和前驱元素的迭代器  

```C++
// 删除奇数元素
forword_list<int> flst = {0, 1, 2, 3, 4, 5, 6, 7};
auto prev = flst.before_begin(); // 前驱
auto curr = flst.begin(); // 当前
while (curr != flst.end())
{
    if (*curr % 2)
        curr = flst.erase_after(prev);
        // 删除 curr 指向的元素，并将 curr 移至被删元素的后面
    else
    {
        prev = curr;
        ++curr;
        // 两个迭代器均向前移动
    }
}
```

## 12. resize

改变容器大小，（不改变容量）`array` 不支持  
若使容器缩小，则会删除后面的元素，反之在后部添加元素  
添加元素时，若提供值则用提供的值拷贝初始化，否则值初始化  

## 13. 内存管理

```C++
// vector、string 和 deque 支持
c.shrink_to_fit() // 将容量减小为和 size 相同，但也可能不执行

// vector、string 支持
c.capacity() // 返回容量，若存的元素大于此值，则需重新分配内存
c.reserve(n) // 分配至少能容纳 n 个元素的空间
```

空 `vector` 的容量为 0  
分配空间的原则是：确保 `push_back` 有效率

## 14. string 的特殊操作

```C++
// 构造函数
string s(cp, n); 
/* cp 指向一个字符数组，数组大小至少为 n，
 * 拷贝 cp 的前 n 个字符到 s
 */
string s(cp);
/* 须注意 cp 必须以 '\0' 结尾 */
string s(s2, pos2);
/* s2 为一个 string 对象，pos2 应小于等于 size，否则抛 out_of_range 异常
 * 拷贝 s2 的从 pos2 下标开始的那一部分字符
 */
string s(s2, pos2, len2)
/* 拷贝 s2 的从 pos2 下标开始的 len2 个字符，pos2 应小于等于 size
 * 最多拷贝 size - pos2 个字符
 */

// 子字符串
s.substr(pos, n);
/* 返回一个包含 s 从 pos 开始 n 个字符的拷贝，pos 默认为 0，
 * n 默认值为 size - pos，若 pos 大于 size，则抛 out_of_range 异常
 */
```

### 修改 string 的操作

```C++
s.insert(pos, args);
/* 在 pos 前插入 args 指定的字符，若 pos 是下标，则返回 s 的引用
 * 若 pos 是迭代器，则返回指向第一个插入字符的迭代器
 */
s.erase(pos, len);
/* 删除从 pos 开始 len 个字符，如果未提供 len，则删除 pos 开始到最后的
 * 所有字符，返回 s 的引用
 */
s.assign(args);
/* 将字符串替换为 args 指定的字符，返回 s 的引用 */
s.append(args);
/* 将 args 追加到 s 后面，返回 s 的引用 */
s.replace(range, args);
/* 删除 range 范围内的字符，替换为 args 指定的字符，返回 s 的引用 */
```

`args` 可使用的形式：  

- `str`： 字符串，不能与 `s` 相同
- `str, pos, len`： `str` 从 `pos` 开始最多 `len` 个字符
- `cp`： 指向以空字符结尾的字符数组
- `cp, len`： 指向的字符数组的前最多 `len` 个字符
- `n, c`： `n` 个字符 `c`
- `b, e`： 迭代器范围，不能是指向 `s` 的迭代器
- 花括号字符列表

`args` 形式|`replace(pos, len, args)`|`replace(b, e, args)`|`insert(pos, args)`|`insert(iter, args)`
:-:|:-:|:-:|:-:|:-:
`str`|o|o|o|x
`str, pos, len`|o|x|o|x
`cp, len`|o|o|o|x
`cp`|o|o|x|x
`n, c`|o|o|o|o
`b, e`|x|o|x|o
花括号字符列表|x|o|x|o

`append` 和 `assign` 可以使用所有 `args` 形式

### string 搜索操作

```C++
// 以下搜索操作返回 size_type 类型的下标，或 npos

s.find(args); // 查找 args 第一次出现的位置
s.rfind(args); // 查找 args 最后一次出现的位置
s.find_first_of(args); // 查找 args 中的任意一个字符第一次出现的位置
s.find_last_of(args); // 查找 args 中的任意一个字符最后一次出现的位置
s.find_first_not_of(args); // 查找非 args 中的字符第一次出现的位置
s.find_last_not_of(args); // 查找非 args 中的字符最后一次出现的位置
```

`args` 可使用的形式：  

- `c, pos`：从 `s` 的 `pos` 位置开始查找字符 `c`，`pos` 默认为 0
- `s2, pos`：从 `s` 的 `pos` 位置开始查找字符串 `s2`，`pos` 默认为 0
- `cp, pos`：从 `s` 的 `pos` 位置开始查找 `cp` 指向的 C 字符串，`pos` 默认为 0
- `cp, pos, n`：从 `s` 的 `pos` 位置开始查找 `cp` 指向的字符数组的前 `n` 个字符，`pos` 和 `n` 无默认值

```C++
// 搜索所有数字的出现位置

string nums("0123456789")
string::size_type pos = 0;
string s;
std::cin >> s;
while ((pos = s.find_first_of(nums, pos) != string::npos))
{
    /* 处理 s.at(pos) */
    ++pos;
}
```

### compare

与 `cstring` 中的 `strcmp` 类似，返回正数（大于），负数（小于），或 0（等于）  

参数可使用的形式：

- `s2`：比较 `s` 和 `s2`
- `pos1, n1, s2`：将 `s` 的从 `pos1` 开始的 `n1` 个字符与 `s2` 比较
- `pos1, n1, s2, pos2, n2`：将 `s` 的从 `pos1` 开始的 `n1` 个字符与 `s2` 从 `pos2` 开始的 `n2` 个字符比较
- `cp`：比较 `s` 和 `cp` 指向的 C 字符串
- `pos1, n1, cp`：将 `s` 的从 `pos1` 开始的 `n1` 个字符与 `cp` 指向的 C 字符串比较
- `pos1, n1, cp, n2`：将 `s` 的从 `pos1` 开始的 `n1` 个字符与 `cp` 指向的字符数组的前 `n2` 个字符比较

### string 与数值间的转换

若无法转换为数值，则抛出 `invalid_argument` 异常，若转换出的数值无法用任何一种类型来表示，则抛出 `out_of_range` 异常  

```C++
to_string(val); // val 可以是任意算术类型，小整型会被提升
stoi(s, p, b); 
/* 返回 s 的起始字串（表示数值内容）的数值，返回类型是 int
 * p 是 size_t 指针，用来存储第一个非数值字符的下标
 * b 是 int，表示转换所用的基数，默认为 10
 */
stol(s, p, b); // long
stoul(s, p, b); // unsigned long
stoll(s, p, b); // long long
stoull(s, p, b); // unsigned long long
stof(s, p, b); // float
stod(s, p, b); // double
stold(s, p, b); // long double
```

## 15. 顺序容器适配器

> 适配器：使容器、迭代器和函数的行为看起来像另一种事物  
> 容器适配器：使容器的行为看似不同的类型  
> 顺序容器适配器：`stack`、`queue` 和 `priority_queue`  

### 容器适配器的类型成员

`size_type`：大小类型  
`value_type`：元素类型  
`container_type`：适配器的底层容器类型  

### 容器适配器的操作

```C++
A a; // 创建空适配器
A a(c); // 创建带有容器 c 的拷贝的适配器

// 可使用关系运算符比较适配器，返回底层容器的比较结果

a.empty();
a.size();
a.swap(b); // 须注意交换时不光 a 和 b 的类型要完全相同，底层容器类型也要完全相同
swap(a, b);
```

适配器使用的容器必须支持添加、删除和访问尾元素的操作，`array` 和 `forword_list` 不可用  
可使用顺序容器类型作为第二个类型参数来重载适配器的默认实现类型，如

```C++
vector<int> vec;
stack<int, vector<int>> stk(vec);
```

只能使用适配器规定的操作，不能使用底层容器的操作  

### 栈适配器

```C++
// 要求容器支持 push_back, pop_back 和 back 操作
// 默认基于 deque 实现，可以基于 vector, string 和 list 实现
s.pop(); // 删除栈顶元素，不返回值
s.push(item); // 将 item 压入栈顶，该元素是 item 的移动或拷贝，无返回值
s.emplace(args); // 在栈顶依据 args 创建新元素，无返回值
s.top(); // 返回栈顶元素的引用，不删除值
```

### 队列适配器

```C++
// queue 要求容器支持 back, push_back, front 和 push_front
// priority_queue 要求容器支持 front, push_back, pop_back 和随机访问
// queue 默认基于 deque 实现，可以基于 vector, string 和 list 实现
// priority_queue 默认基于 vector 实现，可以基于 string 和 deque 实现
q.pop(); // 删除队首元素，无返回值
q.front(); // 返回队首元素的引用，不删除值，只适用于 queue
q.back(); // 返回队尾元素的引用，不删除值，只适用于 queue
q.top(); // 返回最高优先级元素的引用，不删除值，只适用于 priority_queue
q.push(item); 
/* 在 queue 队尾或 priority_queue 的合适位置插入 item，
 * 该元素是 item 的移动或拷贝，无返回值
 */
q.emplace(args); // 同上，但根据 args 创建新元素
```

`priority_queue` 默认根据 `<` 运算符来确定优先级，大元素优先级高  
