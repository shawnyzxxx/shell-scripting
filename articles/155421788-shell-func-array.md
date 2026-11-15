# RHCSE Shell 05：函数与数组详解

## 文章导读
函数与数组是 Shell 编程进阶的两大核心。本文系统讲解 **Shell 函数的定义、调用、参数、返回值、作用域、函数库文件**，以及**索引数组与关联数组的创建、增删改查、遍历**，并附主机在线判断、用户存在性判断等实战案例，适合已掌握 Shell 基础、希望进阶脚本编程的读者。

## 一、函数

### 1. 什么是函数
函数几乎是学习所有程序设计语言都必须过的一关。所谓函数，就是把**完成特定功能、并且多次使用的一组命令或语句封装在一个固定的结构中**，这个结构就叫函数。
从本质上讲，函数是将一个函数名与某个代码块进行映射。用户在定义函数之后，就可以通过函数名来调用其对应的一组代码。

**使用 Shell 函数的优势**
- 把相同的程序段定义为函数，可以减少整个程序段代码量，提升开发效率
- 增加程序段可读性、易读性，提升管理效率
- 实现程序功能模块化，使程序具备通用性（可移植性）

### 2. 函数语法
```bash
function 函数名() {
    指令
    return
}
```

**简化写法1：**
```bash
function 函数名 {
    指令
    return
}
```

**简化写法2：**
```bash
函数名() {
    指令
    return
}
```

### 3. 函数的调用与参数

**函数调用语法**
```bash
函数名称 参数1 参数2 …
```

**函数参数**
Shell 的函数参数语法比较特殊。实际上，Shell 将脚本参数和函数参数做了统一处理，采用相同方法处理脚本参数和函数参数。
- **调用函数**：直接执行函数名即可：`函数名`
- **带参数的函数执行**：`函数名 参数`

与 Shell 脚本一样，用户可以在 Shell 函数中使用位置变量获取参数值：`$0` 表示脚本名称，`$#` 获取参数个数，`$1` 表示第 1 个参数，`$2` 表示第 2 个参数，以此类推。还可通过系统变量 `$@` 和 `$*` 获取所有参数的值。

```bash
[root@server ~]# vim fun1.sh
#!/bin/bash

func() {
    echo "the function has $# parameters"
    echo "all parameters are $*"
    echo "all parameters are $@"
    echo "the script name is $0"
    echo "the first parameter is $1"
}

func hello world
```

```bash
[root@server ~]# bash fun1.sh
the function has 2 parameters
all parameters are hello world
all parameters are hello world
the script name is fun1.sh
the first parameter is hello
```

### 4. 函数的返回值
**方法1**：在函数内用 `return` 退出函数并返回函数的值，在函数外用 `echo $?` 获取返回值。
> 注意：返回值的范围只能在 0~255，超过部分需除以 256 取余。

**方法2**：在函数内用 `echo` 输出值，在函数体外用 `变量=$(函数名)` 获取函数的返回值。

**示例1：**
```bash
# 第一种方法
sum1() {
    sum=$[$1 + $2]
    echo $sum
}
read -p "输入第一个数" first
read -p "输入第二个数" second
sum1 $first $second

# 第二种方法
sum() {
    s=$[$1 + $2]      # $1 $2 传递位置参数
    echo $s
}
sum $1 $2            # $1 $2 代表执行脚本函数时命令后面跟的两个参数位置参数
bash 1.sh 1 3
```

**示例2：判断数字是否大于 10**
```bash
#!/bin/bash
# 函数：判断数字是否大于 10
is_bigger_than_10() {
    if [ $1 -gt 10 ]; then
        return 0    # 条件成立，返回状态码 0（成功）
    else
        return 1    # 条件不成立，返回状态码 1（失败）
    fi
}

# 调用函数（传入参数 15）
is_bigger_than_10 15
echo $?   # 输出：0（表示函数判断结果为“是”）

# 调用函数（传入参数 8）
is_bigger_than_10 8
echo $?   # 输出：1（表示函数判断结果为“否”）
```

### 5. 函数案例

**例1：判定 192.168.48.130~192.168.48.140 之间的主机哪些在线**
要求：使用函数实现一台主机的判定过程，在主程序中调用此函数判定指定范围内所有主机的在线情况。

**法1：直接使用函数实现（无参数，无返回值）**
```bash
[root@server ~]# cat fun4.sh
#!/bin/bash

online() {
    for i in {130..140}; do
        if ping -c 2 192.168.48.$i &>/dev/null
        then
            echo "192.168.48.$i is up"
        else
            echo "192.168.48.$i is unknown"
        fi
    done
}

online
```

**法2：使用函数传参（有参数，无返回值）**
```bash
[root@server ~]# vim fun5.sh
#!/bin/bash

online() {
    if ping -c 2 $1 &>/dev/null
    then
        echo "$1 is up"
    else
        echo "$1 is unknown"
    fi
}

for i in {130..140}
do
    online 192.168.48.$i
done
```

**法3：使用函数返回值判断（有参数，有返回值）**
```bash
[root@server ~]# vim fun6.sh
#!/bin/bash

online() {
    if ping -c 2 $1 &>/dev/null
    then
        return 0
    else
        return 1
    fi
}

for i in {130..140}
do
    online 192.168.48.$i
    if [ $? -eq 0 ]; then
        echo "192.168.48.$i is up"
    else
        echo "192.168.48.$i is unknown"
    fi
done
```

**例2：判断用户是否存在并返回信息**
函数接受一个用户名参数，判断用户是否存在：存在则返回该用户的 shell 和 UID 并返回正常状态值；不存在则提示并返回错误状态值，最后在主程序中调用。
```bash
[root@server ~]# vim fun7.sh
#!/bin/bash

user() {
    if id $1 &>/dev/null
    then
        echo "`grep ^$1 /etc/passwd | cut -d: -f3,7`"
        return 0
    else
        echo "$1 does not exist"
        return 1
    fi
}

read -p "please input username:" username
until [ "$username" = "quit" -o "$username" = "exit" -o "$username" = "q" ]; do
    user $username
    if [ $? == 0 ]; then
        read -p "please input again:" username
    else
        read -p "no $username, Please input again:" username
    fi
done
```

```bash
[root@server ~]# bash fun7.sh
please input username:redhat
1000:/bin/bash
please input again:root
0:/bin/bash
please input again:dakuang
dakuang does not exist
no dakuang,Please input again:hehe
hehe does not exist
no hehe,Please input again:quit
```

### 6. 函数变量作用域
在函数内部，如果没有使用 `local` 关键字修饰，那么函数中的变量也是**全局变量**。

**例1：函数的变量是全局变量**
```bash
[root@server ~]# vim fun8.sh
#!/bin/bash

var="hello world"
func() {
    var="orange"
    echo $var
    var2="hello"
}
echo "$var"
func
echo "$var"
echo "$var2"
```
```bash
[root@server ~]# bash fun8.sh
hello world
orange
orange
hello
```

**例2：函数的变量使用 local 指定为局部变量**
```bash
[root@server ~]# vim fun9.sh
#!/bin/bash

var="hello world"
func() {
    local var="orange"
    echo $var
    local var2="hello"
}
echo "$var"
func
echo "$var"
echo "$var2"
```
```bash
[root@server ~]# bash fun9.sh
hello world
orange
hello world
```
> 对比可见：使用 `local` 后，函数内对 `var` 的修改不影响函数外的变量，`var2` 也只在函数内可见。

### 7. 函数库文件

**概念**
为方便重用功能，可以创建一些可重用的函数，单独放在函数库文件中。创建函数库文件的过程类似于编写 Shell 脚本，**区别在于函数库文件通常只包括函数**，而脚本中既可包括函数和变量的定义，又包括可执行代码（位于函数外部的代码，脚本载入后会立即执行，无需另外调用）。

**函数库文件的调用**
库文件定义好后，用户可在程序中载入并调用其中的函数。Shell 中载入库文件的命令为 `.`（圆点），语法如下：
```bash
. ./filename
# 或者
source filename
```
> 其中 `filename` 为库文件名称，必须是合法文件名。库文件可用相对路径或绝对路径，圆点命令与库文件名之间要有一个空格。

**示例：创建函数库文件**
```bash
[root@server ~]# vim function_library.sh
#!/bin/bash

addition() {
    echo $[$1 + $2]
}

subtraction() {
    echo $[$1 - $2]
}

multiplication() {
    echo $[$1 * $2]
}

division() {
    if [ $2 -eq 0 ]; then
        echo 'Divisor cannot be 0'
    else
        echo $[$1 / $2]
    fi
}
```

**在脚本中加载函数库文件并使用**
```bash
[root@server ~]# vim clr_test.sh
#!/bin/bash

# 加载函数库文件
source /root/function_library.sh

v1=10
v2=5

r1=$(addition $v1 $v2)
r2=$(subtraction $v1 $v2)
r3=$(multiplication $v1 $v2)
r4=$(division $v1 $v2)

echo "$v1+$v2=$r1"
echo "$v1-$v2=$r2"
echo "$v1*$v2=$r3"
echo "$v1/$v2=$r4"
```

## 二、数组

### 1. 数组应用场景

**方案1：通过数组统一存储同类数据**
将多个变量整合到单个数组变量中，用索引访问元素：
```bash
# 定义日志路径数组
logs=(
    "/var/log/httpd/access_log"   "/var/log/nginx/error_log"   "/var/log/mysql/slow.log"
)

# 动态添加新日志（无需修改变量名）
logs+=("/var/log/audit/audit.log")

# 遍历所有日志（自动适应元素数量）
for log in "${logs[@]}"; do
    grep "ERROR" "$log" > "${log}.errors"
done
```

**方案2：动态生成不确定数量的元素**
通过命令输出或循环动态填充数组：
```bash
# 从命令输出生成IP地址数组（数量未知）
ips=($(awk '{print $1}' /var/log/access_list))

# 统计IP数量
echo "共有 ${#ips[@]} 个IP需要处理"

# 批量执行ping测试
for ip in "${ips[@]}"; do
    ping -c 1 "$ip" &> /dev/null && echo "$ip 存活" || echo "$ip 宕机"
done
```

### 2. 数组的创建

**创建数组的 5 种方法**
```bash
# 方法1：直接赋值
array=(value1 value2 value3 ... )

# 方法2：指定下标直接赋值
array=([0]=one [2]=three ... [n]=valuen)

# 方法3：逐元素赋值
array[0]=a; array[1]=b; array[2]=c

# 方法4：用命令输出创建
array=(命令)

# 方法5：先声明，后赋值
```
> 注意：方法 1-4 默认是创建**索引数组**，要创建**关联数组**就要先声明。
> 想按「序号」存数据（如处理文件列表、服务器 IP 清单）→ 用**索引数组**；想按「名字」存数据（如用户姓名-ID 映射、应用配置项 key-value）→ 用**关联数组**。

```bash
# 创建索引数组
declare -a array

# 创建关联数组
[root@localhost mnt]# declare -A arry1
[root@localhost mnt]# arry1=([xixi]="buhaoyisi" [hehe]="wuyu" [haha]="kaixin")
[root@localhost mnt]# echo ${arry1[@]}
kaixin buhaoyisi wuyu
[root@localhost mnt]# echo ${!arry1[@]}
haha xixi hehe
```

**索引数组与关联数组对比**

| 特性 | 索引数组（Indexed Array） | 关联数组（Associative Array） |
|---|---|---|
| 索引类型 | 非负整数（0、1、2...） | 字符串（如 "name"、"ip"） |
| 索引分配方式 | 默认自动递增，也可手动指定 | 必须手动指定字符串键 |
| 定义方式 | 直接赋值（`arr=(a b c)`）或 `declare -a` | 必须用 `declare -A` 声明 |
| 适用场景 | 存储有序列表（文件名、IP 列表） | 存储键值映射（用户信息、配置） |
| 遍历方式 | 按索引顺序遍历 | 按键的哈希表顺序遍历（不固定） |

**`declare -a logs` 的作用**：显式声明 `logs` 为索引数组（普通数组）。敲与不敲的核心区别在于「是否提前定义变量类型」，命令行和脚本中差异不大。

示例1：不敲 `declare -a logs`（默认状态）
```bash
# 1. 查看 logs 变量状态
declare -p logs      # 输出：declare: logs: not found（变量不存在）
# 2. 直接赋值数组（自动创建数组变量）
logs=("access.log" "error.log")
declare -p logs      # 输出：declare -a logs=([0]="access.log" [1]="error.log")（自动变为数组）
# 3. 误赋值为字符串（允许，数组变普通变量）
logs="test.log"
declare -p logs      # 输出：declare -- logs="test.log"（变成普通字符串变量）
```

示例2：敲 `declare -a logs`（显式声明）
```bash
# 1. 执行声明命令
declare -a logs
# 2. 查看 logs 变量状态
declare -p logs      # 输出：declare -a logs=()（空数组，变量已存在）
# 3. 赋值数组（正常生效）
logs=("access.log" "error.log")
declare -p logs      # 输出：declare -a logs=([0]="access.log" [1]="error.log")（数组内容正常）
# 4. 误赋值为字符串（禁止，报错）
logs="test.log"     # 输出：bash: logs: 数组变量赋值时不能使用普通赋值（防止数组被意外改成字符串）
```

### 3. 查看数组

**1、查看数组里的所有元素**
```bash
[root@server ~]# echo ${array[*]}    # * 和 @ 都是代表所有元素
banana apple
[root@localhost ~]# echo ${smile[@]}
xixi hehe haha
```

**2、查看数组里指定的元素**
```bash
[root@localhost ~]# echo ${!smile[@]}   # 查看所有下标
0 1 2
[root@localhost ~]# echo ${smile[2]}   # 查看指定下标的元素
haha
```

**3、查看数组的长度**
```bash
[root@server ~]# echo ${#array[*]}
```
> 常用查看符号：`${arr[*]}`/`${arr[@]}` 取所有元素，`${!arr[@]}` 取所有下标，`${#arr[@]}` 取元素个数，`${arr[n]}` 取第 n 个元素。

### 4. 修改数组

**1、修改数组的名字**
```bash
old_array=("RHEL" "Fedora" "CentOS")
new_array=("${old_array[@]}")   # 完整复制元素
unset old_array                 # 删除原数组
echo ${new_array[@]}            # 输出：RHEL Fedora CentOS
```

**2、修改数组里的元素**
```bash
# 索引数组添加元素
[root@localhost mnt]# arry+=(ahah)
# 显示数组里的内容
[root@localhost mnt]# echo ${arry[@]}
kaixin ahah
# 显示下标索引
[root@localhost mnt]# echo ${!arry[@]}
0 1
# 显示数组里的元素个数
[root@localhost mnt]# echo ${#arry[@]}
2

# 关联数组添加元素
[root@localhost mnt]# arry1+=(ahah)
[root@localhost mnt]# echo ${arry1[@]}
kaixin buhaoyisi wuyu
[root@localhost mnt]# echo ${!arry1[@]}
haha xixi hehe ahah
[root@localhost mnt]# arry1[ahah]="xiangmingbaile"
[root@localhost mnt]# echo ${arry1[@]}
kaixin buhaoyisi wuyu xiangmingbaile
[root@localhost mnt]# echo ${!arry1[@]}
haha xixi hehe ahah
[root@localhost mnt]# echo ${#arry1[@]}
4
```

**3、修改数组里的下标**
```bash
# 场景1：删除元素并重新索引
langs=("Java" "C++" "Rust")    # 原始数组（下标0,1,2）
unset langs[1]                  # 删除下标1的元素
# 重新生成连续下标的新数组
reindexed=("${langs[@]}")
echo ${!reindexed[@]}           # 输出新下标：0 1
echo ${reindexed[@]}            # 输出元素：Java Rust

# 场景2：转换为关联数组（Bash 4+）
declare -A color_codes         # 声明关联数组
color_codes=([red]="#FF0000" [green]="#00FF00")
echo ${color_codes[green]}     # 输出：#00FF00
```

### 5. 删除数组
```bash
# 1、删除数组里的指定元素
[root@server ~]# unset array[shuju1]
# 2、删除整个数组
[root@server ~]# unset array
```

**案例演示**
```bash
[root@localhost mnt]# echo ${!xixi[@]}
0 1 2 3
[root@localhost mnt]# xixi[2]=
[root@localhost mnt]# echo ${!xixi[@]}
0 1 2 3
[root@localhost mnt]# echo ${xixi[@]}
heeh jj /mnt
[root@localhost mnt]# echo ${#xixi[@]}
```
> 说明：将某个下标置空后，下标仍占用位置（`${!xixi[@]}` 仍显示 0 1 2 3），元素个数与剩余元素需用 `${#xixi[@]}` 观察。

## 三、小结
函数通过封装实现代码复用与模块化，重点掌握其**调用与参数传递、两种返回值方式、local 作用域控制、函数库文件的 source 载入**；数组则分为**索引数组**与**关联数组**两类，掌握其创建方式、增删改查、遍历与长度统计。函数与数组结合，能够编写出结构清晰、可复用、可维护的 Shell 脚本。