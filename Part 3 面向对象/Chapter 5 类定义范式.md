# Chapter 5 类定义范式

## 1. 行为像值的类

```C++
class HasPtr
{
public:
    HasPtr(const std::string &s = std::string()): 
        ps(new std::string(s)), i(0) {}
    
    HasPtr(const HasPtr& p):
        ps(new std::string(*p.ps)), i(p.i) {}
    
    HasPtr(HasPtr &&p) noexcept: ps(p.ps), i(p.i) {p.ps = nullptr;}
    
    ~HasPtr() {delete ps;}

    HasPtr &operator=(const HasPtr &rhs)
    {
        /* 如果不定义 newPs 变量，直接给 ps 动态分配内存，
         * 则当 v 和自身是同一个对象时，会导致 ps 指向无效内存
         */
        auto newPs = new std::string(*rhs.ps);
        delete ps;
        ps = newPs;
        i = rhs.i;
        return *this;
    }

    // 使用 swap 进行赋值的版本，由于左边对象被交换到右边，非自赋值时会调用析构函数
    // 该函数同时是拷贝和移动赋值运算符
    // 当 p 的实参是左值时，调用拷贝构造函数进行初始化
    // 当 p 的实参是右值时，调用移动构造函数进行初始化
    HasPtr &operator=(HasPtr p) noexcept
    {
        swap(*this, p);
        return *this;
    }

private:
    std::string *ps;
    int i;
};
```

## 2. 行为像指针的类

```C++
class HasPtr
{
public:
    friend void swap(HasPtr&, HasPtr&);
    # define SWAP

    // 转换和默认构造函数将计数器置 1
    HasPtr(const std::string &s = std::string()):
        ps(new std::string(s)), i(0), use(new std::size_t(1)) {}
    
    // 拷贝构造函数递增计数器
    HasPtr(const HasPtr &p):
        ps(p.ps), i(p.i), use(p.use) {++*use;}
    
    #ifndef SWAP
    // 赋值运算符递增右边对象计数器，递减左边对象计数器，必要时会先释放左边指向的对象
    HasPtr &operator=(const HasPtr &rhs)
    {
        ++*p.use;
        if (--*use == 0)
        {
            delete ps;
            delete use;
        }
        ps = p.ps;
        i = p.i;
        use = p.use;
        return *this;
    }
    #endif

    // 使用 swap 进行赋值的版本，由于左边对象被交换到右边，非自赋值时会调用析构函数
    HasPtr &operator=(HasPtr p) noexcept
    {
        swap(*this, p);
        return *this;
    }

    ~HasPtr()
    {
        // 递减引用计数，若递减到 0，则说明当前对象为唯一指针
        if (--*use == 0)
        {
            delete ps;
            delete use;
        }
    }
private:
    std::string *ps;
    int i;
    std::size_t *use; // 引用计数，使用动态内存可以使不同的 HasPtr 对象使用相同的计数器
};

inline void swap(HasPtr &p, HasPtr &q)
{
    using std::swap; 
    // 成员类型无 swap 函数时，才用 std::swap()
    // 否则用自带的 swap
    // 自带 swap 的优先级高于此 using 声明的优先级
    swap(p.ps, q.ps);
    swap(p.i, q.i);
}
```

## 3. 动态内存管理类

```C++
class StrVec
{
public:
    #define MOVE
    StrVec():
        elements(nullptr), first_free(nullptr), cap(nullptr) {}
    
    StrVec(const StrVec &s)
    {
        std::pair<std::string*, std::string*> newData = alloc_n_copy(s.begin(), s.end());
        elements = newData.first;
        first_free = cap = newData.second;
    }

    StrVec(StrVec &&s) noexcept: elements(s.elements), first_free(s.first_free), cap(s.cap)
    {
        // 防止当前的空间被销毁
        s.elements = s.first_free = s.cap = nullptr;
    }

    ~StrVec()
    {
        free();
    }

    // 分配一块新的内存，释放自身内存，再将新内存赋给自身，而不是直接赋值，以处理自赋值
    StrVec &operator=(const StrVec &rhs)
    {
        std::pair<std::string*, std::string*> data = alloc_n_copy(rhs.begin(), rhs.end());
        free();
        elements = data.first;
        first_free = cap = data.second;
        return *this;
    }

    StrVec &operator=(StrVec &&rhs) noexcept
    {
        if (this != rhs)
        {
            free();
            elements = rhs.elements;
            first_free = rhs.first_free;
            cap = rhs.cap;
            rhs.elements = rhs.first_frss = rhs.cap = nullptr;
        }
        return *this;
    }

    StrVec &operator=(initializer<string> il)
    {
        std::pair<std::string*, std::string*> data = alloc_n_copy(il.begin(), il.end());
        free()
        elements = data.first;
        first_free = cap = data.second;
        return *this;
    }

    std::string &operator[](std::size_t n)
    {
        return elements[n];
    }

    const std::string &operator[](std::size_t n) const
    {
        return elements[n];
    }

    void push_back(const std::string &s)
    {
        chk_n_alloc();
        alloc.construct(first_free++, s);
    }

    template <typename... Args>
    void emplace_back(Args &&...args)
    {
        chk_n_alloc();
        alloc.construct(first_free++, std::forward<Args>(args)...)
    }

    std::size_t size() const
    {
        return first_free - elements;
    }

    std::size_t capacity() const
    {
        return cap - elements;
    }

    std::string *begin() const
    {
        return elements;
    }

    std::string *end() const
    {
        return first_free;
    }

private:
    // 分配内存并将范围内元素拷贝到新内存空间中
    // 返回新内存空间指针和拷贝后的元素的尾后指针
    std::pair<std::string*, std::string*>
    alloc_n_copy(const std::string* b, const std::string* e)
    {
        std::string *data = alloc.allocate(e - b);
        return {data, uninitialized_copy(b, e, data)};
    }

    // 逆序销毁元素并释放内存
    void free()
    {
        // 检查序列是否为空
        if (elements)
        {
            for (std::string *p = first_free; p != elements;)
                alloc.destroy(--p);
            alloc.deallocate(elements, cap - elements);
        }
    }

    #ifndef MOVE
    // 获取更多内存并将已有元素拷贝到新内存空间中
    void reallocate()
    {
        // 新内存大小为原来 size 的 2 倍或 1
        std::size_t newCapacity = size() ? 2 * size() : 1;

        std::string 
        *newData = alloc.allocate(newCapacity), // 新内存空间
        *dest = newData, // 当前移动目标位置
        *elem = elements; // 当前被移动元素位置
        
        // 将旧空间中的元素移动到新空间中
        for (size_t i = 0; i != size(); ++i)
            alloc.construct(dest++, std::move(*elem++))
        
        // 释放旧空间
        free();

        // 更新成员
        elements = newData;
        first_free = dest;
        cap = elements + newCapacity;
    }
    #endif

    // 使用移动迭代器的版本
    void reallocate()
    {
        std::size_t newCapacity = size() ? 2 * size() : 1;
        std::string 
        *first = alloc.allocate(newCapacity),
        *last = uninitialized_copy(
            make_move_iterator(begin()),
            make_move_iterator(end()),
            first
        );
        free();
        elements = first;
        first_free = last;
        cap = elements + newCapacity;
    }
    
    // 若空间已满，则分配新内存空间
    void chk_n_alloc()
    {
        if (size() == capacity())
            reallocate();
    }
    static std::allocator<std::string> alloc;
    std::string *elements; // 指向首元素
    std::string *first_free; // 指向第一个空闲元素
    std::string *cap; // 指向当前分配空间的尾后位置
};
```
