# Chapter 10 泛型算法概览

## 1. 参数说明

`beg` 和 `end` 表示第一个输入序列迭代器范围  
`beg2` 表示第二个输入序列的开始位置，若无 `end2`，则表示与第一个迭代器范围大小相同，否则由 `end2` 指定  
`beg` 和 `beg2` 如无特殊要求则类型不必匹配  
`dest` 表示目的序列的开始位置，要求目的序列足够大以支持保存算法生成的元素  
`unaryPred` 表示一元谓词，`binaryPred` 表示二元谓词  
`comp` 表示用于比较的二元谓词  
`unaryOp` 和 `binaryOp` 表示可调用对象  

## 2. 查找算法

默认使用 `==` 进行比较  

### 简单查找算法

要求输入迭代器  

```C++
// 返回指向输入序列中第一个值为 val 元素的迭代器，未找到时返回 end
find(beg, end, val);
// 返回指向输入序列中第一个满足 unaryPred 的元素的迭代器，未找到时返回 end
find_if(beg, end, unaryPred);
// 返回指向输入序列中第一个不满足 unaryPred 的元素的迭代器，未找到时返回 end
find_if_not(beg, end, unaryPred);

// 以下两个函数返回 iterator_traits<InputIterator>::difference_type 值
// 返回输入序列中值为 val 的元素个数
count(beg, end, val);
// 返回输入序列中满足 unaryPred 的元素的个数
count_if(beg, end, unaryPred);

// 以下三个函数返回 bool 值
// 是否所有元素都满足 unaryPred，序列为空时返回 true
all_of(beg, end, unaryPred);
// 是否有任意一个元素满足 unaryPred，序列为空时返回 false
any_of(beg, end, unaryPred);
// 是否所有元素都不满足 unaryPred，序列为空时返回 true
none_of(beg, end, unaryPred);
```

### 查找重复值的算法

以下函数要求前向迭代器，其中的二元谓词用于比较两个元素是否相等  

```C++
// 返回指向第一对相邻重复元素的第一个元素的迭代器，若无相邻重复元素则返回 end
adjacent_find(beg, end);
adjacent_find(beg, end, binaryPred);
// 返回一个迭代器，从此位置开始的 count 个元素均为 val，若不存在这样的位置则返回 end
search_n(beg, end, count, val);
search_n(beg, end, count, val, binaryPred);
```

### 查找子序列的算法

`find_first_of` 的第一个序列要求输入迭代器，第二个序列要求前向迭代器，其他函数均要求前向迭代器  
第二个序列为空或未找到时均返回 `end1`，二元谓词用于比较两个元素是否相等  

```C++
// 返回第二个序列在第一个序列中第一次出现的位置
search(beg1, end1, beg2, end2);
search(beg1, end1, beg2, end2, binaryPred);
// 返回第二个序列中的任意一个元素在第一个序列中第一次出现的位置
find_first_of(beg1, end1, beg2, end2);
find_first_of(beg1, end1, beg2, end2, binaryPred);
// 返回第二个序列中的任意一个元素在第一个序列中最后一次出现的位置
find_end(beg1, end1, beg2, end2);
find_end(beg1, end1, beg2, end2, binaryPred);
```

## 3. 其他只读算法

以下函数要求输入迭代器，默认使用 `==` 进行比较，二元谓词用于比较两个元素是否相等  

```C++
// 对每个元素使用 unaryOp，若迭代器允许写入值，则 unaryOp 可能改变元素的值，返回 unaryOp
for_each(beg, end, unaryOp);
// 返回一个迭代器 pair，表示第一个不匹配位置，若均匹配，则返回的第一个迭代器为 end1，
// 第二个迭代器为 beg2 + (end1 - beg1)
mismatch(beg1, end1, beg2, end2);
mismatch(beg1, end1, beg2, end2, binaryPred);
// 比较两个序列是否相等
equal(beg1, end1, beg2);
equal(beg1, end1, beg2, binaryPred);
```

## 4. 二分搜索算法

要求前向迭代器，但前向迭代器性能低，若提供随机访问迭代器则性能高  
默认使用 `<` 进行比较，`comp` 用于指定小于关系，需满足“严格弱序”  
要求序列中的元素必须是有序的（升序或 `comp` 对应的顺序）  

```C++
// 返回指向最后一个小于等于 val 的元素的迭代器，若不存在则返回 end
lower_bound(beg, end, val);
lower_bound(beg, end, val, comp);
// 返回指向第一个大于 val 的元素的迭代器，若不存在则返回 end
upper_bound(beg, end, val);
upper_bound(beg, end, val, comp);
// 返回 make_pair(lower_bound 返回值, upper_bound 返回值)
equal_range(beg, end, val);
equal_range(beg, end, val, comp);
// 返回 bool 值，表示序列中是否能找到 val，判定 x 和 y 相等的方式是
// !(x < y) && !(y < x)
binary_search(beg, end, val);
binary_search(beg, end, val, comp);
```

## 5. 只写不读元素的算法

要求输出迭代器  
`Gen` 是可调用对象，表示生成器，每次调用返回不同的值  

```C++
// 向序列中每一个元素赋 val 值，无返回值
fill(beg, end, val);
// 向 dest 指向位置赋 cnt 个 val 值，返回写入元素的尾后位置
fill_n(dest, cnt, val);
// 对序列中每一个元素调用一次 Gen，赋 Gen 的返回值，无返回值
generate(beg, end, Gen);
// 对 dest 指向位置调用 cnt 次 Gen，赋 cnt 个 Gen 的返回值，返回写入元素的尾后位置
generate(dest, cnt, Gen);
```

## 6. 使用输入迭代器的写算法

输入迭代器用于表示输入范围，`dest` 为输出迭代器  
均返回写入元素的尾后迭代器  

```C++
// 拷贝 [beg, end) 内所有元素到 dest
copy(beg, end, dest);
// 拷贝 [beg, end) 内所有满足 unaryPred 的元素到 dest
copy_if(beg, end, dest, unaryPred);
// 拷贝从 beg 开始的 n 个元素到 dest
copy_n(beg, n, dest);
// 使用 std::move 移动 [beg, end) 内所有元素到 dest
move(beg, end, dest);
// 对 [beg, end) 内所有元素调用 unaryOp，并将结果写入 dest 中
transform(beg, end, dest, unaryOp);
// 对两个输入序列内所有元素对调用 binaryOp，并将结果写入 dest 中
transform(beg, end, beg2, dest, binaryOp);
// 对 [beg, end) 内所有值为 old_val 的元素替换为 new_val 后将替换后的输入序列写入 dest 中
replace_copy(beg, end, dest, old_val, new_val);
// 对 [beg, end) 内所有满足 unaryPred 的元素替换为 new_val 后将替换后的输入序列写入 dest 中
replace_copy_if(beg, end, unaryPred, old_val, new_val);
// 将两个有序（升序）序列合并后写入 dest 中，默认使用 < 运算符
merge(beg1, end1, beg2, end2, dest);
// 将两个有序序列合并后写入 dest 中，使用 comp 比较
merge(beg1, end1, beg2, end2, dest, comp);
```

## 7. 使用前向迭代器的写算法

```C++
// 交换 iter1 和 iter2 指向的元素，无返回值
iter_swap(iter1, iter2);
// 交换两个迭代器范围内的元素，两个范围不能有重叠，返回递增过的 beg2，即交换的元素的尾后位置
swap_ranges(beg1, end1, beg2);
// 替换值为 old_val 的元素为 new_val，无返回值
replace(beg, end, old_val, new_val);
// 替换满足 unaryPred 的元素为 new_val，无返回值
replace_if(beg, end, old_val, new_val);
```

## 8. 使用双向迭代器的写算法
