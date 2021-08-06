# Chapter 11 C字符串

## 1. cstring库函数

```C++
size_t strlen(const char *s);
int strcmp(const char *s1, const char *s2);
// strcmp 大于返回正，小于返回负，相等返回0
char *strcat(char *__restrict__ dest, const char *__restrict__ src);
// strcat 把src附在dest之后，返回dest
char *strcpy(char *__restrict__ dest, const char *__restrict__ src);
// strcpy 把src赋给dest，若src长度大于dest会被截断（原/0位也会被填充）,若src
// 长度小于dest则会用/0替换掉dest[strlen(src)]上的字符，下标超过strlen(src)
// 部分的字符不变

// 不可直接比较两个char*变量，否则会变成指针的比较
```

## 2. 混用string和C字符串

1. `string s("hello")` 使用C字符串初始化string对象
2. string的加法可以使其中一个为C字符串
3. string转化成C字符串可以使用 `const char *std::string::c_str() const` 成员函数，但是当原string变化时，对应的C字符串也会变化
