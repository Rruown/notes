# 概述

- 预备   Linux
- 第一章 Linux系统编程入门
- 第二章 Linux多进程开发
- 第三章 Linux多线程开发
- 第四章 Linux网络编程
- 第五章 项目实战

# Linux系统编程入门

## 基础命令

### 系统管理

> 如何（自动）启动/关闭服务

计算机中，一个正在执行的程序或命令，被叫做“进程”（process）。
启动之后一只存在、常驻内存的进程，一般被称作“服务”（service）。

#### service服务管理

**基本语法**

```bash
service 服务名 start|stop|restart|status
```

**常用案例**

```bash
/etc/init.d/ # 查看服务

service ssh start # 启动ssh服务
```

#### systemctl

> 更加强大的service

**基本语法**

```bash
systemctl start | stop | restart | status 服务名 # 启动| 关闭 | 重启 | 查看 服务
systemctl list-unit-files # 查看服务开机启动状态
systemctl enable 服务名 # 开启指定服务的自动启动
systemctl disable 服务名 # 关闭指定服务的自动启动
```

**常用案例**

```bash
/usr/lib/systemd/system # 查看服务
systemctl enable ssh # 开启ssh服务的自动启动
systemctl disable ssh # 关闭ssh服务的自动启动
```



### 搜索查找

#### find

从指定目录递归地遍历满足条件的文件

**基本语法**

```bash
find [搜索范围，默认当前目录][选项]
```

**选项说明**

- `-name`：指定文件名查找文件

- `-user`：查找指定用户的文件

- `-size`：查找指定文件大小文件,单位略；

  - > 可以用来排查大文件

#### locate

利用事先建立的系统中所有文件名称及路径的locate数据库快速定位文件

> 由于locate指定基于数据库查询，第一次运行前，必须使用`updatedb`指令创建locate数据库

**基本语法**

```bash
locate 搜索文件
```



### 进程管理

#### ps

**选项说明：**

- a: 列出带有终端的所有用户的进程
- x: 列出当前用户的所有进程，包括没有终端的进程
- u: 面向用户友好的显示风格
- e: 列出所有进程
- u: 列出某个用户关联的所有进程
- f: 显示完整格式的进程列表

常用命令：

```bash
ps aux # 查看系统所有进程
ps -ef # 查看子父进程之间的关系
```

#### kill

常用命令

```bash
kill -9 PID # 向PID发送SIGINT信号，强制关闭进程
killall 进程名 # 通过进程名杀死进程
```

#### pstree 查看进程树

**基本语法**

```bash
pstree [-p/-u]
```

**常用选项：**

- p: 显示进程的PID
- u: 显示进程的所属用户

#### top

#### netstat 

**显示网络状态和端口占用信息**

**基本语法**

```bash
netstat -anp | grep 进程号 # 查看进程网络信息
netstat -nlp | grep 端口号 # 查看网络端口占用情况
```

**常用选项：**

- a: 显示所有正在监听（listen）和未监听的套接字（socket）
- n: 拒绝显示别名，能显示数字的全部转化成数字
- l: 仅列出在监听的服务状态
- p: 表示显示哪个进程在调用

### 文件目录管理

**常用命令**

```bash
pwd # 显示当前工作目录
ls -al # 列出当前目录下的所有文件信息（包括隐藏文件）
cd 
mkdir
rmdir	
touch
cp -r source target # -r表示递归复制文件目录下的所有文件
rm -rf 文件名 # -f 强制执行
mv 文件1 文件2 # 移动或重名文件(夹)
cat，more,less, head, tail # 查看文件内容

```

#### 输出重定向与追加

**基本语法**

```bash
ls -l > 文件 # 将ls -l的内容写入文件中（覆盖写）
ls -l >> 文件 # 追加写
```

#### 软连接

> ln 命令默认硬链接，-s表示软链接

**基本语法**

```bash
ln -s 原文件（目录） 连接名
```

### 权限管理

#### chmod

> 修改文件/目录权限

![image-20220827093257686](..\images\image-20220827093257686.png)

> u：所有者，g：所有组，o：其他人，a：所有人（u+g+o）

**第一个方式**

```bash
chmod [{ugo}{+-=}{rwx}] 文件或目录
```

**第二种方式**

```bash
chmod [mod=421] 文件或目录
```

#### chown

> 修改所属者

```bash
chown [选项] 最终用户名 文件或目录
```



#### chgrp

> 修改所属组

```bash
chrgp [选项] 最终所属组 文件或目录
```



### 用户管理

> /etc/passwd 

**常用命令**

```bash
useradd 用户名 # 添加用户
userdel 用户名 # 删除用户
passwd 用户名 # 为用户设置密码
id 用户名 # 判断用户名是否存在
su 用户名 # 切换用户
```



### 用户组管理

**常用命令**

```bash
groupadd 组名 # 新增组
groupdel 组名 # 删除组
groupmod -n 新名字 旧名字 # -n 指定工作组名字
usermod -g 用户组 用户名 # 修改用户的所属组
```



## Linux开发环境搭建

**公钥私钥** 
> 避免频繁登录账号密码

```shell
ssh-keygen -t ras//.ssh目录下生成公钥和私钥
```

将本地的公钥注册到服务器`.ssh`目录下的`authorized_keys`文件中



## 编译及调试工具

### 静态库的制作

静态库是程序在链接阶段被复制到了程序里

动态库是程序运行时动态加载内存中供程序调用

**命名规则**：

- Linux：`libxxx.a`
  - lib前缀固定
  - xxx库名
  - .a后缀固定
- Windos:libxxxx.lib    

**Linux静态库的制作**
- `gcc`获得`.o`文件
- 将`.o`文件打包，使用`ar`工具（archive）
  `ar rcs libxxx.a xxx.o xxx.o`
  - r 将文件插入备存文件中
  - c 建立备存文件
  - s 索引

`gcc`参数
- c：只汇编不链接
- I：指定头文件路径
- l：指定库名
- L：指定库文件路径

### 动态库的制作

**命名规则**：

- Linux：`libxxx.so`
  - lib前缀固定
  - xxx库名
  - .so后缀固定
- Windos：libxxxx.dll   

**Linux动态库的制作**
- `gcc`获得`.o`文件，得到和位置无关的代码
  `gcc -c -fpic/-fPIC a.o b.o`
- `gcc`得到动态库
  `gcc -shared a.o b.o -o libcalc.so`
  
- 可以通过`ldd`查看动态库依赖关系
- 在`LD_LIBRARY_PATH`中配置动态库路径

### Makefile

> Makefile文件定义了一系列规则来指定哪些文件先编译，哪些文件后编译，以及哪些文件重新编译等

**Makefile规则**:

- 一个Makefile文件可以有一个或多个规则
  目标 ...: 依赖 ...
       命令 (Shell 命令)
       ...
  - 目标：最终要生成的文件
  - 依赖：生成目标所需要的文件或是目标
  - 命令：通过执行命令对依赖操作生成目标
- Makefile中的其他规则都是为第一条规则服务的

**Makefile工作原理**：

- 查找目标的依赖是否存在
  - 存在则执行命令
  - 不存在，向下查找能生成该依赖的规则
- 检测更新
  - 如果依赖文件比目标文件时间晚，则重新执行命令
  - 否则不执行

**Makefile扩展**：

- 预定义变量：
  - `AR`
  - `CC`
  - `$@`：目标的完整名称
  - `$<`：第一个依赖文件的名称
  - `$^`：所有依赖文件的名称
- 函数
  - `$(wildcard PATTERN...)` 获取指定目录下指定类型的文件列表
  - `$(patsubst <pattern>, <replacement>, <text>)` 查找`<text>`中的单词是否匹配`<pattern>`如果匹配则替换成`<replacement>`

### GDB

## docker

### [命令接口](https://docs.docker.com/engine/reference/commandline/cli/)

#### [docker run](https://docs.docker.com/engine/reference/commandline/run/)

```bash
 docker run [OPTIONS] IMAGE [COMMAND] [ARG...]
```

`--runtime`指定容器的runtime

`-e,--env,--env-file`设置容器的环境变量

`--name`指定容器的名字

`-it`分配容器的伪tty

`-v， --volume`指定容器挂载点

```bash
docker -v a:b # 将a目录挂载到容器的b目录
```

> 解决`docker run`无法启动：
>
> - `-itd` 指定伪tty并设置后台运行，避免主线程停止运行从而停止容器

## 文件描述符

PCB（进程控制块）存放在内核区，其中存放了**文件描述符表**
文件描述表默认1024个文件，前三个分别是：
- `STDIN_FILENO` 标准输入
- `STDOUT_FILENO` 标准输出
- `STDERR_FILENO` 标准错误
均指向当前终端，默认是打开状态

## Linux IO函数

### **打开关闭文件函数**

```c
#include <sys/types.h>
#include <sys/stat.h>
#include <fcntl.h>
//打开一个存在的文件
int open(const char *pathname, int flags);
    /*
      参数：
          flags: O_RDONLY, O_WRONLY, O_RDWR

      返回值：文件描述符
            失败返回- 1
        errno是Linux的一个全局变量，记录最近的错误序号
        perror(char*)打印错误信息
    */

//创建一个新文件
int open(const char *pathname, int flags, mode_t mode);
```

### 文件信息

```c
#include <sys/types.h>
#include <sys/stat.h>
#include <unistd.h>

int stat(const char *pathname, struct stat *statbuf);
```

参数：

- pathname：文件或文件夹路径
- statbuf：返回参数，文件的基本信息

返回值:

- 0，成功
- -1 失败

#### stat结构体定义

```c
 struct stat {
               dev_t     st_dev;         /* ID of device containing file */
               ino_t     st_ino;         /* Inode number */
               mode_t    st_mode;        /* File type and mode */
               nlink_t   st_nlink;       /* Number of hard links */
               uid_t     st_uid;         /* User ID of owner */
               gid_t     st_gid;         /* Group ID of owner */
               dev_t     st_rdev;        /* Device ID (if special file) */
               off_t     st_size;        /* Total size, in bytes */
               blksize_t st_blksize;     /* Block size for filesystem I/O */
               blkcnt_t  st_blocks;      /* Number of 512B blocks allocated */

               /* Since Linux 2.6, the kernel supports nanosecond
                  precision for the following timestamp fields.
                  For the details before Linux 2.6, see NOTES. */

               struct timespec st_atim;  /* Time of last access */
               struct timespec st_mtim;  /* Time of last modification */
               struct timespec st_ctim;  /* Time of last status change */

           #define st_atime st_atim.tv_sec      /* Backward compatibility */
           #define st_mtime st_mtim.tv_sec
           #define st_ctime st_ctim.tv_sec
 };

```



### **读写文件函数**

```c
  #include <unistd.h>
  ssize_t read(int fd, void *buf, size_t count);
  /*
      参数：
          fd:文件描述符，用来操作文件通过open函数获取
          buf：指定一个缓存区（通常是数组）存放读取的数据
          count：指定缓存区的大小
      返回值：
          成功: > 0 读到数据
               = 0 已经读完数据
          失败：-1 
  */
  #include <unistd.h>
  ssize_t write(int fd, const void *buf, size_t count);
  /*
      参数：
          fd:文件描述符，用来操作文件，通过open函数获取
          buf:指定一个缓存区，将其中的内容写入到fd指定的文件中
          count:缓存区大小
      返回值：
          成功：> 0
          失败： - 1
  */
```



### 高级读写函数

在一次函数调用中：

- `writev`以顺序`iov[0]`、`iov [1]`至`iov[iovcnt-1]`从各缓冲区中聚集输出数据到`fd`
- `readv`则将从`fd`读入的数据按同样的顺序散布到各缓冲区中，`readv`总是先填满一个缓冲区，然后再填下一个



### **lseek函数**

```c
  #include <sys/types.h>
  #include <unistd.h>
  off_t lseek(int fd, off_t offset, int whence);
  /*
      参数： 
          fd：文件描述符，通过open函数获取
          offset：偏移量，用来指定文件读写指针的偏移
          whence: 
                SEEK_SET: 将文件偏移设置到offset处
                SEEK_CUR: 将文件偏移设置到当前位置+offset处
                SEEK_END: 将文件偏移设置到文件末尾+offset处
      返回值：文件当前偏移

      作用：1、获取文件头
            lseek(fd, 0, SEEK_SET)
            2、获取文件长度
            lseek(fd, 0, SEEK_END)
            3、扩展文件
            lseek(fd, offset, SEEK_END)
  */
```



# 开发工具

## cmake

cmake**命令使用：**

```cmake
cmake -S . -B build # -S 指定编译的源目录 -B 指定build目录
```

```cmake
mkdir build && cd build # 创建一个build目录
cmake .. && make -j$(nproc) # cmake ..将源目录（./build/..）编译放在build目录下， 多线程make -j$(nproc)
```

### 基础配置

1 项目名称及版本(必选)

```cmake
cmake_minimum_required(VERSION 3.0) # 指定cmake的最低支持版本
project(项目名 version 版本号 language c cxx)
```

**2 CMake变量及环境变量设置**（可选）

`set`为变量和环境变量设置值，语法如下：

> 环境变量是CMake预定义的变量是全局的，普通变量作用范围当前CMakeLists.txt文件

```cmake
set(<varible>/ENV{<varible>} <value> <value> ... [PARENT_SCOPE]) # PARENT_SCOPE 向上层返回变量值，这会修改上层变量的值
```

*普通变量在**`add_subdirectory()`、`function()`**相当于值传递，可以通过`PARENT_SCOPE`修改值, 会复制一份父CmakeLists文件中的变量定义并初始化。*

*普通变量在**`include()`、`macro()`**类似于预处理，相当于引用传递*



`set`设置Cache变量：

> Cache 变量相当于一个全局变量，在同一个 cmake 工程中都可以使用

```cmake
set(<variable> <value>... CACHE <type> <docstring> [FORCE])

# 设置 Cache 变量
set(MY_GLOBAL_VAR "666" CACHE STRING INTERNAL )

# 修改 Cache 变量
set(MY_GLOBAL_VAR "777" CACHE STRING INTERNAL FORCE)
```



指定编程语言版本：

```cmake
set(CMAKE_C_STANDARD 99)
set(CMAKE_CXX_STANDARD 11)
```

指定cmake构建类型：

```cmake
 set (CMAKE_BUILD_TYPE "Debug") # debug模式
 set (CMAKE_BUILD_TYPE "Relese") # Release模式
```

**重要的一些环境变量:**

- `PROJECT_SOURCE_DIR`为包含`PROJECT()`的最近一个`CMakeLists.txt`文件所在的文件夹。
- `CMAKE_SOURCE_DIR`指顶层的CMakeLists.txt路径
- `CMAKE_CURRENT_SOUR_DIR`值cmake文件当前的路径
- `CMAKE_MODULE_PATH`值cmake搜索.cmake脚本的路径



**3 配置编译选项**（可选）

```cmake
add_compile_options(-Wall -Wextra -pedantic -Werror) // 为所有编译器配置选项
set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -pipe -std=c99") // 针对C编译器配置
set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -pipe -std=c++11") // 针对C++编译器配置
```

**4 配置编译类型**（可选）

```cmake
set(CMAKE_BUILD_TYPE Debug) // 可设置为：Debug、Release、RelWithDebInfo、MinSizeRel等
```

**6 添加include目录**（必选）

```cmake
include_directories(src/c)
```

**7 添加link目录（可选）**

指定链接器查找库的目录

```cmake
link_directories(${PADDLE_PATH}/lib)
```

### 编译目标文件

一般来说，编译目标(target)的类型一般有静态库、动态库和可执行文件。 这时编写`CMakeLists.txt`主要包括两步：

1. 编译：确定编译目标所需要的源文件
2. 链接：确定链接的时候需要依赖的额外的库

**编译target的三种命令**:

- `add_executable`: 可执行的目标文件（必须有main函数）
- `add_library`: 编译静态库、动态库
- `add_custom_target`

**`add_library`与** **`add_depencies`** **区别:**

- `add_depencies`编译时检查依赖关系，先编译依赖的目标

> 动态库由Linux中的**ld**自动加载，`ldd`查看程序依赖库。**ld**除了从标准路径加载共享库之外，还会从非标准路径（由`LD_LIBRARY_PATH`系统环境变量指定）加载动态库。

**1 编译库或添加库路径**

```cmake
file(GLOB_RECURSE <variable> [FOLLOW_SYMLINKS]
     [LIST_DIRECTORIES true|false] [RELATIVE <path>] # RELATIVE 代表是否记录相对路径，默认false
     [<globbing-expressions>...])

     
aux_source_directory (< dir > < variable >) # 将目录下的所有源文件存放在变量中       

add_library(math STATIC ${MATH_LIB_SRC}) # 静态链接 SHARED是动态连接

link_directories(${PADDLE_PATH}/lib) # 指定链接库目录
```

**2 链接库**

```cmake
add_executable(可执行文件名 main.cxx)
target_link_libraries(可执行文件名 链接库)
```

### 模块管理

> include、find_package和add_subdirectory都可以模块管理，区别在于：
>
> （1）include、find_packge方式类似，都是加载.cmake脚本文件。add_subdirectory是添加子目录，加载CMakeLists.txt文件
>
> （2）include和find_package的类似于#include宏定义，变量是共享的，而add_subdirectory的子文件是拷贝了一份父文件内的变量，相当于值传递

**1 通过CMake内置模块引入依赖包**

> 为了方便在项目中引入外部依赖包，cmake官方预定义了许多寻找依赖包的Module，他们存储在`path_to_your_cmake/share/cmake-<version>/Modules`目录下。每个以`Find<LibaryName>.cmake`命名的文件都可以帮我们找到一个包。在官方文档中可以查看到哪些库官方已经为我们定义好了，我们就可以直接使用find_package函数进行引用[官方文档：Find Modules](https://link.zhihu.com/?target=https%3A//cmake.org/cmake/help/latest/manual/cmake-modules.7.html)。

以curl库为例，假设项目需要引入这个库，从网站中请求网页到本地，官方已经定义好了FindCURL.cmake

```cmake
find_package(CURL)
add_executable(curltest curltest.cc)
if(CURL_FOUND)
    target_include_directories(clib PRIVATE ${CURL_INCLUDE_DIR})
    target_link_libraries(curltest ${CURL_LIBRARY})
else(CURL_FOUND)
    message(FATAL_ERROR ”CURL library not found”)
endif(CURL_FOUND)
```

每一个模块都会预定义以下几个变量：

```
<LibaryName>_FOUND
<LibaryName>_INCLUDE_DIR or <LibaryName>_INCLUDES <LibaryName>_LIBRARY or <LibaryName>_LIBRARIES
```



**2 通过find_package引入非官方的库**

> 该方式只对支持cmake编译安装的库有效

安装方法以glog库为例，之后便可以通过与引入curl库一样的方式引入glog库了

```cmake
# clone该项目
git clone https://github.com/google/glog.git 
# 切换到需要的版本 
cd glog
git checkout v0.40  

# 根据官网的指南进行安装
cmake -H. -Bbuild -G "Unix Makefiles"
cmake --build build
cmake --build build --target install
```

**3 Module模式与Config模式**

在Module模式中，cmake需要找到一个叫做`Find<LibraryName>.cmake`的文件。这个文件负责找到库所在的路径，为项目引入头文件路径和库文件路径。cmake搜索这个文件的路径有两个，一个是上文提到的cmake安装目录下的`share/cmake-<version>/Modules`目录，另一个是`CMAKE_MODULE_PATH`指定的目录。

如果Module模式搜索失败，没有找到对应的`Find<LibraryName>.cmake`文件，则转入Config模式进行搜索。它主要通过`<LibraryName>Config.cmake` or `<lower-case-package-name>-config.cmake`这两个文件来引入需要的库。以安装的glog库为例，在安装之后，它在`/usr/local/lib/cmake/glog/`目录下生成了`glog-config.cmake`文件，而`/usr/local/lib/cmake/<LibraryName>/`正是find_package函数的搜索路径之一。



常见的用法是将.cmake文件的放入`PRJECT_SOURCE_DIR/cmake`文件下，并加入cmake Module的搜索路径

```cmake
set(CMAKE_MODULE_PATH "${CMAKE_SOURCE_DIR}/cmake;${CMAKE_MODULE_PATH}")
add_executable(addtest addtest.cc)
find_package(ADD)
if(ADD_FOUND)
    target_include_directories(addtest PRIVATE ${ADD_INCLUDE_DIR})
    target_link_libraries(addtest ${ADD_LIBRARY})
else(ADD_FOUND)
    message(FATAL_ERROR "ADD library not found")
endif(ADD_FOUND)
```

### 控制流

**命令或函数定义**

> 参数都是map类型，`options`是<string, bool>映射。oneValueArgs是<string, string>。
>
> multiValueArgs是`map<string, vector<string>>`映射


```cmake
function(test_parse)
    set( options op1 op2 op3 ) # 设置options可选项， 类型是Bool
    set( oneValueArgs v1 v2 v3 ) # 设置一个值的参数名
    set( multiValueArgs m1 m2 )
    message( STATUS "options = ${options}" )
    message( STATUS "oneValueArgs = ${oneValueArgs}" )
    message( STATUS "multiValueArgs = ${multiValueArgs}" )
    cmake_parse_arguments( MYPRE "${options}" "${oneValueArgs}" "${multiValueArgs}" ${ARGN} )
    message("op1  = ${MYPRE_op1}")
    message("op2  = ${MYPRE_op2}")
    message("op3  = ${MYPRE_op3}")

    message("v1  = ${MYPRE_v1}")
    message("v2  = ${MYPRE_v2}")
    message("v3  = ${MYPRE_v3}")

    message("m1  = ${MYPRE_m1}")
    message("m2  = ${MYPRE_m2}")
endfunction()

message( STATUS "\n" )
test_parse( op1 op2 op3 v1 aaa v2 111 v3 bbb m1 1 2 3 4 5 m2 a b c )
# 输出结果：
# op1 = TRUE
# op2 = TRUE
# op3 = TRUE
# v1 = aaa
# v2 = 111
# v3 = bbb
# m1 = 1; 2; 3; 4; 5
# m2 = a; b; c;

message( STATUS "\n" )
test_parse(op1 op3 v1 aaa v2 111 v3 bbb m1 1 2 3 4 5 m2 a b c )
# 输出结果：
# op1 = TRUE
# op2 = False
# op3 = TRUE
# v1 = aaa
# v2 = 111
# v3 = bbb
# m1 = 1; 2; 3; 4; 5
# m2 = a; b; c;

message( STATUS "\n" )
test_parse(op1 op3 v1 aaa v2 v3 bbb m1 1 2 3 4 5  )
# 输出结果：
# op1 = TRUE
# op2 = False
# op3 = TRUE
# v1 = aaa
# v2 = 
# v3 = bbb
# m1 = 1; 2; 3; 4; 5
# m2 = 

message( STATUS "\n" )
test_parse( op1  v1 aaa v2 op2 111 v3 bbb m1 1 2 3 4 5 m2 a b c op3 )
# 输出结果：
# op1 = TRUE
# op2 = TRUE
# op3 = TRUE
# v1 = aaa
# v2 = 
# v3 = bbb
# m1 = 1; 2; 3; 4; 5
# m2 = a; b; c;
```

## git

### Git规范化提交-CZ工具

[Cz工具集使用介绍](https://juejin.cn/post/6844903831893966856)





# 进程

## 进程创建

```c
  #include <sys/types.h>
  #include <unistd.h>
  pid_t fork(void);
  /*
      参数：无
      返回值: 返回两次

            父进程：成功时，返回子进程的p_id，失败返回-1
            子进程: 返回 0
  */

```

## 父子进程虚拟地址空间

从**低地址到高地址**，一个程序由**代码段**、**数据段**、**BSS段**、**堆**、**共享区**、**栈**等组成。

- 代码段：存放程序的机器指令
- 数据段：存放**已被初始化的**全局变量和静态变量
- BSS段：存放**未被初始化的**全局变量和静态变量
- 运行时有**堆**、**栈**
- 共享库：位于堆和栈中间

父进程`fork()`一个子进程时，子进程与父进程具有相同的用户区空间，不同的内核区（比如pid不同，但是文件描述符相同）。

`fork()`函数采用了**读时共享写时复制**的技术。即读数据的时候，子进程与父进程共享一个用户区空间，只有写数据时，系统再给子进程复制一份父进程的用户区空间

**总结**：

- 不同点:
  - 返回值不一样
    - 父进程 > 0 : 返回的时子进程的pid
    - 子进程 = 0
  - 内核区一些数据
    - pid、ppid等
- 相同点：某些状态下相同
  - 读时共享，写时拷贝

## GDB多线程调试

**添加gdb调试**
`gcc xxx.c -o xxx -g`

**gdb常见命令**：

- l：显示代码
- b x : 在x行添加断点
- i b ：显示断点信息
- r ：调试运行
- n : 下一步
- c : 继续调试
  

**设置调试父进程或子进程**：`set follow-fork-mod`

**设置调试模式**：`set detach-on-fork [on | off]`

- `on` 表示调试当前进程的时候，其它的进程继续进行
- `off`表示调试当前进程的时候，其它进程被GDB挂起

**查看调试的进程**：`info inferiors`
**切换当前调试的进程**：`inferiors id`
**使进程脱离GDB调试**：`detach inferiors id`

## exec函数族

exec函数族的作用是根据指定的文件名找到可执行文件，并用它来取代调用进程的内容

```c
#include <unistd.h>
int execl(const char *path, const char *arg, ...)//l(list) 参数地址列表，以空指针结尾
/*
    参数:
        path: 文件路径（绝对路径或相对路径）
        arg: 可执行文件的参数，第一个参数无意义通常是文件名， 最后要以NULL结尾
    返回值: 失败时返回-1
*/



int execlp(const char *file, const char *arg, ...)//p(path) 存有各参数地址的指针数组的地址
/*
    参数:
        file: 文件名(从环境变量中找可执行文件)
        arg: 可执行文件的参数，最后要以NULL结尾
    返回值: 失败时返回-1
*/
```

## 孤儿进程、僵尸进程、进程回收

孤儿进程：子进程尚未运行结束，但父进程已经运行结束了。将子进程的管理委托给init进程。无危害

僵尸进程：子进程结束后，父进程还未来得及回收子进程的PCB资源。有危害

```c
#include <sys/types.h>
#include <sys/wait.h>

pid_t wait(int *wstatus);
/*
  wait函数将调用的父进程挂起，直到其某一个子进程状态改变
  参数：
      *wstatus： 退出相关的宏函数
  返回值:
      > 0 ： 状态改变的子进程的pid
      - 1 : 所有的子进程都已经结束
*/

pid_t waitpid(pid_t pid, int *wstatus, int options);
  /*
    参数：
        pid : > 0 指定回收的pid
              = -1 回收任意一个子进程pid(wait函数)
              = 0  回收调用进程所在组的pid
              > - 1 指定回收的进程组pid
        options : = 0 阻塞
                WNOHANG 不阻塞，立刻返回
                。。。

  */
```

**总结** :

- 进程的创建
  - fork()
  - 父子进程的虚拟地址空间
  - 多线程调试
- exec函数族
- 孤儿进程、僵尸进程、进程回收
  - wait
  - waitpid

## 进程通信

### 1. 匿名管道

UNIX 系统 IPC （进程间通信）的最古老形式

#### 基本原理

管道是内核空间的缓存区，由写端描述符和读端描述符

> **猜测**：内核空间中有一个数组是从描述符到缓存区地址的映射



通信原理依赖于**子进程会复制父进程的虚拟地址空间**

#### 优点

- 简单方便

#### 缺点

- **半双工**：在管道中的数据的传递方向是单向的，一端用于写入，一端用于读取。全双工通信，需要两个管道
- 匿名管道**只能在具有公共祖先的进程**（父进程与子进程，或者两个兄弟进程，具有亲缘关系）之间使用。

- 存在**用户态到内核态的数据拷贝开销**
- **只能以先进先出的方式接受数据**

#### 使用样例

```c
#include <unistd.h>
int pipe(int pipefd[2]);
/*
    参数 : 
        pipefd[2] ： 返回值， pipefd[0]读端, pipefd[1]写端

    返回值:
        成功 : 0
        失败 : -1 
*/

```

Shell中的`|`就是一种匿名管道，每个Shell初始化的时候就创建了一个匿名管道。

```bash
ls -al | grep test
```



### 2. 有名管道

>  **注意**：一个为只读打开管道的进程会被阻塞，直到另一个进程为只写打开管道；一个为只写打开管道的进程会被阻塞，直到另一个进程为只读打开管道；

#### 基本原理

有名管道是一个**特殊的文件**（伪文件-不占用硬盘的空间，只是在内存中作用。通过内核去管理调用）。

#### 优点

- 相比于匿名管道。不存在亲缘关系的进程，可以通过 FIFO 相互通信（相当于读写文件一样）

#### 缺点

- **半双工**：在管道中的数据的传递方向是单向的，一端用于写入，一端用于读取。全双工通信，需要两个管道

- 存在**用户态到内核态的数据拷贝开销**
- **只能以先进先出的方式接受数据**

#### 使用样例

`mkfifo [option]... NAME...`创建有名管道



### 3. 消息队列

#### 基本原理

**消息队列是保存在内核中的消息链表**。消息体是实现约定好的数据结构，并且**消息体必须要有类型号**（>=0）。接收方可以从消息队列中有选择地读取消息体。

#### 优点

- 可以用在不同的进程之间通信（针对于匿名管道）
- 可以有**选择**的读取消息（针对于管道通信）
- 全双工通信（针对于管道通信）

#### 缺点

- 会造成**通信不及时**
- **消息队列不适合比较大数据的传输**，因为在内核中每个消息体都有一个最大长度的限制，同时所有队列所包含的全部消息体的总长度也是有上限。
- **存在用户态与内核态之间的数据拷贝开销**

#### 使用样例

#### 消息的数据格式

```c
struct Msg{
     long type; // 消息类型。这个是必须的，而且值必须 > 0，这个值被系统使用
     // 消息正文，多少字节随你而定
     // ...
 };
```

##### **创建队列**

```c++
#include <sys/types.h>
#include <sys/ipc.h>
#include <sys/msg.h>

int msgget(key_t key, int msgflg);

```

参数：

- key: 某个消息队列的名字
- msgflg:由九个权限标志构成，用法和创建文件时使用的mode模式标志一样

返回值：

- 成功msgget将返回一个非负整数，即该消息队列的标识码；
- 失败则返回“-1”

##### **发送消息**

```c++
int  msgsnd(int msgid, const void *msg_ptr, size_t msg_sz, int msgflg);
```

参数：

- **msgid**: 由msgget函数返回的消息队列标识码
- **msg_ptr**:是一个指针，指针指向准备发送的消息，
- **msg_sz**:是msg_ptr指向的消息长度，消息缓冲区结构体中mtext的大小,不包括数据的类型
- **msgflg**:控制着当前消息队列满或到达系统上限时将要发生的事情
  如：
  msgflg = **IPC_NOWAIT** 表示队列满不等待，返回EAGAIN错误

返回值：

- 成功返回0
- 失败则返回-1

##### 读取消息

```c++
int  msgrcv(int msgid, void *msg_ptr, size_t msgsz,
		 long int msgtype, int msgflg);
```

**参数**：

- **msgid**: 由msgget函数返回的消息队列标识码
- **msg_ptr**:是一个指针，指针指向准备接收的消息，
- **msgsz**:是msg_ptr指向的消息长度，消息缓冲区结构体中mtext的大小,不包括数据的类型
- **msgtype**:它可以实现接收优先级的简单形式
  msgtype=0返回队列第一条信息
  msgtype>0返回队列第一条类型等于msgtype的消息　
  msgtype<0返回队列第一条类型小于等于msgtype绝对值的消息
- **msgflg**:控制着队列中没有相应类型的消息可供接收时将要发生的事
  msgflg=IPC_NOWAIT，队列没有可读消息不等待，返回ENOMSG错误。
  msgflg=MSG_NOERROR，消息大小超过msgsz时被截断

**返回值**：

- 成功返回实际放到接收缓冲区里去的字符个数
- 失败，则返回-1

##### 消息队列控制

```c++
int  msgctl(int msqid, int command, strcut msqid_ds *buf);
```

**参数**：

- msqid: 由msgget函数返回的消息队列标识码
- command:是将要采取的动作,（有三个可取值）分别如下
  - IPC_STAT
  - IPC_SET
  - IPC_

### 4. 共享内存

#### 内存映射

> mmap函数用于申请一段内存空间。munmap函数则用于释放mmap创建的这段内存空间。

```c
#include <sys/mman.h>

void *mmap(void *start, size_t length, int prot, int flags,int fd, off_t offset);
/*
    在进程的虚拟地址空间创建文件的内存映射或申请一段内存空间
    参数: 
        start: 允许用户使用某个特定的地址作为这段内存的起始地址。NULL表示由操作系统指定映射区，
        length: 内存段的长度
        prot: 内存段的权89限
              PROT_EXEC  Pages may be executed.
              PROT_READ  Pages may be read.
              PROT_WRITE Pages may be written.
              PROT_NONE  Pages may not be accessed.
        flags: 
              MAP_SHARED: 在进程间共享这段内存，对该都内存段的修改将反映到被映射的文件中。
              MAP_PRIVATE: 内存段为进程所私有
              MAP_ANONYMOUS: 表示这段内存不是从文件映射来的。其内容被初始化为全0。在这种情况下，后面两个参数可以被忽略
        fd: 映射的文件描述符
        offset: 0， 一般不指定
    返回值:
        映射区的内存地址
        - 1: 映射区创建失败
*/

int munmap(void *addr, size_t length);

/*
    关闭创建的内存映射
*/
```

### 5. 信号

> 中断机制——由设备硬件发送中断信号，cpu在每个指令周期结束后，会检查中断寄存器中的内容。
>
> 信号——由内核向进程注册信号（PCB（`task_struct`中有两个数据成员即未决信号集和阻塞信号集）），如果进程处于阻塞状态，则内核唤醒进程，然后内核向正在运行的进程发送中断，处理信号

信号是事件发生时对进程的通知机制，有时也称之为**软件中断**，它是在**软件层次上对中断机制的一种模拟**，是一种**异步通信**的方式。

**使用信号的两个主要目的**是：

- 让进程知道已经发生了一个特定的事情。
- 强迫进程执行它自己代码中的**信号处理程序**。

#### 信号函数

**`signal`函数**——为一个**信号设置处理函数**

```c
#include＜signal.h＞
_sighandler_t signal(int　sig,_sighandler_t _handler)

参数：
    sig：指出捕获的信号
    _handler: 一个函数指针。用于指定信号sig的处理函数
返回值:
	成功: 前一次调用signal函数时传入的函数指针或默认处理函数指针SIG_DEF
    失败: 返回SIG_ERR
```

**`sigaction`函数——更健壮的设置处理函数**

```c
#include＜signal.h＞
int sigaction(int sig,const struct sigaction*act,struct sigaction*oact);

struct sigaction
{ 
    #ifdef__USE_POSIX199309
        union
        { 
            _sighandler_t
            sa_handler;
            void(*sa_sigaction)(int,siginfo_t*,void*);
        } 
        _sigaction_handler;
    #define sa_handler __sigaction_handler.sa_handler
    #define sa_sigaction__sigaction_handler.sa_sigaction
    #else
    	_sighandler_t sa_handler;
    #endif
        _sigset_t sa_mask;
        int sa_flags;
        void(*sa_restorer)(void);
};
```

`sigaction`的**`sa_hander`**成员指定**信号处理函数**。**`sa_mask`**成员**设置进程的信号掩码**（确切地说是在进程原有信号掩码的基础上增加信号掩码），以指定哪些信号不能发送给本进程

#### 挂起信号

设置进程信号掩码后，被屏蔽的信号将不能被进程接收。如果给进程发送一个被屏蔽的信号，则操作系统将该信号设置为进程的一个被挂
起的信号。如果我们取消对被挂起信号的屏蔽，则它能立即被进程接收到。

#### 信号集

在 **PCB** 中有两个非常重要的信号集。一个称之为 “**阻塞信号集** ”，另一个称之为“**未决信号集**“ 。这两个信号集都是内核使用位图机制来实现的。

信号的 “未决 是一种状态，指的是从信号的产生到信号被处理前的这一段时间。

信号的 “阻塞 是一个开关动作，指的是阻止信号被处理。

```c
// 信号集相关的处理函数

// 清空信号集
int sigemptyset(sigset_t *set);
// 所有信号标志位设为1
int sigfillset(sigset_t *set);
// 添加signum信号
int sigaddset(sigset_t *set, int signum);
// 删除signum信号
int sigdelset(sigset_t *set, int signum);
// 判断signum是否是阻塞信号
int sigismember(const sigset_t *set, int signum);
// 将用户区的信号集set设置到内核中的阻塞信号集中
int sigprocmask(int how, const sigset_t *set, sigset_t *oldset);

int sigpending(sigset_t *set);
```

#### 信号捕捉过程

<img src="C:\Users\zhuang\AppData\Roaming\Typora\typora-user-images\image-20220823095643667.png" alt="image-20220823095643667" style="zoom: 67%;" />



#### 优点

- 异步通信机制

  

### 6. socket套接字

# 线程

## 线程创建

**线程之间共享的资源**：

- 共享
  - 进程ID和父进程ID
  - 文件描述表
- 非共享资源
  - 线程ID
  - 线程特有的数据


**pthread**

```c
#include <pthread.h>
int pthread_create(pthread_t *thread, const pthread_attr_t *attr, void *(*start_routine) (void *), void *arg);
```

- 作用: 创建一个线程
- 参数: 
  - *thread: 传出参数， 创建线程的tid
  - *attr: 选择创建线程的属性，NULL表示默认属性
  - *start_routine: 子线程执行的入口函数地址
  - *arg: 子线程执行的入口函数的参数
- 返回值：
  - 成功: 0
  - 失败: errnum
  - 获取错误: char* strerror(int errnum)
  
## 线程终止

```c
#include <pthread.h>

void pthread_exit(void *retval);
```
跟子线程中`return`一样

## 线程分离

```c
#include <pthread.h>
int pthread_detach(pthread_t thread);
```

- 功能: 分离一个线程，该线程资源由系统自动回收。
- 注意点: 不可以重复分离一个线程，不可以连接已经分离的线程

**线程运行结束后，系统自动回收资源，解决了僵尸线程。但是一旦主线程退出则进程退出，子线程可能也不会运行**

## 线程取消

```c
int pthread_cancel(pthread_t thread);
```

- 功能：取消一个线程

## 线程join

```c++
int pthread_join()
```

- 功能：等待线程执行结束，回收线程资源

**解决了僵尸线程问题，但是主线程会阻塞**

## 线程属性

```c
int pthread_attr_init(pthread_attr_t *attr);
int pthread_attr_destroy(pthread_attr_t *attr);
....
```

- `man pthread_attr_` 查看线程属性

## 互斥锁

```c
pthread_mutex_t=xx

...

```

## 读写锁

```c
pthread_rwlock_xx
```

## 条件变量

```c
pthread_cond_xxx
```
不是锁，满足条件的线程唤醒，不满足则阻塞

## 信号量

```c
sem_xxx
```

## 总结

- 线程创建`int pthread_create(pthread_t *thread, const pthread_attr_t *attr, void *(*start_routine) (void *), void *arg);`
- 线程连接
- 线程分离
- 线程取消
- 线程属性
- 锁
  - 互斥锁
  - 读写锁
  - 信号量
  - 条件变量

# 网络编程

## socket

> 在Linux环境下，`socket`用来表示进程间网络通信的特殊文件类型（伪文件）。
>
> socket是内核当中的缓存区，有发送和接受缓存区，用套接字标识

```c
主机A——socket
  ip:xxx, port:xxx
  //内核
  读缓存区
  写缓存区

主机B——socket
  ip:xxx, port:xxx
  //内核
  读缓存区
  写缓存区
```

**Linux系统将其封装成文件的目的是为了统一接口，使得读写套接字与读写文件一致**

## socket地址

### 主机字节序与网络字节序

现代PC大多采用小端字节序，因此小端字节序又被称为主机字节序。为了不同主机能过够正确地接收数据，

> `约定`**网络字节序总是大端字节序** 

```c
//从主机字节序到网络字节序
uint16_t htons(uint16_t hostshort); // host to network short
uint16_t htonl(uint16_t hostlong); // host to network long 长整型（long类型）的网络字节序转化为网络字节序


//从网络字节序转换到主机字节序
uint16_t ntohs(uint16_t netshort);
uint16_t ntohl(uint16_t netlong);

```

### 通用socket地址

socket网络编程接口中表示socket地址的是结构体`sockadrr`

```c
#include <bits/socket.h>
struct sockaddr{
  sa_family_t sa_family;
  char sa_data[14];
};
typedef unsigned short int sa_family_t;
```

sa_family成员是地址族类型`sa_family_t`的变量。地址族类型同样与协议族类型对应

| 协议族      | 地址族    | 描述 |        地址值含义和长度|
| ----------- | ---------|------| --------------------|
| PF_UNIX     | AF_UNIX  | UNIX本地域协议族| 文件的路径名，长度可达108字节|
| PF_INET     | AF_INET  | TCP/IPV4协议族|16bit端口号，32bit IPV4地址，共6字节 |
| PF_INET6    | AF_INET6 | TCP/IPV6协议族|16bit端口号，32bit 流标识，128bit IPV6，32bit范围ID, 共26字节|

宏PF\_\*和AF\_*都定义在bits/socket.h头文件中

### 新通用socket地址

`sockaddr`只能支持到IPV4地址，`sockaddr_storage`能够支持IPV6，而且是内存对齐的

```c
#include <bits/socket.h>
struct sockaddr_storage{
  sa_family_t sa_family;
  unsigned long int __ss_align;
  char __ss_padding[128 - sizeof(__ss_align)];
};
typedef unsigned short int sa_family_t;
```

### 专用socket地址

为了向以前兼容，`sockaddr`退化成了(void*)的作用， 传递一个地址给函数， 至于这个函数是`sockaddr_in`还是`sockaddr_in6`,由地址族决定，然后函数内部再强制类型转化为所需要的地址类型。

![](../iamges/../images/20220323105416.png)


### IP地址转换函数

编程的过程中需要的ip地址是二进制的形式，而记录日志时我们需要的是点分十进制形式

```c
#include <arpa/inet.h>
//从点分十进制转换为二进制
int inet_pton(int af, const char *src, void* dst);
//从二进制转换为点分十进制
char* inet_ntop(int af, const char *src, char *dst, socklen_t size);
```

**`inet_pton`函数**将用字符串表示的IP地址src转换成用网络字节序整数表示的IP地址，并把转换结果存储于dst指向的内存中。其中，af参数指定地址族，可以是AF_INET或者AF_INET6。

**`inet_ntop`函数**进行相反的转换，前三个参数的含义与inet_pton的参数相同，最后一个参数size指定目标存储单元的大小。下面的两个宏能
帮助我们指定这个大小（分别用于IPv4和IPv6）：

```c
#include＜netinet/in.h＞
#define INET_ADDRSTRLEN 16
#define INET6_ADDRSTRLEN 46
```

## TCP通信流程

![](../images/20220323134447.png)
**服务端通信流程**：

- 创建一个监听的套接字
	- 套接字 : Linux特殊的文件
- 套接字绑定 IP 和 端口
- 设置监听
- 阻塞等待，当客户端发起连接时，解除阻塞，接收客户端的连接，得到一个和客户端通信的套接字
- 通信
	- 发送数据
	- 接收数据
- 通信结束，断开连接

**客户端通信流程**：

- 创建一个连接的套接字
- 连接服务器，需要指定服务器的 IP 和端口
- 通信
- 通信结束，断开连接

### 创建socket

> 自Linux内核版本2.6.17起，type参数可以接受与下面两个重要的标志相与的值：SOCK_NONBLOCK和SOCK_CLOEXEC。
>
> SOCK_NONBLOCK：将新创建的socket设为非阻塞的。
>
> SOCK_CLOEXEC：用fork调用创建子进程时在子进程中关闭该socket。

```c
#include <sys/types.h>          /* See NOTES */
#include <sys/socket.h>
int socket(int domain, int type, int protocol);
```

- 作用：创建一个用于通信的套接字文件
- 参数：
  - domain：指定通信使用的协议族，如`AF_INET6`(IPV6)、`AF_INET`(IPV4)
  - type: 指定通信协议，`SOCK_STREAM`、`SOCK_DGRAM`等
  - protocol : 一般为0
- 返回值：
  - 成功：返回套接字文件的fd
  - 失败：-1

### 命名socket

```c
#include <sys/types.h>          /* See NOTES */
#include <sys/socket.h>

int bind(int sockfd, const struct sockaddr *addr, socklen_t addrlen);
```

- 作用：socket命名
- 参数：
  - sockfd：需要被绑定的socket文件描述符
  - addr：绑定地址
  - addrlen：socket地址的长度
- 返回值：0，-1

### 监听socket

> 内核版本2.2之后，backlog表示处于ESATABLISHED状态的socket的上限

`listen`系统调用来创建一个监听队列以存放待处理的客户连接:

```c++
#include<sys/socket.h>
int listen(int socketfd, int backlog);
```

参数：

- socketfd：指定被监听的socket
- backlog：指定内核监听的最大长度

### 接收连接

> `accept阻塞函数`只是从监听队列中取出连接，而不论连接处于何种状态

accpet系统调用从listen监听队列中接受一个连接:

```c++
#include＜sys/types.h＞
#include＜sys/socket.h＞
int accept(int sockfd,struct sockaddr*addr,socklen_t*addrlen);
```

参数：

- sockfd:监听socket的文件描述符
- addr和addrlen：用来获取被接受连接的远端socket地址

返回值:

- 成功：新的连接socket
- 失败：-1

### 发起连接

客户端需要通过connect系统调用来主动与服务器建立连接：

```c++
#include＜sys/types.h＞
#include＜sys/socket.h＞
int connect(int sockfd,const struct sockaddr*serv_addr,socklen_t addrlen);
```

参数：

- sockfd:socket文件描述符，用于连接
- serv_addr和addrlen：连接的服务器socket地址

返回值:

- 成功：0，sockfd就唯一标识了这个连接
- 失败：-1

### 关闭连接

关闭一个连接实际上就是关闭该连接对应的socket，通过close或shutdown关闭普通文件描述符的系统调用来完成：

```c++
#include＜unistd.h＞
int close(int fd);

#include＜sys/socket.h＞
int shutdown(int sockfd,int howto);// 立即关闭连接
```

### 数据读写

#### TCP数据读写

对文件的读写操作read和write同样适用于socket。但是socket的读写接口增加了对数据读写的控制。

```c++
#include＜sys/types.h＞
#include＜sys/socket.h＞
ssize_t recv(int sockfd,void*buf,size_t len,int flags);
ssize_t send(int sockfd,const void*buf,size_t len,int flags);
```

#### UDP数据读写

```c++
#include＜sys/types.h＞
#include＜sys/socket.h＞
ssize_t recvfrom(int sockfd,void*buf,size_t len,int flags,struct sockaddr*src_addr,socklen_t*addrlen);
ssize_t sendto(int sockfd,const void*buf,size_t len,int flags,const struct sockaddr*dest_addr,socklen_t addrlen);
```

因为UDP通信没有连接的概念，所以我们每次读取数据都需要获取发送端的socket地址

#### 通用数据读写

```c++
#include＜sys/socket.h＞
ssize_t recvmsg(int sockfd,struct msghdr*msg,int flags);
ssize_t sendmsg(int sockfd,struct msghdr*msg,int flags);
```



## 多线程实现并发服务器

## I/O多路复用

I/O多路复用使程序能够同时监听多个文件描述符，能够提高程序的性能，Linux下实现I/O多路复用的系统调用主要有`select`、`poll`和`epoll`

### 几种常见的I/O模型

> 一次I/O经历**数据准备**和**数据读写**

#### **1. 阻塞等待.BIO**

比如`receive`、`accept`函数

##### 基本概念

比如，使用`read`系统调用，进程会被阻塞，直到**数据准备好**，内核将数据从内核区拷贝到用户区

##### 优点

- 不占用CPU资源

##### 缺点

- 同一时刻只能处理一个操作，效率低 **(多线程解决，每个线程负责监听一个套接字)**
- 有一定的**线程切换**开销

##### 使用样例

```c
//服务端
int lfd = socket(...);
bind(lfd, ...);
listen(ldf, ...);
int cfd = accept(lfd, ...);//阻塞
receive(cfd,...)//阻塞
```

#### **2. 非阻塞，忙轮询.NIO**

##### 基本概念

**数据未准备好**时不阻塞线程，而是直接返回-1，`errno`会被设置为EWOULDBLOCK或EAGAIN。

轮询重试，直到**数据准备好**时，内核将数据从内核区拷贝到用户区

##### 优点

- 没有额外线程切换的开销，适用于频繁的网络I/O

##### 缺点

- 同一时刻只能处理一个操作

- 需要占用更多的CPU资源

##### 使用样例

```c
//服务端
int lfd = socket(...);
bind(lfd, ...);
listen(ldf, ...);
int cfd = accept(lfd, ...);//非阻塞
if has client connect:
  read(cfd,...)//非阻塞
  if read data:
    print data
  else:
    do othres
else:
  do others
```

#### **3. IO复用**

> 由一个线程监听事件的发生，再将所有发生的事件交给主线程处理

##### 基本概念

由内核检测是否有数据准备好。

如`select`系统调用，内核以轮询的方式遍历监听socket集合，只要有socket的数据准备好，就记录下准备好的socket，最后返回用户态。

##### 优点

- 一次可以处理多个操作

##### 缺点



##### 使用样例



第一种:`select/poll`

select/poll 委托内核检测是否有数据到达。只返回几个数据到达，不会返回哪几个到达

第二种:`epoll`
epoll 委托内核检测是否有数据到达。返回几个数据到达，并返回哪几个到达了



#### **4. 信号驱动IO**

##### 基本概念

信号驱动IO不再用主动询问的方式去确认数据是否就绪，而是向内核注册SIGIO的信号处理程序(调用`sigaction`的时候建立一个SIGIO的信号)，然后应用用户进程可以去做别的事，不用阻塞。当内核数据准备好后，再通过`SIGIO`信号通知应用进程，数据准备好后的可读状态。应用用户进程收到信号之后，立即调用recvfrom，去读取数据。	

#### **5. AIO**





### select

> 内核检测事件发生是通过轮询的方式

> 主旨思想：
> -  首先构造一个关于文件描述符的列表，将要监听的文件描述符添加列表中
> - 调用`select`系统函数，监听列表中的文件描述符，直到这些描述符中的一个或者多个进行I/O操作时，函数才返回
>   - **select函数是阻塞**，也可以设置超时
>   - 函数对于文件描述符的检测操作是由内核完成
> - 在返回时，会告知进程有多少描述符进行了I/O

```c
#include <sys/time.h>
#include <sys/types.h>
#include <unistd.h>

int select(int nfds, fd_set *readfds, fd_set *writefds, fd_set *exceptfds, struct timeval *timeout);
/*
  参数：
      nfd: 最大的文件描述符号+1
      readfds: 要检测的读文件描述集合
      writefds: 要检测的写文件描述集合
      excepetfds: 检测发送异常的文件描述集合
      timeout: 
              NULL： 永久阻塞，直到检测到I/O有变化
              
  返回值:
        - 1: 失败
        > 0： 检测到的文件描述符数量

*/

//清空fd在集合中的标志
void FD_CLR(int fd, fd_set *set);
//返回fd在集合中的标志
int  FD_ISSET(int fd, fd_set *set);
//设置fd在集合中的标志
void FD_SET(int fd, fd_set *set);
//清空集合中所有的标志
void FD_ZERO(fd_set *set);

```

**缺点**：

- 每次调用select，都需要将**fd集合从用户态拷贝到内核态，再拷贝回来**，需要一定开销
- select**只支持1024个文件描述符数量**
- fds集合不能重用，每次都需要重置

### poll

```c
 #include <poll.h>
struct pollfd {
    int   fd;         /* file descriptor */
    short events;     /* requested events */
    short revents;    /* returned events */

    //POLLIN检测读
    //POLLOUT检测写
};
int poll(struct pollfd *fds, nfds_t nfds, int timeout);
/*
    参数: 
        fds:要检测的文件描述符结构体数组
        nfds: 数组大小
        timeout: 
              0: 不阻塞
              - 1： 阻塞
              ...
    返回值:
        - 1: 失败
        > 0 : 检测到的文件描述符数量
*/
```

`poll`改进了`select`的第三和第四个缺点

### epoll

#### 基本原理

##### file_operations

Linux对文件的操作做了很高层的抽象，它并不知道每个文件应该如何打开、读写，Linux让每种设备类型自己实现`struct file_operations`结构体中定义的函数

```c
struct file_operations{
    loff_t (*llseek) (struct file *, loff_t, int);
    ssize_t (*read)(struct file*, char _user*, size_t, loff_t *);
    ssize_t (*write)(struct file*, const char __user*, size_t, loff_t *);
    unsigned int (*poll) (struct file*, struct poll_table_struct *);
}
```

`poll`主要做两件事：

- 将当前线程加入到设备驱动的等待队列，并设置回调函数。这样设备上有事件发生时才知道唤醒通知哪些线程，调用这些线程的什么方法
- 检查此刻已经发生的事件，POLLIN、POLLOUT、POLLERR等，以掩码的形式返回

##### 等待队列

等待队列的功能时将进程（`task_struct`）加入设备的等待队列，等超时或有事件发生时，唤醒进程并回调特定回调钩子函数。

当执行`epoll_wait`时，实际上就是将当前进程加入到设备fd的等待队列上

<img src="..\images\image-20220827112646124.png" alt="image-20220827112646124" style="zoom: 50%;" />

##### epoll工作流程

<img src="..\images\vBY6AT7enLNC4IA.png" alt="img" style="zoom: 67%;" />

`epoll_create`在内核创建一个`eventepoll`对象，包含一个等待队列，一个就绪的文件列表，一个红黑树（管理监听的文件事件）。

`epoll_ctl`在红黑树插入/删除`epitem`(监听的文件描述符及事件)，并调用socket文件的`poll`函数，将`epitem`添加到对应的等待队列并设置回调函数。

`epoll_wait`如果就绪文件列表中没有数据就会阻塞，否则就返回。当文件描述符的事件发生后，内核将调用对应等待队列中的回调函数，这个回调函数会将`epitem`加入到就绪文件描述符列表中

##### epoll

想要支持`epoll`等函数，文件类型就必须要实现`file_operations`函数列表里的`poll`

`poll`函数的作用是：

- 将当前线程加入到设备驱动的等待队列，并设置回调函数。这样设备上有事件发生时才知道唤醒通知哪些线程，调用这些线程的什么方法
- 检查此刻已经发生的事件,POLLIN、POLLOUT、POLLERR等，以掩码的形式返回

#### 使用样例

1. **零拷贝**：`epoll_create`系统调用在内核区创建一个eventpoll（结构体）， 避免了select/poll将fds从用户区拷贝内核区的开销
2. `epoll_ctl`注册需要检测（监听）的事件（对文件描述符读/写的监听）
3. `epoll_wait`系统内核进行检测(监听)，将发送的事件记录在rdlist中，并将其拷贝回用户区返回结果

```c
#include <sys/epoll.h>
int epoll_create(int size);
/*
    参数: 
        size: 在内核区创建epoll实例，包含需要监听的事件(红黑树)， 检测结果（双向链表）
    返回值：
        -1: 失败
        > 0: fd, 一个用于epoll操作的文件描述符
*/
 typedef union epoll_data {
               void    *ptr;
               int      fd;
               uint32_t u32;
               uint64_t u64;
} epoll_data_t;
 struct epoll_event {
               uint32_t     events;    /* Epoll events */
               epoll_data_t data;      /* User data variable */
};
int epoll_ctl(int epfd, int op, int fd, struct epoll_event *event);
/*
    参数:
        epfd: 指明需要操作的fd
        op: 对epoll实例中的红黑树进行增加、删除、修改等。
        fd: 需要op操作的文件描述符
        event: 需要操作的文件描述符的事件（读、写、。。。）
*/
int epoll_wait(int epfd, struct epoll_event *events, int maxevents, int timeout);
/*
    参数：
        events: 返回所有发生的事件，数组
*/
```

#### LT.水平触发

简单来说就是一种高效的`poll`,`epoll`的默认工作模式是`LT`

当`epoll_wait`检测到其上有事件发生并将此事件通知应用程序后，应用程序可以不立即处理该事件。当应用程序下一次调用`epoll_wait`时，`epoll_wait`还会再次向应用
程序通告此事件，直到该事件被处理。

#### ET.边缘触发

当`epoll_wait`通知的后，应用程序必须要处理，因为后续调用`epoll_wait`将不再通知该事件

> 每个使用ET模式的文件描述符都应该是**非阻塞的**。如果文件描述符是阻塞的，那么读或写操作将会因为没有后续的事件而一直处
> 于阻塞状态（饥渴状态）。

**ET模式要求应用程序一次性读取所有数据**，或者一次性写完所有数据。经典`EPOLLIN`的如下:

```c++
while (true) {
    int len = recv(...);
    if (len < 0) {
        if (errno == EAGAIN || errnor = EWOULDBLOCK){
            break;
        }
        close(fd);
    }
}
```

如果文件文件描述符是**阻塞**的，读/写进程在读/写完后将一直阻塞于`recv`/`send`函数。

#### EPOLLONESHOT事件

> 即使使用ET模式，一个socket上的某个事件还是可能被触发多次。这在并发程序中就会引起一个问题。比如一个线程在读取完某个socket上的数据后开始处理这些数据，而在数据的处理过程中该socket上又有新数据可读（EPOLLIN再次被触发），此时另外一个线程被唤醒来读取这些新的数据。于是就出现了两个线程同时操作一个socket的局面

对于注册了EPOLLONESHOT事件的文件描述符，操作系统最多触发其上注册的一个可读、可写或者异常事件，且只触发一次，除非**使用`epoll_ctl`函数重置该文件描述符上注册的EPOLLONESHOT事件**。

# 高性能服务器

## 阻塞和非阻塞、同步与异步

> 典型的一次IO的两个阶段是**数据准备**和**数据读写（从内核缓存区到用户缓存区的拷贝或从用户缓存区到内核缓存区的拷贝）**

**数据准备阶段**

- 阻塞：调用IO方法的线程进入阻塞状态
- 非阻塞：不会改变线程状态，通过返回值判断

**数据读写阶段**

- 同步：A向B请求调用一个网络IO接口时，数据读写都是A自己完成
- 异步: A向B请求调用一个网络IO接口时，向B传入请求的事件以及通知方式，A可以继续做其他事情，当B监听到事件处理完成后，用通知方式通知A处理结果。

## 服务器编程基本框架

<img src="..\images\20220615194449.png" style="zoom: 50%;" />

**I/O处理单元: **处理客户端连接，接收发送网络数据

**逻辑单元:** 业务线程

**网络存储单元:** 数据库、文件或缓存

**请求队列:** 单元间的通信方式

I/O处理单元是服务器管理客户端连接的模块，主要负责等待和接收新的客户端连接，接收客户端数据，将服务端的处理结果返回给客户端。但是数据的收发也可能在逻辑单元进行，具体看哪种事件处理模式。

一个逻辑单元通常是一个线程。它负责处理客户端请求数据，并将结果交给I/O处理单元。也可以直接交付给客户端，具体看哪种事件处理模式。

请求队列通常被实现为池的部分

### 两种高效的事件处理模式

服务器通常需要处理三个事件:I/O事件、信号及定时事件

#### Reactor模式（重点）

##### 单Reactor多线程模型

主线程（I/O处理单元）只负责监听文件描述符上是否有事件发生，将事件的任务添加到请求队列并通知子线程处理事件。数据的读写，接收新的连接以及处理请求均由工作线程独立完成。

<img src="..\images\image-20220615202556392.png" alt="image-20220615202556392" style="zoom:50%;" />

**使用同步I/O模型实现Reactor模式的工作流程：**

1. 主线程往epoll内核注册socket上的读就绪事件
2. 主线程调用epoll_wait等待事件发生
3. 当读就绪事件发生时，epoll_wait通知主线程将可读事件插入请求队列
4. “可读事件”请求队列上的线程被唤醒，从socket读取数据并处理请求，往epoll内核注册socket上的写就绪事件
5. 主线程调用epoll_wait等待可写事件发生
6. 当事件发生时，epoll_wait通知主线程。主线程将socket“可写事件”插入请求队列
7. “可写事件”请求队列上的线程被唤醒，往socket上写入服务器处理客户请求的结果。

###### 优点

- **单Reator多线程（本项目使用的方案）**的方案优势在于**能够充分利用多核 CPU 的性能**

#### proactor模式

> 异步网络模式，感知已完成的读写事件

与Reactor模式不同，Proactor模式将所有I/O操作都交给**主线程**和**内核**来处理，工作线程仅仅负责业务逻辑

![image-20220708133352230](..\images\image-20220708133352230.png)

**使用异步I/O模型（以aio_read和aio_write为例）实现的Proactor模式的工作流程是：**

1. 主线程调用aio_read函数向内核注册socket上的读完成事件，并告诉内核用户读缓冲区的位置，以及读操作完成时如何通知应用程序
2. 主线程继续处理其他逻辑。
3. 当socket上的数据被读入用户缓冲区后，内核将向应用程序发送一个信号，以通知应用程序数据已经可用。
4. 应用程序预先定义好的信号处理函数选择一个工作线程来处理客户请求。工作线程处理完客户请求之后，调用aio_write函数向内核注册socket上的写完成事件，并告诉内核用户写缓冲区的位置，以及写操作完成时如何通知应用程序。
5. 主线程继续处理其他逻辑。
6. 当用户缓冲区的数据被写入socket之后，内核将向应用程序发送一个信号，以通知应用程序数据已经发送完毕
7. 应用程序预先定义好的信号处理函数选择一个工作线程来做善
   后处理，比如决定是否关闭socket。

#### 模拟proactor模式（了解）

其原理是：主线程执行数据读写操作，读写完成之后，主线程向工作线程通知这一“完成事件”。那么从工作线程的角度来看，它们就直接获得了数据读写的结果，接下来要做的只是对读写的结果进行逻辑处理。



## 日志系统

`rsyslogd`守护进程既能接收用户进程输出的日志，又能接收内核日志。用户进程是通过调用syslog函数生成系统日志的。该函数将日志输出到一个UNIX本地域socket类型（AF_UNIX）的文件/dev/log中， rsyslogd则监听该文件以获取用户进程的输出。



`syslog`函数

```c++
#include＜syslog.h＞
void syslog(int priority,const char*message,...);

// 优先级选项
#include＜syslog.h＞
#define LOG_EMERG 0/*系统不可用*/
#define LOG_ALERT 1/*报警，需要立即采取动作*/
#define LOG_CRIT 2/*非常严重的情况*/
#define LOG_ERR 3/*错误*/
#define LOG_WARNING 4/*警告*/
#define LOG_NOTICE 5/*通知*/
#define LOG_INFO 6/*信息*/
#define LOG_DEBUG 7/*调试*/

```

此外，**日志的过滤也很重要**。程序在开发阶段可能需要输出很多调 试信息，而发布之后我们又需要将这些调试信息关闭。解决这个问题的 方法并不是在程序发布之后删除调试代码（因为日后可能还需要用 到），而是简单地设置日志掩码，使日志级别大于日志掩码的日志信息 被系统忽略。下面这个函数用于设置syslog的日志掩码：

```c++
#include＜syslog.h＞
int setlogmask(int maskpri);
```

### **Where：不清楚在何处打印日志**

  **1.程序入口**:在入口打印日志是因为这个时候传递进来的参数没有经过任何处理，将它打印在日志文件中能一眼就知道程序的原始数据是否符合我们的预期，是不是传递进来的原始数据就出现 的问题。

　**2.异常捕获**

　**3.重要信息**

### **Who：不清楚打印什么级别的日志**

**INFO级别的日志应该是能帮助测试人员判断这是否是一个真正的bug，而不是自己操作失误造成的。**

**DEBUG级别的日志应该是能帮助开发人员分析定位bug所在的位置。**

**ERROR级别的日志最为常见的就是捕获异常时所打印的日志。**

### **What：不清楚日志应该包含什么内容**

打印的内容一定要从实际出发。也就是说如果在实际的生产环境中，你的用户量很大，日志在不停地刷新，如何定位某个用户的整个登录以及后续的操作呢？当然就是根据用户名来跟踪。所以打印内容的第一要素就是要能便于定位；定位过后也许用户在好几个模板中进行操作，还是定位，这个时候定的是模块的位；还有一点当然就是用户操作时的具体参数；最后一点就是用户干了什么。

　　总结就是，[id, module, params, content]（关键字，模块，参数，内容）

## 定时器

定时事件，比如定期检测一个客户连接的活动状态。

将每个定时事件分别封装成定时器，并使用某种容器类数据结构，比如链表、排序链表和时间轮，将所有定时器串联起来，以实现对定时事件的统一管理。

### 定时方法

**1.socket选项`SO_RCVTIMEO`和`SO_SNDTIMEO`**

分别用来设置socket**接收数据超时时间**和**发送数据超时时间**。因此，这两个选项仅对与数据接收和发送相关的socket专用系统调用
有效，这些系统调用包括send、sendmsg、recv、recvmsg、accept和connect。

```c
int ret = setsockopt(sockfd, SOL_SOCKET, SO_SNDTIMEO, ＆timeout, len); // 设置sockfd的发送数据超时时间
```

**`2.SIGALRM信号`**

由`alarm`和`setitimer`函数设置的实时闹钟一旦超时，将触发SIGALRM信号。利用该信号的信号处理函数来处理定时任务。

alarm函数设置一个alarm时钟发送信号，一次alarm调用触发一次SIGALRM信号

```c
#include <unistd.h>
unsigned int alarm(unsigned int seconds);

/*
参数：
	seconds： 参数是0就取消alarm
*/
```

**注册alrm信号处理函数**

```c++
// 添加信号捕捉
void add_sig(int sig, void(handler)(int)) {
    struct sigaction sa;
    memset(&sa, 0, sizeof (sa));
    sa.sa_handler = handler;
    sigfillset(&sa.sa_mask);
    sigaction(sig, &sa, NULL);
}
```

**时间堆**

```c++
class timer{
    
    void add_timer(int,...) {
        // 如果堆中没有该节点，则加入
        // 如果堆中已经有这个节点，则更新过期时间，向下调整。
    }
    void tick(){ // alarm信号处理函数中放置的函数
        ...
        // 从堆中取出最早过期时间，执行它的回调函数（释放连接）
        ...
    }
};
```



**3.I/O复用系统调用的超时参数**

Linux下的3组I/O复用系统调用都带有超时参数，因此它们不仅能统一处理信号和I/O事件，也能统一处理定时事件。

## 无锁编程

### C++内存模型

> 通过保证操作顺序，间接保证了可见性

| `memory_order_relaxed` | 宽松操作：没有同步或顺序制约，仅对此操作要求原子性。         |
| ---------------------- | ------------------------------------------------------------ |
| `memory_order_consume` | 有此内存顺序的加载操作，在其影响的内存位置进行*消费操作*：当前线程中依赖于当前加载的该值的读或写不能被重排到此加载前。其他释放同一原子变量的线程的对数据依赖变量的写入，为当前线程所可见。 |
| `memory_order_acquire` | 有此内存顺序的加载操作，在其影响的内存位置进行*获得操作*：当前线程中读或写不能被重排到此加载前。其他释放同一原子变量的线程的所有写入 |
| `memory_order_release` | 有此内存顺序的存储操作进行*释放操作*：当前线程中的读或写不能被重排到此存储后。当前线程的所有写入，可见于获得该同一原子变量的其他线程（见下方[释放获得顺序](https://zh.cppreference.com/w/cpp/atomic/memory_order#.E9.87.8A.E6.94.BE.E8.8E.B7.E5.BE.97.E9.A1.BA.E5.BA.8F)），并且对该原子变量的带依赖写入变得对于其他消费同一原子对象的线程可见（见下方[释放消费顺序](https://zh.cppreference.com/w/cpp/atomic/memory_order#.E9.87.8A.E6.94.BE.E6.B6.88.E8.B4.B9.E9.A1.BA.E5.BA.8F)）。 |
| `memory_order_acq_rel` | 带此内存顺序的读修改写操作既是*获得操作*又是*释放操作*。当前线程的读或写内存不能被重排到此存储前或后。所有释放同一原子变量的线程的写入可见于修改之前，而且修改可见于其他获得同一原子变量的线程。 |
| `memory_order_seq_cst` | 有此内存顺序的加载操作进行*获得操作*，存储操作进行*释放操作*，而读修改写操作进行*获得操作*和*释放操作*，再加上存在一个单独全序，其中所有线程以同一顺序观测到所有修改 |

<img src="..\images\image-20220901140706228.png" alt="image-20220901140706228" style="zoom: 50%;" />

### 无锁的同步队列

**如何实现一个“多进多出的无锁队列”？**

- 读线程和读线程之间使用`atomic<int> hh`避免冲突
- 写线程和写线程之间使用`atomic<int> tt`避免冲突
- 读线程和写线程使用`atomic<bool> full`避免冲突，每个元素都有一个`atomic<bool> full`，full是false不能读。full是true不能写

**编译器优化：**

- 有依赖关系的读写操作，编译器优化后依旧保证其顺序

`aotmic`底层应该由`CAS`实现，可以使用`compare_exchange_weak`尝试修改值。`CAS`是乐观锁的另一种实现技术，当多个线程尝试使用`CAS`同时更新同一个变量时，只有其中一个线程能更新变量的值，而其它线程都失败，失败的线程并不会被挂起，而是被告知这次竞争中失败，并可以再次尝试。



**`compare_exchange_weak`与`compare_exchange_strong`区别：**

weak性能与strong要高，所以weak和strong的区别在于，weak仍然在`*ptr == expected`的时候，执行依然会有小概率失败。也就是说， 即使`*ptr == expected`，此时也不会发生值的设置，也会返回`false`。（不会是设置成功了，但是返回的是false）。



**无锁队列的`push`实现：**

```c++
bool put(const T &value) {
        size_t _tt = 0;
        do {
            _tt = tt.load(std::memory_order_relaxed);
            if ((_tt + 1) >= hh.load(std::memory_order_relaxed)) {
                return false;
            }
            if (_data[_tt].full.load(std::memory_order_relaxed)){
                return false;
            }
        }
        while (!tt.compare_exchange_weak(_tt, _tt + 1, std::memory_order_release, std::memory_order_relaxed));

        _data[_tt].data = std::move(value);
        _data[_tt].full.store(true, std::memory_order_release);
        return false;
}
```

**无锁队列的`take`实现：**

```c++
bool take(T &value) {
        size_t _hh = 0;
        do {
            _hh = hh.load(std::memory_order_relaxed);
            if (_hh == tt.load(std::memory_order_relaxed)) {
                return false;
            }
            if (!_data[_hh].full.load(std::memory_order_relaxed)){
                return false;
            }
        }
        while (!tt.compare_exchange_weak(_hh, _hh + 1, std::memory_order_release, std::memory_order_relaxed));
        
        value = std::move(_data[_hh].data);
        _data[_hh].full.store(false, std::memory_order_release);
        return false;
}
```



## 服务压力测试

### 修改linux系统用户资源限制

ulimit：显示（或设置）用户可以使用的资源的限制（limit），这限制分为软限制（当前限制）和硬限制（上限），其中硬限制是软限制的上限值，应用程序在运行过程中使用的系统资源不超过相应的软限制，任何的超越都导致进程的终止。

### Webbench

Webbench 是 Linux 上一款知名的、优秀的 web 性能压力测试工具。它是由Lionbridge公司开发。

> 测试处在相同硬件上，不同服务的性能以及不同硬件上同一个服务的运行状况。
> 展示服务器的两项内容：每秒钟响应请求数和每秒钟传输数据量。

基本原理：Webbench 首先 fork 出多个子进程，每个子进程都循环做 web 访问测试。子进程把访问 的
结果通过pipe 告诉父进程，父进程做最终的统计结果。

**测试示例**

```c++
webbench -c 1000 -t 30 http://192.168.110.129:10000/index.html
参数：
-c 表示客户端数
-t 表示时间
```

### 压力测试结果

8CPU

|          | 10   | 1000  | 1e4    | 1e5           |
| :------- | ---- | ----- | ------ | ------------- |
| 阻塞队列 | 9513 | 9630  | 11.8   | 16.3 (超负荷) |
| 无锁队列 | 6861 | 11000 | 50.6   | 70.6 (超负荷) |
|          | -28% | 14.2% | 328.8% | 333.1%        |



# 软件测试

## 软件测试的目的

尽可能的发现错误

**软件测试应贯穿于软件定义与开发的整个期间**。需求分析、概要设计、详细设计以及程序编码等各阶段所得到的文档，包括需求规格说明、概要设计规格说明、详细设 计规格说明以及源程序，都应成为软件测试的对象

## 软件测试的方法

### 从是否关心软件内部结构和具体实现角度分类

#### 黑盒方法

将测试对象看作黑盒子，测试人员不需要考虑程序内部的逻辑结构和内部特性。黑盒测试又叫功能测试或者数据驱动测试。

**1.等价类**

测试的目的是进行**完备的测试**，**同时避免测试用例冗余**

**等价类是指**在该子集合中，各个输入数据对于找出程序bug的能力是等效的。

等价类分为**有效等价类**、**无效等价类**

> **等价类测试的关键，就是选择确定的等价关系，必须区分弱和强等价类**

**2.边界值**

> 从测试工作经验得知，**大量的错误是发生在输入或输出范围的边界上，而不是在输入范围的内部**

**3.决策表**

#### 白盒方法

将测试对象看作一个透明的盒子，允许测试人员利用程序内部逻辑设计测试用例。白盒测试又叫结构测试或逻辑驱动测试。

测试覆盖代码、分支、路径和条件



逻辑覆盖可分为：

**1.语句覆盖**

每条语句至少执行一次

**2.分支覆盖**

每条判定的分支至少执行一次

**3.条件覆盖**

分支判定语句中包含多个条件，每个条件至少执行一次

**4.分支条件覆盖**

选取足够多的测试数据，使判断中每个条件都取得各种可能值，并使每个判断表达式也取到各种可能的结果。

**5.条件组合覆盖**

使得每个判断中条件的各种可能组合都至少出现一次

**6.路径覆盖**

程序图/控制流图中的所有路径至少执行一次

### 从软件测试过程角度分类

#### 单元测试

白盒测试的一种，对软件设计中的单元模块进行测试

#### 集成测试

在单元测试的基础上，对单元模块之间的连接和组装进行测试。

#### 系统测试

应用整体运行时的测试阶段

#### 负载测试

通过逐步增加系统负载，最终确定在满足性能指标的情况下，系统能够承受的最大负载量的测试。

#### 强度测试

#### 容量测试

#### 压力测试

## 软件测试的步骤

开始是单元测试，集中对用源代码实现的每一个程序单元进行测试，检查各个程序模块是否正确地 实现了规定的功能。 

3) 集成测试把已测试过的模块组装起来，主要对与设计相关的软件体系结构的构造进行测试。 
4) **确认测试则是要检查已实现的软件是否满足了需求规格说明中确定了的各种需求**，以及软件配置是 否完全、正确

## 测试结束的标准

1、用例全部通过。2、覆盖率达到标准。3、缺陷率达到标准。4、其他指标达到标准





## Bug的缺陷的优先级和严重程度



### 缺陷的严重性

 **1 – 非常严重的缺陷**

软件的意外退出甚至操作系统崩溃，造成数据丢失。 

**2 – 较严重的缺陷**

例如，软件的某个菜单不起作用或者产生错误的结果； 

**3 - 软件一般缺陷**

例如，本地化软件的某些字符没有[翻译](http://www.haosou.com/s?q=翻译&ie=utf-8&src=wenda_link)或者翻译不准确； 

**4 - 软件界面的细微缺陷**

例如，某个控件没有对齐，某个[标点符号](http://www.haosou.com/s?q=标点符号&ie=utf-8&src=wenda_link)丢失等。

### 缺陷的优先性

**1 –最高优先级**

例如，软件的主要功能错误或者造成软件崩溃，数据丢失的缺陷。 

**2 – 较高优先级**

例如，影响软件功能和性能的一般缺陷； 

**3 -一般优先级**

例如，本地化软件的某些字符没有翻译或者翻译不准确的缺陷； 

**4 – 低优先级**

例如，对[软件的质量](http://www.haosou.com/s?q=软件的质量&ie=utf-8&src=wenda_link)影响非常轻微或出现几率很低的缺陷。

## 测试模型

### 瀑布模型

仅仅把测试过程作为编码之后的一个阶段，忽视了测试对需求分析,系统设计的验证，如果前面 设计错误，得一直到后期的验收测试才被发现，耗时耗力

### v模型

测试与开发同时进行，在 V 模型的基础上，增加了在开发阶段的同步测试。

需求分析阶段-设计系统测试策略（策略包括确定测试技术和工具、测试完成标准等）

概要设计-设计集成测试策略

详细设计-设计单元测试策略

编码-单元测试

集成-集成测试

实施-系统测试

交付-验收测试

### w模型

测试对象不仅仅是程序，还包括需求和设计



需求分析-需求测试

概要设计-概要设计测试

详细设计-详细设计测试

编码-单元测试

模块集成-集成测试

系统构建与实施-系统测试

交付运行-验收测试

## 测试分类

<img src="D:\Repositories\notes\images\image-20220913093321763.png" alt="image-20220913093321763" style="zoom: 67%;" />



## 什么是好的测试框架

gtest的好处:

- 测试应该是**独立的**和**可重复的**。gtest通过是每个测试运行在不同的对象中从而使测试隔离
- 测试应该有**良好的组织**以反映被测试代码的结构。gtest将相关的测试划分到一个测试组内，并且测试组内的测试能共享数据和子例程。
- 测试应该是**可移植**和**可复用**。
- 测试应该在测试失败时尽可能地提供关于问题的信息。gtest在第一次测试失败时不会停止。相反，它只能停止当前测试，并继续下一个测试。您还可以设置报告非致命故障的测试，然后进行当前测试。因此，您可以在单个编辑过程中检测和修复多个错误。
- 测试框架应该让测试人员不再需要编写那些琐碎的代码。gtest会自动跟踪所有定义的测试，并且不需要用户枚举它们以运行它们。
- 测试要求快速。使用gtest，可以在测试中重复使用共享资源，而不是让测试相互依赖。

# [GoogleTest](https://google.github.io/googletest/)



## Test Case

### [Assertions Reference](https://google.github.io/googletest/reference/assertions.html)

`ASSERT_*`断言失败时将立刻从当前结束，可能会跳过清理代码从而造成内存泄漏

`EXPECT_*`断言允许多个错误



所有的断言宏都支持使用`<<`符号自定义错误报告

```c++
EXPECT_TRUE(my_condition) << "My condition is not true";
```

#### 泛化断言

> 验证`value`是否匹配[matcher](https://google.github.io/googletest/reference/matchers.html)

```c++
EXPECT_THAT(value, matcher)
ASSERT_THAT(value, matcher)
```

**使用样例**

```c++
#include "gmock/gmock.h"

using ::testing::AllOf;
using ::testing::Gt;
using ::testing::Lt;
using ::testing::MatchesRegex;
using ::testing::StartsWith;

...
EXPECT_THAT(value1, StartsWith("Hello"));
EXPECT_THAT(value2, MatchesRegex("Line \\d+"));
ASSERT_THAT(value3, AllOf(Gt(5), Lt(10)));
```

### 布尔条件

#### EXPECT_TRUE

#### EXPECT_FALSE

### 二元对比

#### EXPECT_EQ

#### EXPECT_NE

#### EXPECT_LT

> 验证小于

#### EXPECT_GE

> 验证大于

### 字符串对比

#### EXPECT_STREQ

#### EXPECT_STRNE

#### EXPECT_STRCASEEQ

> 验证两个字符串是否具有相同的内容，忽略大小写

#### EXPECT_STRCASENE

### 浮点数对比

#### EXPECT_FLOAT_EQ

#### EXPECT_DOUBLE_EQ

#### EXPECT_NEAR

> 验证两个浮点数不超过误差值

```c++
EXPECT_NEAR(val1, val2, abs_error)
ASSERT_NEAR(val1, val2, abs_error)
```



### 异常断言

#### EXPECT_THROW

> 验证语句抛出了指定类型的异常

```c++
EXPECT_THROW(statement, expection_type)
ASSERT_THROW(statement, expection_type)
```

#### EXPECT_ANT_THROW

> 验证语句抛出了任意类型的异常

```c++
EXPECT_ANY_THROW(statement)
ASSERT_ANY_THROW(statement)
```

#### EXPECT_NO_THROW

### 谓词断言

#### EXPECT_PRED*

> 验证谓词为真

```c++
EXPECT_PRED1(pred,val1)
EXPECT_PRED2(pred,val1,val2)
EXPECT_PRED3(pred,val1,val2,val3)
EXPECT_PRED4(pred,val1,val2,val3,val4)
EXPECT_PRED5(pred,val1,val2,val3,val4,val5)

ASSERT_PRED1(pred,val1)
ASSERT_PRED2(pred,val1,val2)
ASSERT_PRED3(pred,val1,val2,val3)
ASSERT_PRED4(pred,val1,val2,val3,val4)
ASSERT_PRED5(pred,val1,val2,val3,val4,val5)
```

`pred`是一个**函数**或者是**仿函数**且返回值是`bool`类型

**使用样例**

```c++
// Returns true if m and n have no common divisors except 1.
bool MutuallyPrime(int m, int n) { ... }
...
const int a = 3;
const int b = 4;
const int c = 10;
...
EXPECT_PRED2(MutuallyPrime, a, b);  // Succeeds
EXPECT_PRED2(MutuallyPrime, b, c);  // Fails
```

#### EXPRE_PRED_FORMAT*

```c++
EXPECT_PRED_FORMAT1(pred_formatter,val1)
EXPECT_PRED_FORMAT2(pred_formatter,val1,val2)
EXPECT_PRED_FORMAT3(pred_formatter,val1,val2,val3)
EXPECT_PRED_FORMAT4(pred_formatter,val1,val2,val3,val4)
EXPECT_PRED_FORMAT5(pred_formatter,val1,val2,val3,val4,val5)

ASSERT_PRED_FORMAT1(pred_formatter,val1)
ASSERT_PRED_FORMAT2(pred_formatter,val1,val2)
ASSERT_PRED_FORMAT3(pred_formatter,val1,val2,val3)
ASSERT_PRED_FORMAT4(pred_formatter,val1,val2,val3,val4)
ASSERT_PRED_FORMAT5(pred_formatter,val1,val2,val3,val4,val5)
```

略

### 死亡断言

> **死亡测试**(Death Test),即程序因为各种原因挂掉的测试,只测试挂掉这一种现象,并且捕捉挂掉原因.



生成新进程,在新进程中进行死亡测试(进程间不共享资源,保证安全性).

生成进程取决于平台，通过编译选项`--gtest_death_test_style`，指定到`::testing::GTEST_FLAG(death_test_style)`。具体生成进程命令如下:

- POSIX 系统，使用`fork()`
  - `--gtest_death_test_style`值为`fast`,立即执行执行死亡测试的语句.此方式为默认.
  - `--gtest_death_test_style`值为`threadsafe`,安装测试代码设定的那样运行,目的是在线程安全性与测试执行效率间取得平衡.
- Windows系统 `CreateProcess()`API,只有`threadsafe`模式.

引入`GTEST_FLAG_SET`的原因是考虑线程安全. 有可能fork出来的线程(threads started by statically-initialized modules)无法被释放. Google Test 采用了三种策略应对这个问题.

- 当死亡测试遇到多线程环境就会WARNING.
- `DeathTest`结尾的test suite会在其他test之前进行.
- 在Linux上使用`clone`取代`fork`.

在死亡测试的 statement 里进行多线程是没问题的,因为死亡测试是在新进程里的.

`GTEST_FLAG_SET`的设置方式非常灵活,可以设置全局的也可以随用随设置

#### EXPECT_DEATH

> 验证`statement`语句造成进程以非0异常终止并输出`stderr`和`matcher`匹配内容

```c++
EXPECT_DEATH(statement,matcher)
ASSERT_DEATH(statement,matcher)
```

`matcher`要么是[matcher](https://google.github.io/googletest/reference/matchers.html)要么是正则表达式

**使用样例**

```c++
EXPECT_DEATH(DoSomething(42), "My error");
```

#### EXPECT_DEATH_IF_SUPPORTED

> 如何支持死亡测试，则行为与`EXPECT_DEATH`一样，否则就什么都不做

#### EXPECT_DEBUG_DEATH

> 在debug模式的时候验证。否则就只是执行语句

#### EXPECT_EXIT

> 验证进程退出的状态满足谓词

```c++
EXPECT_EXIT(statement,predicate,matcher)
ASSERT_EXIT(statement,predicate,matcher)
```

**使用样例**

```c++
// Returns true if the program exited normally with the given exit status code.
::testing::ExitedWithCode(exit_code);

// Returns true if the program was killed by the given signal.
// Not available on Windows.
::testing::KilledBySignal(signal_number);
EXPECT_EXIT(NormalExit(), testing::ExitedWithCode(0), "Success");
```



### **测试数据参数化**

**1.添加类继承自**`public::testing::TestWithParam<T>`

**使用样例**

```c++
class isPrimerParamTest: public::testing::TestWithParam<int>{
    
}.
```

2.告诉gtest你的测试参数

```c++
INSTANTIATE_TEST_CASE_P(TrueReturn, isPrimerParamTset,
                       testing::Values(3, 5, 7, 9, 100));
```

第三个参数还可以是

- Range(begin, end, step)
- ValuesIn(容器或c数组),如ValuesIn(v.begin(), v.end())
- Bool()分别取true和false
- Combin(g1, g2,...,gn) 排列组合

3.获取参数并测试

```c++
TEST_P(isPrimerParamTest, ExpectTrueReturn){
    int x = GetParam();
    EXPECT_TRUE(isPrime(x));
}
```



## Test Suit



## 事件机制

### test case事件

> 每次案例前后，多次对类初始使用测试

也就是TestFixtures的内容



### test suit 事件

> 在某一批测试用例中，第一个执行到最后一个执行后都只是对类初始化一次，最后一次销毁
>
> 一般用于类行为测试或者其他有联系的多个方法测试

**使用样例**

```c++
class SuitTestSmpl: public testing::Test{
protected:
    static void SetUpTestCase(){
  		share = new T;
        ...
    }
    static void TearDownTestCase(){
        ...
         delete _share;
    }
    static T *share;
}
```

```c++
// t1和t2共享share
TEST_F(SuitTestSmpl, t1){
    ...
}
TEST_F(SuitTestSmpl, t2){
	..    
}

```



### Global 事件

> 所有用例共享
>
> 可用于组合类行为测试

```c++
class GlobalTestSmpl: public testing::Environment {
protected:
    virtual void SetUp(){
        // 准备工作
    }
    virtual void TearDown(){
        // 清理工作
    }
};

int _tmain(int argc, char* argv[]){
    // 注册全局事件GlobalTestSmpl
    testing::addGlobalTestEnviroment(new GlobalTestSmpl);
    testing::InitGoogleTest(&argc, argv);
    return RUN_ALL_TESTS();
}
```





### Test Fixtures

如果编写了两个或多个在类似数据上运行的测试，则可以使用test fixture。可以重用对象的相同配置进行多个不同的测试。

**创建一个fixture：**

- 创建一个类继承自`::testing::Test`，body部分使用`protected`，gtest将从子类访问成员
- 定义默认构造函数或`SetUp()`函数
- 定义析构函数或`TearDown()`函数

**使用fixture**

```c++
TEST_F(Test Fixtures类名, TestName) {
  ... test body ...
}
```

当测试运行时，gtest将创建一个TestFixtureName对象t1，用SetUp函数初始化t1，测试，用TearDown函数并销毁t1

**main()函数**

大多数情况下不需要自己写`main`函数，它们会链接到`gtest_main`。如果要写自己的main函数必须要返回`RUN_ALL_TESTS()`的值

```c++
int main(int argc, char **argv) {
  // The ::testing::InitGoogleTest() function parses the command line for googletest flags, 
  // and removes all recognized flags.
  ::testing::InitGoogleTest(&argc, argv); 
  return RUN_ALL_TESTS();
}
```



# RPC

## 基础知识

### RPC

**RPC作用：**

- 屏蔽远程调用跟本地调用的区别，让我们感觉就是调用项目内的方法
- 隐藏底层网络通信的复杂性，更专注于业务逻辑

**RPC在架构中位置**

RPC是解决应用间通信的一种方式，应用架构最终会从“单体”演化成“微服务化”，整个系统会被拆分为多个不同功能的应用，而应用之间通过RPC进行通信。

**RPC框架能够帮助我们解决系统拆分后的通信问题，并让我们像调用本地一样调用远程方法。**

![RPC通信流程1](D:\Repositories\notes\images\image-20220727204227201.png)



### 协议

RPC负责应用间的通信，所以性能要求更高，HTTP协议一般很难满足需求，所以RPC基本会选择设计更紧凑的私有协议。

**可扩展的协议**

![image-20220727204017932](C:\Users\zhuang\AppData\Roaming\Typora\typora-user-images\image-20220727204017932.png)

### 序列化

> 首选Hessain与Protobuf，因为性能、时空开销、通用性、兼容性和安全性都满足了要求

#### JSON

是一种文本型序列化框架

#### Hessain

是动态类型、二进制、紧凑的，可跨语言移植的序列化框架。

#### Protobuf

Google内部的混合语言数据标准，是一种轻便、高效的结构化数据存储格式，用于结构化数据序列化

**优点:**

- 序列化后体积比JSON、Hessian少很多
- IDL能够清晰地描述语义，所以能保证应用程序之间的类型不会丢失，无需类似XML解析器
- 序列化反序列化速度很快，不需要动态获取类型
- 消息格式升级和兼容性不错，可以做到向后兼容

#### 需要注意的问题

1.对象构造得过于复杂：越复杂，序列化和反序列化就越浪费性能，严重影响RPC框架整体性能

2.对象过于庞大： 太大的对象序列化同样浪费时间

3.使用序列化框架不支持的类作为入参类

4.对象有复杂的继承关系

### 网咯通信

RPC框架的开发与使用过程中，倾向于使用**IO多路复用**技术，并且要尽量做到**零拷贝**
