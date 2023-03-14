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

### 整数比较

1. `-eq` equal
2. `-ne` not equal
3. `-lt` less than
4. `-le` less equal
5. `-gt` greater than
6. `ge` greater equal

### 布尔运算

1. `!` 非
2. `-a` and
3. `-o` or

### 文件权限

1. `-r` read
2. `-w` write
3. `-x` execute

### 文件类型

1. `-e` existence
2. `-f` file
3. `-d` directory

```bash
# &&表示前一条命令执行成功，才执行后一条命令，||表示上一条命令执行失败，才执行下一条命令
$[ sdfadf ] && echo OK || echo notOK
```

## 流程控制

> `；`用来分隔命令

### if判断

### 基本语法

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

### 基本语法

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

### 基本语法

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

### 基本语法

```bash
while [ 条件 ]
do
	程序
done
```

## 数组

Shell 数组用括号来表示，元素用"空格"符号分割开，语法格式如下

```bash
array_name=(value1 value2 ... valuen)
```

**读取数组：**

```bash
${array_name[index]}
```

**数组长度：**

```bash
${#my_array[*]}
${#my_array[@]
```

**添加元素：**

```bash
array_name=("${array_name[@]}" v1 v2 ...)
array_name+=(v1 v2...)
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

### 基本语法

```bash
awk [选项] '/正则表达式/{actions}'... '/正则表达式/{actions}' 文件
```

### 数据字段变量

> 默认的字段分隔符是任意的空白字符，也可以通过-f指定

`awk` 的主要特性之一是其处理文本文件中数据的能力，它会自动给一行中的每个数据元素分配一个变量。

### 关键字

BEGIN：在处理数据前运行一些脚本命令



END：在读完数据后执行脚本命令

### 内置变量

`NF`: number of field  一条记录的字段的数目

`NR`: number of recent 已经读出的记录数，就是行号，从1开始

### Actions

> 设计跟c语言类似

### 运算符

> 以下没有的运算符，使用方式与c语言一样

```bash
空格 # 字符串连接
| |& # 管道运算符，用在getline, print, printf函数
in # 数组成员
？： # 三元运算符
~ !~ # 正则表达式匹配, 注意要用/regex/
```

### 控制语句

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

### 输入/输出语句

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

### 函数

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

### 基本语法

```bash
wc [-clw] [文件...]
```

- `-c` --chars 只显示字节数
- `-l` --lines 只显示行数 
- `-w` --words 只显示单词数

### `cat`

### `uniq`

忽略重复行

### **基本语法**

```bash
uniq [OPTION] [INPUT [OUTPUT]]
```

- c: 行数

- d: 只打印重复行

- u: 只打印唯一行

- i: 忽略大小写

  

### `sort`

按行排序文件数据	

### 基本语法

```bash
sort [OPTION] [FILE]
```

- r: 反转排序结果
- k：通过key排序，1表示第1列，2表示第2列

## 