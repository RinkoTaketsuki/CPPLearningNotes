# Chapter 11 语句

## 1. 分类

- 表达式语句：最终返回值被丢弃
- 空语句：使用时最好添加注释（null statement）
- 复合语句（块）：不加分号，本身是一个作用域，if，switch，while，for语句的控制循环部分中定义的变量和循环体是同一个作用域

## 2. if语句的悬垂else问题

> 悬垂else：if语句的数量多于else时，不同语言的if-else匹配方式不同

```C++
if (condition1)
    if (condition2)
        statement1;
else
    statement2;
// 从缩进来看，看似是else与第一个if语句匹配，但根据C++的就近匹配原则，else会与
// 第二个if语句匹配
```

## 3. switch语句

```C++
switch (expr)
{
    case value1:
        statement1;
    case value2:
        statement2;
    case value3:
        statement3;
    ...
    default
        statementn;
}
```

1. 执行顺序：expr可以是表达式，也可以是声明，将expr的值转化为整型，再和case中的值比较，假设与第n个case匹配成功，则遇到break为止运行第n个case及以下的case和default中的全部语句
2. case标签必须是整型常量表达式，任意两个标签的值不能相同
3. case标签内不能初始化变量（可以在代码块内初始化变量），但可以定义和声明变量，且声明语句下面的case和default均可使用该变量
4. 巧用不带break的switch语句

    ```C++
    switch (ch)
    {
        // 检查是否是小写元音字母
        case 'a': case 'e': case 'i': case 'o': case 'u':
        ++cnt; break;
    }
    ```

## 4. while语句执行顺序

```C++
while (condition) 
{
    statement;
}
```

1. 检查condition是否为真。若为假，则结束，若为真，则继续
2. 执行statement，跳转回步骤 1

> 若是do-while语句则从2开始，即statement至少执行一次，注意while()后面要加分号

## 5. for语句执行顺序

小括号中的部分为循环头，大括号中的部分为循环体  
循环头定义的变量和循环体中定义的变量作用域相同

```C++
for (init-statement; condition; expression) 
{
    statement;
}
```

1. 先执行init-statement
2. 检查condition是否为真。若为假，则结束，若为真，则继续
3. 执行statement
4. 执行expression，跳转回步骤2

## 6. 范围for语句

```C++
for (declaration: expression) 
{
    statement;
}
```

declaration：循环控制变量，expression：迭代序列  
相当于Python中的 `for declaration in expression:`  
如需修改declaration中的值则需声明为引用类型  
范围for语句是先赋值循环控制变量，再执行statement

## 7. try语句

```C++
try
{
    /* program-statements */
}
catch(/* exception declaration1 */)
{
    /* handler-statements1 */
}
catch(/* exception declaration2 */)
{
    /* handler-statements2 */
}
catch(/* exception declaration3 */)
{
    /* handler-statements3 */
}
...
```

1. program-statements部分声明和定义的变量在外部无法访问，在catch子句内也无法访问
2. 异常处理时从里往外寻找catch子句，若最终找不到catch则会调用std::terminal()函数终止程序运行

## 8. 异常类

> 在此只讨论stdexcept头文件中定义的异常类

1. 只能默认初始化exception类，其他必须用string或C字符串初始化
2. what()无参数，返回const char*

类名|说明
:-:|:-:
exception|异常类
runtime_error|运行时错误类
range_error|
overflow_error|
underflow_error|
logic_error|逻辑错误类
domain_error|
invalid_argument|
length_error|
out_of_range|
