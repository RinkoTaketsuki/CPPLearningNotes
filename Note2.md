# Chapter 2 变量和基本类型
## 1. 内置类型
```
算术类型
    |---整型
        |----一般整型
        |----布尔类型
        |----字符类型
    |---浮点型
空类型
```
|布尔类型|
|:----:|
|bool|

|字符类型 | | |
|:----:|:----:|:----:|
|char|signed char|unsigned char|
|wchar_t|
|char16_t|
|char32_t|

注：char可能算signed也可能算unsigned，由编译器决定

| 整型 | |
|:----:|:----:|
|short|unsigned short|
|int|unsigned int = unsigned|
|long|unsigned long|
|long long|unsigned long long|


| 浮点型 | |
|:----:|:----:|
|float|unsigned float|
|double|unsigned double|
|long double|unsigned long double|

## 2. 内存地址
字节：8比特<br>
字：一般是32或64比特<br>
地址：索引一个字节

## 3. 类型转换