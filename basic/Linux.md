# 基础命令

## 系统管理

> 如何（自动）启动/关闭服务

计算机中，一个正在执行的程序或命令，被叫做“进程”（process）。
启动之后一只存在、常驻内存的进程，一般被称作“服务”（service）。

### service服务管理

**基本语法**

```bash
service 服务名 start|stop|restart|status
```

**常用案例**

```bash
/etc/init.d/ # 查看服务

service ssh start # 启动ssh服务
```

### systemctl

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



## 搜索查找

### find

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

### locate

利用事先建立的系统中所有文件名称及路径的locate数据库快速定位文件

> 由于locate指定基于数据库查询，第一次运行前，必须使用`updatedb`指令创建locate数据库

**基本语法**

```bash
locate 搜索文件
```



## 进程管理

### ps

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

### kill

常用命令

```bash
kill -9 PID # 向PID发送SIGINT信号，强制关闭进程
killall 进程名 # 通过进程名杀死进程
```

### pstree 查看进程树

**基本语法**

```bash
pstree [-p/-u]
```

**常用选项：**

- p: 显示进程的PID
- u: 显示进程的所属用户

### top

### netstat 

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

## 文件目录管理

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

### 输出重定向与追加

**基本语法**

```bash
ls -l > 文件 # 将ls -l的内容写入文件中（覆盖写）
ls -l >> 文件 # 追加写
```

### 软连接

> ln 命令默认硬链接，-s表示软链接

**基本语法**

```bash
ln -s 原文件（目录） 连接名
```

## 用户管理

> /etc/passwd 

**常用命令**

```bash
useradd 用户名 # 添加用户
userdel 用户名 # 删除用户
passwd 用户名 # 为用户设置密码
id 用户名 # 判断用户名是否存在
su 用户名 # 切换用户
```



## 用户组管理

**常用命令**

```bash
groupadd 组名 # 新增组
groupdel 组名 # 删除组
groupmod -n 新名字 旧名字 # -n 指定工作组名字
usermod -g 用户组 用户名 # 修改用户的所属组
```



# Shell编程

shell脚本开头如下：

```bash
#!/bin/bash
```

脚本运行方法有以下两种方式:

```bash
bash 脚本 # 创建一个子shell运行脚本
. 脚本 # 在当前shell中运行脚本
```

单引号与双引号

```bash
var=1
echo '$var'
>>> $var # 单引号直接输出内部字符串，不解析特殊字符

echo "$var"
>>> 1 # 双引号解析特殊字符
```

内部字段分隔符（Internal Field Separator）,默认情况下，bash shell会将下面的字符当做字段分隔符：空格、制表符、换行符。

```bash
#!/bin/bash
# test IFS

# 以逗号分割字段
IFS=','
for number in $(cat file)
do
  echo "Number $number"
done

```

```bash
arr=(${line//./ }) # 以.作为分隔符划分字符串为数组
```



## 变量

变量按照两个角度分类:

1. 系统变量与用户自定义变量
2. 全局变量与局部变量

```bash
env # 查看系统全局变量
set | less # 查看全部的变量
```

### **基本语法**

1. 定义变量：变量名=变量值

   > 注意，=前后没有空格

2. 撤销变量:`unset` 变量名

3. 声明静态变量：`readonly` 变量

   > 注意，不能unset

### 全局变量

> 在子shell中更改的，父shell没有变化。比如子shell声明一个全局变量，父shell无法访问

将局部变量升级为全局变量

```bash
export 变量
```

`$PATH`环境变量是shell查找命令的目录。添加**自定义命令路径**，就可以直接调用shell脚本

### 特殊变量

1. `$n`:代表第n个参数，9以上用大括号，如`${10}`

2. `$#`:获取参数个数
3. `$*`和`$@`：代表命令行中的所有参数
   1. `$*`：`"$*"`把所有参数看作一个整体
   2. `$@`：`"$@"`把所有参数作为一个集合
4. `$?`：最后一次执行命令的返回状态
   1. 返回0：命令执行成功
   2. 返回非0：命令执行失败

## 运算符

> 命令替换：指将命令的标准输出值赋值给某个变量。
>
> 命令替换方式：
>
> - `'命令'`
> - `$(命令)`

### 基本语法

```bash
$((运算式)) 或 $[运算式]
```

## 条件判断

### 基本语法

1. `test condition`

2. `[ conditioin ]`

   > condition前后要有空格

### 常用判断条件

> 字符串之间的比较用`=`和`!=`

#### 整数比较

1. `-eq` equal
2. `-ne` not equal
3. `-lt` less than
4. `-le` less equal
5. `-gt` greater than
6. `ge` greater equal

#### 布尔运算

1. `!` 非
2. `-a` and
3. `-o` or

#### 文件权限

1. `-r` read
2. `-w` write
3. `-x` execute

#### 文件类型

1. `-e` existence
2. `-f` file
3. `-d` directory

```bash
# &&表示前一条命令执行成功，才执行后一条命令，||表示上一条命令执行失败，才执行下一条命令
$[ sdfadf ] && echo OK || echo notOK
```

## `流程控制`

> `；`用来分隔命令

### if判断

#### 基本语法

1. 单支

```bash
if [ 条件 ]; then
	程序
fi
```

或

```bash
if [ 条件 ]
then
	程序
fi
```

2. 多分支

```bash
if [ 条件 ]
then
	程序
elif
then
	程序
else
	程序
fi
```

### case语句

#### 基本语法

```bash
case $变量名 in
"值1")
	程序
;;
"值2")
	程序
;;
*)
	程序
esac
```

`;;`相当于break，`*`相当于default

### for循环

#### 基本语法

```bash
for ((初始值;循环条件;变量变化))
do
	程序
done
```

```bash
for 变脸 in 值1 值2 ...
do
	程序
done
```

> `{}`在Linux表示序列，如`{1..100}`表示1-100的序列

求1到100的和

```bash
for i in {1..100}
do
	sum=$[$sum+$i]
done
```

### while循环

#### 基本语法

```bash
while [ 条件 ]
do
	程序
done
```

## 读取控制台输入

### 基本语法

```bash
read [选项] [参数] 变量名
```

选项:

- `-p`:指定读取时的提示符
- `-t`:指定读取时的等待事件，单位秒

## 文本处理工具

### `awk`

#### 基本语法

```bash
awk [选项] '/正则表达式/{actions}'... '/正则表达式/{actions}' 文件
```

#### 数据字段变量

> 默认的字段分隔符是任意的空白字符，也可以通过-f指定

`awk` 的主要特性之一是其处理文本文件中数据的能力，它会自动给一行中的每个数据元素分配一个变量。

#### 关键字

BEGIN：在处理数据前运行一些脚本命令



END：在读完数据后执行脚本命令

#### 内置变量

`NF`: number of field  一条记录的字段的数目

`NR`: number of recent 已经读出的记录数，就是行号，从1开始

#### Actions

> 设计跟c语言类似

##### 运算符

> 以下没有的运算符，使用方式与c语言一样

```bash
空格 # 字符串连接
| |& # 管道运算符，用在getline, print, printf函数
in # 数组成员
？： # 三元运算符
~ !~ # 正则表达式匹配, 注意要用/regex/
```

##### 控制语句

```bash
if,while,do-while,for,for(a in b),break, continue 与c语言一致
delete array[index]
delete array
exit [可选的表达式] {语句}
switch (表达式) {
	case 值或正则表达式: 
		语句
	...
	default: 
		语句
}

```

##### 输入/输出语句

```bash
close(file) # 关闭文件或管道
getline # 将下一行记录放入$0,并设置NF,NR, FNR, RT
getline <file # 将文件中的下一行记录放入$0,并设置NF,RT
getline var # 将下一行记录放入var,并设置NR, FNR, RT
getline var <file # 将文件中的下一行记录放入var,并设置RT
command | getline var # 运行command命令，管道输出到var或$0
next # 跳到下一行记录
nextfile # 跳到下一个文件进行处理


print # 输出当前行记录
print expr-list # 输出表达式列表，每个表达式由OFS分割，直到ORS结束(expr-list形如 a,"123",1)
print expr-list >file # 输出到文件
print fmt, expr-list # 格式化输出
print fmt, expr-list >file # 格式化输出到文件

sytem(cmd-line) # 执行一个命令返回运行结果
fflush([file]) #

print ... >> file # 追加写入文件
print ... | command # 

```

##### 函数

**数组函数**

```bash
cos,sin,int,log,rand,sqrt...
```

**字符串函数**

```bash
index(s,t) # 返回字符串t在s中的索引位置， 从1开始，0是错误
length([s]) # 返回字符串s或$0的长度
substr(s,i[,n]) # 截取s的子串，从i开始截取n个字符
待定...
```



### `wc`

#### 基本语法

```bash
wc [-clw] [文件...]
```

- `-c` --chars 只显示字节数
- `-l` --lines 只显示行数 
- `-w` --words 只显示单词数

### `cat`

### `uniq`

忽略重复行

#### **基本语法**

```bash
uniq [OPTION] [INPUT [OUTPUT]]
```

- c: 行数

- d: 只打印重复行

- u: 只打印唯一行

- i: 忽略大小写

  

### `sort`

按行排序文件数据	

#### 基本语法

```bash
sort [OPTION] [FILE]
```

- r: 反转排序结果
