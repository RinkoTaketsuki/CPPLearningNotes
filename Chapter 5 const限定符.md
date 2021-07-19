# Chapter 5 const限定符

## 1. const对象（顶层const）

- const对象不能改变，故必须初始化

- const和extern同时使用时，extern写在前面

## 2. const引用（对const的引用）(底层const)

1. 常量引用可以指向常变量，也可以指向普通变量

	```C++
	const int ci = 1024; const int &cri = ci; // 正确
	int j = 512; const int &crj = j; // 正确
	int &ri = ci; // 错误，非常量引用不可引用常量
	crj = 1; // 错误，不可通过常量引用修改变量的值
	j = 1; // 正确
	```

2. 常量引用可以引用字面值，表达式和类型不同的变量

	```C++
	int i = 42; double d = 3.14;
	const int &r1 = 2 * i; // 等价于：const int temp = i; const int &r1 = temp;
	const int &r2 = d; // 等价于：const int temp = d; const int &r2 = temp;
	```

## 3. 指向const的指针（底层const）

和常量引用类似（但不必初始化）

```C++
const double pi = 3.14; const double *cp = &pi; // 正确
double d = 1.0; const double *cq = &d; // 正确
double *p = pi; // 错误，非常量指针不可指向常量
*cq = 2.0; // 错误，不可通过常量指针修改该内存空间中的值
*p = &d; *p = 2.0; // 正确
```

## 4. const指针（顶层const）

指针本身是const的，必须初始化

```C++
int i = 42; int j = 43;
int *const pc = &i;
*pc = 1; // 正确，指针指向的内容可修改
pc = j; // 错误，指针本身不可修改
```

## 5. 顶层const和底层const混用例子

```C++
int i = 0; const int ci = 42;
int *const pc = &i;
const int *cp = &ci;
const int *const cpc = cp;
const int &r = ci;
i = ci; // 正确，可以将常变量的值赋给普通变量
cp = cpc; // 正确，cp本身的值可以改变，且二者都是指向const int类型的指针
```

## 6. constexpr限定符

> 常量表达式：指编译时便可确认其值为常量的表达式，参与运算的元素只有常变量和字面值和常量表达式
>
> 常量表达式中各个元素的类型必须是字面值类型，即算术类型，指针和引用；标准库和自定义类型则不属于

1. 使用constexpr声明的常变量必须用常量表达式初始化

	```C++
	constexpr int pi = 3;
	constexpr int pim2 = pi * 2;
	
	int i = 2;
	const int ci = i; // 正确
	constexpr int cei = i; // 错误
	constexpr int cej = ci + 1; // 正确
	```

2. constexpr指针必须指向地址固定的对象（如全局变量）或为0，且总是顶层const，指针指向的对象可变，可以指向常变量也可以指向普通变量

	```C++
	int i = 0;
	constexpr int ci = 42;
	constexpr const int *p = &i;
	constexpr int *q = &i;
	```
