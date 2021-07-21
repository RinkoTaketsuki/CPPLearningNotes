# Chapter 2 基本类型

## 1. 内置类型

- 算术类型  
    - 整型  
        - 一般整型  
        - 布尔类型  
        - 字符类型  
    - 浮点型  
- 空类型  

| 布尔类型 |
| :------: |
|   bool   |

| 字符类型 |             |               |
| :------: | :---------: | :-----------: |
|   char   | signed char | unsigned char |
| wchar_t  |
| char16_t |
| char32_t |

注：char 可能算 signed 也可能算 unsigned，由编译器决定

|   整型    |                         |
| :-------: | :---------------------: |
|   short   |     unsigned short      |
|    int    | unsigned int = unsigned |
|   long    |      unsigned long      |
| long long |   unsigned long long    |

|   浮点型    |
| :---------: |
|    float    |
|   double    |
| long double |

## 2. 内存地址

字节：8 比特  
字：一般是 32 或 64 比特  
地址：索引一个字节

## 3. 类型转换

### 非布尔 => 布尔

0 false  
其他 true

### 布尔 => 非布尔

false 0  
true 1

### 浮点数 => 整数

截断

### 整数 => 浮点数

小数部分为 0，精度可能损失

```C++
// 以下注释为GDB调试结果
unsigned long ll_v = 0x7999999999999999;
//8762203435012037017
float f_v = ll_v;
// 8.76220333e+18
double d_v = ll_v;
// 8.7622034350120366e+18
```

### 很大的正值 => 无符号类型

取余

### 带符号数和无符号数混合运算

先将带符号数转化为无符号数

### 其他注意事项

1. 不要把超出范围的值赋给带符号类型
2. if、while、for 等语句中的条件表达式的值会自动转换为布尔型
3. 勿混用带符号类型和无符号类型

## 4. 字面值常量的类型

|         例子         |        类型        |
| :------------------: | :----------------: |
|          20          |        int         |
|      9999999999      |        long        |
| -9999999999999999999 | unsigned long long |
|         024          |        int         |
|         0x10         |        int         |
|         1.1          |       double       |
|         .23          |       double       |
|       4.414E2        |       double       |
|       4.414e2        |       double       |
|       4.414E-2       |       double       |
|       4.414e-2       |       double       |
|         'a'          |        char        |
|       "Hello"        |   const char[6]    |
|         L'a'         |      wchar_t       |
|    u8"中文字符串"    |   const char[16]   |
|        1L 2l         |        long        |
|       1LL 2ll        |     long long      |
|        1U 2u         |    unsigned int    |
|      1ULL 2ull       | unsigned long long |
|      1.0F 2.0f       |       float        |
|      1.0L 2.0l       |    long double     |
|         u'a'         |      char16_t      |
|         U'a'         |      char32_t      |
|      true false      |        bool        |

注：  

1. 字符串字面值可写成多行

    ```C++
    char* str = "Hello "
        "world!";
    ```

2. 过长的负整数(long long 及以上的级别)会忽略负号只保留正值

## 5. 转义序列

|     序列      |          字符          |
| :-----------: | :--------------------: |
|      \0       |        (char)0         |
|      \n       |    换行符（Enter）     |
|      \r       | 回车符（光标回到行首） |
|      \b       | 退格符（光标回退一格） |
|      \t       |   横向制表符（Tab）    |
|      \v       |       纵向制表符       |
|      \\?      |           ?            |
|     \\\\      |           \\           |
|      \\"      |           "            |
|      \\'      |           '            |
| \正整数字面值  |  当前编码集对应的字符   |
