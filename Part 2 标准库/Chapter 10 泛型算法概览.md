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

```C++
// dest 是输出序列的尾后迭代器，拷贝或移动顺序是从后向前，输出序列和输入序列的元素排列顺序相同
// 若输入范围为空返回 dest，否则返回指向输出序列首元素的迭代器  
copy_backward(beg, end, dest);
move_backward(beg, end, dest);
// 将同一序列中两个有序子序列 [beg, mid) 和 [mid, end) 合并为一个有序序列
// 有序的定义为升序（使用默认的 < 运算符）或 comp 对应的顺序，返回 void
inplace_merge(beg, mid, end);
inplace_merge(beg, mid, end, comp);
```

## 9. 划分算法

通常把输入序列分为两组，一组满足一元谓词，另一组不满足  
稳定版本通常比不稳定版本效率更低且消耗内存更多  
除 `is_partitioned` 要求输入迭代器外均要求双向迭代器  

```C++
// 若输入序列为空或所有满足条件的元素都在不满足条件的元素之前，则返回 true
is_partitioned(beg, end, unaryPred);
// 将输入序列中满足条件的元素拷贝到 dest1，不满足条件的元素拷贝到 dest2
// 输入序列与两个目的序列不能重叠，返回迭代器 pair，即两个目的序列的尾后迭代器
partition_copy(beg, end, dest1, dest2, unaryPred);
// 输入序列必须满足 is_partitioned(beg, end, unaryPred)，
// 返回满足条件的子序列的尾后迭代器，可能是 end
partition_point(beg, end, unaryPred);
// 对序列进行原地划分，返回划分后满足条件的子序列的尾后迭代器，若所有元素都不满足则返回 beg
partition(beg, end, unaryPred);
stable_partition(beg, end, unaryPred);
```

## 10. 排序算法

均要求随机访问迭代器，默认使用 `<` 运算符比较，默认升序排序  
稳定版本通常比不稳定版本效率更低且消耗内存更多  
`partial_sort` 和 `nth_element` 系列函数只进行部分排序工作，效率比排序整个输入序列要高  
除 `partial_sort_copy` 以外其他函数均无返回值  

```C++
// 对输入范围原地排序
sort(beg, end);
sort(beg, end, comp);
stable_sort(beg, end);
stable_sort(beg, end, comp);
// 返回 bool 值，表示整个输入序列是否有序
is_sorted(beg, end);
is_sorted(beg, end, comp);
// 返回从首元素开始的最长有序子序列的尾后迭代器
is_sorted_until(beg, end);
is_sorted_until(beg, end, comp);
// 将输入序列的前 mid - beg 小的元素升序放在序列最前面，其他元素放在后面
// 未排序部分的元素的相对位置不确定
partial_sort(beg, mid, end);
partial_sort(beg, mid, end, comp);
// 将输入序列的前 destEnd - destBeg 小的元素升序放在 [destBeg, destEnd) 中
// 若目的序列范围比输入序列大，则将整个输入序列的排序结果放在目的序列中
// 返回目的序列的已排序部分的尾后迭代器
partial_sort_copy(beg, end, destBeg, destEnd);
partial_sort_copy(beg, end, destBeg, destEnd, comp);
// nth 是随机访问迭代器，执行函数后，nth 指向的元素的值是整个序列排序后对应位置的值
// 且 nth 之前的元素均小于等于 *nth，nth 之后的元素均大于等于 *nth
nth_element(beg, nth, end);
nth_element(beg, nth, end, comp);
```

## 11. 使用前向迭代器的重排算法

拷贝版本的目的序列迭代器要求输出迭代器  

```C++
// 把等于 val 或满足 unaryPread 的元素放在前面，前面的序列的元素的相对位置不变
// 返回前面的序列的尾后迭代器
remove(beg, end, val);
remove_if(beg, end, unaryPred);
// 把等于 val 或满足 unaryPread 的元素放在目的序列中，前面的序列的元素的相对位置不变
remove_copy(beg, end, dest, val);
remove_copy_if(beg, end, dest, unaryPred);
// 把相邻的重复元素排在后面，前面的序列的元素的相对位置不变
// 返回前面的序列的尾后迭代器
unique(beg, end);
unique(beg, end, binaryPred);
// 把相邻的重复元素排在后面，把前面的序列拷贝给 dest，前面的序列的元素的相对位置不变
unique_copy(beg, end, dest);
unique_copy(beg, end, dest, binaryPred);
// 把 [mid, end) 放在前面，把 [beg, mid) 放在后面
// 返回指向原来的 beg 指向的元素的迭代器
rotate(beg, mid, end);
// 把 [mid, end) 放在 dest 前面，把 [beg, mid) 放在 dest 后面
// 返回目标序列的指向第二次拷贝的首元素的迭代器
rotate_copy(beg, mid, end, dest);
```

## 12. 使用双向迭代器的重排算法

拷贝版本的目的序列迭代器要求输出迭代器  

```C++
// 翻转元素，无返回值
reverse(beg, end);
// 翻转元素，返回目的序列的尾后迭代器
reverse_copy(beg, end, dest);
```

## 13. 使用随机访问迭代器的重排算法

均返回 void  

```C++
// 打乱元素
random_shuffle(beg, end);
// 打乱元素，rand 是可调用对象，其接受一个正整数输入，返回 0 到该值之间均匀分布的随机数
random_shuffle(beg, end, rand);
// 打乱元素，UniformRand 是均匀分布随机数引擎
shuffle(beg, end, UniformRand);
```

## 14. 排列算法

> 排列即数学中排列的定义，排列的顺序是指字典顺序  
> 例：“ABC” 的 6 个排列按照顺序写出  
> “ABC”，“ACB”，“BAC”，“BCA”，“CAB”，“CBA”  

排列算法假定输入序列都是无重复值的，均要求双向迭代器  

```C++
// 若第二个输入序列的某个排列等于第一个输入序列，则返回 true
// 默认使用 == 运算符
is_permutation(beg1, end1, beg2);
is_permutation(beg1, end1, beg2, binaryPred);
// 将输入序列重排为下一个排列，返回 true
// 如果是最后一个排列则重排为第一个排列，返回 false
// 默认使用 < 运算符
next_permutation(beg, end);
next_permutation(beg, end, comp);
// 类似上一个函数，但重排为上一个排列
prev_permutation(beg, end);
prev_permutation(beg, end, comp);
```

## 15. 有序序列的集合算法

注意只能处理有序序列，均默认使用 `<` 运算符  
输入序列要求输入迭代器，目的序列要求输出迭代器，有目的序列的均返回目的序列的尾后迭代器  

```C++
// 包含：若第一个序列包括第二个序列中的所有元素，则返回 true，否则返回 false
includes(beg1, end1, beg2, end2);
includes(beg1, end1, beg2, end2, comp);
// 并：将两个有序序列合并成一个有序序列并去重，结果写入目的序列
set_union(beg1, end1, beg2, end2, dest);
set_union(beg1, end1, beg2, end2, dest, comp);
// 交：将两个有序序列的公共有序序列写入目的序列
set_intersection(beg1, end1, beg2, end2, dest);
set_intersection(beg1, end1, beg2, end2, dest, comp);
// 差：将在第一个有序序列中而不在第二个有序序列中的元素组成的有序序列写入目的序列
set_difference(beg1, end1, beg2, end2, dest);
set_difference(beg1, end1, beg2, end2, dest, comp);
// 对称差：将只在其中一个序列中出现的元素组成的有序序列写入目的序列
set_symmetric_difference(beg1, end1, beg2, end2, dest);
set_symmetric_difference(beg1, end1, beg2, end2, dest, comp);
```

## 16. 最值算法

默认使用 `<` 运算符  
接受序列的要求输入迭代器  

```C++
// 返回两个对象的最值
min(val1, val2);
min(val1, val2, comp);
max(val1, val2);
max(val1, val2, comp);
// 返回花括号列表的最值
min(il);
min(il, comp);
max(il);
max(il, comp);
// 返回最值 pair：{min, max}
minmax(val1, val2);
minmax(val1, val2, comp);
minmax(il);
minmax(il, comp);
// 处理迭代器范围输入的版本，返回指向对应元素的迭代器或迭代器 pair
min_element(beg, end);
min_element(beg, end, comp);
max_element(beg, end);
max_element(beg, end, comp);
minmax_element(beg, end);
minmax_element(beg, end, comp);
```

## 17. 字典序比较算法

默认使用 `<` 运算符，要求输入迭代器，返回 `bool`

> 字典序比较：假设两个序列名为 a 和 b  
> 如果两个序列的 [0, i) 范围内的元素都相等，但 a[i] < b[i]，则 a < b  
> 假定 a 序列更短，两个序列的 [0, a 的长度) 范围内的元素都相等，则 a < b  

```C++
// 如果字典序小于则返回 true，大于等于则返回 false
lexicographical_compare(beg1, end1, beg2, end2);
lexicographical_compare(beg1, end1, beg2, end2, comp);
```

## 18. 数值算法

定义在 `<numeric>` 中，输入序列要求输入迭代器，目的序列要求输出迭代器  

> 部分和：s[i] = accumulate(a.cbegin(), a.cbegin() + i + 1, 0)  
> 相邻差：d[i] = a[i] - a[i - 1]，d[0] = a[0]  

```C++
// 返回输入序列求和 + init，默认使用 + 运算符
accumulate(beg, end, init);
accumulate(beg, end, init, binaryOp);
// 返回输入序列内积 + init，默认使用 + 和 * 运算符
// binaryOp1 代替 +，binaryOp2 代替 *
inner_product(beg, end, beg2, init);
inner_product(beg, end, beg2, init, binaryOp1, binaryOp2);
// 将输入序列的部分和写入 dest，默认使用 + 运算符，返回目的序列的尾后迭代器
partial_sum(beg, end, init);
partial_sum(beg, end, init, binaryOp);
// 将输入序列的相邻差写入 dest，默认使用 - 运算符，返回目的序列的尾后迭代器
adjacent_difference(beg, end, init);
adjacent_difference(beg, end, init, binaryOp);
// beg[i] = val++，无返回值
iota(beg, end, val);
```
