# LNMP + LAMP 脚本化部署实验

## 文章导读
本篇是 RHCSE Shell 课程的 LNMP/LAMP 部署实验，通过编写 Shell 脚本**自动化安装 httpd/nginx、mariadb、php 等软件**，完成 Discuz 论坛的部署。实验分 LAMP（Apache）与 LNMP（Nginx）两条路线，适合学习用 Shell 脚本实现运维自动化部署的读者。

## 一、LAMP 实验基本步骤

**1. 下载所需软件**：`lrzsz`、`zip`、`httpd`、`mariadb-server`、`php`、`php-fpm`、`php-mysqlnd`

**2. 转换到 httpd 目录**
```bash
cd /var/www/html/
```

**3. 将 Discuz 软件压缩包拖入并解压**
```bash
[root@shawn2 html]# rz -E
rz waiting to receive.
[root@shawn2 html]# unzip Discuz_X3.5_SC_UTF8_20250901.zip
```

**4. 关闭防火墙并设置宽容模式**
```bash
[root@shawn2 html]# systemctl stop firewalld.service
[root@shawn2 html]# setenforce 0
```

**5. 开启 httpd 功能**
```bash
[root@shawn2 html]# systemctl start httpd
```

**6. 给 upload/ 目录下的文件添加权限**
```bash
[root@shawn2 upload]# chmod -R 777 config/ data/ uc_*
```

**7. 启用 mariadb 服务并创建 luntan 数据库**

**8. 在浏览器上部署完成。**

## 二、编写部署脚本

### 1. mysql.sh（数据库配置脚本）
```bash
#!/bin/bash
mysql_config() {
mysql <<end
alter user 'root'@'localhost' identified by 'Redhat123!';
create database luntan;
exit
end
}
```

### 2. lib.sh（实验环境部署脚本）
```bash
#!/bin/bash

# 安装软件
install_soft() {
    yum install httpd mariadb-server php php-fpm php-mysqlnd -y > /dev/null
    yum install lrzsz -y > /dev/null
    yum install zip -y > /dev/null
}

# 配置 http
http() {
    # 提前把论坛软件放到 /root
    cp /root/Dis* /var/www/html
    cd /var/www/html
    unzip Discuz* > /dev/null && echo "解压成功" || echo "解压失败"
    cd upload/
    chmod -R 777 config/ uc_* data
}

# 配置数据库（示例，注释保留）
# mysql() {
#     systemctl start mariadb.service
#     mysql <<end
#     alter user 'root'@'localhost' identified by 'Redhat123!';
#     create database luntan;
#     exit
# end
# }

# 安全
safe() {
    systemctl stop firewalld
    setenforce 0
}
```

### 3. do.sh（总流程判断脚本）
```bash
#!/bin/bash
source /http/lib.sh
source /http/mysql.sh

install_soft
systemctl start mariadb.service
systemctl start httpd
if [ $? -eq 0 ]; then
    echo 软件安装 is 成功
else
    echo 软件安装 is not 成功
fi

http
if [ $? -eq 0 ]; then
    echo http配置 is 成功
else
    echo http配置 is not 成功
fi

mysql_config
if [ $? -eq 0 ]; then
    echo mysql配置 is 成功
else
    echo mysql配置 is not 成功
fi

safe
if [ $? -eq 0 ]; then
    echo 防火墙、selinux关闭 is 成功
else
    echo 防火墙、selinux关闭 is not 成功
fi
```

## 三、验证脚本

先恢复环境，再启用脚本验证。

**1. 恢复环境**
```bash
[root@shawn2 shell]# yum remove httpd mariadb-server php php-fpm php-mysqlnd -y > /dev/null
[root@shawn2 shell]# yum remove lrzsz -y > /dev/null
[root@shawn2 shell]# yum remove zip -y > /dev/null
[root@shawn2 shell]# rm -rf /var/lib/mysql/     # 特别注意要删除，移除 mariadb 后会有文件残留
```

同时将 `/var/www/html/` 目录下的文件移除，在 `/root` 下新建 `http` 目录，将脚本和论坛软件放入，把软件包复制到 `/root`：

```bash
[root@shawn2 http]# mv Discuz_X3.5_SC_UTF8_20250901.zip /root/
```

**2. 启用脚本**
（http_start 函数已删除，忽略）

**3. 浏览器验证**

## 四、LNMP 实验
恢复环境，关闭防火墙并设置宽容模式，基础步骤如下：

**1. 下载所需软件**：`zip`、`nginx`、`mariadb-server`、`php`、`php-fpm`、`php-mysqlnd`
> `php-fpm.conf` 和 `php.conf` 文件载入 server。

**2. 在 /root 下创建 /web 目录并复制软件包**
```bash
[root@shawn2 ~]# mkdir /web
[root@shawn2 ~]# mv /root/Discuz_X3.5_SC_UTF8_20250901.zip /web/
```

**3. 在 /web 目录下解压软件包，创建 /etc/nginx/conf.d/nginx.conf**
```
server {
    listen 192.168.60.133:80;
    root /web/upload;
    include /etc/nginx/default.d/php.conf
}
```
重启 nginx 服务。

**4. 在浏览器测试**

默认权限不够，更改权限：
```bash
[root@shawn2 ~]# cd /web/upload/
[root@shawn2 upload]# chmod -R 777 config/ data/ uc_*
```

**5. 启用 mariadb 服务并完善数据库部署**

## 五、小结
本实验通过 `mysql.sh`、`lib.sh`、`do.sh` 三个脚本，把 LAMP/LNMP 环境的安装、配置、数据库初始化、安全设置串联成自动化流程，并用 `$?` 判断每步执行结果。这种「函数拆分 + 总流程调度 + 结果判断」的脚本组织方式，是 Shell 运维自动化的典型写法。