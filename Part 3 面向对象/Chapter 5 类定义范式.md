# Chapter 5 类定义范式

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

```C++
class ValuePtr
{
public:
    friend void swap(ValuePtr&, ValuePtr&);
    ValuePtr(const Value &v = Value()):
        memA(typeA()), memB(new Value(b)), use(new size_t(1)) {}
    ValuePtr(const ValuePtr &p):
        memA(p.memA), memB(new Value(*p.memB)), use(p.use) {++*use;}
    ValuePtr &operator=(const ValuePtr &p)
    {
        ++*p.use; // 自赋值安全
        if (--*use == 0)
        {
            delete memB;
            delete use;
        }
        memA = p.memA;
        memB = p.memB;
        use = p.use;
        return *this;
    }
    ValuePtr &operator=(ValuePtr p)
    {
        swap(*this, p);
        return *this;
    }
    ~ValuePtr()
    {
        if (--*use == 0)
        {
            delete memB;
            delete use;
        }
    }
private:
    typeA memA;
    Value *memB;
    size_t *use;
};
inline void swap(ValuePtr &p, ValuePtr &q)
{
    using std::swap; 
    // typeA 和 Value 类无 swap 函数时，才用 std::swap()
    // 否则用自带的 swap
    // 自带 swap 的优先级高于此 using 声明的优先级
    swap(p.memA, q.memA);
    swap(p.memB, q.memB);
}
```

## 3. 动态内存管理类

```C++
class ValueVec
{
    // TODO P464
};
```
