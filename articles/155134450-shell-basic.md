# RHCSE Shell 基础详解

## 文章导读
Shell 是用户与操作系统内核之间的桥梁，也是脚本编程的基础。本文系统讲解 Shell 的**操作系统层次模型、发展历程、脚本基本元素（Shebang、注释、命令、变量、输入输出）、read/echo/printf 命令、输入输出重定向、编写规范、脚本执行方式、退出状态码**，并附系统信息报告与用户密码初始化两个实验，适合 Shell 入门学习者。

## 一、Shell 概述

### 1. 操作系统层次模型
```
┌─────────────────────────────────────────┐
│           应用程序 (Applications)         │  ← 用户直接使用的软件
├─────────────────────────────────────────┤
│            Shell (命令行解释器)           │  ← 用户与系统内核的"翻译官"
├─────────────────────────────────────────┤
│            系统调用接口 (System Calls)    │  ← 应用程序与内核的桥梁
├─────────────────────────────────────────┤
│           操作系统内核 (Kernel)           │  ← 核心资源管理器
├─────────────────────────────────────────┤
│             硬件资源 (Hardware)          │  ← CPU、内存、磁盘等
└─────────────────────────────────────────┘
```

### 2. Shell 的核心作用
- **用户与内核的桥梁**：内核直接管理硬件资源（CPU、内存、磁盘、网络等），但普通用户无法直接与内核交互。Shell 作为「命令解释器」，接收用户输入的命令，翻译成内核能理解的指令，再将内核的执行结果返回给用户。
- **保护系统安全**：Shell 对用户命令进行解析和验证，防止恶意或错误的命令直接损害内核，通过权限机制控制用户对系统资源的访问。
- **提供统一的交互接口**：无论底层硬件如何变化，Shell 为用户提供一致的命令行操作体验，隐藏复杂的硬件操作细节，简化系统使用。

> 简单比喻：Shell 就像计算机系统的「前台接待员」或「翻译官」——用户告诉接待员想要什么，接待员转达给后台（内核），再把结果返回给用户。

**Shell 的重要性体现在：**
- **必要性**：保护内核安全，提供统一接口，简化系统操作
- **脚本化**：通过编写脚本实现自动化，提高效率
- **强大功能**：历史、补全、重定向、管道等特性提升使用体验
- **灵活性**：变量、控制结构等编程特性支持复杂逻辑实现

掌握 Shell 脚本编程是系统管理、DevOps 和自动化运维的基础技能，建议通过实际编写脚本巩固这些概念。

### 3. Shell 的发展历史
- **1971 年**：Ken Thompson 开发了第一个 Unix Shell（Thompson Shell）
- **1977 年**：Stephen Bourne 在贝尔实验室开发了 Bourne Shell（sh），成为 Unix 的标准 Shell
- **1978 年**：Bill Joy 在伯克利开发了 C Shell（csh），引入了类 C 语法
- **1983 年**：David Korn 开发了 Korn Shell（ksh），结合了 sh 和 csh 的优点
- **1989 年**：Brian Fox 为 GNU 项目开发了 Bash（Bourne-Again Shell）
- **1990 年**：Paul Falstad 开发了 Z Shell（zsh）

根据 2024 年的数据：Bash 约 80%（Linux 默认，最流行）；Zsh 约 15%（macOS 默认，快速增长）；其他约 5%（ksh、csh、fish 等）。

## 二、Shell 脚本的基本元素

一个完整的 Shell 脚本通常包含以下基本元素：

### 1. Shebang（释伴）
```bash
#!/bin/bash
```
- 必须是脚本的第一行，以 `#!` 开头
- 指定执行此脚本的解释器路径
- 常见的有：`#!/bin/bash`、`#!/bin/sh`、`#!/usr/bin/python` 等

### 2. 注释
```bash
# 这是单行注释

: '
注意：冒号后面要加上空格
这是多行注释
可以写多行内容
'
```

### 3. 命令
- 系统命令：`ls`、`cp`、`mkdir` 等
- 控制结构：`if`、`for`、`while` 等
- 函数调用等

### 4. 变量
```bash
name="value"
```

### 5. 输入输出
**输入：**
1. 直接赋值：`name="li si"`
2. `read` 命令：`read name`
3. 使用位置参数（`$1 $2 $3…`）：`name=$1`
4. 命令输入：`name=$(whoami)`
> 注意：在命令行中定义的变量，退出当前进程后即失效，其他终端上无法使用（切换到其他用户、退出重新登录后变量丢失）。

```bash
var='xiaoming2'
var="$(cmd)"
ip1=192.168.1.251
school="Peking University"
today1=$(date +%F)     # 推荐使用 $() 形式
today2=$(date +%F)
```
> 说明：`$(cmd)` 是推荐的命令替换写法，等价于反引号 `cmd`，但更清晰、更易嵌套。

**输出：** `echo`、`printf`

## 三、输入：read 命令
`read` 命令用于从标准输入（键盘）或文件中读取数据，并将其赋值给变量。

### 1. 基本 read 语法
```bash
# 基本读取
read variable_name

# 带提示信息的读取
read -p "提示信息: " variable_name

# 读取多个变量
read var1 var2 var3

# 从文件描述符读取
read -u fd variable_name
```

### 2. 基本读取示例
```bash
#!/bin/bash

# 基本读取
echo "请输入您的名字:"
read name
echo "你好, $name!"

# 一行完成提示和读取
read -p "请输入您的年龄: " age
echo "您今年 $age 岁"

# 读取多个值
echo "请输入三个数字（用空格分隔）:"
read num1 num2 num3
echo "您输入的数字是: $num1, $num2, $num3"
```

### 3. read 命令的常用选项

**`-p`：显示提示信息**
```bash
#!/bin/bash

# 使用 -p 选项显示提示
read -p "请输入用户名: " username
read -p "请输入密码: " -s password   # -s 用于隐藏输入
echo
echo "用户: $username, 密码已设置"
```

**`-t`：设置超时时间**
```bash
#!/bin/bash

# 5秒超时
if read -t 5 -p "请在5秒内输入内容: " input; then
    echo "您输入了: $input"
else
    echo "时间到，未输入任何内容"
    exit 1
fi
```

**`-n`：限制输入字符数**
```bash
#!/bin/bash

# 读取固定长度
read -n 5 -p "请输入5个字符: " text
echo
echo "您输入了: $text"
```

## 四、输出：echo 命令
`echo` 命令用于输出文本到标准输出。

### 1. 基本 echo 语法
```bash
# 基本输出
echo "Hello World"

# 输出变量
echo "变量值: $variable"

# 输出多个参数
echo "第一个" "第二个" "第三个"

# 输出到文件
echo "内容" > file.txt
echo "追加内容" >> file.txt
```

### 2. echo 选项
**`-n`：不输出尾随换行符**
```bash
#!/bin/bash

echo -n "正在处理..."
sleep 2
echo "完成"
# 输出：正在处理...完成（在同一行）
```

**`-e`：启用反斜杠转义**
```bash
#!/bin/bash

# 启用转义字符解释
echo -e "第一行\n第二行"
echo -e "制表符:\t前\t后"
echo -e "回退\b字符"   # 删除前一个字符
```

转义序列：

| 序列 | 含义 |
|---|---|
| `\b` | 退格 |
| `\c` | 不继续输出（相当于 -n） |
| `\e` | 转义字符 |
| `\f` | 换页 |
| `\n` | 换行 |
| `\r` | 回车 |
| `\t` | 水平制表符 |
| `\v` | 垂直制表符 |
| `\\` | 反斜杠 |

**`-E`：禁用反斜杠转义（默认）**
```bash
#!/bin/bash

# 禁用转义（默认行为）
echo -E "第一行\n第二行"
# 输出：第一行\n第二行（原样输出）
```

### 3. echo 高级用法（彩色文本）
```bash
#!/bin/bash

# 输出彩色文本
echo -e "\033[31m红色文字\033[0m"
echo -e "\033[32;44m绿字蓝底\033[0m"

# 定义颜色函数
red() { echo -e "\033[31m$@\033[0m"; }
green() { echo -e "\033[32m$@\033[0m"; }

red "这是错误信息"
green "这是成功信息"
```

## 五、输出：printf 命令
`printf` 命令提供更强大的格式化输出功能，类似于 C 语言的 printf 函数。

### 1. 基本 printf 语法
```bash
printf format-string [arguments...]
```

### 2. 格式说明符

| 说明符 | 描述 | 示例 |
|---|---|---|
| `%s` | 字符串 | `printf "%s" "hello"` |
| `%d` | 十进制整数 | `printf "%d" 100` |
| `%f` | 浮点数 | `printf "%.2f" 3.14159` |
| `%x` | 十六进制整数 | `printf "%x" 255` |
| `%c` | 字符 | `printf "%c" 65`（输出 A） |
| `%b` | 字符串（解释反斜杠转义） | `printf "%b" "line1\nline2"` |

### 3. printf 基本用法
```bash
#!/bin/bash

# 基本格式化
printf "姓名: %s, 年龄: %d\n" "张三" 25

# 宽度和对齐
printf "%-10s %5d\n" "Alice" 30    # 左对齐
printf "%10s %5d\n" "Bob" 25       # 右对齐

# 浮点数格式化
printf "π ≈ %.2f\n" 3.14159
# 前导零
printf "带前导零: %05d\n" 42
# 千位分隔符（某些系统支持）
printf "千位分隔: %'d\n" 1234567
```

对齐示例：
```bash
[root@rhce ~]# a=Alice
[root@rhce ~]# b=3
[root@rhce ~]# printf "%-10s %5d\n" "$a" "$b"
Alice          3
```

## 六、输入输出重定向
```bash
#!/bin/bash

# 输出重定向
echo "内容" > file.txt          # 覆盖写入
echo "追加内容" >> file.txt     # 追加写入

# 输入重定向
read var < file.txt              # 从文件读取
wc -l < file.txt                # 统计行数

# 错误输出重定向
command 2> error.log             # 错误信息重定向
command > output.log 2>&1        # 标准输出和错误都重定向
command &> all_output.log        # 同上（更简洁）
```

**here document（here-doc）示例**
```bash
[root@rhce ~]# passwd << end
redhat
redhat
end
```
> 通过 here document 可向命令批量传入多行输入。

## 七、Shell 脚本编写规范
良好的编写规范能提高脚本的可读性和可维护性。

### 1. 文件命名规范
- 使用 `.sh` 扩展名，如 `backup_files.sh`
- 文件名应见名知意，使用小写字母和下划线

### 2. 脚本头部信息
```bash
#!/bin/bash
# 脚本名称: system_info.sh
# 描述: 显示系统基本信息
# 作者: Your Name
# 创建日期: 2024-10-28
# 版本: 1.0
```

编辑 `.sh` 文件时自动生成脚本说明注释，可在 `~/.vimrc` 配置：
```vim
autocmd BufNewFile *.py,*.cc,*.sh,*.java exec ":call SetTitle()"
func SetTitle()
  if expand("%:e") == 'sh'
    call setline(1,"#!/bin/bash")
    call setline(2,"#########################")
    call setline(3,"#File name:".expand("%"))
    call setline(4,"#Version:v1.0")
    call setline(5,"#Email:admin@test.com")
    call setline(6,"#Created time:".strftime("%F %T"))
    call setline(7,"#Description:")
    call setline(8,"#########################")
  endif
endfunc
```

### 3. 代码格式规范
- 运算符两边加空格：`name="value"` → `name = "value"`
- 流程控制语句格式规范：
```bash
# 好的格式
if [ condition ]; then
    commands
fi

for item in list; do
    commands
done
```

### 4. 变量命名规范
- 局部变量使用小写：`file_name`
- 常量使用大写：`MAX_RETRY=3`
- 环境变量使用大写：`DATABASE_URL`
> 注意：变量只能由字母、数字、下划线组成，不能有其他特殊字符，并且不能以数字开头。

## 八、Shell 脚本执行方法

### 1. 作为可执行文件执行（推荐）
```bash
# 添加执行权限
chmod +x script.sh

# 执行脚本
./script.sh
```

### 2. 指定解释器执行
```bash
# 不需要执行权限
bash script.sh
sh script.sh
```

### 3. 使用 source 或点号执行
```bash
# 在当前 Shell 环境中执行，变量会保留
source script.sh
. script.sh     # 点号+空格+脚本名
```

### 4. 三种执行方式的区别

| 执行方式 | 语法 | 是否创建子 Shell | 环境变量影响 |
|---|---|---|---|
| `./script.sh` | 需要执行权限 | 是 | 不影响父 Shell |
| `bash script.sh` | 直接执行 | 是 | 不影响父 Shell |
| `source script.sh` | 在当前 Shell 执行 | 否 | 影响当前 Shell |

**示例验证**
```bash
# 创建测试脚本 test_env.sh
echo "当前Shell PID: $$"
export TEST_VAR="hello"
```
> 使用 `export` 导出，变量可被子进程继承。

```bash
# 方式1：创建子Shell
./test_env.sh       # 显示子Shell的PID，TEST_VAR不会影响当前环境
# 方式2：在当前Shell执行
source test_env.sh  # 显示当前Shell的PID，TEST_VAR会在当前环境生效
echo $TEST_VAR      # 输出: hello
```

实测：
```bash
[root@rhce shell]# /shell/test.sh
当前Shell PID: 15183
[root@rhce shell]# source /shell/test.sh
当前Shell PID: 14656
[root@rhce shell]# bash /shell/test.sh
当前Shell PID: 15184
```
> `source` 执行时 PID 与当前 Shell 一致（14656），而 `./`、`bash` 会创建子 Shell（PID 不同）。

## 九、Shell 脚本退出状态码

### 1. 什么是退出状态码
- 每个命令执行后都会返回一个数字状态码
- `0` 表示成功，非 0 表示失败
- 通过 `$?` 变量获取上一个命令的退出状态

### 2. 常见状态码含义

| 状态码 | 含义 | 说明 |
|---|---|---|
| `0` | 成功 | 命令执行成功 |
| `1` | 一般性错误 | 未知错误 |
| `2` | 误用 Shell 命令 | 权限不足等 |
| `126` | 命令不可执行 | 没有执行权限 |
| `127` | 命令未找到 | 命令不存在 |
| `128+N` | 信号 N 终止 | 如 130=128+2（Ctrl+C） |
| `255` | 退出状态越界 | 超出 0-255 范围 |

### 3. 自定义退出状态码
```bash
#!/bin/bash

check_file() {
    if [ -f "$1" ]; then
        echo "文件存在"
        return 0    # 成功
    else
        echo "文件不存在"
        return 1    # 失败
    fi
}

check_file "/etc/passwd"
echo "检查结果: $?"

# 脚本退出时指定状态码
if [ $? -eq 0 ]; then
    exit 0    # 成功退出
else
    exit 1    # 失败退出
fi
```

## 十、实验

### 实验1：系统信息报告脚本
创建脚本输出基本系统状态信息（当前登录用户、报告生成时间、磁盘使用情况、内存使用情况）：
```bash
#!/bin/bash
# 简单的系统信息报告脚本

# 定义变量
REPORT_TITLE="=== 系统基本信息报告 ==="
CURRENT_USER=$(whoami)
CURRENT_TIME=$(date)
DISK_USAGE=$(df -h /)
MEMORY_INFO=$(free -h)

# 输出报告
echo $REPORT_TITLE
echo "报告生成时间: $CURRENT_TIME"
echo "当前用户: $CURRENT_USER"
echo ""
echo "磁盘使用情况 (根分区):"
echo "$DISK_USAGE"
echo ""
echo "内存使用情况:"
echo "$MEMORY_INFO"
echo ""
echo "=== 报告结束 ==="
```

### 实验2：用户存在性判断并初始化密码
提示用户输入用户名称，判断用户是否存在，无论存在与否都初始化用户密码为 redhat；如果用户不存在就创建用户，然后初始化密码。
```bash
mima() {
    echo "input user:$name new passwd:"
    passwd $name << end
redhat
redhat
end
    echo "$name 密码修改成功"
}

read -p "please input name:" name
id $name
if [ $? -eq 0 ]; then
    echo "$name 已经存在"
    mima
else
    echo "$name 不存在，正在创建"
    useradd $name
    echo "$name 创建成功"
    mima
fi
```

## 十一、小结
本文从 Shell 的操作系统层次模型出发，系统讲解了 Shell 的发展历程、脚本基本元素、read/echo/printf 输入输出、重定向、编写规范、执行方式与退出状态码，并通过两个实验巩固应用。掌握这些基础，是进一步学习 Shell 流程控制、函数与文本处理的前提。