# Chapter 10 数组

## 1. 定义和初始化

1. 数组初始化维度必须是constexpr，const也不行
2. 数组元素的类型不能用auto
3. 数组元素不能是引用类型
4. 大括号和字符串字面值初始化数组时可省略维度
5. 大括号中的初始值数量不能超过数组维度
6. 数组不能赋值（拷贝）
7. 数组在随机访问时[]运算符中的数字类型为size_t

## 2. 复杂数组定义举例

```C++
int arr[10];
int *ptrs[10]; // 数组元素为int*类型
int (*pArr)[10] = &arr; // pArr为指向int [10]类型的指针（一种二级指针）
int (&rArr) = arr; // rArr为指向arr（int [10]类型）的引用
int *(&rPtrs) = ptrs; // rPtrs为指向ptrs（int * [10]）类型的引用
```

## 3. 数组与指针的使用注意事项

1. `auto var = arr;` 此时var为指针类型
2. `decltype(arr) var;` 此时var为数组类型
3. &arr[0] 和 &arr[数组维度] 相当于迭代器的begin和end
4. C++标准库中的begin和end函数

    ```C++
    constexpr int *std::begin(arr); // 返回首元素指针
    constexpr int *std::end(arr); // 返回尾元素下一位置的指针
    ```

5. 指针相减的返回类型是ptrdiff_t
6. 指向同一个对象的指针和空指针才能使用比较运算符
7. 当指针指向数组的非首位时，可以使用负数下标

    ```C++
    int *p = &arr[2];
    cout << p[-1]; // arr[1]
    ```

## 4. 使用数组初始化vector

```C++
int arr[10] = {0};
vector<int> v1(begin(arr), end(arr));
vector<int> v2(arr + 1, arr + 3); // 切片
```

## 5. 多维数组初始化

```C++
int arr[3][4] = {
    {0, 1, 2, 3},
    {4, 5, 6, 7},
    {8, 9, 10, 11}
};
int arr[2][3] = {0, 1, 2, 3, 4, 5};
int arr[3][4] = {{0}, {4}, {8}}; // 显式初始化每行第一个元素，其他默认初始化
int arr[3][4] = {0, 1, 2, 3} // 显式初始化第一行
```

## 6. 多维数组的处理

1. 表达式的下标运算符数量少于数组维度时返回子数组，可以引用

    ```C++
    int arr[3][4] = {0};
    int (&row)[4] = arr[1];
    ```

2. 范围for语句遍历多维数组

    ```C++
    int arr[3][4] = {0};
    for(auto &row : arr)
    {
        for(auto &item : row)
        {
            /* 处理 item */
        }
    }
    ```

    > 须注意不可省略外层循环的引用符号，否则row将是指针，失去数组大小信息

3. 指向数组的指针的运用

    ```C++
    int arr[3][4] = {0};
    int (*p)[4] = &arr[2]; // p是int[4]指针，指向arr的第3行
    auto p = arr; // p是int[4]指针，指向arr的第1行

    // 使用指针遍历输出数组
    for (auto p = begin(arr); p != end(arr); ++p)
    {
        for (auto q = begin(*p); q != end(*p); ++q)
        {
            // q是int指针
            cout << *q << ' ';
        }
        cout << endl;
    }
    ```

4. 使用类型别名简化指针

    ```C++
    using intArr4 = int[4];
    // 等价于 typedef int intArr4[4];

    int arr[3][4] = {0};

    // 使用指针遍历输出数组
    for (intArr4 *p = begin(arr); p != end(arr); ++p)
    {
        for (int *q = begin(*p); q != end(*p); ++q)
        {
            cout << *q << ' ';
        }
        cout << endl;
    }
    ```
