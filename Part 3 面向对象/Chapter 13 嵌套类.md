# Chapter 13 嵌套类

嵌套类在外层类中定义了一个类型成员，该类型的访问权限由外层类决定，即如果嵌套类是 `protected` 的，则只有外层类内部、外层类的派生类和外层类的友元可以访问  
注意类型成员必须先声明后使用，即便是在外层类的内部，但只声明了的嵌套类是不完全类型，不能在外层类中使用  
嵌套类只能使用外层类的静态成员和类型成员  
外层类和嵌套类相互独立，其对象不互相包含成员  

```C++
class OUT
{
    class IN;
    IN mem; // 错误，IN 是不完全类型
    int mem2;
    static int smem;
    typedef int integer;
};

OUT::smem = 42;

class OUT::IN
{
    integer m1 = mem2; // 错误
    integer m2 = smem; // 正确
public:
    IN(integer, integer);
};

OUT::IN::IN(OUT::integer n1, OUT::integer n2)
{
    /* construct function */
}
```
