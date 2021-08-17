# Chapter 4 关联容器

## 1. 注意事项

1. 不支持与位置有关的操作，如 `push_back` 等  
2. 提供的迭代器均为双向迭代器  
3. 初始化 `map` 时，每个键值对用一个花括号  
4. 对于有序的关联容器，默认使用关键字类型的 `<` 运算符进行比较  
5. 若向容器提供比较函数，该函数必须满足“严格弱序”  

    > 严格弱序（类似小于）：  
    > if a < b, then b < a is false  
    > if a < b and b < c, then a < c is true  
    > if a < b is false and b < a is false, then a = b  
    > if a = b and b = c, then a = c

6. 通常使用如下方式提供比较函数：

    ```C++
    bool compare(const A &obj1, const A &obj2);
    set<A, decltype(compare)*> set_A(&compare); 
    // 参数也可以不带取地址符，compare 在传入构造函数时会自动转化成函数指针类型
    ```

## 2. pair

默认构造函数对数据成员进行值初始化  
返回  `pair` 类型时，可以直接使用花括号返回两个值  

### pair 支持的操作

```C++
// 初始化
pair<T1, T2> p;
pair<T1, T2> p(v1, v2);
pair<T1, T2> p = {v1, v2};
make_pair(v1, v2); // 返回值为 v1 和 v2 的 pair，自动推断类型

// 公有数据成员
p.first;
p.second;

// 关系运算符
// 以 < 为例，当 p1.first < p2.first 或 p1.first == p2.first && 
// p1.second < p2.second 时为真
// 仅当两个元素都相等时 pair 才相等，否则为不等
```

## 3. 类型成员

下面只列出关联容器的特有类型：  

`key_type`：键类型  
`mapped_type`：值类型  
`value_type`：对于 `set` 等价于键类型，对于 `map` 则为 `pair<const key_type, mapped_type>`  

## 4. 迭代器

`map` 返回指向 `value_type` 元素的迭代器，关键字不能更改  
`set` 总是返回 `const_iterator` 类型的迭代器，不能修改元素  
迭代器按关键字升序遍历元素（有序容器）  
通常不对关联容器使用泛型算法  

## 5. 插入元素

```C++
c.insert(v); // v 是 value_type 对象
c.emplace(args);
c.insert(b, e);
c.insert({v1, v2, ...});
c.insert(p, v);  // p 用于提示搜索被插入元素的存储位置的开始位置
c.emplace(p, args);

/* map 和 set 返回一个 pair，第一个元素是指向某个元素的迭代器，
 * 第二个元素是 bool 值，表示是否插入成功
 * 若 map 和 set 插入成功，则返回指向被插入元素的迭代器
 * 若 map 和 set 插入失败，则返回指向同名关键字元素的迭代器
 * multiset 和 multimap 总会插入值，返回指向被插入元素的迭代器 
 */
```

## 6. 删除元素

```C++
c.erase(k); // 删除每个关键字值为 k 的元素，返回删除数量（size_type）
c.erase(p); // 删除迭代器 p 指定的元素，返回指向 p 后面的元素的迭代器（或为尾后迭代器）
c.erase(b, e); // 删除指定范围内的元素，返回 e
```

## 7. 访问元素

### map 和 unordered_map 的下标和 at 操作

1. 须注意其他所有关联容器均不支持下标和 `at` 操作  
2. 当给定关键字不在 `map` 中时，下标运算符会创建新元素，其对应的 `second` 元素会进行值初始化，而 `at` 操作会抛出 `out_of_range` 异常；因此，`const` 的 `map` 不能使用下标运算符  
3. 下标运算符返回 `mapped_type` 类型，而解引用迭代器返回 `value_type`  

### find 操作

输入一个关键字，返回指向关键字相等的元素的迭代器，若找不到则返回尾后迭代器  
对 `map` 使用 `find` 代替下标运算符可以避免插入元素  

### count 操作

输入一个关键字，返回关键字相等的元素个数（`size_type`）

### 有序关联容器特有操作

```C++
c.lower_bound(k); // 返回指向第一个关键字不小于 k 的元素的迭代器
c.upper_bound(k); // 返回指向第一个关键字大于 k 的元素的迭代器
c.equal_range(k); // 返回关键字等于 k 的元素的迭代器 pair
```

须注意若 k 是最大关键字，则 `upper_bound` 返回尾后迭代器  
若 k 在 c 中不存在且大于所有关键字，则 `lower_bound` 返回尾后迭代器  
`equal_range` 等价于 `[lower_bound, upper_bound)`  
若 k 在 c 中不存在，则 `equal_range` 返回的两个迭代器均为 k 可插入的位置  

## 8. 无序关联容器

组织元素时默认方法不是使用 `<` 运算符，而是使用 `==` 运算符和 `hash` 函数

```C++
c.bucket_count(); // 正在使用的桶的数量（size_type）
c.max_bucket_count(); // 容器最大容纳的桶数量（size_type）
c.bucket_size(n); // 编号为 n（size_type） 的桶中的元素数量（size_type）
c.bucket(k); // 包含关键字 k（key_type） 的桶编号（size_type）
c.begin(n); c.end(n); // 返回编号为 n 的桶的迭代器（local_iterator）
c.cbegin(n); c.cend(n); // 返回编号为 n 的桶的迭代器（const_local_iterator）
c.load_factor(); // 返回每个桶的平均元素数量（float）
c.max_load_factor(); // 返回试图维护的平均桶大小（float）
// c 会添加桶使得 load_factor <= max_load_factor
c.rehash(n); // 重组存储，使得 bucket_count >= n，且 bucket_count > size/max_load_factor
c.reserve(n); // 重组存储，使得 c 可以保存 n 个元素且不进行 rehash
```

内置类型和一些标准库类型（`string`，智能指针）等具有 `hash` 模板，不具有 `hash` 模板的需提供 `hash` 函数  

```C++
class A
{
private:
    string s = "";
public:
    A() = default;
    string& get() {return s;}
};

size_t hasher(const A &obj)
{
    // hash<string> 是可调用对象
    return hash<string>()(obj.get());
}

bool eqOp(const A &obj1, const A &obj2)
{
    return obj1.get() == obj2.get();
}

// 三个模板参数是：元素类型，hash 函数指针类型，比较函数指针类型
// 三个初始化参数是：桶数量，hash 函数指针，比较函数指针
unordered_multiset<A, decltype(hasher)*, decltype(eqOp)*>
umset_A(100, hasher, eqOp);
```
