



# 1. 语言基础

## 1.1 关键字

### `const`与`volatile`

#### **类型限定符**

> `const` 定义类型为常量
> volatile 定义类型为易变

**const对象**——对象不能被修改。直接尝试修改对象内容，编译器报错。

> const只有用在成员函数时可以算作函数签名的一部分

**volatile对象**——类型为 `volatile-`限定的对象，通过 `volatile`限定的类型的泛左值表达式的每次访问（读或写操作、成员函数调用等），都被当作对于优化而言是可见的副作用（即在单个执行线程内，volatile 访问不能被优化掉，或者与另一[按顺序早于](https://zh.cppreference.com/w/cpp/language/eval_order)或按顺序晚于该 volatile 访问的可见副作用进行重排序）

#### **限定成员函数**

在有 cv 限定符的函数体内，`*this` 有同样的 cv 限定

#### `const T&`

`const T&`的初始化可以是左值也可以是右值。

```c++
double &dr = 1; // 错误： 需要左值
const double &cdr {1}; // 正确
```

`const T&`初始化的解释可能如下:

```c++
double tmp = double{1}; // 首先创建一个右值的临时变量
const double &cdr{tmp}; // 然后用临时变量初始化cdr
```

### extern

**1. 语言链接**

> 提供以不同程序语言编写的模块间的链接。

**基本语法**

```c++
extern 语言链接名 [声明, {声明序列}]
```

由于**C++支持函数重载，而C语言不支持**，因此函数被C++编译后在**`符号库`**中的函数签名是与C语言不同的；C++编译后的函数需要加上参数的类型才能唯一标定重载后的函数，而加上extern "C"后，是为了向编译器指明这段代码按照C语言的方式进行编译

**2. 静态存储期和外部链接**

extern关键字可以访问到其他翻译单元中的变量或函数，只需要在当前翻译单元声明即可

```c++
//a.cpp
extern int a;//定义和声明
int add();//默认是extern的


//b.cpp
int a = 0;
int add() {
    ...
}


```

**3. 显示模板实例化声明（不常用）**

```c++
template<class T>
struct Z // 模板定义
{
    void f() {}
    void g(); // 永远不会定义
};
 
extern template struct Z<double>; // 显式实例化 Z<double>
Z<int> a;                  // 隐式实例化 Z<int>
Z<char>* p;                // 此处不实例化任何内容
 
p->f(); // 隐式实例化 Z<char> 且 Z<char>::f() 在此出现。
        // 始终不需要且不实例化 Z<char>::g()：不必对其进行定义
```



### static

> **静态存储期**和**内部链接**

静态存储期：程序开始时分配，并在程序结束时解分配。这类对象只存在一个实例

内部链接:名字可从当前翻译单元（文件）中的所有作用域使用。在**命名空间作用域**声明的下列任何名字均具有内部链接。

**简单来说，当前翻译单元就是内部链接名字的命名空间**

**在块作用域声明的下列任何名字均无链接**

#### 面向过程中的static

**1. 静态全局变量**

- 存放在静态区（BSS段或数据段）
- 无法对其他文件可见(*区别于全局变量*)

**2. 静态局部变量**

- 存放在静态区
  
    > 静态区特性1：变量在运程序开始时分配内存，并且只存在一个
- 局部变量，作用于函数内部

**3. 静态函数**

- 声明函数名是内部链接的，其他文件可以使用同名。

#### 面向对象中的static

**1. 静态数据成员**

静态成员函数不关联到任何对象。*整个程序中只有一个拥有静态存储期的静态数据成员实例*。

**2. 静态成员函数**

静态成员函数不关联到任何对象。调用时，它们没有 this 指针。
静态成员函数的地址可以存储在常规的函数指针中，但不能存储在成员函数指针中。

## 静态多态与动态多态

静态多态（函数重载和运算符重载 +类），是在编译的时候，就确定调用函数的类型；动态多态（虚函数实现），在运行的时候，才能确定调用的是哪个函数，动态绑定。运行基类指针指向派生类的对象，并调用派生类的函数。

## 函数类型与函数声明

```c++
void say_hello(const char *str); // 函数
void (*fptr)(const char *); // 函数类型

```



## C语言

### 实现一个`memcpy`函数

> 注意点:`void*`**不可以进行算术运算**，++--必须要要知道指针的类型

```c
void* memcpy(void* dest, void* src, size_t len) {
    if (dest ==     NULL || src == NULL) {
        return NULL;
    }

    //判断是否内存重叠
    if (dest <= src || (char*) dest > (char*)src + len) {
        //无重叠，从低地址往高地址复制
        while (len --) {
            *(char*)dest = *(char*)src;
            (char*) src ++;
            (char*) dest ++;
        }
    }
    else{
        //有重叠，从高地址往低地址复制
         while (len --) {
            *(char*)dest = *(char*)src;
            (char*) src --;
            (char*) dest --;
        }
    }
}

```



## 异常处理与异常安全

> 用异常和断言是避免程序错误

### 异常设计——契约

> Eiffel观点: 契约破坏才会有Exception 

<img src="C:\Users\zhuang\AppData\Roaming\Typora\typora-user-images\image-20220719211549822.png" alt="image-20220719211549822" style="zoom:50%;" />



面向对象程序设计中经常讨论的一个设计方法是契约设计，它指出方法是**客户（方法的调用者）和声明方法的类之间的契约**。这个契约包括客户必须满足的前置条件（precondition）和方法本身必须满足的后置条件（postcondition）等，具体如上图所示。

**比如String类的charAt(index)方法**

**前置条件**: 客户传入的index参数的最小取值是0，最大取值是在该String对象上调用length()方法的结果减去1。如果客户违反契约，它将抛出异常`StringIndexOutOfBoundsException`

**后置条件:** `String`类的`charAt(int index)`方法的后置条件要求返回值必须是该字符串对象在index位置上的字符数据

### C++中的异常类

<img src="http://c.biancheng.net/uploads/allimg/190218/114Q24150-0.jpg" alt="C++ exception类层次图" style="zoom:80%;" />

### 异常安全

以下是四个被广泛认可的异常保证等级:

- 不抛出异常保证: 函数始终不会抛出异常
- 强异常保证: 如果函数抛出异常，那么程序的状态会恰好被回滚到该函数调用前的状态
- 基础异常保证
- 无异常保证

# 2. 内存管理

**c++内存管理工具：**

|           分配           |            回收            |   类属    |            可否重载            |
| :----------------------: | :------------------------: | :-------: | :----------------------------: |
|          malloc          |            free            |   C函数   |              不可              |
|           new            |           delete           | C++表达式 |              不可              |
|     ::operator new()     |    ::operator delete()     |  C++函数  |               可               |
| allocator<T>::allocate() | allocator<T>::deallocate() | C++标准库 | 可自由设计并以之搭配人任何容器 |



## 2.1. 基本构建

### `new`、`delete`表达式

```c++
MyClass *A = new MyClass(...);
```

**1. 表达式`new`可能会被编译器拆分为三步：**

> 只有编译器才能像第三步一样调用构造函数，想直接调用ctor，可运行`placement new`

```c++
// 1. 调用::operator new
void* ptr = ::operator new(sizeof(A));
// 2. 类型转换
MyClass *A = static_cast<MyClass*>(ptr);
// 3. 调用构造函数
A->MyClass::MyClass(...)
```

**VC6版的`::operator new`函数：**

```c++
void* operator new(size_t size) {
    // try to malloc size bytes
    void* p;
    while ((p = malloc(size)) == 0) {
        _TRY_BEGIN
            if (_callnewh(size) == 0) break; // 自定义的函数，给应用机会释放不必要内存
        _CATCH(std::bad_malloc)
        _CATCH_END
    }
    return p;
}
```

> 当`operator new`无法分配出申请的内存时，会抛出`std::bad_alloc`异常。在抛出异常之前会不止一次调用可由**用户指定的handler**

handler的定义和使用方法如下

```c++
typedef void(*new_handler)();
new_handler set_new_handler(new_handler p) throw();
```

设计良好的new handler应该做到两件事:

- 释放更多的memory可用
- 调用`abort()`或`exit()`

**2. 表达式`delete`可能会被编译器拆解为两步：**

```c++
// (1) 先析构
pc->~Complex(); // 
// (2) 然后释放内存
operator delete(pc) // 
```

### arry new/delete表达式

> `arry new`只能使用默认的构造函数

```c++
MyClass *ptr = new MyClass[3]; // 调用3次ctor，只能使用默认的构造函数
...
delete[] ptr; // 调用3次dtor
```

**`malloc`分配内存**的时候带有**`cookie信息(cookie大小8字节)`**包括记录了分配几个对象等，`free`函数根据cookie信息去回收内存

<img src="D:\mygit\notes\images\20220629164152.png" style="zoom:33%;" />

#### 内存泄漏的情况

对于含有指针的类, **一次delete能够回收3个内存，但只调用了一次dtor**，会导致内存泄漏

```c++
MyClass *ptr = new MyClass[3]; // 调用3次ctor
...
delete ptr; // 调用1次dtor，内存泄漏
```

### `placement new`

placement new 允许我们将object构建于allocated memory中

```c++
#include <new>
char *buf = new char(sizeof(Complex) * 3);
Complex* pc = new(buff)Complex(1,2);
...
delete[] buff;
```

编译器可能会将`new(buff)Complex(1,2)`拆解为：

```c++
// 1.
void* ptr = operator new(sizeof (Complex), buff);
// 2.
pc = static_cast<Complex*>(ptr);
// 3.
pc->Complex(1,2);


void* operator new(size_t, void* loc) {
    return loc;
}
```

### 重载基本构建

#### 重载全局::operator new/delele`, `::operator new[]/delete[]`

**注意**：这个影响很大

```c++
void* operator new (size_t size){
    printf("my global new() \n");
    return myMalloc(size);
}
void operator delete (void* ptr){
    printf("my global delete() \n");
    myFree(ptr);
}
void* operator new[] (size_t size){
    printf("my global new[]() \n");
    return myMalloc(size);
}
void operator delete[] (void* ptr){
    printf("my global delete[]() \n");
    myFree(ptr);
}
```

#### 重载成员`operator new/delete`，`operator new[]/delte`

```c++
class Foo {
  
   void* operator new (size_t size){
        printf("my member new() \n");
        return myMalloc(size);
    }
    void operator delete (void* ptr){
        printf("my member delete() \n");
        myFree(ptr);
    }
    void* operator new[] (size_t size){
        printf("my member new[]() \n");
        return myMalloc(size);
    }
    void operator delete[] (void* ptr){
        printf("my member delete[]() \n");
        myFree(ptr);
    }
};
```

#### 重载`operator new()`/`delete()`

> 重载多个class member operator new()，前提是每个版本的的声明**第一个参数必须是size_t**，其余参数以new所指定的placement argument为初值

```c++
class Foo {
  	// (1)这个就是一般的operator new的重载
    void* operator new (size_t size){
        return myMalloc(size);
    }
    // (2)这个是标准库已提供的placement new()的重载
	void* operator new (size_t size, void* buffer) {
        return buffer;
    }
    // (3)新的placement new
    void* operator new (size_t size, long extra) {
        return malloc(size + extra);
    }
};
```

可以这么理解：使用`new`**表达式**时编译器会调用`operator new`，第一个参数会被自动绑定为类的大小，其余参数相当于重载，**placement new看作是operator new的重载别名**。

**我们也可以重写`placement delete`，但是只有构造对象的时候抛出异常，才会调用相应的`delete`动作**。主要是为了避免内存分配好后，构造对象失败后可以回收已经分配的内存。

## 2.2. 内存池设计

每次申请一次内存，都需要调用一次malloc并且malloc的内存块带有**cookie信息**。如果**一次性申请一大块内存**，可以**降低调用malloc的次数**的同时，也会**减少cookie信息**的消耗。即内存池设计的目的：**提高速度和节约内存**

### pre-class allocator

> union很好的复用了对象空间，即将其作为freelist上的对象指针，又作为使用的对象。

**基本思路**：重写`operator new`一次性申请一大块内存，并用free list管理内存。operator delete删除的对象重新回收加入到free list中

```c++
class Airplane {
private:
    struct AirplaneRep{
        unsigned long miles;
        char types;
    };
private:
    union {
        AirplaneRep rep; // 使用的对象
        Airplane* next; // free list上的对象指针
    };
private:
    static Airplane* headOfFreeList;
    static const int BLOCK_SIZE;
public:
    void* operator new(size_t size) {
        Airplane* p = headOfFreeList;
        if (p) {
            headOfFreeList = p->next;
        }
        else {
            Airplane* newBlock = static_cast<Airplane*>
            (::operator new(BLOCK_SIZE * sizeof (Airplane)));
            // 将一大块串成一个free list
            // 跳过0，因此它被传回做本次结果
            for (int i = 1; i < BLOCK_SIZE - 1; ++ i) {
                newBlock[i].next = &newBlock[i + 1];
            }
            newBlock[BLOCK_SIZE - 1].next = 0;
            p = newBlock;
            headOfFreeList = &newBlock[1];
        }
        return p;
    }
    // 将回收的对象重新插入到free list
    void operator delete(void* deadObject) {
        if (deadObject == 0) return;
        Airplane* carcass = static_cast<Airplane*>(deadObject);
        carcass->next = headOfFreeList;
        headOfFreeList = carcass;
    }


};
```

### static allocator

前一个方法要为不同的类重写近乎一样的`member operator new`和`member operator delete`。static allocator将memory allocator的概念包装起来，更加容易使用

```c++
class allocator{
private:
    struct obj{
      struct obj* next;  
    };
private:
    obj* freeStore = nullptr;
    const int CHUNK = 20; // 标准库默认的是20
public:
    void* allocate(size_t size) {
        obj* p;
        if (!p){
            size_t chunk = size * CHUNK;
            freeStore = p = (obj*)malloc(chunk);
        
            // 将一大块串成一个free list
            // 跳过0，因此它被传回做本次结果
            for (int i = 01; i < CHUNK - 1; ++ i) {
                p->next = (obj*)(char*(p) + size);
                p = p->next;
            }
            p->next = nullptr;
        }
        p = freeStore;
        freeStore = freeStore->next;
        return p;
    }
    // 将回收的对象重新插入到free list
    void deallocate(void* p, size_t size) {
    	((obj*)p)->next = freeStore;
        freeStore = (obj*)p;
    } 
};
```

其他类比如Foo使用效果如下:

```c++
class Foo {
public:
    long l;
    string str;
    int i;
	static allocator myAlloc;
public:
    void* operator new (size_t size) {
        return myAlloc.allocate(size);
    }
    void operator delete(void* p, size_t size) {
        myAlloc.deallocate(p, size);
    }
};
```

### macro for static allocator

为static allocator设计宏定义，为了更加方便地使用

```c++
#define DECLARE_POOL_ALLOCATOR()
public:
void* operator new (size_t size) {
        return myAlloc.allocate(size);
}
void operator delete(void* p, size_t size) {
    myAlloc.deallocate(p, size);
}
protected:
static allocator myAlloc;

#define IMPLEMENT_POOL_ALLOC(class_name)
allocator::class_name::myAlloc;
```

在类中的使用如下

```c++
class Foo {
  	DECLARE_POOL_ALLOCATOR()  
public:
    long l;
    string str;
    int i;
	static allocator myAlloc;

};
IMPLEMENT_POOL_ALLOC(Foo)
```

## 2.3. 标准库分配器

#### VC6标准分配器实现

VC6的分配器实现只是将operator new和operator delete封装在allocate和deallocate下，其他什么都没有做

```c++
template<typename _Ty>
class allocator {
public:
    typedef _Ty* pointer;
public:
    pointer allocate(size_t _N, const void*) {
        return (_Allocate((diiference_type)_N, (pointer)0);
    }
    void deallocate(void _FARQ *_p, size_t){
        operator delete(_p);
    }
};

                
template<typename _Ty>
_FARQ* _allocate(...) {
    return operator new (_N * sizeof (_Ty));
}
```

### G2.9标准分配器实现

G2.9的allocator只是以`::operator new`和`::operator delete`完成`allocate()`和`deallocate()`

### G4.9标准分配器实现

G4.9的allocator只是以`::operator new`和`::operator delete`完成`allocate()`和`deallocate()`，没有特殊设计

### G2.9 std_alloc运行模式

> `alloc`负责管理8~128字节的小块区域，小块内存的带有cookie信息会导致空间利用率较低。而大于128字节的区域直接由malloc申请

<img src="D:\mygit\notes\images\20220703165436.png" style="zoom: 67%;" />

free List是一个16个大小的数组，记录从$8\sim16\times8$个字节freeList

申请x字节（不是8的倍数会对齐）：

- 从x字节对应的free List中取一块数据返回。
- 如果free List为空，则从pool（战备池）中取$x\times\{1...20\}$（至少要能取出一个，最多取20个）返回一块数据，其余挂在free List。
- 如果战备池也不够的话，先做战备池的**碎片处理**（剩下多少就放到哪个freelist中），然后调用`malloc`函数申请$x\times20\times2+ RoundUp(已申请的内存大小>> 4)$大小的空间，一半当作战备池，一半挂在free List。

总结：顺序：free List->战备池->malloc

**申请失败**：从第一个大的freeList中取一块数据返回

## 2.4. malloc/free



## 内存分配方式

在C++中，内存分成5个区，他们分别是堆、栈、自由存储区、全局/静态存储区和常量存储区。

**栈**： 函数内局部变量的存储单元都可以在栈上创建，函数执行结束时这些存储单元自动被释放。
**堆**：`new`申请的空间
**自由存储区**
**全局/静态存储区**：全局变量和静态变量
**常量存储区**：特殊存储区，不允许被修改

## 程序内存模型

从**低地址到高地址**，一个程序由**代码段**、**数据段**、**BSS段**、**堆**、**共享区**、**栈**等组成。

- 代码段：存放程序的机器指令
- 数据段：存放**已被初始化的**全局变量和静态变量
- BSS段：存放**未被初始化的**全局变量和静态变量
- **堆**
- 共享库：位于堆和栈中间
- **栈**
  

**常量存储区** ：存放常量，不允许修改。

# 3. 面向对象设计

## 3.1. 简介

**目标**

- 好的方式编写类
  - 带有指针的类  （基于对象）
  - 不带有指针的类（面向对象）
- 学习class之间的关系
  - 继承
  - 复合
  - 委托

## 3.2. 头文件与类的声明

**1. 头文件的声明方式**

```c++
#ifndef __COMPLEX__//如果编译器未定义过，则新定义一个COMPLEX
#define __COMPLEX__
....

#endif
```

**2. 头文件的布局**

头文件中尽量只写声明和简单的函数定义， 定义放在源文件中

*模板的头文件要包含定义*

```c++
#ifndef __COMPLEX__//如果编译器未定义过，则新定义一个COMPLEX
#define __COMPLEX__
------------------------------------------------------------
//前置声明
#include <cmath>
class complex;
.....
------------------------------------------------------------
//类声明
class complex {
public:
    complex(double r= 0, double i = 0):re(r), im(i){}
    complex& operator += (const complex&);//body外定义函数，body只有声明
    double real() const {return re;}//body内直接定义函数
    double imag() const {return im;}
private:
    double re, im;

    friend complex& __doapl(complex*, const complex &);
};

------------------------------------------------------------
//类定义
complex::funct
ion...
------------------------------------------------------------
#endif
```

**3. 类模板**

> 避免重复编写相似的类，比如仅有类的字段类型不一样

```c++
template<typename T>
class complex {
public:
    complex(T r= 0, T i = 0):re(r), im(i){}
    complex& operator += (const complex&);
    double real() const {return re;}
    double imag() const {return im;}
private:
    T re, im;

    friend complex& __doapl(complex*, const complex &);
};
```

```c++
complex<double> c1(2.5, 1.5);
complex<int> c2(2, 1);
```

## 3.3 面向对象

### 构造函数

**1. inline函数**

成员函数默认都是**inline函数**，也可以自己用**inline**关键字定义

**2. 访问级别**

数据：private

**3. 构造函数**

```c++
class complex {
public:
    complex(double r = 0, double i = 0) // 参数默认值，其他函数也一样
        :re(r), im(i)//初值列：初始化
        {
            //re = i, im = i;
        }//赋值
    complex& operator += (const complex&);//body外定义函数，body只有声明
    double real() const {return re;}//body内直接定义函数
    double imag() const {return im;}
private:
    double re, im;

    friend complex& __doapl(complex*, const complex &);
};
```

```c++
complex c1(2, 1);
complex c2;
complex *p = new complex(4);
```

**一个变量值的设定经历两个阶段**：

- 初始化：初值列直接初始化。
- 赋值

**4. 函数重载**

```c++
class complex{
public:
	double real()  { return re;}
    void real(double r) { re = r;}
};
```

real函数编译后的实际名称可能是（取决于编译器）:

```c++
?real@Complex@@QBENXZ
?real@Complex@@QAENABEN@Z
```

以下两种构造函数重复，1和2都是默认初始化的构造函数，编译器无法区别

```c++
complex(double r= 0, double i = 0):re(r), im(i){}
```

```c++
complex():re(r), im(i){}
```

**5. 作用域符号**

- class::member
- 全局作用域::

------

### 参数传递和返回值

**1. 常量成员函数**

**不改变数据内容的函数，加上const关键字**

```c++
double real() const {return re;}
```

如果对象是const的，但是成员函数没有加上const，则编译器无法确定成员函数是否会修改数据，报错

```c++
{
    const complex c1(2, 1);
    cout << c1.real();
}
```

**2. 参数传递**

- 值传递
- 引用传递(to const **不修改引用的内容**)
  - **参数传递尽量使用引用传递**
  - **返回值尽量使用引用**

引用传递的一个好处在于**传递者无需知道接受者是以什么形式接受**

```c++
void func_by_pointer(Complex *complex);
void func_by_reference(Complex &complex);

Complex complex();
//对比于指针
func_by_pointer(*complex);
//对比于引用
func_by_reference(complex);//无需关心接收的参数类型
```

**3. 友元**

友元可以自由取得friend的private成员，**比使用函数调用速度要快——不需要类型检查和安全性检查等**

定义在外部的函数或类，需要在类（可以是多个类）中声明

```c++
class complex {
private:
    double re, im;
    friend complex& __doapl(complex*, const complex &);
};
```

```c++
inline complex::complex&  _doapl(complex* this, const complex& r){
    this->re += r.re;
    this->im += r.im;
    return *this;
}
```

>  **注意**:
>
> - 友元关系不可以被继承
> - 单向的
> - 不具有传递性



### 操作符重载

 操作符就是一种函数，可以自定义。类似于java的接口

**1. 成员函数**

**所有的成员函数一定带有一个隐藏的this参数**——调用成员函数的对象指针

```c++
inline complex& complex:: operator += (this, const complex &r){//this是一个指针， this不需要写出来
    return __doapl(this, r);
}
```

**2. 非成员函数与临时对象**

**无this**，均为全局函数

**typename() 临时对象**
为了应对用户的三种可能用法，需要开发三种函数：

```c++
inline complex operator + (const complex& x, const complex& y){
    return complex(real(x) + real(y)), imag(x) + imag(y));//typename() 临时对象
}
```

```c++
inline complex operator + (const complex& x, double y){
    return complex(real(x) + y), imag(x));
}
```

```c++
inline complex operator + (double x, const complex& y){
    return complex(x + real(y)), imag(y));
}
```

```c++
{
    c2 = c1 + c2;
    c2 = c1 + 5;
    c2 = 7 + c1;
}
```

**注意点**：*以上函数绝不可以使用引用传递，因此它们返回的都是局部变量*

> 局部变量的生命周期仅是当前函数体内。如果返回局部变量的引用，则会出现不可知的错误。

3. **一元操作符重载**

- ++ class: `class operator++(){}`
- class ++: `class operator++(int){}`

### body外的各种定义

**1. +，-， ==， !=符号重载**
编译器看参数的个数判断其含义

**2. << 符号重载**

```c++
#include <iostream>
ostream&
operator << (ostream& os, const complex& x){//ostram每一次输入都改变os
    return os << '(' << read(x) << ',' << imag(x) << ')';
}
```

### 总结

编写类，**提高速度**需要注意的点：

- 类的布局
  - 常量成员函数加**const**
  - 类的body
    - body内部定义的函数默认是**inline函数**
    - 构造函数常被重载——尽量用**初值列赋值**
  - 参数和返回值尽量用**引用传递**，该不该加**const**
  - 友元

什么时候可以传引用？
什么时候可以传参数？

- **不可以将局部变量作为引用返回**， 局部变量在函数执行完毕后结束生命周期

## 3.4. 基于对象

### 三大函数：拷贝构造，拷贝赋值，析构函数

> 编译器默认的拷贝，是一个一个bit复制
>
> 编译器默认提供**拷贝构造**、**拷贝赋值**和**析构函数**。如果类中含有指针，需要重写

### 构造函数与析构函数

含有指针的对象死亡前，需要释放指针，否则造成内存泄露

**析构函数在对象死亡前被调用**

```c++
inline String::String(const char* cstr = 0){
    if (cstr){
        m_data = new char[strlen(cstr) + 1];
        strcopy(m_data, cstr);
    }
    else {
        m_data = new char[1];
        *m_data = '\0';
    }
}
//
inline String::~String(){
    delete[] m_data;
}
```

### 拷贝构造与拷贝复制

**带有指针的类一定要写拷贝构造与拷贝赋值**！

编译器默认的拷贝赋值会造成内存泄露（浅拷贝）

```c++
class A = b;//拷贝构造
class A(b); //拷贝构造
```

**拷贝赋值**

- 1.自我复制检查
- 2.删除自身
- 3.申请内存空间
- 4.复制过去

```c++
inline String& operator=(const String &str){
    if (this == &str) return *this;//检查自我复制
    delete[] m_data;
    m_data = new char[strlen(str.m_data) + 1];
    strcpy(m_data, str.m_data);
    return *this;
}
```

### 浅谈内存管理

**栈**：存在于某作用域（scope）的一块内存空间。比如函数本身就会形成一个stack存放它接受的参数以及返回地址。
**函数本体内声明的任何变量其所使用的内存都来自于stack**

**堆**：操作系统提供的全局内存空间，程序可动态分配获得若干区块（**blocks**）

```c++
class Complex{...};
{
    Complex c1(1, 2);
    Complex *p = new Complex(3,2);
}
```

*上述代码中，c1占用的空间来自于stack，p的空间是动态申请的，需要自己释放*

#### 对象的生命周期

**1. 栈中对象的生命周期**

比如c1, 其生命在作用域结束之后结束，会自动调用析构函数。栈中变量会被自动清理

**2. 栈中静态对象的生命周期**

```c++
{
    static Complex c3(2,4);
}
```

一直到程序终止

**3. 全局对象的生命周期**

```c++
class Complex{...};
...
Complex c4(1,3);
...
{
    Complex c1(1, 2);
    Complex *p = new Complex(3,2);
}
```

一直到程序终止与静态对象类似

#### new与delete

**new 动态申请的过程：**

1. 分配内存**malloc**

2. 转型

3. 调用构造函数

![](./images/../../images/new.png)

**delete**：

1. 调用析构函数

2. 施放内存**free**

![](./images/../../images/delete.png)

#### VC中的动态分配的内存块

> 调用malloc时，实际分配的内存大小

第一个位置用来确定对象的大小,末尾0表示未分配，1表示已分配
![](./images/../../images/vc.png)

**重点：array new 一定要搭配array delete**：
array delete将调用n次析构函数，使用delete只会调用一次析构函数，从而造成内存泄露

### 虚继承

为了解决多继承时的命名冲突和冗余数据问题，C++ 提出了虚继承，使得在派生类中只保留一份间接基类的成员。

```c++
class A{};
class B:virtual A{};
```

## 3.6. 补充知识

### static

类内存模型中，**成员函数只有一份**，依靠隐藏的**this**指针告诉函数处理一个对象的私有数据

静态成员函数与非静态成员函数：

- 都只有一份，但是**静态成员函数没有this指针**
- **静态成员函数只能处理静态数据**

**静态数据需要在类外写定义**:

```c++
class Account{
public:
    static double m_rate;//声明
    static void set_rate(const double& x){m_rate = x};
};

double Account::set_rate(5.0);//定义
```

**调用静态函数的方式**：

- 通过对象调用
- 通过类调用



### 类模板

**函数模板**:函数对于任何类都是一个样的写法

> 编译器会对function template作参数推导，所以不必用<>写出class的类型

比如min函数,编译器会调用class::operator <

```C++
template<class T>
inline const T& min(const T& a, const T& b){
    return a < b ? a : b;
}
```

### 命名空间

将所有的东西包装在命名空间中，可以**防止重名**

```c++
namespace std{

}
using namespace std;//直接使用

using std::cout;//使用声明

std::cout
```

## 3.7. 类之间的关系

### Composition(复合)——has a

![](./iamges/../../images/adapter.png)

容器与适配器：deque的功能几乎满足queue的需求，只是接口和名字不同，需要改造一下deque

```c++
template<calss T>
class queue{
    deque<T> c;
    ....
public:  return "456";
    bool empty(){return c.empty();}
    ....
}
```

**复合关系下的构造和析构**——*编译器内部调用默认的构造函数和析构函数*：

<img src="D:\mygit\notes\images\20220614162419.png" style="zoom: 50%;" />

- **构造由内而外**

  **Container的构造函数首先会调用Component的default构造函数**

  ```c++
  Container::Container(...): Component(){...}; // Component()是编译器自动帮我们加上的
  ```

  > Component()是编译器自动添加的default构造函数，也可以自己指定构造函数

- **析构由外而内**

  **Container的析构函数首先执行自己，然后才调用Component的析构函数**
  
  ```c++
  Container::~Container(...){... ~Component()};
  ```

### Delegation(委托)——Composition by references

![](./../images/delegation.png)

**与复合的区别：** 生命周期不一致。复合的生命周期是一起出现，而委托只有在引用使用的时候才会出现

**（经典）好处：** pimpl(pointer to implemetation)指针的实现无需关心， 指针指向的类修改无需改动

### Inheritance(继承)——is a

数据可以被继承，**函数继承其实是一种调用权**

![](./images/../../images/inher.png)

继承有public继承、private继承、protect继承

```c++
struct _List_node_base{
    _List_node_base* _M_next;
    _List_node_base* _M_prev;
};

template<typename T>
struct _List_node
    :public _List_node_base
{
    T _M_data;
};
```

**继承关系的构造和析构**：*编译器自动调用默认函数*

- 构造由内而外:Derived的构造函数首先调用Base的default构造函数，然后再执行自己

  ```c++
  Derived::Derived(...):Base(){...};
  ```

- 析构由外而内:Drived的析构函数首先执行自己，然后调用Base的析构函数

  - **base class的析构函数必须是virtual**

  ```c++
  Derived::~Derived(...){...~Base()};
  ```

### 虚函数与多态

> 类似于abstract

**Inheritance(继承) with virtual functions(虚函数)**：

- non-virtual函数：不希望子类重写该方法
- virtual函数：希望子类重新定义该方法
- pure virtual函数: 子类一定要重新定义该方法

```c++
class Shape{
public:
    virtual void draw() const = 0;//pure virtual
    virtual void error(const std::string& msg);//virtual
    int objectID() const;//non-virtual
}
```

### Delegation(委托) + Inheritance(继承)

**观察者模式**

```c++
class Subject
{
    int m_value;
    vector<Observe*> m_views;
public:
    void attach(Observer* obs){
        m_views.push_back(obs);
    }
    void set_val(int value){
        m_value = value;
        notify();
    }
    void notify(){
        for (auto ob : m_views)
            ob->update(this, m_value);
    }
}
class Observer{
public:
    virtual void update(Subject* sb, int m_value = 0);
}
```

# 4. C++ 程序设计兼谈对象模型

## 4.1. 目标

- 泛型编程（template）
- 深入继承关系所形成的对象模型：
  - this指针
  - vptr虚指针
  - vtbl虚表
  - virtual mechanism 虚机制
  - virtual functions 虚函数造成的**多态**效果



## 4.2. 隐式转换——用户自定义转换

### conversion function——转换函数

> 对象->其他类型

比如分数转换为double

```c++
class Fraction
{
public:
    Fraction(int num, int den=1)
        :m_numerator(num), m_denominator(den){}
  
    operator double() const{    //转换不需要参数，返回值可以不用写
        return (double) (m_numerator / m_denominator);
    }
private:
    int m_numerator;
    int m_denominator;
}
```

编译器可能会先找全局的operator+，然后找double的转换函数

```c++
{
    Fraction f(3, 5);
    double d = 4 + f;
}
```

**标准库例子——代理模式**

```c++
template<class Alloc>
class vector<bool, Alloc>{
public:
    typedef __bit_reference reference;
protected:
    reference operator[] (size_type n){///重载了operator[] 本应该返回bool类型，却使用reference做代理返回
        return * (begin() + difference_type(n));
    }
...
}

struct __bit_reference{
    unsigned int *p;
    unsigned int mask;
    ...
public:
    operator bool() const {//reference应当含有bool的转换函数
        return !(!(*p & mask));
    }
};
```

### non-explicit-one-argument ctor

> 其他类型-->对象

two parameters, one argument
两个参数， 一个有默认值， 一个是实参

```c++
class Fraction
{
public:
    Fraction(int num, int den=1)  //一个参数默认值，构造函数实际是可以当成一个实参
        :m_numerator(num), m_denominator(den){}
  
    Fraction operator + (const Fraction &f){
        return Fraction(...)
    }
private:
    int m_numerator;
    int m_denominator;
};
```

编译器找到operator +，但是不符合形式，尝试能否将4转换为Fraction
调用non-**explicit** ctor将4转为Fraction，然后调用opertor+

```c++
{
    Fraction f(3, 5);
    double d = 4 + f;
}
```

> **`注意`：conversion function vs. non-explicit-one-argument ctor：如果同时存在上述两种情况，编译器找到多条路线可行，编译器会不知道怎么办，报错**

### explicit-one-argument ctor

**禁用**编译器自动调用*non-explicit-one-argument ctor*,从而避免隐式转换
大部分情况下用在构造函数前

## 4.3. pointer-like classes——关于智能指针

> 从面向对象编程的角度看，智能指针的设计是面向对象的**复合关系**，内部含有一个指针并重写了`*`和`->`
>

`->`操作符使用后可以被一直使用下去

```c++
template<class T>
class shared_ptr{
public:
    T& operator *() const{return *px;}
    T* operator->() const{return px;}
    shared_ptr(T* p): px(p){}
private:
    T* px;
    long *pn;
}
```

**总结**： 智能指针（比如迭代器）内部包含一个指针，必须要实现*、-> 操作符

**list 迭代器**

```c++
template <class T>
struct __list_node{
void* prev;
void* next;
T data;
};

template<class T>
struct __list_iterator{
typedef __list_node* link_type;
link_type node; //智能指针内必须包含一个真正的指针
reference operator*() const{return (*node).data;}
pointer operator->() const{return &(operator*());}
};
```

## 4.4. functon-like classes——仿函数

类似于python的call，将类作为函数调用

```c++
template <class T>
struct identity {
    const T& operator()(const T& x) const {return x;}
};
```

## 4.5. template模板

### 类模板

### 函数模板

> 编译器会被函数模板进行**参数推导**，不必指明类型

```c++
template<typename T>
inline
const min(const T& a, const T& b) {
    return a < b ? a : b;
}
```

### 成员模板

>  模板类的构造函数一般设计为成员模板

```c++
template<class T1, class T2>
struct pair{
    template<class U1, class U2>
    pair(const pair<U1, U2> &p):first(p.first), second(p.second){}
};
```

**3. specialization模板特化**

锁定模板泛化的某一typename/class

**泛化-full specialization**

```c++
template<class Key>
struct hash{};
```

**特化**

```c++
template<>
struct hash<char>{
  size_t operator() (char x) const {return x;}  
};
```

**4. partial specialization.偏特化**

- 个数上的偏

```c++
template<typename T, typename Alloc=...>
class vector{
    ...
};
```

为第一个模板参数指定类型

```c++
template<typename Alloc=...>
class vector<bool, Alloc>{
    ....
}
```

- 范围上的偏

```c++
template<typename T>
class C{
    ...
}
```

```c++
template<typename U>
class C<U*>{
    ...
}
```

**5. template template parameter.模板模板参数**

参数模板参数本身是一个模板

```c++
template<typename T, template<typename> class Container>
class Xcls
{
private:
    Container<T> c;
public:
    ... 
}
```

## 4.6. 三个主题

> **variadic templates.模板参数可变**

``...``就是一个所谓的pack包
用于模板参数就是模板参数包
用于函数参数类型，就是函数参数类型包
用于函数参数，就是函数参数包

```c++
template<typename T, typename...Type>
void print(const T& firstArg. const Types& ...args){//分解为一个参数和一包参数
    cout << firstArg << endl;
    print(args...);//每次递归的输出
}
```

*参数包数量为0，需要专门写一个空函数用作调用*

```c++
void print(){}
```

size...获取包的大小

``size...of(args)``

### reference.引用

object对象和它reference的大小相同且地址也相同（ 逻辑上是，物理上不是）

```c++
MyClass a; // MyClass的对象
MyClass &alias_a = a; // 对象的引用

// 逻辑上a和alias_a代表的是一个对象
// 但实现上alias_a是一个指针
sizeof (a) == sizeof (&alias_a) 
&a == &alias_a					
```

> 指针与引用的区别：
>
> - 引用是指对象的别名，代表原对象，它表现得形式与对象一致；但是指针是指向对象的地址
> - 引用无法重新指向其他对象；指针可以，容易出现空指针的现象
>
> `引用不算作函数签名`

**好处**：

- 避免一些复制不可复制的对象
- 参数使用引用时， 调用端接口相同

### 对象模型：vptr和vtbl

**虚函数是面向对象最重要的部分**

![](./imagse/../../images/vptr.png)

八个函数各自占了内存的某块内存

基类A有两个虚函数 ``vfunction1``、``vfuntion2``，派生类B重写了A的vfunction1继承了vfunciton2。

vptr指向vtbl，vtbl存放虚函数的指针

``new C()``得到p指针，``(*(p->vptr)[n])(p)``



**重点的内容**：

- 派生类继承了父类的函数调用权限，同时也**继承了虚表**，因此派生类中的虚函数索引与基类中的虚函数索引一致。
- 只要类中含有一个虚函数，内存中就有一个vptr
- 静态绑定： 汇编 ``call.XXXXX(地址)``
- 编译器看到符合3个条件就会动态绑定：根据p指针指向的对象不一样，调用的函数也不一样
  - **通过指针调用**
  - **指针向上转型（将派生类基类的部分赋值给基类）**
  - **调用虚函数**

基类不使用虚函数的情况如下，`基类指针只调用基类的析构函数`，从而可能造成内存泄漏

```c++
class A{
public:
    ~A(){
        cout << "A dtor" << endl;
    }
};
class B: public A{
public:
    ~B(){
        cout << "B dtor" << endl;
    }
};
int main(){
    A *a = new B; // 输出 Adtor
    delete a;
    return 0;
}
```



### 对象模型：this.概念

**虚函数两种用法**：

- 多态
- 模板方法（**设计模式**）

应用框架

```c++
CDocument::OnFileOpen(){
    ....
    Serialize();
    ....
}
virtual Serialise();
```

应用

```c++
class CMyDoc: public CDocument
{
    virtual Serialize(){...}
}
```

```c++
int main(){
    CMyDoc myDoc;
    myDoc.OnFileOpen();
}
```

**虚函数的实现与抽象函数实现基本类似，派生类调用非虚函数时实际上调用的是父类的非虚函数， 而派生类实现了父类的虚函数时，调用的则是自己的虚函数——动态绑定实现**

## 4.7. 重载、重写与隐藏

|              | 返回类型 | 参数                         | 说明                                         | 范围             |
| ------------ | -------- | ---------------------------- | -------------------------------------------- | ---------------- |
| 重载         | 无关     | （类型、个数、顺序）必须不同 | 是指同一可访问区内的同名函数                 | **同一类中**     |
| 重写（覆盖） | 相同     | 相同                         | 是指派生类重新定义了基类中的同名函数         | 派生类与基类     |
| 隐藏         | 无关     | 无关                         | 派生类屏蔽了基类的同名函数，只要同名就会隐藏 | **派生类与基类** |

### 隐藏

派生类的函数屏蔽了与其同名的基类函数，注意只要同名函数，不管参数列表是否相同，基类函数都会被隐藏。

```c++
class Base
{
public:
    void fun(double ,int ){ cout << "Base::fun(double ,int )" << endl; }
};

class Derived : public Base
{
public:
    void fun(int ){ cout << "Derived::fun(int )" << endl; }
};

int main()
{
    Derived pd;
    pd.fun(1);//Derived::fun(int )
    pb.fun(0.01, 1);//error C2660: “Derived::fun”: 函数不接受 2 个参数

    return 0;
}
```





# 5. Effective C++

## 5.1. 让自己习惯C++

### 01. 将C++视为语言联邦

- C
- 面向对象
- STL
- 泛型编程

### 02. 尽量以`const`、`enum`和 `inline`代替 `#define`

### 03.尽可能使用`const`

- 帮助编译器侦查错误
- 尽量避免`const`版本与`non-const`版本的重复, 令`non-cast`版调用`const`版避免代码重复

### 04.尽量在对象使用前确定已被初始化

- 内置类型尽量手动初始化
- 构造函数用初值列初始化（按照声明顺序）
- 为免除“跨编译单元之初始化次序”问题，用`local static`对象替换 `non-local static`对象

## 5.2. 析构/构造/赋值

### 05. 了解c++编译器默认生成版的函数

- 编辑器可以暗自为class创建`default`构造函数、`copy`构造函数、`copy assignment`操作符以及析构函数
  
### 06. 如果不想使用编辑器自动生成的函数，就该明确拒绝

- 可将成员其声明为`private`，或者`=delete`

### 07. 为多态基类声明`virtual`析构函数

### 08. 析构函数绝对不要吐出异常

### 09. 绝不在构造和析构期间调用`virtual`函数

- 因为这种调用从不会下降至derived class

### 10. 令`operator =`返回一个`reference to *this`

### 11. 在`operator =`中处理“自我赋值”

- 判断“证同”
- `copy and swap`

### 12. 赋值对象时勿忘记每一个成份

- copying函数应该确保复制“对象内所有成员变量”及“所有base class 成分”

## 5.3. 资源管理

### 13. 以对象管理资源

- 为防止资源泄漏，请使用**RAII对象**，它们在**构造函数中获得资源**并在**析构函数中释放资源**。
- 两个常被使用的RAII class分别是**shared_ptr**和aut_ptr。**前者通常是较佳选择**，因为其copy行为比较直观

### 14. 在资源管理中小心copying行为

- 复制RAII对象必须一并复制它所管理的资源，所以资源的copying行为决定RAII对象的copying行为
-  普通而常见的RAII class copying行为是：抑制copying、施行引用计数法（reference counting）。不过其他行为也都可能被实现

### 15. 在资源管理类中提供对原始资源的访问

-  API 往往要求访问原始资源（raw resources），所以每个RAII class应该提供一个“取得其所管理之资源”的办法。

### 16. 成对使用`new`和`delete`时要采用相同形式

- (new, delete), (new[], delete[])

### 17. 以独立语句将newed对象置入智能指针

- 以独立语句将newed对象存储于（置入）智能指针内，一旦异常被抛出，有可能导致难以察觉的资源泄漏。

## 5.4. 设计与声明

### 18. 让接口容易被正确使用，不易被误用

-  “促进正确使用”的办法包括**接口的一致性**，以及与**内存类型的行为兼容**。
- “阻止误用”的办法包括**建立新类型**、**限制类型上的操作**，**束缚对象值**，以及**消除客户的资源管理责任**。
- shared_ptr支持定制型删除器。这可防范DLL问题，可被用来自动解除互斥锁等等。

### 19. 设计class犹如设计type

- class的设计就是type的设计。在定义一个新的type之前，请确定你已经考虑过本条款覆盖的所有讨论主题。

### 20. 宁以pass-by-reference-to-const替换pass-by-value

- 尽量以pass-by-reference-to-const 替换 pass-by-value。前者通常比较高效，并可避免切割问题。
- 以上规则并不适用于内置类型，以及STL的迭代器和函数对象。对它们而言pass-by-value往往比较适当。

### 21. 必须返回对象时，别妄想返回其引用

-  绝不要返回pointer或reference指向一个**local stack对象（临时对象）**，或返回一个heap-allocated（已分配堆）对象，或返回pointer或reference指向一个local static对象而有可能同时需要多个这样的对象。

### 22. 将成员变量声明为private

- protected并不比public更具有封装性，只有private提供封装。

### 23. 宁以non-member、non-friend替换member函数

- 宁可拿non-member、non-friend 函数替换member函数。这样做可以增加封装性、包裹弹性和机能扩充性。
- 将所有便利函数放在多个头文件内但隶属同一个命名空间（namespace），这是C++标准程序库的组织方式。

### 24. 如果所有参数都需要类型转换，请为此采用non-member函数

- 如果你需要为某个函数的所有参数（包括被this指针所指的那个隐喻参数）进行类型转换（例如：实现operator*交换律），那么这个函数必须是个non-member。

### 25. 考虑写出一个不抛出异常的swap函数

- 当`std::swap`对你的类型效率不高时，提供一个**swap成员函数**，并确定这个函数不抛出异常。
- 如果你提供一个`member swap`，也该提供一个`non-member swap`用来调用前者，对于class（而非template），也请特化std::swap。
- 调用swap时应针对std::swap使用using声明式，然后调用swap并且不带任何“命名空间资格修饰”。
-  为“用户定义类型”进行std template全特化是好的，但千万不要尝试在std内加入某些对std而言全新的东西。

## 5.5. 实现

### 26. 尽可能延后变量定义式的出现时间

- 尽可能延后变量定义式的出现，这样可以增加程序的清晰度并改善程序效率。

### 27. 尽量少做转型动作

- 如果可以，尽量避免转型，特别是在注重效率的代码中避免dynamic_cast。如果有个设计需要转型动作，试着发展无需转型的替代设计。
- **如果转型是必要的，试着将它隐藏于某个函数背后**。客户随后可以调用该函数，而不需要将转型放进他们自己的代码中。
- 宁可使用C++style（新式）转型，不要使用旧式转型。前者很容易辨识出来，而且也比较有着分门别类的职掌。

### 28. 避免返回handle指向对象内部成分

- 避免返回handle（用来取得某个对象，包括reference、指针、迭代器）指向对象内部。遵守这个条款可增加封装性，帮助const成员函数的行为像个const，并将发生“虚吊号码牌”的可能性降至最低。

### 29. 为“异常安全”而努力是值得的

### 30. 透彻了解inline的里里外外

- 将大多数inline限制在小型、被频繁调用的函数身上。这可使日后的调试过程和二进制升级更容易，也可使潜在的代码膨胀问题最小化，使程序的速度提升机会最大化。
- 不要只因为function template出现在头文件，就将它们声明为inline。

# 6. STL—体系结构与内核分析

## 1. 目标

- 使用C++标准库
- 认识C++标准库
- 良好使用C++标准库
- 扩充C++标准库

## 2. STL体系结构基础

STL 六大部件：

- 容器
- 分配器
- 算法
- 迭代器
- 适配器
- 仿函数



**STL容器：采用`前闭后开`区间**

**容器间的关系与分类**: 缩进代表复合关系

![](../images/211123.png)

## 3. 分配器

`operator new()`和 `malloc`。内存分配最终都调用到 `malloc`,根据不同操作系统 `malloc`最终会使用不同的系统调用。`malloc`拿到的内存空间比申请的空间要大称为额外开销，具体细节看**内存管理**

> **总结：** `allocator`只是以 `::operator new`和 `::operator delete`完成 `allocate()`和 `deallocate()`

G4.9的版本分配器默认使用的是 `alloctor`，G2.9较好的设计 `alloc`被命名为 `__pool_alloc`

## 4. 容器

> 均按照handle-body的设计思想

**1. 分类**

- sequence（序列）容器
  - array
  - vector
  - deque
  - list（双链表）
  - forward list（单链表）
- associative（关联）容器
  - set/mutiset
  - map/mutimap
- unordered（无序关联） 容器

#### list

> 最有代表性

双向循环链表，**刻意在尾端加一个空白结点，用以符合STL“前闭后开”区间**

**GNU2。9**

![](../images/20211125_111224.png)

*__list_node*

```c++
template<class T>
struct __list_node{
    typedef void* void_pointer;
    void_pointer prev;
    void_pointer next;
    T data;
}
```

*__list_iterator*

所有的智能指针都要实现 `*`和 `->`操作，`++，--`操作按照容器逻辑重载

```c++
template<class T, class Ref, class Ptr>
struct __list_iterator{
    typedef bidrectional_iterator_tag iterator_category;//1
    typedef T value_type;//2
    typedef Ptr pointer;//3
    typedef Ref reference;//4
    typedef ptrdiff_t difference_type;//5
    link_type node;
    //上述五个是必要的
    reference operator*() const {return (*node).data;}
    pointer operator->() const {return &(operator*());}
    self& operator++(){node = (link_type)((*node).next); return *this;}
    self operator++(int){self tmp = *this//调用拷贝构造，不会调用operator*; ++*this; return tmp;}
};
```

*list*

```c++
template<class T, class Alloc = alloc>
struct list{
protected:
    typedef __list_node<T> list_node;
public:
    typedef list_node* link_type;
    typedef __list_iterator<T,T&,T*> iterator;
protected:
    link_type node;
...
};
```

#### vector

按照GNU2.9设计实现vector

```c++
template<class T>
class vector;

template<class T>
void construct(typename vector<T>::iterator I, const T &x);

template<class T>
class vector {
public:
    typedef T value_type;
    typedef T* iterator;
    typedef T& reference;
    typedef size_t size_type;
protected:
    iterator start = nullptr;
    iterator finish = nullptr;
    iterator end_of_storage = nullptr;
public:
    ~vector(){delete[] start;}
    const iterator begin() const {return start;}
    const iterator end() const {return  finish;}
    void push_back(const T &value){
        if (finish == end_of_storage){
            insert_aux(end(), value);
        }
        else{
            construct(end(), value);
            finish ++;
        }
    }
    void pop_back(){
        if (!empty()){
           finish --;
        }
    }
    const T back() const {return *(end() - 1);}
    const T front()const {return *(start);}
    bool empty(){
        return begin() == end();
    }
    const reference operator[](size_t n) const{ return *(begin() + n);}
    reference operator[](size_t n);
    const size_type size() const {return size_type(end() - begin());}//通过函数调用的好处之一是其他地方发生变化时，哪怕是end和begin变化了，size（）函数也不需要变化
    const size_type capacity(){return size_type(end_of_storage - begin());};
private:
    void insert_aux(iterator position, const T& x);
};

template<class T>
void construct(typename vector<T>::iterator I, const T &x){
    *(I) = x;
}
template<class T>
typename vector<T>::iterator uninitialized_copy(typename vector<T>::iterator &from_start, typename vector<T>::iterator &from_end, typename vector<T>::iterator &to_start){
    auto from = from_start, to = to_start;
    while (from != from_end){
        construct(to, *from);
        from ++;
        to ++;
    }
    return to;
}
template<class T>
void vector<T>::insert_aux(iterator position, const T& x) {
    const size_type old_size = size();
    const size_type len = old_size != 0 ? old_size * 2 : 1;

    iterator new_start = new T[len];
    iterator new_finish = new_start;
    try {
        new_finish = uninitialized_copy<T>(start, position, new_start);
        construct(new_finish, x);
        new_finish ++;
    }
    catch (std::exception &e){

    }
    delete[] start;
    start = new_start;
    finish = new_finish;
    end_of_storage = new_start + len;
}
```

**GNU.4.9**

![](../images/20211125_135732.png)

#### array

**TR1**技术报告1

> 没有构造函数与析构函数

#### forward_list

![](../images/20211202_192841.png)

#### deque

**逻辑**上是一个双端可入的数组

**物理**上分段连续的数组

![](../images/20211202_193134.png)

- map由一个vector实现，buffer是一个连续的数组（node），当buffer空间不足时，新创建一个新的buffer把其指针加入到vector中
- iterator是一个类，node指向map， iterator中的 `first`、和 `last`分别指向 `cur`（当前结点）的首尾指针

```c++
template <class T, class Alloc=std::allocator, size_t BufSize=0>
class deque{
    template<class T, size_t BufSize>//BufSize是指buffer容纳元素的个数
    struct __deque_iterator{
        typedef T value_type;
        typedef T** map_pointer;
        typedef ptrdiff_t different_type;
        T* cur;
        T* first;
        T* last;
        map_pointer node;
    };

public:
    typedef T value_type;
    typedef T* pointer;
    typedef size_t size_type;
    typedef __deque_iterator<T, BufSize> iterator;

protected:
    typedef pointer* map_pointer;
protected:
    iterator start;
    iterator finish;
    map_pointer map;
    size_type map_size;
public:
    iterator begin(){return start;}
    iterator end(){return finish;}
    size_type size() const { return finish - start;}
    //deque的insert函数能以最小的移动次数插入元素
    iterator insert(iterator position, const value_type& x){
        if (position.cur == start.cur){ //如果插入的是前端交给push_front()做
            push_front)x;
            return start;
        }
        else if (position.cur == finish.cur){//如果插入的是尾端，交给push_back()做
            push_back(x);
            iterator tmp = finish;
            -- tmp;
            return tmp;
        }
        else return insert_aux(position, x);
    }
    iterator insert_aux(iterator position, const value_type &x){
        iterator::different_type index = pos - start; //安插点之前的元素个数
        value_type x_copy = x;
        if (index < size() / 2){ // 如果按插点之前的元素个数较少
            push_front(front());//最前端加入第一个元素
            ...
            copy(front2, pos1, front1);//元素般移
        }
        else {
            push_back(back());
            ...
            copy_backward(pos, back2, back1);
        }

        *pos = x_copy;//安插点设定新值
        return pos;
    }
};
```

**deque如何模拟连续空间**：全都是deque iterator的功劳

```c++
reference  operator*() const{return *cur;}
        pointer operator->() const {return &(operator*());}
        //两个iterator距离相当于
        //（1）两个iterator间buffer的总长度 +
        //（2）iterator1至buffer末尾的长度+
        //（3）terator2至buffer起头的长度
        different_type operator - (const self& x) const{
            return different_type (buffer_size()) * (node - x.node - 1) + (cur - first) + (x.last - x.cur);
        }
        void set_node(map_pointer p){
            node = p;
            first = *p;
            last = *(p + different_type(BufSize));
        }

        self& operator ++ (){
            ++ cur;  //切换下一个元素
            if (cur == last){//如果抵达buffer尾部
                set_node(node + 1);//跳至下一个buffer的起点
                cur = first;
            }
            return *cur;
        }
        self& operator++(int){
            self tmp = *this;
            ++*this;
            return tmp;
        }
        self& operator--(){
            if (cur ==first){
                set_node(node - 1);
                cur = last;
            }
            --cur;
            return *cur;
        }
```

**当map满了以后，会将其复制到新空间的中间，以让其前插入**

#### 红黑树

红黑树是一种平衡二插树

提供“遍历”操作及iterators，按正常的遍历，得到排序状态。

红黑树提供两种插入操作：`insert_unique`、`insert_equal`

```c++
template<class Key,
         class Value,
         class KeyOfValue,
         class Compare,
         class Alloc=alloc>
class rb_tree{
protected:
    typedef __rb_tree_node<Value> rb_tree_node;
    ...
public:
    size_type node_count;//rb_tree的大小
    link_type header;
    Compare key_compare;//key的大小比较规则，某个仿函数
};
```

**红黑数调用**

```c++
rb_tree<int,int,identity<int>,less<int>> myTree;
```

```c++
template<class Arg, class Result>
struct unary_function{
    typedef Arg argument_type;
    typedef Result result_type;
}

template<Class T>
struct identity:public unary_function<T,T>{
    const T& operator()(const T& x) const {return x;}
}
```

#### set与multiset

set/multiset以rb_tree为底层结构，因此有元素自动排序特性。

排序依据是 `key`，而**set/multiset元素的data和key合一为value，value就是key**

```c++
template<class Key,
         class Compare = less<Key>, class Alloc = alloc>
class set{
public:
    typedef Key key_type;
    typedef Key value_type;
    typedef Compare key_compare;
    typedef Compare value_compare;
private:
    typedef rb_tree<key_type, value_type,
                identity<value_type>, key_compare, Alloc> rep_tree;
    rep_type t;

};
```

#### map与multimap

map/multimap以rb_tree为底层结构，因此有元素自动排序特性

排序依据是 `key`

```c++
template<class Key,
         class T,
         class Compare = less<Key>, class Alloc = alloc>
class map{
public:
    typedef Key key_type;
    typedef T data_type;
    typedef T mapped_type;
    typedef pair<const Key, T> value_type;
    typedef Compare key_compare;
    typedef Compare value_compare;
private:
    typedef rb_tree<key_type, value_type,
                select1st<value_type>, key_compare, Alloc> rep_tree;
    rep_type t;
public:
    typedef typename rep_type::iterator iterator;

};
```

#### hashtable

**实现方式**：“拉链法”

当插入的元素数量超过 `buckets vector`的大小，增加容量，需要**rehashing**

#### unordered_*

## 5. 迭代器与算法

### Iterator Traits

#### **Iterator需要遵循的原则**

迭代器是算法和容器之间的**桥梁**，帮助做算法设计

<img src="../images/20211125_143335.png" style="zoom: 50%;" />

**算法提问的方式**

```c++
void algorithm(I first, I end){
    I::iterator_category;
    I::value_type;
    ...
}
```

**迭代器必须要提供五种关联的type**, 全部都是`typedef`的

- `iterator_category`
- `value_type`
- `pointer`
- `reference`
- `difference_type`

> **迭代器是一种泛化的指针，native指针应该也是一个迭代器，但是native指针无法根据上述内容回答算法的问题**

#### Iterator or native pointer

traits机器必须要有能力分辨它所获得`iterator`是 `class iterator T`还是 `native pointer to T`。利用**偏特化**可以达到目的

**加一个中间层即可解决**

```c++
template<class I>
struct iterator_traits{
    typedef typename I::iterator_category iterator_category;
    typedef typename I::value_type value_type;
    typedef typename I::pointer pointer;
    typedef typename I::reference reference;
    typedef typename I::difference_type difference_type;
    ...
}
```

**偏特化：**

```c++
template<class T>
struct iterator_traits<T*>{
    typedef random_access_iterator_tag iterator_category;
    typedef T value_type;
    typedef T* pointer;
    typedef T& reference;
    typedef ptrdiff_t difference_type;

}
template<class T>
struct iterator_traits<const T*>{
    typedef random_access_iterator_tag iterator_category;
    typedef T value_type;
    typedef T* pointer;
    typedef T& reference;
    typedef ptrdiff_t difference_type;
}
```

```c++
template<typename I,...>
void algorithm(...){
    iterator_traits<I>::value_type v1;
    ...
}
```

### 迭代器分类

<img src="../images/1640054969566.png" alt="1640054969566.png" style="zoom:50%;" />

```c++
//五种迭代器类别
struct input_iterator_tag{};
struct output_iterator_tag{};
struct forward_iterator_tag{}:public input_iterator_tag{};
struct bidirectional_iterator_tag:forward_iterator_tag{};
struct random_access_iterator_tag:bidirectional_iterator_tag{};
```

为什么不用`枚举`表示

```c++
void _display(random_access_iterator_tag){...}
void _display(input_iterator_tag){...}
void _display(forward_iterator_tag){...}
void _display(bidirectional_iterator_tag){...}
void _display(output_iterator_tag){...}

template<typename I>
void display_category(I itr){
    typename iterator_traits<I>::iterator_category cagy;
    _display(cagy);
}
```

### 迭代器对算法的影响

> 注意 `difference_type`不能随便地写成整数

算法通过“询问”迭代器，对不同的迭代器种类分开设计算法

```c++
template<class InputIterator>
inline iterator_traits<InputIterator>::difference_type
distance(InputIterator first, InputIterator last){
    typedef typename iterator_traits<InputIterator>::iterator_category category;
    return __distance(first, last, category())
}

template<class RandomAccessIterator>
inline iterator_traits<RandomAccessIterator>::difference_type
__distance(RandomAccessIterator first, RandomAccessIterator last,
            random_access_iterator_tag){//模板函数接受random_access_iterator_tag类
      return last - first;
}

template<class InputIterator>
inline iterator_traits<InputIterator>::difference_type
__distance(InputIterator first, InputIterator last,
            input_iterator_tag){//模板函数接受input_iterator_tag类
    iterator_traits<InputIterator>::difference_type n = 0;
    while (last != first){
      ++ first, ++ n;
    }
    return n;
}
```

> 静态多态: 编译器在**编译期间完成的**，编译器会根据实参类型来选择调用合适的函数
>
> 主要发生在:
>
> - 函数重载
> - 模板函数

```c++
template<class Iterator>
inline tyepname iterator_traits<Iterator>::category
    iterator_category(const Iterator &i){
     typedef iterator_traits<Iterator>::category category;
    return category();
}

template<class InputIterator, class Distance>
inline void advance(InputIterator &i, Distance n){
    __advance(i, n, iterator_category(i));
}
template<class RandomAccessIterator, class Distance>
inline void __advance(RandomAccessIterator &i, Distance n,
        random_access_iterator_tag){
      i += n;
}
template<class BidirectionalIterator, class Distance>
inline void __advance(RandomAccessIterator &i, Distance n,
        random_access_iterator_tag){
        if (n > 0)
            while (n --) ++ i;
        else 
            while (n ++) -- i;
}
```

面向对象中，`farword_iterator`类型is a `input_iterator`，最终会调用`__advance(InputIterator,n)`

### 算法源码中对`iterator_category`的“暗示”

语法上不支持模板函数只接受特定类

![1640058514482.png](../images/1640058514482.png)

### 算法样例

#### accumulate

**蓝色的binary_op被称为`callable entity`（可被调用的实体），传进去任何东西只要能被小括号作用起来就可以**

> `callable entity`如果不是仿函数的话，就只能作用于函数模板中

```c++
template<class InputIterator, class T>
T accumulate(InputIterator first, InputIterator last, T init){
    for (; first != last; ++ first) {
        init = init + *first;
    }
    return init;
}

template<class InputIterator, 
        class T，
        class BinaryOperation>
T accumulate(InputIterator first, InputIterator last, T init, BinaryOperation op){
    for (; first != last; ++ first) {
        init = binary_op(init, *first);
    }
    return init;
}
```

#### replace, replace_if, replace_copy

```c++
template<class ForwardIterator,
        class T>
void replace(ForwardIterator first,
            ForwardIterator last,
            const T& old_value,
            const T& new_value){

    for (; first != last; ++ first){
        if (*first == *old_value){
            *first = new_value;
        }
    }
}
```

```c++
template<class ForwardIterator,
        class Predicate,
        class T>
void replace(ForwardIterator first,
            ForwardIterator last,
            Predicate pred,
            const T& new_value){

    for (; first != last; ++ first){
        if (pred(*first)){
            *first = new_value;
        }
    }
}
```

#### find, find_if

**不具有`find`成员函数**的容器:vecotr,list,deque...
**具有`find`成员函数的容器**：关联式容器

泛化的find函数$O(N)$

```c++
template<class InputIterator,
        class T>
InputIterator find(InputIterator first,
                   InputIterator last,
                   consts T& value)
{
    while (first != last && *first != value)
        ++ first;
        return first;
}
```

## 6. 仿函数

算术类

```c++
template<class T>
struct plus:public binary_function<T,T>{
    T operator()(const T&x, const T& y){ return x + y; }
}
```

逻辑运算类

```c++
template<class T>
struct logical_and:public binary_function<T,T>{
    bool operator()(const T&x, const T& y) }{return x && y;}
}
```

相对关系类

```c++
template<class T>
struct equal_to:public binary_function<T,T>{
    bool operator()(const T& x, const T& y){return x == y;}
}
```

**仿函数的可适配（adaptable）条件**

STL规定每个Adaptable Function都应该挑选适当者继承，因为Function Adapter将会提问红色问题

```c++
template<class Arg, class Result>
struct unary_function{
    typedef Arg argument_type;
    typedef Result result_type;
};
```

```c++
template<class Arg1, class Arg2, class Result>
struct binary_function{
    typedef Arg1 first_argument_type;
    typedef Arg2 second_argument_type;
    typedef Result result_type;  
};
```

### 函数适配器.Function Adapter

`binder2nd`已经被`bind`取代

```c++
cout << count_if(vi.begin(), vi.end(),
        not1(bind2nd(less<int>(), 40))) << endl;
```

```c++
//辅助函数，让用户更加方便使用
//编译器会自动推导Op的类型
template<class Operation, class T>
inline binder2nd<Operation> bind2nd(const Operation& op, const T& x){
    typedef typename Operation::second_argument_type arg2_type;
    return binder2nd<Operation>(op, arg2_type(x));
}
```

```c++

template<class Operation>
class binder2nd
:public unary_function<typename Operation::first_argument_type,
                       typename Operation::result_type>
{
protected:
    Operation op;
    typename Operation::second_argument_type value;
public:
    //constructor
    binder2nd(const Operation& x, const typename Operation::second_argument_type &y)
        :op(x), value(y){}
    typename Operation::result_type operator()(const typename Operation::first_argument_type &x) const {return op(x, value);}

}
```

### 新型适配器.bind(c++11)

`std::bind`可以绑定：

- functions
- function objects
- member function, _1必须是某个object地址
- data members, _1必须是某个object地址

### 迭代器适配器.reverse_iterator

![](../iamges/../images/20220104130808.png)

reverse_iterator是与iterator相反的iterator：

- reverse_iterator的begin()就是iterator的end()
- reverse_iterator的end()就是iterator的begin()
- iterator的++就是reverse_iterator的--
- iterator的--就是reverse_iterator的++

```c++
template<class Iterator>
class reverse_iterator
{
protected:
    Iterator current;//对应的正向迭代器
public:
    //5种trait的typedef
    typedef Iterator iterator_type;//正向迭代器
    typedef reverse_iterator<Iterator> self;//逆向迭代器

public:
    explict reverse_iterator(iterator_type x) : current(x) {}
    reverse_iterator (const self& x) : current(x.current){}
    self& operator ++ (){return --current;}//前进变成后退
    self& operator -- (){return ++ current;}//后退变成前进
    reference operator *() const { return *current;}
    pointer operator->() const {return &current;}
    self operator + (different_type n) {return current - n;}
    self operator - (different_type n) {return current + n;}
}
```

### ostream_iterator

![](../images/20220104140014.png)



## 7. Type Traits

c++ 11 type traits
![](../images/20220121112243.png)

**is_void**

```c++
template<typename _Tp> 
struct is_void:
    public __is_void_heler<typename remove_cv<_Tp>>::type {};//remove const and volatile
```

**is_void_helper**

```c++
template<typename>
struct is_void_helper
    :public false_type
{};

template<typename void>
struct is_void_helper
    :public true_type
{};
```

**remove const**

```c++
template<tyename _Tp>
struct remove_const{
    typedef _Tp type;
};

template<tyename _Tp>
struct remove_const<_Tp const>{
    typedef _Tp type;
};
```

**remove volatile**

```c++
template<typename _Tp>
struct remove_volatile{
    typedef _Tp type;
}
template<typename _Tp>
struct remove_volatile<_Tp volatile>{
    typedef _Tp type;
}
```

**remove const and volatile**

```c++
template<typename _Tp>
struct remove_cv{
    typedef typename remove_const<remove_volatile<_Tp>::type>::type type;
}
```



# 7. 新特性

## C++11

### 1. 智能指针

> 基于RAII原理实现：对象销毁时，自动调用析构函数

#### auto_ptr（舍弃)

> 独占所有权。 `auto_ptr`的**拷贝复制**的语义是移动复制的语义，容易造成对象所有权的转移而客户却不知道。

```c++
template<T>
class auto_ptr{
    //数据
private:
    T* M_ptr;
   
public:
    //构造函数
    auto_ptr(T* __p = 0) : _M_ptr(__p) { }

    //拷贝构造
    auto_ptr(auto_ptr& __a): _M_ptr(__a.release()) { }

    //什么都没做，只是将指针置空
    T* release() 
    {
        T* __tmp = M_ptr;
        M_ptr = 0;
        return __tmp;
    }
    //释放内存
    voi reset(T* __p = 0) 
      {
        if (__p != _M_ptr)
        {
            delete _M_ptr;
            _M_ptr = __p;
        }
      }
    //重点
    auto_ptr& operator=(auto_ptr& __a)
    {
        reset(__a.release());
        return *this;
    }
};

```

`auto_ptr`**指针的复制或分配会更改所有权**

```c++
auto_ptr<string> p1 = auto_ptr<string>(new string("123"));
auto_ptr<string> p2;
p2 = p1;
```

`p2`智能指针的拷贝复制会更改资源的所有权，导致原来的智能指针**悬空**

#### unique_ptr

> 独占所有权。对auto_ptr做了改进，删除了默认的拷贝复制函数和拷贝构造函数，新增了移动复制和移动构造函数

```c++
template<T>
class unique_ptr{
    unique_ptr(const unique_ptr&) = delete;
    unique_ptr& operator=(const unique_ptr&) = delete;

};
```

`unique_ptr`采用了**严格所有权**，删除了拷贝构造和拷贝复制，支持**移动构造**和**移动复制**。

#### share_ptr

shared_ptr实现**共享式**拥有概念。多个智能指针可以指向相同对象，该对象和其相关资源会在“最后一个引用被销毁”时候释放。

**计数器**的可以通过一个指针实现，指针指向计数器管理类，`share_ptr`的拷贝构造和拷贝复制，同时复计数器管理类指针即可。

> 只有通过拷贝构造和拷贝复制其值给另一个`shared_ptr`，才能将对象的所有权共享

#### weak_ptr

> 循环引用可能会导致`shared_ptr`无法正确的释放资源，而`weak_ptr`不影响计数器，只提供访问资源的方式。所以每次访问前，需要先判断资源是否被删除

`shared_ptr`循环引用

```c++

class Son;
class Father {
public:
    shared_ptr<Son> son_;
    Father() {
        cout << __FUNCTION__ << endl;
    }
    ~Father() {
        cout << __FUNCTION__ << endl;
    }
};
class Son {
public:
    shared_ptr<Father> father_;
    Son() {
        cout << __FUNCTION__ << endl;
    }
    ~Son() {
        cout << __FUNCTION__ << endl;
    }
};
int main()
{
    auto son = make_shared<Son>();
    auto father = make_shared<Father>();
    son->father_ = father;
    father->son_ = son;
    cout << "son: " << son.use_count() << endl; // son: 2
    cout << "father: " << father.use_count() << endl; // father: 2
    return 0;
}

```

**weak_ptr主要使用的函数:**

```c++

int use_count();
bool expired();//检查被引用的对象是否已经删除
share_ptr<int> lock();//创建管理被引用对象的share_ptr
```



### 2. 四种类型转换

#### static_cast

> 用隐式和用户定义转换的组合在类型间转换。
>
> 注意: static_cast是静态转换，`dynamic_cast`是运行时，根据运行时的动态类型转换

指针向上转型在`static_cast`和`dynamic_cast`都是有效的，即使没有cast，也会自动的转换

```c++
struct A{
    int a;
    virtual void f(){}
};
struct B: public A{
    void f() override {}
};
struct C: public A{
    void f() override {}
};
int main(){
    A *a = new B;

    if (B *b = static_cast<B*>(a); b != nullptr) { // b success
        cout << "b success" << endl;
    }
    if (C *c = static_cast<C*>(a); c != nullptr) { // c success
        cout << "c success" << endl;
    }
    return 0;
}
```

`static_cast`可以用在隐式转换，如基本类型之间的转换。也可以用在类指针的向上转型（安全的），向下转型（不安全的）

#### dynamic_cast

>  用于含有**`虚函数`**的类之间向上、向下和侧向之间的转

在不知道指针的动态类型时使用

```c++
struct A{
    int a;
    virtual void f(){}
};
struct B: public A{
    void f() override {}
};
struct C{
    virtual void f(){}
};
int main(){
    A *a = new B;

    if (B *b = dynamic_cast<B*>(a); b != nullptr) { // b success
        cout << "b success" << endl;
    }
    if (C *c = dynamic_cast<C*>(a); c != nullptr) {
        cout << "c success" << endl;
    }
    return 0;
}
```

#### const_cast

> 只能用于指针或引用，不可作用于函数指针和成员函数指针

在有不同 cv 限定的类型间转换。

```c++
int i = 3;                 // 不声明 i 为 const
const int& rci = i; 
const_cast<int&>(rci) = 4; // OK：修改 i
```

#### reinterpret_cast

`重新解释`，几乎什么都能转，但是可能会出问题。

### 3. 可变模板参数

> 变化的是：1.参数的个数。2.参数的类型

`...`就是一个pack(包)
用于模板参数就是模板参数包
用于函数参数类型就是函数参数类型包
用于函数参数就是函数参数包

`sizeof...(args)`获得参数数量
数量不定的模板参数

**`...`语法用在如下三个地方：**

```c++
template<typename T, typename... Types> // 1
void print(const T& firstArg, const Types&... args){ // 2
    cout << firstArg << endl;
    print(args...); // 3
}
```

#### 递归使用方法

##### 入口

```c++
template<typename... Types>
inline size_t hash_val(const Types&... args){
    size_t seed = 0;
    hash_val(seed, args...);
    return seed;
}
```

##### 递归主体

```c++
template<typename T, typename... Types>
inline void hash_val(size_t& seed, const T& val, const Types&... args){
    hash_combine(seed, val);
    hash_val(seed, args...);
}
```

##### 出口

```c++
template<typename T>
inline void hash_val(size_t& seed, const T& val){
    hash_combine(seed, val);
}
```

**从入口函数进入，通过递归函数对参数包解包直到调用到出口函数为止**

#### 递归继承.Tuple

```c++
template<typename... Values> class tuple;//泛化模板
template<> class tuple<> {};//出口

template<typename Head, typename... Tail>
class tuple<Head, Tail...>
    :private tuple<Tail...>
{
    typedef tuple<Tail...> inherited;
public:
    tuple(){}
    tuple(Head v, Tail... vtail)
        :m_head(v), inherited(vtail...){}
    
    typename Head::type head() { return m_head;}
    inherited& tail() {return *this;}
        
protected:
    Head m_head;
}
```

#### 示例1

> 实现一个打印函数`print`：将任意类型任意个参数打印出来

递归主体实现如下：


```c++
template<typename &T, typename... Types>
void printX(const T& firstArg, const Types&... args) {
	cout << firstArg << endl;    
    printX(args...);
}
```

递归出口如下：

```c++
void printX(){}
```



```c++
printX(7.5, "hello", 1) // 输出: 7.5 hello 1
```

#### 示列2

> 实现一个printf("%d..", ..)格式化输出函数

#### 示列4

> 标准库`std::maximum`函数

递归主体：

```c++
template<tyename... Args>
int maximum(int n, Args... &args) {
    return std::max(n, maximum(args))
}
```

递归出口:

```c++
int maximum(int n) {
    return n;
}
```

#### 示例5

> 以异于一般的方式处理first元素和last元素

```c++
cout << make_tuple(7.5, string("hello"), 42);
// 期望运行结果: [7.5, hello, 42]
```

入口：

```c++
template<typename... Args>
ostream& operator << (ostream& os, const tuple<Args...> &t) {
    os << "[";
    PRINT_TUPLE<0, sizeof...(Args), Args..>::print(os, t);
    os << "]";
}
```

递归主体:

```c++
略
```

### 4. 统一初始化

直接在变量名后加大括号

```c++
int values[] {1, 2, 3, 4};
vector<int> v{2, 3, 5, 5};
vector<string> cities{"sdf", "af", "adsf"};
```

编译器看到`{t1, ...,tn}`内部做出一个`initializer_list<T>`。 构造函数有一个版本接受这种形式。
**如果构造函数中不接受这种形式，编译器会将`initializer_list<T>`拆解**，一个个丢进构造函数

#### Initializer List

```c++
int i; // 未定义
int j{}; // 初始化未为0
int *p; // 未定义
int *p{}; // 初始化未nullptr
```

C++11提供了类模板`std::initializer_List<>`，可以被用来支持初始化，或者任何你想用的地方

```c++
void print(std::initializer_List<int> vals) {
    for (auto p = vals.begin(); p != vals.end(); p ++) {
        std::cout << *p << "\n";
    }
}

print({1, 2, 3, 4, 5, 6}) // 传递一个处置里处理
```

> 注意：`initializer_List<>`与`variadic templates`有很大不同的，
>
> - 都支持任意数量参数。
>
> - `variadic templatees`还支持**任意的类型**，而`initializer_List<>`只支持一种类型

`initializer_List<T>`实际上就是一个`array`

> **拷贝动作是浅拷贝**

```c++
template<class _E>
class initializer_list{
public:
    typedef _E* iterator;
private:
    iterator _M_array;
    size_type _M_len;
    //私有的构造函数由编译器调用
    ctor()
    ....

};
```

### 5. Lambdas

Lambdas相当于一个匿名的**仿函数**，其返回值是一个对象

**语法**：$[...](...) mutable\ throwSpec\ \rightarrow retType \{...\}$

<img src="D:/mygit/notes/images/20220216094150.png" style="zoom: 50%;" />

#### `decltype` + `auto`

得到一个表达式(可以是对象)的类型

**auto + decltype**

```c++
map<string, float> mp;
decltype<mp>::value_type elem;
```

- 声明返回类型

  ```c++
    template<typename T1, typename T2>
    auto add(T1 x, T2, y)->decltype(x + y);
  ```

- 模板编程中使用

- 传递lambda的类型

  ```c++
    auto cmp=[](const Person &p1, const Person &p2)
    {
        return p1.name < p2;
    };
    ....
    std::set<Person, decltype(cmp)> coll(cmp);
  ```



### 6. 右值引用

> 解决不必要的拷贝，但是它实际还是一个引用
>
> **用于含有指针的面向对象设计，避免重复copy指针内容**

比如vector、string

```c++
vector<A> res;
// 1.调用一次默认构造函数；2.调用一次移动构造函数
res.push_back(A());

// 1.调用一次默认构造函数
A a;
// 2.调用一次copy构造函数
res.push_back(a);
```

#### 拷贝构造与移动构造

<img src="C:\Users\zhuang\AppData\Roaming\Typora\typora-user-images\image-20220626211510971.png" alt="image-20220626211510971" style="zoom:50%;" />

<img src="C:\Users\zhuang\AppData\Roaming\Typora\typora-user-images\image-20220626211553608.png" alt="image-20220626211553608" style="zoom:50%;" />

#### 拷贝赋值与移动赋值

#### move aware class

```c++
class String{
private:
    char* _data;
    size_t _len;
    void _init(char* s, size_t len){
        _data = new char*(len + 1);
        memcpy(s, s + len, _data);
        _data[len] = '\0';
        _len = len;
    }
public:
    String(const String& rhs){ // 拷贝构造函数
        _init(rhs._data, rhs._len);
    }
    String(String&& rhs) noexcept
    	: _data(rhs._data), _len(rhs._len){ // 移动构造
        rhs._data = nullptr;
        rhs._len = 0;
    }
    
    ~String() {  // 析构函数
        if (_data) {
            delete _data;
        }
    }
    
    String& operator = (const String& rhs) { // 拷贝赋值
        if (this == &rhs) return *this;
        if (_data) delete _data;
        _init(rhs._data, rhs._len);
        return *this;
    }
    String& operator = (String &&rhs) { // 移动赋值
        if (this == &rhs) return *this;
        if (_data) = delete _data;
        _init(rhs._data, rhs._len);
        rhs._data = nullptr;
        rhs._len = 0;
        return *this;
    }
};
```

#### 完美转发.(常用于模板)

> C++11 标准中规定，通常情况下右值引用形式的参数只能接收右值，不能接收左值。但对于**函数模板中使用右值引用语法**定义的参数来说，它不再遵守这一规定，**既可以接收右值，也可以接收左值（**此时的右值引用又被称为“**万能引用**”）。

```c++
template <typename T>
void function(T&& t) {
    otherdef(t);
}
```

```c++
int n = 10;
int & num = n;
function(num); // T 的类型是 int&
int && num2 = 11;
function(num2); // T 的类型是 int &&
```

C++ 11标准为了更好地实现完美转发，特意为其指定了新的类型匹配规则，又称为**引用折叠规则**（假设用 A 表示实际传递参数的类型）：

- 当实参为左值或者左值引用（A&）时，函数模板中 T&& 将转变为 `A&`（A& && = A&）；
- 当实参为右值或者右值引用（A&&）时，函数模板中 T&& 将转变为 `A&&`（A&& && = A&&）。



c++11引入`std::forward`将函数模板接收到的参数类型一模一样地传递给被调用地函数

```c++
//实现完美转发的函数模板
template <typename T>
void function(T&& t) {
    otherdef(forward<T>(t));
}
//重载被调用函数，查看完美转发的效果
void otherdef(int & t) {
    cout << "lvalue\n";
}
void otherdef(const int & t) {
    cout << "rvalue\n";
}
```

```c++
function(5); // rvalue
int  x = 1;
function(x); // lvalue
```



### 7. `nullptr`

`nullptr`是一个**关键字**，它会**自动转换为对应的指针类型**。而`NULL`或`0`虽然能够代表空指针的含义，但实际上是整数类型。

```c++
void f(int);
void f(void*);

f(0);//调用f(int)
f(NULL);//调用f(int) 
f(nullptr); //调用f(void*)
```

函数`f(NULL)`表达的含义不清晰，`NULL`一般用作空指针。但其类型是整数类型0，因此编译器会将NULL当作整数类型。

### 8. 区间迭代

>  Range-based for statement

### 9. `=default, =delete`

强制加上`=default`, 重新获得并使用**默认函数**

```c++
class Zoo
{
public:
    Zoo(int i): _i(i){}
    Zoo(Zoo&)=default;
    Zoo(Zoo&&)=default;//move ctor
    Zoo& operator=(const Zoo&)=default;
}

```

#### Big Three

编译器为 **构造函数**、**赋值函数**、**析构函数**设置默认的函数

#### No Copy, NoDtor

```c++
struct NoCopy{
    NoCopy(const NoCopy&)=delete;
    NoCopy& operator()=(const NoCopy&)=delete;
};
```

```c++
struct PrivateCopy{
private:
    PrivateCopy(const PrivateCopy&);
    PrivateCopy& operator()=(PrivateCopy NoCopy&);
public:
    ...
}

```

### 10. Alias

#### Alias Template

`typedef`不接受参数

```c++
template<T>
using vec=vector<T, myAlloc<T>>;
```

#### Type Alias

类似与typedef

### 11. noexpect

函数后`noexpect`表示该函数不会发出异常

```c++
void foo() noexpect;等价于 void foo() noexpect(true)
```

### 12. `override`, `final`

#### override

让编辑器检查父类函数重写是否出错

```c++
class test{

    virtual void funtion(int) override{};
};
```

#### final

- 不允许父类被重写

  ```c++
    struct base final{};
  ```

- 不允许函数被重写

  ```c++
  struct test{
      virtual void function() final;
  };
  ```

### 13. `std::function`

> 类模板 `std::function` 是通用**多态函数**包装器



# 8. 设计模式

**总原则**：开闭原则

开闭原则即**对扩展开放**，**对修改关闭**。在程序需要进行拓展的时候，不能去修改原有的代码，而是要扩展原有代码，实现一个热插拔的效果。所以一句话概括就是：为了使程序的扩展性好，易于维护和升级。想要达到这样的效果，我们需要使用接口和抽象类等，后面的具体设计中我们会提到这点。

**1、单一职责原则**

不要存在多于一个导致类变更的原因，也就是说**每个类应该实现单一的职责**，如若不然，就应该把类拆分。

**2、里氏替换原则**

里氏代换原则（LSP）是面向对象设计的基本原则之一。 里氏替换原则通俗来讲就是：`子类可以扩展父类的功能，但不能改变父类原有的功能`

**3、依赖反转原则**

这个是开闭原则的基础，具体内容：**面向接口编程，依赖于抽象而不依赖于具体**。写代码时用到具体类时，不与具体类交互，而与具体类的上层接口交互。

**4、接口隔离原则**

这个原则的意思是：**要为各个类建立它们需要的专用接口，而不要试图去建立一个很庞大的接口供所有依赖它的类去调用**。使用多个隔离的接口，比使用单个接口（多个接口方法集合到一个的接口）要好。

**5、迪米特法则**（最少知道原则）

就是说：一个类对自己依赖的类知道的越少越好。也就是说无论被依赖的类多么复杂，都应该将逻辑封装在方法的内部，通过public方法提供给外部。这样当被依赖的类变化时，才能最小的影响该类。

最少知道原则的另一个表达方式是：只与直接的朋友通信。类之间只要有耦合关系，就叫朋友关系。耦合分为依赖、关联、聚合、组合等。我们称出现为成员变量、方法参数、方法返回值中的类为直接朋友。局部变量、临时变量则不是直接的朋友。我们要求陌生的类不要作为局部变量出现在类中。

**6、合成复用原则**

原则是尽量首先使用合成/聚合的方式，而不是使用继承。

## 8.1. 创造型

创建型模式的主要关注点是“**怎样创建对象？**”，它的主要特点是“**将对象的创建与使用分离**”。这样可以降低系统的耦合度，使用者不需要关注对象的创建细节，对象的创建由相关的工厂来完成。

### 1. 单例模式

> 不可以使用double check的单例模式，因为内存的重排序导致有线程安全问题

C++11版本以前，线程安全的单例模式

```c++
class Singleton{
private:
    Singleton();
    Singleton(const Singleton&);
    Singleton operator(const Singleton&);
    static Singleton* instance;
    mutex mutex_;
    
public:
    Singleton* getInstance() {
        mutex_.lock();
        if (instance == nullptr) {
            instance = new Singleton();
        }
        mutex_.unlock();
        return instance;
    }
};
```

C++11规定了local static在多线程条件下的初始化行为，要求编译器保证了内部静态变量的线程安全性。在C++11标准下，《Effective C++提出了一种更优雅的单例模式实现，使用函数内的 local static 对象。

```c++
class A{
public :
    static A& getInstance();
private:
    A();
    A(const A&ths);
};
inline A::getInstance(){
    static A a;//只要有人调用a就会一直存在

    return a;
}
```

### 2. 简单工厂模式

#### 模型结构

- Factory：工厂角色

  工厂角色负责实现创建所有实例的内部逻辑

- Product：抽象产品角色

  抽象产品角色是所创建的所有对象的父类，负责描述所有实例所共有的公共接口

- ConcreteProduct：具体产品角色

  具体产品角色是创建目标，所有创建的对象都充当这个角色的某个具体类的实例。

<img src="https://design-patterns.readthedocs.io/zh_CN/latest/_images/SimpleFactory.jpg" alt="../_images/SimpleFactory.jpg" style="zoom: 80%;" />

```c++
// 抽象product
class Fruit{
public:
    virtual void use() = 0;
};
```

```c++
// 具体product
class Apple： public Fruit{
public:
    void use() override {
        printf("apple\n");
    }
};
```

```c++
// 简单工厂
class Factory{
public:
    Fruit createFruit(string s) {
        if (s == "apple"){
			return new Apple();
        }
    }
};
```

#### 模型应用

1. JDK类库中广泛使用了简单工厂模式，如工具类java.text.DateFormat，它用于格式化一个本地日期或者时间。

   ```java
   public final static DateFormat getDateInstance();
   public final static DateFormat getDateInstance(int style);
   public final static DateFormat getDateInstance(int style,Locale locale);
   ```

2. Java加密技术

   ```java
   // 获取不同加密算法的密钥生成器:
   KeyGenerator keyGen=KeyGenerator.getInstance("DESede");
   // 创建密码器:
   Cipher cp=Cipher.getInstance("DESede");
   ```

### 3. 工厂方法模式

> 工厂模式可以在不修改具体工厂类的情况下引进新的产品，只需要为这种新类型创建一个具体的工厂类就可以获得新实例，这一特点无疑使得工厂方法模式具有超越简单工厂模式的优越性，**更加符合“开闭原则”**。

#### 模型结构

工厂方法模式包含如下角色：

- Product：抽象产品
- ConcreteProduct：具体产品
- Factory：抽象工厂
- ConcreteFactory：具体工厂

<img src="https://design-patterns.readthedocs.io/zh_CN/latest/_images/FactoryMethod.jpg" alt="../_images/FactoryMethod.jpg" style="zoom:80%;" />

```c++
// 抽象product
class Fruit{
public:
    virtual void use() = 0;
};
// 具体product
class Apple： public Fruit{
public:
    void use() override {
        printf("apple\n");
    }
};
// 具体product
class Banana: public Fruit {
public:
    void use() override {
        printf("banana\n");
    }	
};
```

```c++
// 抽象工厂
class Factory{
public:
    virtual Fruit factoryMethod() = 0;
};

// 苹果工厂
class AppaleFactory{
public:
    Fruit factoryMethod() {
        return Appale();
    }
};
// 香蕉工厂
class AppaleFactory{
public:
    Fruit factoryMethod() {
        return Banana();
    }
};
```

#### 模型应用

**日志记录器：**某系统日志记录器要求支持多种日志记录方式，如文件记录、数据库记录等，且用户可以根据要求动态选择日志记录方式， 现使用工厂方法模式设计该系统。

<img src="https://design-patterns.readthedocs.io/zh_CN/latest/_images/loger.jpg" alt="../_images/loger.jpg" style="zoom:80%;" />



### 4.抽象工厂模式

<img src="https://design-patterns.readthedocs.io/zh_CN/latest/_images/AbatractFactory.jpg" alt="../_images/AbatractFactory.jpg" style="zoom:80%;" />

#### 模型应用

在很多软件系统中需要更换界面主题，要求界面中的按钮、文本框、背景色等一起发生改变时，可以使用抽象工厂模式进行设计。

### 5. 建造者模式

#### 模型结构

建造者模式包含如下角色：

- Builder：抽象建造者

- ConcreteBuilder：具体建造者

- Director：指挥者

- Product：产品角色

  <img src="D:\mygit\notes\images\Builder.jpg" alt="../_images/Builder.jpg" style="zoom:80%;" />

#### 模型应用

在很多游戏软件中，地图包括天空、地面、背景等组成部分，人物角色包括人体、服装、装备等组成部分，可以使用建造者模式对其进行设计，通过不同的具体建造者创建不同类型的地图或人物。

## 8.2. 结构型

### 1. 外观模式

Facade模式可以为互相关联在一起的错综复杂的类整理出高层API，让系统对外只有一个简单的接口。

**简单来说，是对复杂关系的接口做一个总结。**

### 2. 代理模式

> 只在必要时生成对象实例

#### 模型结构

参与角色:

- 主体(Subject)：定义了Proxy角色和RealSubject角色之间的接口。由于Subject角色的存在，客户不需要在于调用的是Proxy还是RealSubject
- 代理人(Proxy): 只有自己处理不了时，才会将工作交给RealSubject
- 实体主体(RealSubject)

<img src="https://design-patterns.readthedocs.io/zh_CN/latest/_images/Proxy.jpg" alt="../_images/Proxy.jpg" style="zoom:80%;" />



## 8.3. 行为型

### 1. 策略模式

## 8.4. 其他

# 其他

## C/C++程序编译/链接模型

> 所有东西放在一个文件中编译，使得编译速度太慢，修改一个地方就要整个重新编译。
>
> 编译/链接模型，**"分块处理"**，便于程序修改升级

“分开处理”衍生概念:

- 定义/声明
- 头文件/源文件
- 翻译单元
  - 源文件 + 相关头文件（直接/间接） - 应忽略的预处理语句
- 

### 预处理

- 将源文件转换为翻译单元的过程
- 防止头文件被循环展开
  - `#ifdef` 
  - `#pragma once`

比如宏命令`#define a b`，将源程序中的所有替换为b
`#include`命令将头文件的内容添加进来

```bash
gcc a.cpp -E -o a.i
```

### 编译

- 将翻译单元通过**词法和语法分析**，转换为**汇编代码**
- **编译优化**

```bash
gcc a.i -S -o a.s
```

### 汇编

将汇编代码翻译为**机器指令**的目标文件

```bash
gcc a.s -c -o a.o
```

### 链接

- 合并多个目标文件，关联声明与定义
- 链接种类: 内部链接(定义只在翻译单元内部可见)、外部链接(在不同翻译单元可见)、无链接(同样的翻译单元也不可以见，如函数的局部变量)
- 链接常见错误: 找不到定义

变量、函数等分为声明和定义，一个文件中声明定义了，如果其他文件想要使用可以进行声明。在链接阶段，将这些声明关联上。

```bash
gcc a.o -
```

#### 静态链接

在链接的时候将要调用的函数链接到了生成的可执行文件中

**优点：**

- 静态库整个打包到程序中，加载速度快
- 发布程序不需要提供静态库，移植方便

**缺点:**

- 需要给更大的空间
- 更新、部署、发布麻烦

#### **动态链接**

在链接的时候没有把调用的函数链接到文件中，而是在执行的过程中再去链接函数，生成的可执行文件中只包含函数的重定位信息

**优点:**

- 实现进程间共享动态库
- 更新、部署、发布简单

**缺点:**

- 加载速度要比静态库慢
- 发布程序时需要提供依赖的动态库

## 函数调用分析

### 函数调用

**栈帧**：用来记录函数的一次过程信息，被*esp*和*ebp*包裹

```c++
void test1(const String &s);
void test2(int a, double b);
```

> 传递的参数会被拷贝一份压入栈帧中。
>
> 基本数据类型（包括引用，指针等）直接copy，而自定义类型编译器调用其**拷贝构造**函数

### 保存现场

将下一个执行的指令地址压入栈内，然后将ebp入栈

执行函数中断

将当前esp作为新的ebp

### 执行子函数

创建子函数的局部变量（可以传递值）

### 恢复现场

子函数执行完后中断返回，出栈读取pc等的值，将当前的esp赋值为ebp， 将esp出栈，函数继续执行
