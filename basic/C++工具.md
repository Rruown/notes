# 编译工具

## CMake

### 资料

[《CMake菜谱》](https://www.bookstack.cn/read/CMake-Cookbook/README.md)

[《CMake官方文档》](https://cmake.org/cmake/help/latest/index.html)

### 基础构建

> []括号内的是选项，用大写字符表示，可以视作“ON”或“OFF”的布尔类型
> <>尖括号内的是变量

#### 模板代码

```CMake
cmake_minimum_required(VERSION 3.2.0)
project(OC_SORT
        VERSION         1.0
        DESCRIPTION     "The C++ version implementation of OC-SORT"
        HOMEPAGE_URL    ""
        LANGUAGES       CXX)
set(CMAKE_CXX_STANDARD 14) # C++14
set(CMAKE_EXPORT_COMPILE_COMMANDS ON)
```

#### **一 基本配置**

**1.项目名称及版本（必选）**

```Plaintext
cmake_minimum_required(VERSION <min>[...<policy_max>] [FATAL_ERROR])
project(<PROJECT-NAME>
        [VERSION <major>[.<minor>[.<patch>[.<tweak>]]]]
        [DESCRIPTION <project-description-string>]
        [HOMEPAGE_URL <url-string>]
        [LANGUAGES <language-name>...])
```

**2.配置编译选项（可选）**

```Plaintext
# 为所有编译器配置选项
add_compile_options(-Wall -Wextra -pedantic -Werror) 

# 针对C编译器配置
set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -pipe -std=c99") 

# 针对C++编译器配置
set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -pipe -std=c++11")
```

**3.CMAKE变量设置（可选）**

> CMAKE提供了多种的变量，包括普通变量、全局变量和环境变量。并且CMAKE预设了一些环境变量

**（1）设置普通变量**

```Plaintext
set(<variable> <value>... [PARENT_SCOPE]) # PARENT_SCOPE选项指定是否将当前变量返回到父文件

set(MY_LIB_PATH ${CMAKE_CURRENT_BIN_DIR}/xxxx)
```

**（2）设置全局变量**

```Plaintext
set(<variable> <value>... CACHE <type> <docstring> [FORCE]) 

# 设置全局变量
set(MY_GLOBAL_VAR "666" CACHE STRING INTERNAL )

# 设置全局目录变量
set(workspace "/tmp/xxxx" CACHE PATH "this is my workspace" FORCE)
```

`[FORCE]`选项强制修改全局变量的值

`<docstring>`类似于注释

`type`类型有:

-   `BOOL`
    
-   `FILEPATH`：文件路径
    
-   `PATH`：目录
    
-   `STRING`：文本
    
-   `INTERNAL`：文本
    

**（3）设置环境变量**

> 设置CMAKE预设的环境变量**配置**编译选项

指定语言标准

```Plaintext
set(CMAKE_CXX_STANDARD 14) # C++14
set(CMAKE_C_STANDARD 98) # C 98
```

指定构建类型

```Plaintext
set(CMAKE_BUILD_TYPE "Debug") # debug模式
set(CMAKE_BUILD_TYPE "Relese") # Release模式
```

**4.重要的环境变量**

-   `PROJECT_SOURCE_DIR`为包含`PROJECT()`的最近一个`CMakeLists.txt`文件所在的文件夹。
-   `CMAKE_SOURCE_DIR`指顶层的CMakeLists.txt路径
-   `CMAKE_CURRENT_SOURCE_DIR`指cmake文件当前的路径
-   `CMAKE_CURRENT_BINRARY_DUR`指cmake文件当前的二进制（一般位于build目录下）文件路径
-   `CMAKE_MODULE_PATH`指cmake搜索.cmake脚本的路径 
-   `CMAKE_EXPORT_COMPILE`指定编译生成command_compile.json

#### **二 构建库目标**

**（1）构建普通库目标**

> 若干源文件生成一个库目标，一般会配合`aux_source_directory`或`file`函数一起使用

```Plaintext
add_library(<name> [STATIC | SHARED | MODULE]
            [EXCLUDE_FROM_ALL]
            [<source>...])
```

搜索目录下所有源文件

```Plaintext
aux_source_directory(<dir> <variable>)

# 搜索src目录下的所有源文件.cc,.cpp
aux_source_directory(./src SRCS)
```

搜索满足通配符表达式的文件

```Plaintext
file(GLOB <variable> 
     [LIST_DIRECTORIES true|false] [RELATIVE <path>] [CONFIGURE_DEPENDS]
     [<globbing-expressions>...])
file(GLOB_RECURSE <variable> [FOLLOW_SYMLINKS] # GLOB_RECURSE递归搜索
     [LIST_DIRECTORIES true|false] [RELATIVE <path>] [CONFIGURE_DEPENDS]
     [<globbing-expressions>...])
     
# 搜索src目录下所有的.cc文件
file(GLOB_RECURSE SRCS ./src/*.cc)
```

**（2）构建接口库目标**
> 暂时略

```Plaintext
add_library(<name> INTERFACE)

# version 3.19
add_library(<name> INTERFACE [<source>...] [EXCLUDE_FROM_ALL])
```

**（3）导入库目标**

> 导入库目标不需要构建，相比`link_directories`，不仅能有依赖关系，还能够方便地配置属性选项,一般配合`set_property`使用

```Plaintext
add_library(<name> [STATIC | SHARED | MODULE | UNKNOWN] IMPORTED [GLOBAL])

# 导入./InferEngine.so
add_library(InferEngine SHARED IMPORTED)
```

为多个object设置属性

```Plaintext
set_property(<GLOBAL                      |
              DIRECTORY [<dir>]           |
              TARGET    [<target1> ...]   |
              SOURCE    [<src1> ...]
                        [DIRECTORY <dirs> ...]
                        [TARGET_DIRECTORY <targets> ...] |
              INSTALL   [<file1> ...]     |
              TEST      [<test1> ...]     |
              CACHE     [<entry1> ...]    >
             [APPEND] [APPEND_STRING]
             PROPERTY <name> [<value1> ...])

# 导入./InferEngine.so
set_property(TARGET InferEngine PROPERTY IMPORTED_LOCATION ${CMAKE_CURRENT_SOURCE_DIR}/libInferEngine.so)
```

**（4）构建接口目标**

略

#### **三 包含头目录**

**（1）设置所有目标头文件目录**

```Plaintext
include_directories(<dir>...)
```

**（2）设置某个目标头文件目录**

```Plaintext
target_include_directories(<target> [SYSTEM] [AFTER|BEFORE]
  <INTERFACE|PUBLIC|PRIVATE> [items1...]
  [<INTERFACE|PUBLIC|PRIVATE> [items2...] ...])
  
```

#### **四 链接可执行目标**

**（1）构建可执行目标**

```Plaintext
add_executable(<name> [WIN32] [MACOSX_BUNDLE]
               [EXCLUDE_FROM_ALL]
               [source1] [source2 ...])
               
# m1.cc m2.cc main.cc构建可执行目标test，其中main.cc中含有主函数
add_executable(test m1.cc m2.cc main.cc)
```

**（2）可执行目标链接目标库**

> 如果同时将多个target链接到同一个target中，并且这些target之间存在依赖关系，就需要特别注意顺序。因为在CMake中，`target_link_libraries`命令会按照指定的顺序链接库或target，而如果链接顺序不正确，就可能导致链接失败。

```Plaintext
target_link_libraries(<target>
                      <LINK_PRIVATE|LINK_PUBLIC> <lib>...
                     [<LINK_PRIVATE|LINK_PUBLIC> <lib>...]...)

# test链接InferEngine库
target_link_libraries(test InferEngine)

# -Wl in gcc is to call “ld” from gcc
# –start-group and –end-group are for repeatedly search lib for circular dependency.
target_link_libraries(${PROJECT_NAME} "-Wl,--start-group" ${GST_LIBRARIES} "-Wl,--end-group")
```

#### 五 构建

**cmake命令构建**

```Plaintext
cmake -S . -B build # -S 指定编译的源目录 -B 指定build目录
cmake --build build # 编译build目录
```

**bash脚本构建**

```Bash
mkdir build && cd build # 创建一个build目录
cmake .. && make -j$(nproc) # 多线程编译
```

  

### 进阶构建

### 高级构建

高级构建模式主要涉及外部项目管理、

#### ExternalProject

`ExternalProject_Add`的选项用于外部项目源的配置和编译、安装等所有方面，可以分成如下几类：

-   **Directory**
    
    ```CMake
    TMP_DIR = <EP_BASE>/tmp/<name>
    STAMP_DIR = <EP_BASE>/Stamp/<name>
    DOWNLOAD_DIR = <EP_BASE>/Download/<name>
    SOURCE_DIR = <EP_BASE>/Source/<name>
    BINARY_DIR = <EP_BASE>/Build/<name>
    INSTALL_DIR = <EP_BASE>/Install/<name>
    ```
    
-   **Download**
    
-   **Update和Patch**
    
-   **Configure**
    
-   **Build**
    

```CMake
# 指定外部项目源编译的命令
BUILD_COMMAND "./script/build.sh && ..."
BUILD_COMMAND "cmake -S . -B build && cmake --build build"

# 每次外部项目源均要重新构建
BUILD_ALWAYS 1
```

-   **Install**
    

```CMake
# 指定外部项目比安装的命令
INSTALL_COMMAND ""
```

-   **Test**
    

1.  ##### 外部项目源与构建分离
    

在使用googletest时，google官方给出的CMake使用方式如下。这种方法会自动将googletest项目源下载到构建目录下并包含gtest的头文件等。但是，每次清空构建目录时都会重新googletest项目源（外网下载速度慢），然后再编译项目。

```CMake
include(FetchContent)FetchContent_Declare(
  googletest
  URL https://github.com/google/googletest/archive/03597a01ee50ed33e9dfd640b249b4be3799d395.zip
)# For Windows: Prevent overriding the parent project's compiler/linker settings
set(gtest_force_shared_crt ON CACHE BOOL "" FORCE)FetchContent_MakeAvailable(googletest)
```

使用`ExternalProject_Add`添加外部源支持**本地外部源**、**远程外部源**（URL、下载命令、GIT库等）两种方式。本地外部源指定`SOURCE_DIR`属性。

默认情况下，**CMake会假定外部项目是使用CMake配置的**。在`ExternalProject_Add`中配置`CMAKE_ARGS`和`CMAKE_CACHE_ARGS`属性，如下所示。就可以向外部项目源传递CMake参数。

```CMake
CMAKE_ARGS
  -DCMAKE_CXX_COMPILER=${CMAKE_CXX_COMPILER}
  -DCMAKE_CXX_STANDARD=${CMAKE_CXX_STANDARD}
  -DCMAKE_CXX_EXTENSIONS=${CMAKE_CXX_EXTENSIONS}
  -DCMAKE_CXX_STANDARD_REQUIRED=${CMAKE_CXX_STANDARD_REQUIRED}
```

**实行方案：**将外部项目源独立于构建目录之外（事先下载到本地，用本地外部源方式或用远程外部源方式下均可），然后使用`CMAKE_ARGS`属性向外部项目源传递CMake参数：

-   `-DCMAKE_INSTALL_PREFIX=${GTEST_INSTALL_DIR}`，指定安装目录
    

  将安装目录中的头文件目录和库目录和库名合并到当前项目中，至此完成。

# 版本控制

## Git

### 分支管理

  

**查看本地分支**

```Bash
git branch
```

**查看远程分支**

```Bash
git branch -r
```

**查看所有分支**

```Bash
git branch -a
```

**本地创建新分支**

```Bash
git branch new_branch
```

**切换分支**

```Bash
git checkout new_branch
```

**创建分支并切换**

```Bash
git checkout -b new_branch
```