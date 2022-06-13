## Shell.命令解释器

管理计算机硬件的是操作系统的`kernel`，用户通过`shell`与`kernel`沟通

### 格式

- 以`#!/bin/bash`开头
- 拥有可执行权限

### 变量

> 变量名尽量大写

- 系统变量
  - `$HOME`、`$PWD`、`$USER`等
- 用户自定义变量
- 当前shell所有的变量`set`

**1. 基本语法**

定义变是bash

var=value
撤销变量

```是bash
unset var
```
声明静态变量，不能`unset`

```bash
readonly var
```
**将命令的返回值赋给变量**

- `''`单引号
- `$()`

```bash
A='ls -al'
A=$(ls -al)
```
**2. 设置环境变量**

基本语法

- export 变量名=变量值
- source 配置文件
