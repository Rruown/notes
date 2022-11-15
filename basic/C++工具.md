# 编译工具

## cmake

**cmake编译**

```cmake
cmake -S . -B build # -S 指定编译的源目录 -B 指定build目录
cmake --build build # 编译build目录
```



**bash脚本**

```bash
mkdir build && cd build # 创建一个build目录
cmake .. && make -j$(nproc) # cmake ..将源目录（./build/..）编译放在build目录下， 多线程make -j$(nproc)
```



### 基础构建

#### **1.基础配置**

**项目名称及版本（必选）**

```cmake
cmake_minimum_required(VERSION 3.0) #指定cmake的最低支持版本
project(项目名 VERSION 版本号 language c cxx) # 指定项目名
```

**CMAKE变量设置（可选）**

> CMAKE提供了多种的变量，包括普通变量、全局变量和环境变量。并且CMAKE预设了一些环境变量

**（1）设置普通变量**

> []括号内的是选项

```cmake
set(<variable> <value>... [PARENT_SCOPE]) # PARENT_SCOPE选项指定是否将当前变量返回到父文件
```

**（2）设置全局变量**

```cmake
set(<variable> <value>... CACHE <type> <docstring> [FORCE]) 

# 设置 Cache 变量
set(MY_GLOBAL_VAR "666" CACHE STRING INTERNAL )

# 设置全局目录变量
set(workspace "/tmp/xxxx" CACHE PATH "this is my workspace" FORCE)
```

`[FORCE]`选项强制修改全局变量的值

`<docstring>`类似于注释

`type`类型有：

	- `BOOL`
	- `FILEPATH`：文件路径
	- `PATH`：目录
	- `STRING`：文本
	- `INTERNAL`：文本

**（3）设置环境变量**

> 设置CMAKE预设的环境变量配置编译选项

```c++
# 指定语言标准
set(CMAKE_CXX_STANDARD 14) # 指定C++版本
set(CMAKE_C_STANDARD 98) # 指定C版本

# 指定构建类型
set (CMAKE_BUILD_TYPE "Debug") # debug模式
set (CMAKE_BUILD_TYPE "Relese") # Release模式
...
```



#### **2.构建库目标**



#### **3.包含头目录**



#### **4.链接可执行目标（可选）**

### 进阶构建

### 高级构建