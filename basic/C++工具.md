# 编译工具

## CMake

### 资料

[《CMake菜谱》](https://www.bookstack.cn/read/CMake-Cookbook/README.md)

[《CMake官方文档》](https://cmake.org/cmake/help/latest/index.html)



### 基础构建

> []括号内的是选项，用大写字符表示，可以视作“ON”或“OFF”的布尔类型
>
> <>尖括号内的是变量

#### **一 基本配置**

**1.项目名称及版本（必选）**

```cmake
cmake_minimum_required(VERSION <min>[...<policy_max>] [FATAL_ERROR])
project(<PROJECT-NAME>
        [VERSION <major>[.<minor>[.<patch>[.<tweak>]]]]
        [DESCRIPTION <project-description-string>]
        [HOMEPAGE_URL <url-string>]
        [LANGUAGES <language-name>...])
```



**2.配置编译选项（可选）**

```cmake
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

```cmake
set(<variable> <value>... [PARENT_SCOPE]) # PARENT_SCOPE选项指定是否将当前变量返回到父文件

set(MY_LIB_PATH ${CMAKE_CURRENT_BIN_DIR}/xxxx)
```

**（2）设置全局变量**

```cmake
set(<variable> <value>... CACHE <type> <docstring> [FORCE]) 

# 设置全局变量
set(MY_GLOBAL_VAR "666" CACHE STRING INTERNAL )

# 设置全局目录变量
set(workspace "/tmp/xxxx" CACHE PATH "this is my workspace" FORCE)
```

`[FORCE]`选项强制修改全局变量的值

`<docstring>`类似于注释

`type`类型有:

- `BOOL`
- `FILEPATH`：文件路径
- `PATH`：目录
- `STRING`：文本
- `INTERNAL`：文本

**（3）设置环境变量**

> 设置CMAKE预设的环境变量**配置**编译选项

指定语言标准

```cmake
set(CMAKE_CXX_STANDARD 14) # C++14
set(CMAKE_C_STANDARD 98) # C 98
```

 指定构建类型

```cmake
set(CMAKE_BUILD_TYPE "Debug") # debug模式
set(CMAKE_BUILD_TYPE "Relese") # Release模式
```

**4.重要的环境变量**

- `PROJECT_SOURCE_DIR`为包含`PROJECT()`的最近一个`CMakeLists.txt`文件所在的文件夹。
- `CMAKE_SOURCE_DIR`指顶层的CMakeLists.txt路径
- `CMAKE_CURRENT_SOURCE_DIR`指cmake文件当前的路径
- `CMAKE_CURRENT_BINRARY_DUR`指cmake文件当前的二进制（一般位于build目录下）文件路径
- `CMAKE_MODULE_PATH`指cmake搜索.cmake脚本的路径
- `CMAKE_EXPORT_COMPILE`指定编译生成command_compile.json



#### **二 构建库目标**

**（1）构建普通库目标**

> 若干源文件生成一个库目标，一般会配合`aux_source_directory`或`file`函数一起使用

```cmake
add_library(<name> [STATIC | SHARED | MODULE]
            [EXCLUDE_FROM_ALL]
            [<source>...])
```

搜索目录下所有源文件

```cmake
aux_source_directory(<dir> <variable>)

# 搜索src目录下的所有源文件.cc,.cpp
aux_source_directory(./src SRCS)
```

搜索满足通配符表达式的文件

```cmake
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

```cmake
add_library(<name> INTERFACE)

# version 3.19
add_library(<name> INTERFACE [<source>...] [EXCLUDE_FROM_ALL])
```



**（3）导入库目标**

> 导入库目标不需要构建，相比`link_directories`，不仅能有依赖关系，还能够方便地配置属性选项,一般配合`set_property`使用

```cmake
add_library(<name> [STATIC | SHARED | MODULE | UNKNOWN] IMPORTED [GLOBAL])

# 导入./InferEngine.so
add_library(InferEngine SHARED IMPORTED)
```

为多个object设置属性

```cmake
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

```cmake
include_directories(<dir>...)
```



**（2）设置某个目标头文件目录**

```cmake
target_include_directories(<target> [SYSTEM] [AFTER|BEFORE]
  <INTERFACE|PUBLIC|PRIVATE> [items1...]
  [<INTERFACE|PUBLIC|PRIVATE> [items2...] ...])
  
```



#### **四 链接可执行目标**

**（1）构建可执行目标**

```cmake
add_executable(<name> [WIN32] [MACOSX_BUNDLE]
               [EXCLUDE_FROM_ALL]
               [source1] [source2 ...])
               
# m1.cc m2.cc main.cc构建可执行目标test，其中main.cc中含有主函数
add_executable(test m1.cc m2.cc main.cc)
```



**（2）可执行目标链接目标库**

```cmake
target_link_libraries(<target>
                      <LINK_PRIVATE|LINK_PUBLIC> <lib>...
                     [<LINK_PRIVATE|LINK_PUBLIC> <lib>...]...)

# test链接InferEngine库
target_link_libraries(test InferEngine)
```



#### 五 构建

**cmake命令构建**

```cmake
cmake -S . -B build # -S 指定编译的源目录 -B 指定build目录
cmake --build build # 编译build目录
```



**bash脚本构建**

```bash
mkdir build && cd build # 创建一个build目录
cmake .. && make -j$(nproc) # 多线程编译
```



### 进阶构建

### 高级构建