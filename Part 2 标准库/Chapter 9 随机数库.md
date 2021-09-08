# Chapter 9 随机数库

> 随机数引擎类：产生 unsigned 随机数序列，是函数对象类，每次调用产生一个新的随机数  
> 随机数分布类：使用引擎返回服从某种分布的随机数，是函数对象类，每次调用输入引擎产生一个新的随机数  
> 随机数发生器 = 引擎对象 + 分布对象  

## 1. 随机数引擎类

标准库有多个随机数引擎类，编译器会指定一个作为 `default_random_engine` 类  

```C++
// 假设引擎类名为 Engine

Engine::result_type; // 返回的随机数类型

Engine e; // 默认构造函数，使用默认种子
Engine e(s); // 使用整型值 s（Engine::result_type） 作为种子
e.seed(s); // 使用种子 s 重置状态，无返回值，s 默认为 1
e.min(); // static constexpr 成员函数，返回此引擎的最小值（Engine::result_type）
e.max(); // static constexpr 成员函数，返回此引擎的最大值（Engine::result_type）
e.discard(u); // 将引擎向前推进 u（unsigned long long）步，
```

## 2. 随机数分布类

除 `bernoulli_distribution` 外其他均是模板，接受单个内置类型参数，有的只接受浮点类型，有的只接受整型  
接受的类型参数表示产生的随机数的结果类型，`bernoulli_distribution` 返回 `bool` 类型  
这里的整型不包括 `bool` 和各种 `char` 类型，整型的默认参数是 `int`，浮点型的默认参数是 `double`  
每个分布的构造函数的形参代表这种分布的参数，如果这些参数表示一个范围，均指闭区间范围，包括所有端点  
下面列出的各种分布中 `IntT` 表示接受整型，`RealT` 表示接受浮点型

```C++
// 均匀分布
// m 默认为 0，n 默认为 IntT 的最大值，x 默认为 0.0，y 默认为 1.0
uniform_int_distribution<IntT> u(m, n);
uniform_int_distribution<RealT> u(x, y);

// 伯努利分布
// p 表示返回 true 的概率，默认为 0.5
bernoulli_distribution b(p);

// 二项分布
// t 表示抽样次数，默认为 1
// p 表示单次正例概率，默认为 0.5
binominal_distribution<IntT> b(t, p);

// 几何分布
// p 表示单次正例概率，默认为 0.5
geometric_distribution<IntT> g(p);

// 负二项分布：k + r 次实验中，总共成功 k 次且第 k + r 次恰好为成功
// 根据如上描述的概率分布返回 r
// p 表示单次正例概率，默认为 0.5
negative_binominal_distribution<IntT> n(k, p);

// 泊松分布，x 为 double 类型的均值
poisson_distribution<IntT> p(x);

// 指数分布，参数 lam 默认为 1.0
exponential_distribution<RealT> e(lam);

// Γ 分布，参数 α 和 β 均默认为 1.0
gamma_distribution<RealT> g(a, b);

// 韦伯分布，参数 λ 和 k 均默认为 1.0
weibull_distribution<RealT> w(a, b);

// 极值分布，参数 a 默认为 0.0，参数 b 默认为 1.0
extreme_value_distribution<RealT> e(a, b);

// 正态分布，均值默认为 0.0，标准差默认为 1.0
normal_distribution<RealT> n(m, s);

// 对数正态分布，均值默认为 0.0，标准差默认为 1.0
lognormal_distribution<RealT> n(m, s);

// 卡方分布，自由度默认为 1.0
chi_squared_distribution<RealT> c(x);

// 柯西分布，位置参数默认为 0.0，尺度参数默认为 1.0
cauchy_distribution<RealT> c(a, b);

// F 分布，自由度默认均为 1.0
fisher_f_distribution<RealT> f(m, n);

// t 分布，自由度默认为 1.0
student_t_distribution<RealT> s(n);

// 离散分布，i 和 j 表示权重的迭代器范围，il 表示权重列表
// 返回 n 的概率为下标为 n 的元素的权重除以总权重
discrete_distribution<IntT> d(i, j);
discrete_distribution<IntT> d{il};

// 分段常数分布，迭代器范围 [b, e) 表示分段参数 b，迭代器范围 [w, w + (e - b - 1)) 表示权重参数 w
// x 落在 [b[i], b[i + 1]) 区间内的概率为该段的权重除以加权区间长度和，x 在区间内为均匀分布
piecewise_constant_distribution<RealT> pc(b, e, w);

// 分段线性分布，与上一个分布的区别是区间内概率密度函数为线性，左端点值为 w[i]，右端点值为 w[i + 1]
piecewise_linear_distribution<RealT> pl(b, e, w);
```

```C++
// 使用例

// 分布类的构造函数是 explicit 的
// 可生成 0~9 的均匀分布的随机整数，包括端点
uniform_int_distribution<unsigned> u(0, 9);
default_random_engine e;
cout << u(e) << endl;
```

```C++
// 其他方法
d.min(); // 返回最小值（）
d.max(); // 返回最大值（）
d.reset(); // 重置状态
```

## 3. 常见问题

### 生成的随机数总是一样的

```C++
// 错误示例，每次都返回相同序列，因为每次调用该函数都是从默认种子开始
// 正确方法是将引擎和分布类设为 static，这样每次调用都是下一个状态
vector<unsigned> randVec()
{
    default_random_engine e;
    uniform_int_distribution<unsigned> u(0, 9);
    vector<unsigned> ret;
    for (size_t i = 0; i < 100; ++i)
        ret.push_back(u(e));
    return ret;
}
```

可以将 `time(nullptr)` 设置为种子，但这样每一秒内生成的随机数序列都是一样的  

### 生成随机浮点数

```C++
uniform_real_distribution<double> u(0, 1);
```

### 使用分布类的默认结果类型

将模板实参设为空的尖括号即可  

### 使用非均匀分布类

```C++
normal_distribution<> n(4, 1.5); // 正太分布，均值 4，标准差 1.5
bernoulli_distribution b(0.6); // 伯努利分布，非模板类，返回 bool，true 的概率为 0.6，默认为 0.5
```
