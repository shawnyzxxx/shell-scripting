# RHCSE Shell 03：条件测试详解

## 文章导读
Shell 条件测试是脚本编程的核心功能，用于根据条件决定程序流程。本文系统讲解**基本语法（test、`[ ]`、`[[ ]]`、`(( ))`）、文件测试、整数测试、字符串测试、逻辑操作符**，并附大量实战练习与最佳实践，适合 Shell 脚本学习者深入掌握条件判断。

## 一、基本语法

### 1. test 命令
```bash
# 基本语法
test condition

# 示例
test -f "/etc/passwd"
echo $?      # 查看退出状态码（0 为真，非 0 为假）
```

### 2. `[ ]` 方括号语法
```bash
# 语法：注意方括号内的空格是必须的！
[ condition ]

[ -f "/etc/passwd" ]
[ "$name" = "root" ]
[ $count -gt 10 ]
```

### 3. `[[ ]]` 双括号语法（Bash 扩展）
功能更强大，支持模式匹配等：
```bash
[[ $file == *.txt ]]
[[ $str1 > $str2 ]]
[[ -n $var && $var != "" ]]
```

### 4. `(( ))` 双括号算术测试
用于算术比较：
```bash
(( arithmetic_expression ))

(( a > b ))
(( count++ ))
(( result = a + b ))
```

## 二、文件测试
文件测试用于检查文件的各种属性。

| 操作符 | 描述 | 示例 |
|---|---|---|
| `-e file` | 文件是否存在 | `[ -e "/etc/passwd" ]` |
| `-f file` | 是否是普通文件 | `[ -f "data.txt" ]` |
| `-d file` | 是否是目录 | `[ -d "/tmp" ]` |
| `-L file` | 是否是符号链接 | `[ -L "/bin/sh" ]` |
| `-b file` | 是否是块设备 | `[ -b "/dev/sda" ]` |
| `-c file` | 是否是字符设备 | `[ -c "/dev/tty" ]` |
| `-p file` | 是否是命名管道 | `[ -p "fifo" ]` |
| `-S file` | 是否是套接字 | `[ -S "/var/run/socket" ]` |
| `-r file` | 是否可读 | `[ -r "file.txt" ]` |
| `-w file` | 是否可写 | `[ -w "file.txt" ]` |
| `-x file` | 是否可执行 | `[ -x "script.sh" ]` |
| `-s file` | 文件大小是否大于 0 | `[ -s "data.txt" ]` |
| `-u file` | 是否设置了 setuid | `[ -u "/usr/bin/passwd" ]` |
| `-g file` | 是否设置了 setgid | `[ -g "/usr/bin/wall" ]` |
| `-k file` | 是否设置了 sticky 位 | `[ -k "/tmp" ]` |
| `-O file` | 当前用户是否是文件所有者 | `[ -O "$HOME/.bashrc" ]` |
| `-G file` | 当前用户是否属于文件组 | `[ -G "file.txt" ]` |
| `-N file` | 文件自上次读取后是否被修改 | `[ -N "log.txt" ]` |
| `file1 -nt file2` | file1 是否比 file2 新 | `[ "a.txt" -nt "b.txt" ]` |
| `file1 -ot file2` | file1 是否比 file2 旧 | `[ "a.txt" -ot "b.txt" ]` |
| `file1 -ef file2` | 是否指向同一文件（硬链接） | `[ "a" -ef "b" ]` |

### 1. 文件存在性和类型测试
```bash
#!/bin/bash
file="test.txt"

# 创建测试文件
echo "test" > "$file"
mkdir test_dir 2>/dev/null

echo "=== 文件存在性和类型测试 ==="

# 文件存在性
[ -e "$file" ] && echo "$file 存在" || echo "$file 不存在"

# 文件类型测试
[ -f "$file" ] && echo "$file 是普通文件"
[ -d "test_dir" ] && echo "test_dir 是目录"
[ -L "/bin/sh" ] && echo "/bin/sh 是符号链接"

# 特殊文件类型
[ -b "/dev/sda" ] && echo "/dev/sda 是块设备文件"
[ -c "/dev/tty" ] && echo "/dev/tty 是字符设备文件"
[ -p "test_fifo" ] && echo "test_fifo 是命名管道"

# 清理
rm "$file"
rmdir test_dir 2>/dev/null
```

### 2. 文件权限测试
```bash
#!/bin/bash
file="permission_test.txt"
echo "test" > "$file"
chmod 644 "$file"    # 设置权限为 rw-r--r--

echo "=== 文件权限测试 ==="
[ -r "$file" ] && echo "文件可读" || echo "文件不可读"
[ -w "$file" ] && echo "文件可写" || echo "文件不可写"
[ -x "$file" ] && echo "文件可执行" || echo "文件不可执行"

# 特殊权限位
chmod +s "$file" 2>/dev/null && [ -u "$file" ] && echo "设置了 setuid 位"
chmod +s "$file" 2>/dev/null && [ -g "$file" ] && echo "设置了 setgid 位"
chmod +t "$file" 2>/dev/null && [ -k "$file" ] && echo "设置了 sticky 位"

rm "$file"
```

### 3. 文件属性测试
```bash
#!/bin/bash
file1="file1.txt"
file2="file2.txt"

echo "content" > "$file1"
echo "content" > "$file2"

echo "=== 文件属性测试 ==="

# 文件大小
[ -s "$file1" ] && echo "文件非空" || echo "文件为空"

# 文件比较
[ "$file1" -nt "$file2" ] && echo "$file1 比 $file2 新"
[ "$file1" -ot "$file2" ] && echo "$file1 比 $file2 旧"
[ "$file1" -ef "$file2" ] && echo "是同一个文件（硬链接）" || echo "不是同一个文件"

# 文件所有权
[ -O "$file1" ] && echo "当前用户是文件所有者"
[ -G "$file1" ] && echo "当前用户属于文件所属组"

# 文件是否被修改
[ -N "$file1" ] && echo "文件自上次读取后被修改过"

rm "$file1" "$file2"
```

**实验练习：判断文件类型与权限**
让用户输入一个文件名，判断文件类型和执行者对该文件的权限。

```bash
read -p "input a filename:" filename
test -z $filename && echo "you must input a filename" && exit 0
test ! -e $filename && echo "the file $filename do not exist" && exit 0
test -f $filename && filetype="regular file"
test -d $filename && filetype="directory"
test -r $filename && perm="readable"
test -w $filename && perm="$perm writable"
test -x $filename && perm="$perm executable"
echo "the $filename is a $filetype"
echo "and the permissions are: $perm"
```
> 说明：由于 root 在很多权限限制上无效，用 root 执行该脚本时，结果常与 `ls -l` 不一致，建议使用一般用户执行。

## 三、整数测试
整数测试用于比较数值大小。

| 操作符 | 描述 | 示例 | `(( ))` 等效写法 |
|---|---|---|---|
| `-eq` | 等于（equal） | `[ $a -eq $b ]` | `(( a == b ))` |
| `-ne` | 不等于（not equal） | `[ $a -ne $b ]` | `(( a != b ))` |
| `-gt` | 大于（greater than） | `[ $a -gt $b ]` | `(( a > b ))` |
| `-ge` | 大于等于 | `[ $a -ge $b ]` | `(( a >= b ))` |
| `-lt` | 小于（less than） | `[ $a -lt $b ]` | `(( a < b ))` |
| `-le` | 小于等于 | `[ $a -le $b ]` | `(( a <= b ))` |

### 1. 基本整数比较
```bash
#!/bin/bash
a=10
b=20
c=10

echo "=== 整数比较测试 ==="
echo "a=$a, b=$b, c=$c"

[ $a -eq $c ] && echo "a 等于 c" || echo "a 不等于 c"
[ $a -ne $b ] && echo "a 不等于 b" || echo "a 等于 b"
[ $b -gt $a ] && echo "b 大于 a" || echo "b 不大于 a"
[ $b -ge $a ] && echo "b 大于等于 a" || echo "b 小于 a"
[ $a -lt $b ] && echo "a 小于 b" || echo "a 不小于 b"
[ $a -le $c ] && echo "a 小于等于 c" || echo "a 大于 c"
```

### 2. 使用 `(( ))` 进行算术测试
```bash
#!/bin/bash
x=15
y=20

echo "=== 使用 (( )) 进行算术测试 ==="
(( x < y )) && echo "x < y"
(( x == 15 )) && echo "x 等于 15"
(( x != y )) && echo "x 不等于 y"
(( x <= y )) && echo "x <= y"
(( y >= x )) && echo "y >= x"

# 复合算术表达式
(( x + 5 == 20 )) && echo "x + 5 = 20"
(( y % x == 5 )) && echo "y % x = 5"
```

## 四、字符串测试
字符串测试用于比较字符串内容。

| 操作符 | 描述 | 示例 |
|---|---|---|
| `=` 或 `==` | 字符串相等 | `[ "$s1" = "$s2" ]` |
| `!=` | 字符串不等 | `[ "$s1" != "$s2" ]` |
| `<` | 在字典序中小于 | `[[ "$s1" < "$s2" ]]` |
| `>` | 在字典序中大于 | `[[ "$s1" > "$s2" ]]` |
| `-z` | 字符串长度为 0（空） | `[ -z "$str" ]` |
| `-n` | 字符串长度非 0 | `[ -n "$str" ]` |
| `=~` | 正则表达式匹配（仅 `[[ ]]`） | `[[ "$str" =~ ^[0-9]+$ ]]` |

```bash
#!/bin/bash
str1="hello"
str2="world"
str3="hello"
empty=""
spaces="   "

echo "=== 字符串测试 ==="

# 字符串相等比较
[ "$str1" = "$str3" ] && echo "str1 等于 str3" || echo "str1 不等于 str3"
[ "$str1" == "$str3" ] && echo "str1 == str3"   # == 是 = 的同义词

# 字符串不等比较
[ "$str1" != "$str2" ] && echo "str1 不等于 str2"

# 字符串长度测试
[ -z "$empty" ] && echo "empty 是空字符串"
[ -n "$str1" ] && echo "str1 是非空字符串"
[ -z "$spaces" ] && echo "spaces 被视为空字符串"

# 在 [[ ]] 中，有空格的字串不会被分割
filename="file with spaces.txt"
[[ $filename == *"with spaces"* ]] && echo "文件名包含 'with spaces'"
```

## 五、逻辑操作符
逻辑操作符用于组合多个测试条件。

### 1. 基本逻辑操作
```bash
#!/bin/bash
file="test.txt"
number=15
text="hello"

echo "=== 逻辑操作测试 ==="
echo "content" > "$file"

# 逻辑与：-a 或 &&
echo "--- 使用 -a（test 和 [ ] 中）---"
[ -f "$file" -a -r "$file" ] && echo "文件存在且可读"

echo "--- 使用 &&（[[ ]] 中或命令连接）---"
[[ -f "$file" && -r "$file" ]] && echo "文件存在且可读"

# 逻辑或：-o 或 ||
echo "--- 使用 -o ---"
[ ! -f "$file" -o -w "$file" ] && echo "文件不存在或可写"

echo "--- 使用 || ---"
[[ ! -f "$file" || -w "$file" ]] && echo "文件不存在或可写"

# 逻辑非：!
echo "--- 使用 ! ---"
[ ! -d "$file" ] && echo "不是目录"
[[ ! $number -eq 10 ]] && echo "number 不等于 10"

rm "$file"
```

### 2. 复杂逻辑组合
```bash
#!/bin/bash
age=25
score=85
name="张三"
file="data.txt"

echo "=== 复杂逻辑组合 ==="

# 组合测试
if [[ $age -ge 18 && $age -le 60 ]] && [[ $score -gt 80 || $name == "张三" ]]; then
    echo "条件满足：年龄在18-60之间，且分数大于80或是张三"
fi

# 使用括号分组（在 [[ ]] 中）
if [[ ($age -gt 18 && $score -ge 60) || $name == "管理员" ]]; then
    echo "条件满足"
fi

# 文件测试组合
if [ -f "$file" ] && { [ -r "$file" ] || [ -w "$file" ]; }; then
    echo "文件存在且可读或可写"
fi
```

## 六、实战练习

**练习0：读入两个整数并比较大小**

```bash
[root@localhost test3]# cat 2.sh
#!/bin/bash
read -p "please input two number:" a b
[ -z "$a" -o -z "$b" ] && {
    echo "please input 'two' number"
    exit 1
}
expr $a + 10 &>/dev/null
return_a=$?
expr $b + 10 &>/dev/null
return_b=$?
[ "$return_a" -eq 0 -a "$return_b" -eq 0 ] || {
    echo "please input two 'number'"
    exit 2
}
[ "$a" -lt "$b" ] && {
    echo "$a < $b"
    exit 0
}
[ "$a" -eq "$b" ] && {
    echo "$a = $b"
    exit 0
}
[ "$a" -gt "$b" ] && {
    echo "$a > $b"
    exit 0
}
```

**测试：**
```bash
[root@localhost test3]# ./2.sh
please input two number:2 4
2 < 4
[root@localhost test3]# ./2.sh
please input two number:6 6
6 = 6
```

**练习1：检查 /etc/passwd 是否存在且可读**
```bash
#!/bin/bash
if [ -e "/etc/passwd" ] && [ -r "/etc/passwd" ]; then
    echo "文件存在且可读"
else
    if [ ! -e "/etc/passwd" ]; then
        echo "错误：文件不存在"
    else
        echo "错误：文件不可读"
    fi
fi
```
简化写法：
```bash
read -p "please input filename:" file
[ ! -e "$file" ] && echo " $file 不存在"
[ -r "$file" ] && echo "$file 存在且可读"
```

**练习2：检查年龄是否在 1 到 120 之间**
```bash
#!/bin/bash
read -p "请输入年龄: " age
if [ "$age" -ge 1 ] && [ "$age" -le 120 ]; then
    echo "年龄有效"
else
    echo "无效年龄"
fi
```

**练习3：判断当前已登录账户数**
若超过 5 个则输出 "Too many"：

```bash
[root@server ~]# num=$(who | cut -d " " -f1 | sort -u | wc -l)
[root@server ~]# [ $num -gt 5 ] && echo "Too many" || echo "已登录账户数:$num"
已登录账户数:1
```
> `who` 查看已登录账户；`cut -d " " -f1` 以空格分割截取第 1 部分；`sort -u` 去重排序；`wc -l` 计算行数。也可改写为 `(( $num > 5 ))`。

**练习4：读入目录名，不存在则创建**
```bash
[root@server ~]# vim temp2.sh
#!/bin/bash
cd /
ls
read -p "请输入一个目录名：" dir
test -d $dir && ls -l $dir || (echo "目录不存在，开始新建..."; mkdir $dir; ls -l $dir)
```

**练习5：读入 x、y，计算和值输出**
```bash
[root@server ~]# vim temp4.sh
#!/bin/bash
read -p "请输入x的值：" x
read -p "请输入y的值：" y
if [ -n "$x" -a -n "$y" ]; then
    if [[ "$x" =~ ^[0-9]+$ ]] && [[ "$y" =~ ^[0-9]+$ ]]; then
        echo $[x+y]
    else
        echo "请输入数字"
    fi
else
    echo "请输入有效内容"
fi
```

**练习6：判断账户类型**
```bash
[root@server ~]# vim temp5.sh
#!/bin/bash
cat /etc/passwd | cut -d ":" -f1 | sort -u
read -p "请输入一个账户名：" us
us_num=$(id -u $us)
if (($us_num==0)); then
    echo "此用户为管理员账户"
else
    if (($us_num>=1 && $us_num<=999)); then
        echo "此账户为系统账户"
    else
        echo "普通账户"
    fi
fi
```

## 七、最佳实践和注意事项

### 1. 变量引用要加引号
```bash
# 安全的方式（总是加引号）
file="my file.txt"
[ -f "$file" ]     # 正确
[ -f $file ]        # 错误！会被分成两个参数

# 检查变量是否设置
[ -n "${VAR:-}" ]   # 安全，即使 VAR 未设置也不会报错
```

### 2. 使用 `[[ ]]` 代替 `[ ]` 的情况
```bash
# 模式匹配（只能用 [[ ]]）
[[ $file == *.txt ]]

# 正则表达式匹配
[[ $email =~ ^[a-z]+@[a-z]+\.[a-z]+$ ]]

# 字符串比较不需要转义
[[ $str1 < $str2 ]]      # 在 [ ] 中需要转义：[ "$str1" \< "$str2" ]

# 更安全的变量扩展
[[ -n $var ]]            # 即使 var 未设置也不会报错
```

### 3. 避免常见错误
```bash
#!/bin/bash

# 错误1：忘记空格
# [condition]    # 错误
[ condition ]    # 正确

# 错误2：使用错误的操作符
# [ $a > $b ]    # 这是重定向，不是比较
[ $a -gt $b ]    # 正确（整数比较）
[[ $a > $b ]]    # 正确（字符串比较）

# 错误3：测试空变量
unset var
# [ $var = "value" ]       # 错误：展开为 [ = "value" ]
[ "${var:-}" = "value" ] # 正确

# 错误4：在 [ ] 中使用 && ||
# [ cond1 && cond2 ]     # 错误
[ cond1 ] && [ cond2 ]  # 正确
[[ cond1 && cond2 ]]    # 正确
```

## 八、小结
Shell 条件测试是脚本逻辑判断的基础。本文梳理了四种测试语法、文件/整数/字符串三类测试操作符、逻辑操作符组合，以及变量引用加引号、优先使用 `[[ ]]`、避免常见错误等最佳实践。熟练掌握这些测试，就能写出健壮、可靠的 Shell 判断逻辑。