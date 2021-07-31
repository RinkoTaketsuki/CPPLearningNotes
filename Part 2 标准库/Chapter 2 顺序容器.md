# Chapter 2 顺序容器

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
C c(b, e); // 将迭代器 b 到 e 指定的范围内的元素拷贝给 c（array 不支持）
C c{ele1, ele2, ...}; // 列表初始化，若为 array 类型则必须大小相等

// 除 array 以外的顺序容器支持的构造函数
C c(n); 
/* 使容器包含 n （size_type）个元素，每个元素都使用默认构造函数初始化
 * 或值初始化，是 explicit 的，string 不支持，无默认构造函数的元素类型
 * 也不支持
 */
C c(n, t); // 使容器包含 n 个值为 t 的元素

// 赋值
c1 = c2; // 用 c2 中的元素替换 c1 中的元素
c1 = {ele1, ele2, ...}; // 用列表中的元素替换 c1 中的元素（array 不支持）

// 关系运算符：所有容器都支持相等和不等运算符
// 无序关联容器不支持大小关系运算符

// 方法
c1.swap(c2); // 无返回值，交换 c1 和 c2 的元素
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

### 反向迭代器

操作含义会发生颠倒，如 `++` 和 `--` 的方向会变反  

## 5. 类型成员

使用时需显式指定类名  
在不了解元素类型的情况下很有用  

## 6. 初始化注意事项

拷贝初始化必须保证二者元素类型匹配（可转换到目标容器元素类型），容器类型也需相同（教材描述好像有问题）  
使用迭代器初始化注意左闭右开  
array 初始化时元素类型和大小（size_t）同时作为其元素类型的一部分
