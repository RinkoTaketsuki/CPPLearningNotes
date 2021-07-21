# Chapter 8 标准库类型vector

## 1. 定义和初始化vector对象

> 不可用引用类型实例化vector  
> 某些编译器要求当用vector实例化vector时，后面要加空格，如：  
> `vector<vector<int> >`

```C++
vector<T> v1;
vector<T> v2(v1);
vector<T> v2 = v1;
vector<T> v3(n, val); // 用n个T类型的val初始化
vector<T> v4(n); // 用n个T类型的默认值初始化
vector<T> v5{a, b, c}; // 类似于数组的大括号初始化，类内初始化
vector<T> v6 = {a, b, c}; // 同上，但使用拷贝初始化
```

## 2. vector方法和运算

```C++
vector<T> v, v1, v2;
bool v.empty();
size_t v.size();
void v.push_back(T); // 把一个T对象放在vector最后面
v[n] // T&（n是size_t类型）
v1 = v2; // vector<T>&
v1 = {...} // vector<T>&
// 还有各种逻辑比较符号 返回bool
```

1. 赋值运算符会完全把左边的vector替换成右边的vector，包括内容和长度，左边的值会完全丢失
2. 只能对存在的元素进行下标操作  
3. 若使用范围for循环则不要使用push_back()
