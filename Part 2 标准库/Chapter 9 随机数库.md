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
