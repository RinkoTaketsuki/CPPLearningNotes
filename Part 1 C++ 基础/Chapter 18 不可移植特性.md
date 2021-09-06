# Chapter 18 不可移植特性

> 此章节中的特性在使用时需注意其不可移植特性，即如果在多种机器上编译和运行时需要重构代码  

## 1. 位域

类可将其非静态数据成员设置为位域  
位域必须是整型或枚举类型，通常使用无符号类型，不同机器的有符号类型位域的行为不同  

```C++
// Example

typedef unsigned int Bit;
class File
{
    // The number following the colon represents the numbers of bits
    // occupied by such bit-field.
    Bit mode: 2;
    Bit modified: 1;
    Bit prot_owner: 3;
    Bit prot_group: 3;
    Bit prot_world: 3;
public:
    enum modes { READ = 01, WRITE = 02, EXECUTE = 03 };
    File &open(modes);
    void close();
    void write();
    bool isRead() const;
    void setWrite();
};

void File::write()
{
    modified = 1;
    /* other statements */
}

void File::close()
{
    if (modified)
        /* save modified contents */
}

File &File::open(File::modes m)
{
    mode |= READ; // set default open mode: READ

    /* process other open modes */

    if (m & WRITE)
        /* open current file as r/w mode */
    return *this;
}

inline bool File::isRead() const { return mode & READ; }
inline void File::setWrite() { mode |= WRITE; }
```

如果可能的话，类内部连续定义的位域会被压缩在同一个整数的相邻位，具体能否压缩以及如何压缩与机器相关  
取地址运算符不能作用于位域，任何指针也无法指向位域  

## 2. volatile 限定符

## 3. 链接指示 extern "C"
