# 草稿

## 左值引用与右值引用

### `const`引用

> `const`引用接收的**既可以是左值也可以是右值**

> **引用只能接收左值**

```C++
double &dr = 1; // 错误： 需要左值
const double &cdr {1}; // 正确
```

### 右值引用

**右值**有三类：

-   常量
    
-   **只读变量**
    
-   **临时变量**，函数返回值通常是一个临时变量，只被用作拷贝
    

右值引用可以接管临时变量的生命周期。当接管的右值是常量时等价于普通变量

```C++
ClassA &&alias = ClassA(); // 一次对象构造，只有一次构造
ClassA alias = ClassA();// 对象构造、拷贝构造，两次构造
```

## 面向对象

### 派生类向基类转换

派生类的对象（不是指针）向基类的隐士转换不会产生额外的临时对象。

# 并发编程

## 预备

-   线程与进程
    
-   并发与并行
    

### 为什么要使用并发技术？

1.  **分离关注点**
    

归类相关代码，隔离无关代码，程序变得更加清晰，更加容易理解和维护及测试

2.  **提升性能**
    
      任务并行，将一个大任务分解成多个小任务同时运行，降低总体的运行时间
    
      数据并行，同时对多组数据进行同样的操作
    

### 什么时候不能用并发技术？

性能的提升不如预期：大多数任务需要的执行时间小于线程切换的时间

线程是非常重要的有限资源：从空间看，每个线程都需要分配栈空间，4096个线程就会立刻耗光32位计算机的内存。

### 标准线程库

优点：
-   具有可移植性，更容易维护——不需要在不同平台上使用专属地底层API 
	为了可以便捷利用这些工具，同时又能照常使用标准C++线程库，C++线程库的某些类有可能提供成员函数native_handle()，允许它的底层直接运用平台专属的API
-   高效实现，低抽象损失
-   提供了高级工具

## 线程管理

### 线程的基本操作

1.  #### 启动线程
    -   函数
        ```C++
        void test();
        std::thread my_tread(test);
        ```
        
    -   可调动的类型
        
        ```C++
        struct Test{
            void operator ()();
        };
        std::thread my_thread(Test{});
        ```
        
    

2.  #### 等待线程与后台运行线程
    

**一旦启动了线程就需要明确是要等待其结束，还是与其脱离****。**

都不选择，会导致进程结束

> 如果标准选择了自动 detach 线程，考虑到线程可能会捕获局部作用域的资源的引用，在抛出异常、折叠栈帧之后局部作用域的资源被析构，而线程还在执行，整个程序进入未定义的状态。

  

`join()`、`detach`()需要注意的前提条件是`joinable()`满足。

`join()`是一种简单的线程等待方案，更加复杂的方案如超时等待，检查线程是否结束等，需要条件变量与future配合。

3.  #### 异常等待线程
    

如果线程启动以后有异常抛出，而`join()`尚未执行，则该`join()`调用会被略过。

-   使用`try/catch`
    
    ```C++
    struct func;    ⇽---  ①定义见代码清单2.1
    void f()
    {
        int some_local_state=0;
        func my_func(some_local_state);
        std::thread t(my_func);
        try {
            do_something_in_current_thread();
        }
        catch(...) {
            t.join();
            throw;
        }
        t.join();
    }
    ```
    
-   利用RAII机制：将thread用类封装，当类对象销毁时自动调用`thread.join()`
    

## 线程间共享数据

只读数据和私有数据不会引起线程安全问题， 共享的读写数据可能会出现不可预知的错误

《算法》这本书使用“**循环不变式**”分析算法，减少算法缺陷。并发编程也可以借用这一思想，不同的是并发编程是建立在基础算法之上的。

假设算法在单线程满足不变式，在多线程中非原子化的步骤（涉及共享数据的读写）将会破坏**不变式**

### 共享数据会带来什么问题？

共享数据会带来恶性的条件竞争，指多个线程操作其结果的正确与否与线程的相对执行顺序有关。

> 在并发编程中，操作由两个或多个线程负责，它们争先让线程执行各自的操作，而结果取决于它们执行的相对次序，所有这种情况都是条件竞争

### 如何防止条件竞争？

### 用互斥保护共享数据

####   如何使用C++互斥？

#####     方针：优先使用`lock_guard`加锁解锁

  `lock_guard`使用RAII机制封装`lock`对象，在析构时调用`unlock`。

  因此，`lock_guard`能在作用域内自动加解锁，满足在函数的所有退出路径上（异常退出）解锁。

####   使用管程思想保护共享数据

> 管程模型解决互斥问题的方法是：**将共享变量及对共享变量的操作统一封装起来。**

#####     方针：避免将共享数据以指针或引用暴露给外部

    避免成员函数返回共享变量的指针或引用

    避免在成员函数中将共享变量以指针或引用传递给第三方函数

---

  

## 并发操作同步

### 一、使用future等待一次事件发生

future特性：

-   **独占异步结果-move only**：`get`仅能被有效调用一次，数据准备好后`get`会进行移动操作
    
-   **自动推断类型**：使`std::shared_future`更便于使用。std::future具有成员函数share()，直接创建新的std::shared_future对象，并向它转移归属权。
    
    ```C++
    std::promise< std::map<SomeIndexType, SomeDataType, SomeComparator, SomeAllocator>::iterator> p;
    auto sf=p.get_future().share();
    ```
    

#### 异步任务返回结果—async

> 线程级别的一次同步。
> 
> 因为std::thread没有提供直接返回结果的方法，所以std::async应运而生

使用方法与`std::thread`基本类似，传递函数参数是复制到内部空间，传递引用使用`std::ref`

```C++
int test(int);

std::future<int> result = std::async(test, 1);
```

`std::async`支持同步和异步的方式执行任务，默认情况下自行选择

-   `launch::async`：异步执行
    
-   `launch::deferred`：延后执行（同步执行，调用`future`时才执行任务）
    

```C++
// 程序自行选择运行方式
std::async(std::launch::async | std::launch::deferred, ...)
```

#### 关联future与任务—packaged_task

> _任务级别_
> 
> std::packaged_task<>连结了future对象与函数（或可调用对象）。std::packaged_task<>对象在执行任务时，会调用关联的函数，把返回值保存为future的内部数据，并令future准备就绪

```C++
int test(int);

std::packed_task my_task(test);
// future传递给等待线程调用
std::future<int> result = my_task.get_future();
```

`std::packaged_task`是一个可调用类型，可作为函数直接调用，也可以作为线程函数传递给`std::thread`

#### 关联promise与future

> _语句级别_
> `std::promise<T>`给出了一种异步求值的方法，某个`std::future<T>`对象与结果关联，能延后读出需要求取的值。配对的`std::promise`和`std::future`可实现下面的工作机制：等待数据的线程在future上阻塞，而提供数据的线程利用相配的`promise`设定关联的值，使`future`准备就绪

```C++
// my_promise对象传递任务线程
std::promise<函数类型> my_promise;
// 任务线程准备好数据后调用set_value
my_promise.set_value(res);

// my_future 对象传递给等待线程
auto my_future = my_promise.get_future(); // my_future 被推导出future<函数类型>
// 等待线程调用get获取结果
auto result = my_future.get();
```

#### 将异常保存到future

`std::async`和`std::packaged_task`抛出异常会代替结果保存在`std::future`使其准备就绪，只要调用`get`异常就会被再次抛出。

`promise`使用`set_exception`保存异常

#### 多线程一起等待

`future`**只能移动不可复制**。

`share`成员函数将结果的所有权转交给`shared_future`

```C++
auto shared_my_future = my_future.share();

if (shared_my_future.valid()) { // 检查异步状态是否有效
    auto result = shared_my_future.get();
}
```

  

### 二、限时等待

> 有两种超时机制可供选用：
> 
> -   迟延超时，线程根据指定的时长而继续等待
>     
> -   绝对超时，在某特定时间点来临之前，线程一直等待
>     

~~处理迟延超时的函数变体以“_for”为后缀，而处理绝对超时的函数变体以“_until”为后缀。~~

#### 时钟类

C++标准库提供了多种时钟类，每个时钟的**关键信息**：

-   当前时间点：`chrono::some_clock::now()`
    
-   时间值的类型：从该时钟取得的时间以它为表示形式，每个时钟类都有一个`time_point`成员类型~~，~~~~`now`~~~~的返回值类型就是~~~~`time_point`~~
    
-   时钟的计时单元长度：描述时间的单元是`period`的成员类型，用秒的分数形式表示。~~例如以~~~~`ratio<60,1>`~~~~为计时单元，即60秒为一个单位也就是一分钟~~
    
-   是否是恒稳时钟：从`is_steady`静态成员函数获取。恒稳时钟不可以调整计时速率，前后两次调用`now`不会出现后来返回的值早于前一个，**恒稳时钟对于超时时限的计算非常重要**。
    

`system_clock`系统时钟类

`steady_clock`恒稳时钟类

`high_resolution_clock`高精度时钟类

  

#### 时长类

> `std::chrono::duration<>`是标准库中最简单的时间部件，具有两个模板参数，前者指明采用何种类型表示计时单元的数量，后者是一个分数，设定该时长类的计时单元。计时单元的数量可通过成员函数`count()`获取

-   手动设定：`std::chrono::duration<short,std::ratio<60,1>>`
    
-   预定义的typedef声明：
    
    > nanoseconds、microseconds、milliseconds、seconds、minutes和hours，分别对应纳秒、微秒、毫秒、秒、分钟、小时。
    
-   C++14引入了名字空间`std::chrono_literals`，其中预定义了一些字面量后缀运算符（literal suffix operator）
    
    ```C++
    using namespace std::chrono_literals;
    auto one_day=24h;
    auto half_an_hour=30min;
    auto max_time_between_messages=30ms;
    ```
    

显示时长转换： `std::chrono::duration_cast<>`

支持迟延超时的等待需要用到`std::chrono::duration<>`的实例

```C++
std::future<int> f=std::async(some_task);
if(f.wait_for(std::chrono::milliseconds(35))==std::future_status::ready)
    do_something_with(f.get());
```

**超时等待的返回状态：**

-   一旦超时，函数就返回`std::future_status::timeout`
    
-   假如准备就绪，则函数返回`std::future_status::ready`
    
-   future的相关任务被延后，函数返回`std::future_status::deferred`
    

#### 时间点类

> 时间点是一个时间跨度，始于一个称为时钟纪元的特定时刻，终于该时间点本身。典型的时钟纪元包括1970年1月1日0时0分0秒，或运行应用的计算机的启动时刻

~~时间点用于带有后缀“_until”的等待函数的变体~~。为了预先安排操作，我们需计算某一具体未来时刻，最“地道”的方法是在程序代码中的某个固定位置，将some_clock::now()和前向偏移相加得出时间点

```C++
auto timeout = std::chrono::steady_clock::now() + std::chrono::milliseconds(500);
```

## 内存模型与原子操作

### C++内存模型

#### **Synchronizes with**

> _synchronized-with是個發生在兩個不同thread間的同步行為，當A synchronized-with B的時，代表A對記憶體操作的效果，對於B是可見的。而A和B是兩個不同的thread的某個操作。_

#### Happens-before

  

#### 内存顺序

#####   memory_order_seq_cst

> **It provides the same restrictions and limitation to moving loads around that sequential programmers are inherently familiar with, except it applies across threads**.

  `memory_order_seq_cst`顺序在单线程中与程序员熟悉的“顺序”编程一样。编译器可以重排原子操作间的“动作”顺序，但不能跨原子操作排序。

  从另一个角度看，`memory_order_seq_cst`顺序的原子操作，其作用类似于“屏障（barrier）”

#####   memory_order_relaxed

> **This model allows for much less synchronization by removing the** _**happens-before**_ **restrictions.**

#####   memory_order_acquire / memory_order_release

> The third mode is a hybrid between the other two. The acquire/release mode is similar to the sequentially consistent mode, except it **only applies a** _**happens-before**_ **relationship to dependent variables**. This allows for a relaxing of the synchronization required between independent reads of independent writes.

`memory_order_release`顺序禁止将原子操作之前的读或写重排到原子操作之后

`memory_order_acquire`顺序禁止将原子操作之后的读或写重排到原子操作之前

### C++中的原子操作及类别

####   标准原子类型概述

> 原子类的所有操作全是原子化的

  所有原子类（除了`atomic_flag`）都有`is_lock_free`成员函数——这是因为有些是借助锁实现的。

  C++17提供了`static constexpr bool is_always_lock_free`在编译器判断是否属于无锁数据结构。此外标准库内置了宏

```C++
ATOMIC_BOOL_LOCK_FREE、ATOMIC_CHAR_LOCK_FREE、ATOMIC_CHAR16_T_LOCK_FREE、ATOMIC_CHAR32_T_LOCK_FREE、ATOMIC_WCHAR_T_LOCK_FREE、ATOMIC_SHORT_LOCK_FREE、ATOMIC_INT_LOCK_FREE、ATOMIC_LONG_LOCK_FREE、ATOMIC_LLONG_LOCK_FREE和 ATOMIC_POINTER_LOCK_FREE
```

  **`atomic_flag`****是最基本的无锁布尔标志，用它可以实现其他所有原子类型。**其他的原子类型都是`atomic<>`特化，功能更全，但可能不是无锁结构

#####     方针：尽量使用标准原子类的`atomic<>`特化，而不是别名

#####     原子类特点：

-   原子类不可拷贝不可复制——拷贝构造和拷贝复制涉及到两个不同的对象，无法原子化（不用互斥）。
    
-   支持基本类型赋值或构造，也可以隐式转成基本类型
    
-   提供成员函数处理，如load()和store()、exchange()、compare_exchange_weak()和compare_exchange_strong()等
    
-   支持算术运算、赋值运算
    
-   提供三种内存序：
    
    -   读操作：std::memory_order_relaxed、std::memory_order_consume、std::memory_order_acquire或std::memory_order_seq_cst。
        
    -   写操作：std::memory_order_relaxed、std::memory_order_release或std::memory_order_seq_cst
        
    -   读-改-写操作：std::memory_order_relaxed、std::memory_order_consume、std::memory_order_acquire、std::memory_order_release、std::memory_order_acq_rel或std::memory_order_seq_cst。
        

##   设计基于锁的并发数据结构

-   线程安全的数据结构
    
    > 多个线程可以并发读写的数据结构，而不会出现数据不一致或不可预期的结果。
    
-   用互斥保护数据结构不是真正的并发，只是串行化
    

###     设计并发数据结构的指引

-   线程安全
    
    -   确保线程不会破坏数据结构的“不变式”
        
    -   小心接口固有的条件竞争；尽力提供独立，而非零散的接口
        
    -   保证接口是异常安全的
        
    -   避免死锁
        
-   保证高并发
    
    -   尽量减少锁的范围
        
    -   尽量采用细粒度的锁；分离冲突数据；合理选择底层容器
        

# 指导方针

出自[Effective Modern C++](https://cntransgroup.github.io/EffectiveModernCppChinese/Introduction.html)

1.  ## 类型推导
    

### 条款：理解模板类型推导的三种情况

```C++
template<typename T>
void f(ParamType param);

f(expr);                        //从expr中推导T和ParamType
```

  模板类型的推导是在编译期间，编译器根据`expr`的类型与`ParamType`进行**模式匹配**，从而推导`ParamType`的类型和`T`的类型。

  参数有实参和形参的区别，形参可以在实参的基础上增加类型修饰符（引用、指针或const等）

```C++
template<typename T>
void f1(T& param);

const int x = 22;
f1(x);                      // param类型是const int&,  T的类型是const int

template<typename T>
void f2(const T& param);
f2(x);                     // param类型是const int&, T的类型是int
```

**情况1：ParamType是一个指针或引用，但不是万能引用**

-   如果`expr`的类型是一个引用，忽略引用部分
    
-   然后`expr`的类型与`ParamType`进行模式匹配来决定`T`
    

  

**情况2：ParamType是一个万能引用**

-   如果`expr`是左值，`T`和`ParamType`都会被推导为左值引用
    
-   如果`expr`是右值，就使用**情况1**推导规则
    

  

**情况3：ParamType既不是指针也不是引用（值传递）**

-   如果`expr`的类型是一个引用，忽略这个引用部分
    
-   如果忽略`expr`的引用性（reference-ness）之后，`expr`是一个`const`，那就再忽略`const`。如果它是`volatile`，也忽略`volatile`
    

### 条款：了解类型推导的特殊情况：数组形参和函数形参会退化成指针形参

  数组声明允许退化成指针声明

```C++
const char name = "huang zhen";
const  char* s_name = name;
```

特别需要注意：**数组形参与指针形参是等价的**

```C++
void myFunc(int param[]);
// 等价于
void myFunc(int* param);
```

  但是**可以**接受指向数组的**引用**，这样就可以推导出数组类型

```C++
template<typename T>
void f(T& param);                       //传引用形参的模板

const char name = "huang zhen";

f(name); // T被推导成 const char (&) [11]

//在编译期间返回一个数组大小的常量值（//数组形参没有名字，
//因为我们只关心数组的大小）
template<typename T, std::size_t N>                    
constexpr std::size_t arraySize(T (&)[N]) noexcept {return N;}
```

  

### 条款：理解`auto`类型推导及与模板类型推导的不同点

与模板类型推导相同，但是`auto`类型推导假定花括号初始化代表`std::initializer_list`，而模板类型推导不这样做

C++14的返回值`auto`的推导规则使用的是模板类型推导的规则。

  

### 条款：优先考虑`auto`而非显示类型声明

优点：

1.  声明复杂类型非常简单，如配合`decltype`使用lambda表达式
    
2.  **避免一些隐式类型转换，兼容性更好**
    

缺点：

1.  **有些隐式类型转换是必要**，这种情况常见于**代理类**的实现。例如`std::vector<bool>`的`operator[]`操作不会返回`bool&`，它会返回一个全新的对象（MSVC的STL实现中返回的是`std::_Vb_reference<std::_Wrap_alloc<std::allocator<unsigned int>>>`对象）
    
2.  降低了代码的可读性。但是可以配合clangd在代码编辑器中显示编译器的类型推导结果
    

# 面向对象设计

##   草稿

###   方针： 组合优于继承

-   **继承打破了超类的封装**
    
-   **子类与超类紧密的耦合**： 超类中的任何修改都有可能会破坏子类的功能
    
-   **通过继承复用代码可能会导致平行继承体系的产生**：
    

> 每当你为某个类增加一个子类，必须也为另一个类相应增加一个子类。如果你发现某个继承体系的类名前缀和另一个继承体系的类名前缀完全相同，便是闻到了这种坏味道。

![](https://vastaitech.feishu.cn/space/api/box/stream/download/asynccode/?code=NDE3NDFiY2VkM2M3NzNhMThkYzRkY2EwYTc3MzhjMmFfcmVUcGtpT1pCZWFqV0JKTld1SzdGRVc4UnJUaFNFaENfVG9rZW46Ym94Y25xc0czeEt0RzVzUzcyNFRWaG92RGRjXzE2Nzc3NDYzODQ6MTY3Nzc0OTk4NF9WNA)![](https://vastaitech.feishu.cn/space/api/box/stream/download/asynccode/?code=NWQzOGFiMWY4YzAzODFlYmE1YjQ2MDc2ZjA5OTFjNzdfZjZ0ZjFRbUlQQUtuWVRNbDVIRXAzS0hEcUdXU2dod2JfVG9rZW46Ym94Y250SnRQTWpnWUlVUDR1N3Y1Z3U2TmtjXzE2Nzc3NDYzODQ6MTY3Nzc0OTk4NF9WNA)

## 外部多态模式

> https://docs.huihoo.com/ace_tao/c++.html
> 
> 允许没有继承关系的类能够被多态地对待

###   如何做到的？

  如果希望对任意的类实例都能执行某些操作，就需要将所有活动的类实例收入到集合中，然后遍历集合中的实例，执行操作。因为集合是同质的（即要求其中所有的对象是相同类型的），为了维护一个单一的集合，一个共同的基类必须存在。——模板 + 委托

  为了调用统一的接口，就需要接口/函数适配器，将不同的操作投射到一起。

![](https://vastaitech.feishu.cn/space/api/box/stream/download/asynccode/?code=ZjBmM2FiYmY2OTdhNTMxZTVjYWZjMDQzYzM4M2Q2NzNfcnBYYUNCcnVYdjhqVnYxUmRwcnJER1lIRWRhbDk1U0hfVG9rZW46Ym94Y25hcFlObXhpMzJRanlYeTJpQ2tEUEpoXzE2Nzc3NDYzODQ6MTY3Nzc0OTk4NF9WNA)

## 创造型模式

###   原型模式

  

# 外部库

## 数学库

### #Eigen

[Eigen](https://eigen.tuxfamily.org/dox/group__QuickRefPage.html)是线性代数的C ++模板库：**矩阵，向量，数值求解器以及相关算法**。

**Eigen 的一些优化：**
- 针对小目标，使用栈内存相当于原始数组`T []`
- 针对大目标，使用的堆内存，支持多线程`OpenMP`
- 支持向量化`SIMD`

#### 1. 使用 CMake 创建Eigen项目

```cmake
find_package(Eigen3 REQUIRED) // 添加Eigen3库

add_execute(test main.cc)
target_link_libraries(test Eigen3:Eigen)
```

#### 2. 基础实践

##### Array, Matrix, Vector类型
Eigen提供了两种类型：`Array`与`Matrix`，它们的模板类声明如下：
```C++
template<Scalar, RowsAtCompileTime, ColsAtCompileTime, Options>
class Matrix: public MatrixBase<Matrix<Scalar, RowsAtCompileTime, ColsAtCompileTime, Options>>;

template<Scalar, RowsAtCompileTime, ColsAtCompileTime, Options>
class Array: public ArrayBase<Matrix<Scalar, RowsAtCompileTime, ColsAtCompileTime, Options>>;
```
- `Scalar`指容器的数据类型，支持自定义
- `RowAtCompilerTime`和`ColsAtCompileTime`是指在编译期确定的行数和列数
- `Options`有`RowMajor`和`ColMajor`，指容器的数据布局，默认情况下列优先的格式存储

##### 构造方法
> Array与Matrix通用

Fixed-size类型：
```C++
Matrix<int, 1,2> m(); // 构造无初始化

Matrix<int, 1, 4> m(1, 2, 3, 4) // 初始化元素，最多只支持1-4个元素
```

dynamic-size类型：
```C++
Array<int, -1, 1> array(6); // size 6 * 1
Array<int, -1, -1> array(6, 6); // size 6 * 6
```

##### 初始化

```C++
array << 1, 2, 3, 4, 5; // 初始化的元素必须与 array 的 size 一致
array << other_array; // 两个对象的 size 必须一致
```

