# Chapter 13 参数传递

> 引用传递：形参是引用类型，是实参的别名  
> 值传递：实参的值被拷贝给形参，实参与形参相互独立

## 1. 引用形参的好处

1. 可以修改实参的值
2. 避免拷贝引起的性能浪费，而且有些类型的对象不支持拷贝，只能用引用传递
3. 可以用于返回多个信息

## 2. const形参和实参

1. 当形参具有顶层const时，与初始化顶层const对象类似，可以传常量也可以传非常量
2. 顶层const不能作为函数重载的区分条件
3. 底层const形参可以传常量或非常量，但底层const对象不能赋给非底层const对象
4. 引用传参尽量使用常量引用，其适用范围更广

## 3. 数组形参

> 数组不能拷贝，数组变量在值传递时会转换成指针

### 值传递数组的方法

```C++
void f(const int*);
void f(const int[]);
void f(const int[10]);
// 如果不需要修改数组元素的值，则声明const，否则不声明const
```

以上三种函数声明完全等价，都是只传了指针，[]中写的数字只有标记作用，实际数组大小无须与之相同，若需传递完整信息，则需要以下3种方法中的一种

1. 数组本身具有结束标记，如C字符串的'\0'字符
2. 传首尾指针
3. 额外传递一个表示数组大小的形参

### 引用传递数组的方法

```C++
void f(int (&arr)[10]); // 括号不能省，否则报错，不能传大小不等于10的数组
```

### 传多维数组的方法

```C++
void f(int matrix[][10], size_t rowSize);
// matrix是数组元素是int[10]类型的数组
void f(int (*matrix)[10], size_t rowSize);
// matrix是指向int[10]类型的指针，括号不能省，否则就是“指针的数组”
void f(int (&matrix)[3][4]);
// 引用传参
```

## 4. main函数的形参

```C++
int main(int argc, char *argv[]){}
```

argc是当前命令中的字符串总数  
argv是char*的数组，存储命令中的字符串
假设有下面的执行命令

```Shell
./a.out -o fo.txt -i fi.txt
```

则此时argc为5，argv[0]为"./a.out"，argv[1]为"-o"，argv[5]为0

## 5. 可变形参

### initializer_list形参

实参类型一致但数量不定时使用

```C++
// initializer_list的方法
initializer_list<T> lst; // 默认初始化
initializer_list<T> lst{a, b, c, ...}; // 大括号初始化
lst2(lst); lst2 = lst; // 拷贝
constexpr size_t lst.size() const; // 返回元素数量
constexpr const T* lst.begin() const; // 返回首指针 
constexpr const T* lst.end() const; // 返回尾指针的下一位置
// lst中所有元素均为常量类型不可修改
```

```C++
// 传参方法举例
void err_msg(int errCode, initializer_list<string> il); // 声明
err_msg(-1, {"function", "OK"}); // 调用
```

### 省略符形参

1. C和C++通用  
2. 省略符不能传递类类型，容易出错  
3. 需使用C标准库varargs
4. 只能放在形参列表末尾
5. 不检查类型

## 6. 默认实参

1. 定义函数时，默认实参后面的实参必须都有默认实参
2. 调用函数时，若有多个默认实参又要指定位置靠后的实参，必须先指定完前面的实参
3. 声明函数时，若有多次声明，则后面的声明不能修改前面的默认实参，但可以添加默认实参
4. 局部变量不能作为默认实参
