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

## 3. 类型转换注意事项

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
    
    > 字符串字面值可以赋给 char* 变量，但不能通过 char* 变量修改其指向的 C 字符串。通常情况下，const 指针是不能赋给非 const 指针的，但只有字符串字面值是个例外。 

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
