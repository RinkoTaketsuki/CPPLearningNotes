# Chapter 3 泛型算法

> 泛型算法本身不会执行容器的操作，故不会改变容器大小  
> 泛型算法执行迭代器的操作和元素的操作，如元素的 `==` 和 `<` 运算符  
> 泛型算法的范围可以是迭代器，也可以是指针

## 1. 只读算法

> 只读取输入范围内的元素而不改变元素  
> 通常使用 `cbegin` 和 `cend`，除非需要利用算法的返回迭代器改变元素值  
> 接受两个输入序列但第二个输入序列只有一个参数的，都假定第一个序列和第二个序列一样长  

`accumulate` 接受三个参数，求和范围和求和初值，初值决定了其使用的是哪类对象的 `+` 运算符以及返回类型  

```C++
vector<string> v;
string sum = accumulate(v.cbegin(), v.cend(), "");
// 错误，const char[1] 类型无 + 运算符
// 把第三个参数换成 string("") 则正确
```

`find` 接受三个参数，查找范围和要查找的值，需元素支持 `==` 运算符，返回值指向查找的数据  
`equal` 接受三个参数，第一个序列的比较范围和第二个序列的开头位置，需元素支持 `==` 运算符，返回 `bool`  

## 2. 写容器元素的算法

`fill` 接受三个参数，填充范围和填充值，需元素支持赋值运算符，无返回值  
`fill_n` 接受三个参数，开头位置，填充数量（size_type）和填充值，需元素支持赋值运算符，返回被填充的尾元素后面的位置  
`for_each` 接受三个参数，作用范围和作用函数，返回作用函数的移动  
`transform` 接受四个参数，转换范围，写入目标和转换函数，需元素支持赋值运算符，返回被写入的尾元素后面的位置，转换范围和写入目标的迭代器可以指向同一个容器  

## 3. 插入迭代器

通过插入迭代器赋值时，赋值运算符右边的值会被插入容器（拷贝初始化）  
对插入迭代器执行解引用和自增操作返回的是迭代器本身，不会对迭代器做任何事情

### back_inserter

通过此迭代器赋值时，赋值运算符会调用 `push_back` 插值  

### front_inserter

通过此迭代器赋值时，赋值运算符会调用 `push_front` 插值  

### inserter

通过此迭代器赋值时，赋值运算符会调用 `insert` 插值  
该函数接受第二个参数，该参数必须是指向给定容器的迭代器，元素会被插到该迭代器之前  
此迭代器的插值位置不会变，始终为第二个参数的前面

```C++
int var = 1;
vector<int> vec(5);
auto iter = vec.begin() + 3;
auto ins_iter = inserter(vec, iter);

*ins_iter = var;
/* {0, 0, 0, 1, 0, 0}
 *              ^ins_iter
 */
```

## 4. 拷贝算法

`copy` 接受三个参数，拷贝范围和目的序列，目的序列至少要包含与输入序列一样多的元素，需元素支持赋值运算符，返回目的序列拷贝的尾元素后面的位置  
`replace` 接受四个参数，替换范围，查找值和替换值，需元素支持 `==` 运算符和赋值运算符，无返回值  
`replace_copy` 接受五个参数，查找范围，替换后序列的保存位置，查找值和替换值，不会改变原序列的值，需元素支持 `==` 运算符和赋值运算符，返回目的序列拷贝的尾元素后面的位置  

## 5. 重排容器元素的算法

`sort` 接受三个参数，排序范围和排序谓词，默认从小到大排序，默认情况需元素支持 `<` 运算符，无返回值
`stable_sort` 与 `sort` 基本相同，但排序后元素相对位置不变  
`unique` 接受三个参数，去重范围和去重谓词，默认使用 `==` 运算符去重，将不重复的元素放在靠前的位置，返回第一个重复元素的位置  

## 6. lambda

> 可调用对象：定义了 `()` 调用运算符的对象，包括函数，函数指针和 lambda 表达式  

```C++
auto function = [/*capture list*/] (/*parameter list*/) -> /*return_type*/ {/*function body*/};
// 参数列表和返回类型可以不写，捕获列表和函数体可以为空，只能使用尾置返回
// 不写参数列表相当于参数列表为空，不写返回类型则自动判断返回类型
// 自动判断返回类型的规则是，如果函数体不是只有一个 return 语句，无其他语句，则一律为 void
// 空捕获列表表示不使用其所在函数中的任何局部变量，空捕获列表的 lambda 对象可以用函数代替
```

对于所在函数的非 `static` 变量，lamdba 函数体只能使用捕获列表中包含的变量，但可以直接使用所在函数之外的名字，与通常的作用域不同  

```C++
void fun()
{
    int var = 42;
    auto function = [var] {return var}; // 值捕获，被捕获的值是 var 的拷贝，var 不可改变
    auto function = [&var] {return var}; 
    /* 引用捕获，被捕获的值是 var 的引用，如果 function 在函数结束后执行，会捕获不到 var
     * 引用捕获的对象能否修改取决于其指向类型是否是 const 的
     */
    auto function = [=] {return var}; // 隐式值捕获
    auto function = [&] {return var}; // 隐式引用捕获
    auto function = [var] () mutable {return ++var;} // 可变 lambda，使值捕获的对象能改变
}
```

有些变量无法拷贝，只能使用引用捕获  
如果返回 lambda 对象，则返回的 lambda 对象中不能有引用捕获，因为不能返回局部变量的引用  
尽量减少捕获的数据量，避免捕获指针和引用变量  
混合使用隐式和显式捕获时，捕获列表的第一个元素必须是 `=` 或 `&`，显式捕获的方式和隐式捕获必须不同  

### bind

一种函数适配器，接受一个可调用对象，生成新的可调用对象来“适应”原对象的参数列表

```C++
auto newCallable = bind(callable /*, arg_list*/);
// 调用 newCallable 时，会利用 arg_list 中的参数调用 callable
// 占位符 _x 表示 newCallable 的第 x 个参数，占位符在 std::placeholders 命名空间中

// 一个例子
void fun(const double &var1, const int &var2, const string &var3) {}
auto newFun = bind(fun, _2, _1, "hello");
// newFun 接受两个参数，因为 _1 是 arg_list 的第 2 个参数，故 newFun 的第一个参数会赋给 fun 的第二个参数
// 同理，newFun 的第二个参数会赋给 Fun 的第一个参数
// "hello" 会赋给 Fun 的第三个参数
// 这样做使得原本需要三个参数的函数变成了只需要两个参数的函数，可以用于向泛型算法传递合适的谓词参数
// newFun 是新生成的可调用对象，调用 newFun 不会导致 fun 被调用，但非占位符参数会被拷贝到 newFun 对象中
// 因此非占位符参数不能是不可拷贝的，如需使用不可拷贝的参数需要使用 ref 或 cref
void fun2(ostream &os) {}
ostream os_obj;
auto newFun2 = bind(fun2, ref(os_obj));
// ref 返回一个包含有 os 的引用的对象，该对象可以拷贝
// cref 类似，返回包含 os 的 const 引用的对象
```

## 7. iostream 迭代器

将流作为序列来处理  

### istream_iterator

绑定的流必须支持 `>>` 运算符  
默认初始化的 `istream_iterator` 是尾后迭代器  

```C++
// istream_iterator 支持的操作

// 初始化
istream_iterator<T> iter; // 读取 T 类型的尾后迭代器
istream_iterator<T> iter(is); // 输入流为 is，读取类型为 T

// 相等和不等运算符
/* 两个输入流迭代器必须读取类型相同才能比较，当二者绑定的输入流相同
 * 或均为尾后迭代器是二者相等
 */

*iter; // 返回读取的值
iter->mem; // 访问读取的值的 mem 成员

// 自增运算符
/* 读取下一个值，前置和后置版本表达含义与通常情况相同
 */
```

```C++
// 从 cin 读取整数值
istream_iterator<int> iIter(cin);
istream_iterator<int> eofIter;
vector<int> vec;
while (iIter != eofIter)
{
    // 当 iIter 达到 EOF 或 IO 错误时，其值与尾后迭代器相等
    vec.push_back(*iIter++);
}
// 上面的 vector 声明和 while 语句与下面的语句等效
vector<int> vec = (iIter, eofIter);
```

```C++
// 计算标准输入中输入的整数值的和
istream_iterator<int> in(cin), eof;
cout << accumulate(in, eof, 0) << endl;
```

输入流迭代器只保证在第一次解引用之前读取值，当创建了一个未解引用过便销毁的迭代器或两个不同的迭代器读取同一个流时需注意此问题  

### ostream_iterator

绑定的流必须支持 `<<` 运算符  
不允许默认初始化或使用表示尾后位置的输出流迭代器  

```C++
// ostream_iterator 支持的操作

ostream_iterator<T> iter(os);
ostream_iterator<T> iter(os, str);
/* iter 将 T 类型的值写到输出流 os 中，每写一个值后面额外加上 str
 * str 必须是 C 字符串
 */
iter = val; // 向输出流输出 val
*iter; ++iter; iter++; // 这三个操作不会做任何事情，返回 out 本身
```

```C++
// 向 cout 输出值，值之间用空格隔开
vector<int> vec = {1, 1, 4, 5, 1, 4};
ostream_iterator<int> oIter(cout, " ");
for (int val : vec)
    oIter = val; // 或写成 *oIter++ = val;
cout << endl;
// 以上输出语句与下面的语句等效
copy(vec.begin(), vec.end(), oIter);
cout << endl;
```

## 8. 反向迭代器

操作含义会发生颠倒，如 `++` 和 `--` 的方向会变反  
只能从同时支持 `++` 和 `--` 的迭代器定义反向迭代器，故单向链表和流不能建立反向迭代器  

```C++
// 递减排序，把较小元素放在末尾，较大元素放在开头
sort(vec.rbegin(), vec.rend());
```

```C++
// 处理逗号分隔的单词列表
string line("first,middle,last");
auto firstCommaPos = find(line.cbegin()/* 1 */, line.cend()/* 6 */, ','); /* 2 */
string firstWord(line.cbegin(), firstCommaPos); // "first"
auto lastCommaPos = find(line.crbegin()/* 5 */, line.crend()/* 0 */, ','); /* 3 */
string reversedLastWord(line.crbegin(), lastCommaPos); // "tsal"
string LastWord(lastCommaPos.base()/* 4 */, line.cend()); // "last"
//   f i r s t , m i d d l e , l a s t
// 0 1         2             3 4     5 6
```

`base` 相比原位置向后移位一个位置，这使得 `rIter1.base(), rIter2.base()` 和 `rIter1, rIter2` 表示的范围相同，方向相反  
将一个普通迭代器赋给反向迭代器或用于初始化反向迭代器也是同样的效果，二者指向的位置不同  

## 9. 泛型算法要求的迭代器类别

提供给泛型算法的迭代器类型要比要求的迭代器类型的层次更高，否则会出错  

### 输入迭代器

只读不写，单遍扫描，只能递增  

支持的操作：  

- `==` 和 `!=`  
- `++`
- 解引用和访问成员（`*`，`.` 和 `->`）

`*iter++` 操作会导致其他指向流的迭代器失效，故只能单遍扫描  
`find` 和 `accumulate` 要求输入迭代器  
`istream_iterator` 属于输入迭代器  

### 输出迭代器

只写不读，单遍扫描，只能递增  

支持的操作：

- `++`
- 解引用操作，`*iter` 只出现在赋值运算符的左侧，用于写入元素  

`*iter++` 操作会导致其他指向流的迭代器失效，故只能单遍扫描  
`copy` 的第三个参数要求输出迭代器  
`ostream_iterator` 属于输出迭代器  

### 前向迭代器

支持多次读写，只能递增  

`replace` 要求前向迭代器  
单向链表中的迭代器是前向迭代器  

### 双向迭代器

支持多次读写，可递增可递减  

除单向链表外其他容器均提供双向迭代器  

### 随机访问迭代器

可随机访问任意元素，在双向迭代器的基础上还具有如下功能：  

- 大小比较运算符  
- 加减（包括组合赋值）一个整数值  
- 两个迭代器相减  
- 下标运算符，`iter[n]` 和 `*(iter[n])` 等价  

`sort` 要求随机访问迭代器  
`vector`，`string`，`deque`，`array` 提供的都是随机访问迭代器  
访问内置数组的指针也是随机访问迭代器  

## 10. 泛型算法的形参模式  

```C++
alg(beg, end, others);
alg(beg, end, dest, others);
alg(beg, end, beg2, others);
alg(beg, end, beg2, end2, others);
```

### 接受单个 dest 的算法

算法假定向 dest 不管写入多少数据都是安全的  
通常使用插入迭代器或 `ostream_iterator`  

### 接受第二个序列的算法

只接受 `beg2` 的算法假定 `[beg1, end1)` 和从 `beg2` 开始的范围一样大  

## 11. 泛型算法命名规范  

### 使用重载形式传递谓词

接受谓词版本的函数使用谓词来代替默认的 `<` 或 `==` 运算符  

```C++
unique(beg, end);
unique(beg, end, comp);
```

### 附加 _if 后缀

该版本使用谓词而不是元素值进行比较  
从判定二者相等改为以传递的谓词返回非零值为准  

```C++
find(beg, end, val);
find_if(beg, end, pred);
```

### 附加 _copy 后缀

重排元素的算法默认将重排后的结果写回输入序列  
附加后缀的版本指定输出位置  

```C++
reverse(beg, end);
reverse_copy(beg, end, dest);
```

以上几种命名方式可以混用  

```C++
remove(beg, end, val);
remove_if(beg, end, pred);
remove_copy(beg, end, dest, val);
remove_copy_if(beg, end, dest, pred);
```

## 12. 链表和单向链表的特定容器算法  

由于链表和单向链表不提供随机访问迭代器，故一些泛型算法以成员函数的形式在这两类容器中实现  
也可以用通用版本的泛型算法，但性能不如成员函数版本的算法  

```C++
// 这些算法均返回 void

lst.merge(lst2);
lst.merge(lst2, comp);
/* lst 和 lst2 都必须是有序的，将 lst2 中的元素按照由小到大顺序放在 lst 的合适位置
 * 合并之后 lst2 会变为空，第二个版本用给定的谓词代替 < 运算符
 */

lst.remove(val);
lst.remove_if(pred);
/* 调用 erase 删除等于 val 或使得 pred 为真的每个元素
 */

lst.reverse();
/* 反转元素
 */

lst.sort();
lst.sort(comp);
/* 使用 < 运算符或 comp 排序元素
 */

lst.unique();
lst.unique(pred);
/* 调用 erase 删除重复值，使用 == 或二元谓词 pred 判定
 */

lst.splice(p, lst2);
flst.splice_after(p, lst2);
/* p 指向 lst 或 flst，将 lst2 中的元素全部移动到 lst 的 p 前面或 flst 的 p 后面
 * lst2 中的元素会全部被删除，lst2 的类型必须和 lst 或 flst 相同，且不能是同一个链表
 */

lst.splice(p, lst2, p2);
flst.splice_after(p, lst2, p2);
/* 与上面不同的是，p2 指向 lst2，只移动 p2 指向的元素或 p2 后面的一个元素
 * lst2 可以和 lst 相同
 */

lst.splice(p, lst2, b, e);
flst.splice_after(p, lst2, b, e);
/* 与上面不同的是，[b, e) 必须是 lst2 的合法范围，p 不能指向 [b, e) 中的元素
 */
```

链表特有的操作会改变容器，是真正的删除元素，而不是像泛型算法一样使用替换重排的方法
