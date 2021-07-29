# Chapter 24 类定义范式

## 1. 行为像值的类

```C++
class Value
{
public:
    Value(const typeB b = typeB()): 
        memA(typeA()), memB(new typeB()) {}
    Value(const Value &v):
        memA(v.memA), memB(new typeB(*v.memB)) {}
    Value &operator=(const Value &v)
    {
        auto newMemB = new typeB(*v.memB);
        delete memB;
        memB = newMemB;
        memA = v.memA;
        return *this;
    }
private:
    typeA memA;
    typeB *memB;
};
```

如果不定义newMemB变量，直接给memB动态分配内存，则当v和自身是同一个对象时，会导致memB指向无效内存

## 2. 行为像指针的类
