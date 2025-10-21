# [Quillset.com](https://quillset.com)

[TOC]

# 服务器创建笔记

*Written By Dioxgen*

## 使用的硬件：

> 树莓派 5 8GB
>
> PCIe-M.2*2-HAT
>
> UPS-HAT-E
>

## 系统：

> Raspberry Pi OS (64-bit)
>
> Ubuntu Desktop 24.04.3 LTS (64-bit)
>

我将跳过 Raspberry Pi OS 的安装，直接从进入桌面开始。

本笔记若想复现，配合 LLM 体验更佳。

## 基本设置

### 1）修改时区：

```bash
sudo dpkg-reconfigure tzdata
```

找到 `Asia`，然后选择 `shanghai` 即可。

### 2）解除电源限制：

> 接下来的操作将覆盖最大电流值，让系统认为它可以获得完整的 5A 电流，从而获得最佳性能。这通常用在非官方电源、UPS 设备等给树莓派的供电上，属于高级操作，存在一定风险，请务必谨慎。

编辑树莓派 EEPROM：

```bash
sudo rpi-eeprom-config --edit
```

在最后一行添加：`PSU_MAX_CURRENT=5000`，保存并退出。

### 3）设置静态 IP：

[为 Ubuntu 24.04 LTS 设置固定 IP 地址](https://www.sysgeek.cn/set-static-ip-ubuntu/)

查看 IP：

```bash
hostname -I
```

------

## 在 SSD 上运行 Ubuntu

支持的 SSD 列表：[NVMe compatibility list - Pineberry Pi Documentaton](https://docs.pineberrypi.com/nvme-compatibility-list)

### 1）挂载 SSD：

首先编辑 `config` 文件，以开启 PCIe。

```bash
sudo gedit /boot/firmware/config.txt
```

`[CM5]` 和 `[ALL]` 之间添加：

```
dtparam=pciex1
```

> SSD 的分区，格式化并不必要，新的系统会直接安装到 SSD 中。

### 2）配置从 SSD 启动：

```bash
sudo raspi-config
```

按 `6 - A4 - B2 <OK>` 的顺序配置 EEPROM。

用系统自带的 Raspberry Pi Imager 烧录 Ubuntu Desktop 24.04.3 LTS (64-bit)，重启。

------

### # 其他：

#### `raspi-config`：

安装依赖：

```bash
sudo apt install whiptail parted lua5.1 alsa-utils psmisc
```

从 Raspberrypi 官网下载最新的 deb 安装包（2025.09）：

```bash
wget https://archive.raspberrypi.org/debian/pool/main/r/raspi-config/raspi-config_20250902_all.deb
```

安装安装包：

```bash
sudo dpkg -i raspi-config_20200707_all.deb
```

运行 `raspi-config`：

```bash
sudo raspi-config
```
#### 获取 root 权限：

在终端中输入：

```bash
sudo passwd root	#设置密码
su root				#获得 root
```

如果要禁用 root 帐号，那么可以执行：

```bash
sudo passwd -l root
```

#### 创建/删除文件（夹）：

##### `mkdir`:

可以使用 `mkdir` 命令来创建新的文件夹：

```bash
mkdir 目录名
```

创建多个文件夹：

```bash
mkdir 文件夹1 文件夹2 文件夹3
```

创建多级文件夹：

```bash
mkdir -p 父级文件夹/子级文件夹
```

删除文件夹：

```bash
rm -rf 目录名
```

> 以上命令将会删除目录并且向下穿透，其下所有文件、文件夹都会被删除。

##### `touch`：

`touch` 命令是最简单的创建空文件的方法。如果指定的文件不存在，`touch` 会创建一个新的空文件；如果文件已存在，`touch` 会更新文件的访问和修改时间。

例如：

```bash
touch newfile.txt
```

如果要一次性创建多个文件，可以这样操作：

```bash
touch file1.txt file2.txt file3.txt
```

删除文件：

```bash
rm -f /.../.txt
```


#### Screen：

[远程神器 screen 命令的保姆级详解教程](https://blog.csdn.net/weixin_39925939/article/details/121033427)

Screen 是一个全屏窗口管理器，用户可以通过该软件同时连接多个本地或远程的命令行会话，并在其间自由切换。

新建会话：

```bash
screen -S session_name
```

远程 dettach 某个会话：

```bash
screen -d session_name #session_name 也可以是对应的 session id
```

进入 dettached 的会话：

```bash
screen -r session_name #session_name 也可以是对应的 session id
```

进入 attached 的会话：

如果一个会话的状态是 `Attached` 的，而你并不在其中，我们就要输入 `screen -d <screen的pid>` ，来使他 ”`Dettached`”，然后再输入 `screen -r <screen的pid>`，来进入这个 screen。

列出当前所有的 session：

```bash
screen -ls
```

关闭会话：

如果在会话之中，输入 `exit` 或者 `Ctrl+d` 来终止这个会话。成功终止后，如果有其他处于 Attached 状态的 screen 界面，他就会跳到那个界面中，如果没有，他就会跳到默认界面上。

删除会话：

```bash
screen -X -S session_name quit
```

dettach 子窗口：

```bash
Ctrl+a d #进入screen窗口后，想暂时退出
```

#### 系统更新：

```bash
sudo apt update && sudo apt upgrade
```

#### `systemctl`：

```bash
# 停止/开启 service
sudo systemctl stop service
sudo systemctl start service
# 禁用/启用 service 开机启动
sudo systemctl disable service
sudo systemctl enable service
# 检查状态
sudo systemctl status service
```

------

## WiFi 设置

> 若不想在 Ubuntu 设置树莓派自带的 WiFi 网卡，可直接使用以太网接口联网以跳过该设置。

### 网速测试：

```bash
sudo apt install speedtest-cli	#安装 speedtest-cli
speedtest						#开始测试
```



------

## 设置 SSH

接下来介绍基于 **OpenSSH** 从 Windows 连接 Ubuntu 的方法：

### 1）Windows 端：

[适用于 Windows 的 OpenSSH 入门](https://learn.microsoft.com/zh-cn/windows-server/administration/openssh/openssh_install_firstuse)

### 2）Ubuntu 端：

更新软件包列表：

```bash
sudo apt update
```

安装 OpenSSH 服务器：

```bash
sudo apt install openssh-server
```

启动 SSH 服务并设置开机自启：

```bash
sudo systemctl start ssh  	# 启动服务
sudo systemctl enable ssh 	# 设置开机自动启动
```

检查 SSH 服务状态，确认 SSH 服务是否正常运行：

```bash
sudo systemctl status ssh
```

如果看到 `active` 的字样，说明服务已成功启动。

#### 配置防火墙（UFW）：

检查端口：

```bash
sudo lsof -i:端口号
```

检查防火墙状态：

```bash
sudo ufw status
```

若 `inactive`：

```bash
sudo ufw enable     #启动防火墙
sudo ufw allow ssh  #允许端口 SSH（22）
sudo ufw reload     #重新加载防火墙
```

再次检查防火墙状态：

```bash
sudo ufw status
```

应当 “`active`”，且列表展示 `22` 端口 `Anywhere` `ALLOW`。

查找 Ubuntu IP：

```bash
ip addr show
或
hostname -I
```

在 Windows 上打开 `PowerShell` 或 `CMD`，使用以下命令连接：

```CMD
ssh [Ubuntu用户名]@[Ubuntu的IP]
或
ssh -p port username@xxx.com	#适用于内网穿透场景
```

如，如果你的用户名是 `crow`，服务器 IP 是 `192.168.1.3`，则：

```CMD
ssh crow@192.168.1.3
```

随后将要求输入用户在 Ubuntu 上的登录密码。输入时密码不可见，这是正常的安全设计，输完后直接按回车即可。如果密码正确，将成功登录到 Ubuntu 的命令行界面。

### 3）高级配置（可选）

#### 使用 SSH 密钥认证（更安全，免密登录）：

“主机密钥” 是一种用来防止 “中间人攻击” 的机制。

在 Windows 上生成密钥对：

```CMD
ssh-keygen -t rsa -b 4096
```

默认保存在 `~\.ssh\id_rsa`（私钥）和 `~\.ssh\id_rsa.pub`（公钥）。

将公钥上传到 Ubuntu：

在本地 Windows 上安装 `Git`，然后在密钥文件的保存路径右键选择 **Git Bash Here**，使用以下命令将公钥复制到远程 Ubuntu：

```CMD
ssh-copy-id -p port crow@192.168.1.3
```

其中 `-p port` 只在需要指定端口的时候使用（为安全设置别的端口、Doker 容器……），默认的 22 端口不需要指定。

或者手动将公钥（`~\.ssh\id_rsa.pub`）中内容复制到 Ubuntu 的 `~/.ssh/authorized_keys` 文件中。如此，再次连接时，SSH 将优先尝试使用密钥认证。

#### 更改 SSH 端口：

为增强安全性，可以修改 SSH 服务的默认端口（22）。

在 Ubuntu 上编辑 SSH 配置文件：

```bash
sudo nano /etc/ssh/sshd_config
```

找到 `#Port 22` 这一行，删除注释，并改为一个未被使用的端口号，如 `2233`，保存并退出。

重启 SSH 服务：

```bash
sudo systemctl restart ssh
```

使新端口在防火墙中开放：

```bash
sudo ufw allow 2233/tcp
```

连接时指定端口：

```CMD
ssh -p 2222 crow@192.168.1.3
```

#### 退出 SSH 会话：

```bash
exit
或
logout
或
Ctrl + D
```

#### 公网访问：

这里以 **Tailscale** 为例。

##### Windows 端：

[Download | Tailscale](https://tailscale.com/download/windows)

##### Ubuntu：

[Install Tailscale on Ubuntu 24.04 (noble) · Tailscale Docs](https://tailscale.com/kb/1476/install-ubuntu-2404)

添加 Tailscale 的软件包签名密钥和仓库：

```bash
curl -fsSL https://pkgs.tailscale.com/stable/ubuntu/noble.noarmor.gpg | sudo tee /usr/share/keyrings/tailscale-archive-keyring.gpg >/dev/null
curl -fsSL https://pkgs.tailscale.com/stable/ubuntu/noble.tailscale-keyring.list | sudo tee /etc/apt/sources.list.d/tailscale.list
```

下载 Tailscale：

```bash
sudo apt-get update
sudo apt-get install tailscale
```

将你的设备连接到你的 Tailscale 网络，并在浏览器中进行认证：

```bash
sudo tailscale up
```

你可以通过运行以下命令找到你的 Tailscale IPv4 地址：

```bash
tailscale ip -4		#查看 IP
tailscale status	#检查 tailscale 状态
```

> 如：100.103.152.81

如果您添加的设备是服务器或可远程访问的设备，您可能需要考虑禁用[密钥过期](https://tailscale.com/kb/1028/key-expiry)功能，以避免需要定期重新认证。

现在，可以直接使用 SSH 连接：

```CMD
ssh crow@100.103.152.81
```

##### tailscale ssh：

若**直接**尝试以下命令连接：

```CMD
tailscale ssh [user@]<host>		#tailscale 连接
```

将会报错：

```CMD
No ED25519 host key is known for XXX.net. and you have requested strict checking.
Host key verification failed.
```

原因是 Windows 从未见过这台 Linux 服务器，不认识它的身份指纹，因此拒绝连接。想要解决，只需手动运行一次 SSH 命令来触发提示并接受密钥即可：

```CMD
ssh [user@]<host>
```

这里不使用 `tailscale ssh` 命令，先用最基础的 SSH 连接来确保主机密钥被正确记录。

命令执行后，将会看到类似下面的提示：

```CMD
The authenticity of host 'XXX.XXX.XXX.XXX' can't be established.
ED25519 key fingerprint is SHA256:xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx.
Are you sure you want to continue connecting (yes/no/[fingerprint])?
```

选 `yes`，回车即可。

##### 普通 `ssh` 与 `tailscale ssh` 的区别：

| 命令                | 优点                                  | 缺点                                                |
| :------------------ | :------------------------------------ | :-------------------------------------------------- |
| **`tailscale ssh`** | 安全、便捷，无需配置服务端 SSH 密钥   | 功能可能没有原生 SSH 丰富，依赖 Tailscale 基础设施  |
| **`ssh`**           | 功能全面、通用，不依赖 Tailscale 服务 | 需要手动管理 SSH 密钥，分散式管理，安全性配置更复杂 |

对于日常管理和快速访问，推荐使用 `tailscale ssh`，它能带来巨大的便利性和安全性提升。

当你需要更多控制或特定功能时，再回退到传统的 `ssh` 命令即可。

两者可以共存以根据场景自由选择。

------

## 安装 PHP

### 安装 PHP 和 PHP-FPM：

WordPress 要求 PHP 7.3 或更高版本。你需要安装 PHP 本身以及 PHP-FPM（FastCGI Process Manager），因为 Nginx 是通过 PHP-FPM 来处理 PHP 文件的。

```bash
sudo apt install php-fpm php-mysql
```

以上命令安装了 PHP-FPM 和用于连接 MySQL 的 PHP 扩展。

#### 安装其他常用 PHP 扩展：

为了确保 WordPress 及其插件、主题功能完整，你最好再安装一些常用的扩展：

```bash
sudo apt install php-curl php-json php-mbstring php-xml php-zip php-gd php-curl php-xmlrpc
```

安装完成后，你可以通过以下命令检查已安装的 PHP 版本，确保其满足 WordPress 的要求：

```bash
php -v
```

> 如果默认安装的 PHP 版本过低（例如低于7.3），你可能需要通过添加第三方PPA（如 `ondrej/php`）来安装更新的版本。

## 安装 MySQL

### 安装与配置：

在终端中运行以下命令来安装 MySQL：

```bash
sudo apt install mysql-server
```

安装完成后，运行安全脚本以设置 `root` 密码并移除一些不安全配置：

```bash
sudo mysql_secure_installation
```

根据脚本提示，你通常会：

- 设置 **root 用户的密码**。
- **移除匿名用户**。
- **禁止 root 用户远程登录**。
- **移除测试数据库**。
- **重新加载权限表**使设置生效。

### 为 WordPress 创建数据库和用户：

登录 MySQL 并为 WordPress 创建一个专用的数据库和用户，这样更安全。

```bash
sudo mysql -u root -p
```

在 MySQL 提示符下，执行以下命令（请将 `your_password` 替换为一个强密码）：

```sql
CREATE DATABASE wordpress;
CREATE USER 'wordpressuser'@'localhost' IDENTIFIED BY 'your_password';
GRANT ALL PRIVILEGES ON wordpress.* TO 'wordpressuser'@'localhost';
FLUSH PRIVILEGES;
EXIT;
```

------

## Nginx（稳定版）

[在 Ubuntu 24.04 LTS 上安装 Nginx](https://www.sysgeek.cn/install-nginx-ubuntu/)

更新 Ubuntu，然后：

```bash
sudo apt install curl gnupg2 ca-certificates lsb-release ubuntu-keyring dirmngr software-properties-common apt-transport-https
curl -fSsL https://nginx.org/keys/nginx_signing.key | sudo gpg --dearmor | sudo tee /usr/share/keyrings/nginx-archive-keyring.gpg >/dev/null
gpg --dry-run --quiet --import --import-options import-show /usr/share/keyrings/nginx-archive-keyring.gpg
echo "deb [signed-by=/usr/share/keyrings/nginx-archive-keyring.gpg] http://nginx.org/packages/ubuntu `lsb_release -cs` nginx" | sudo tee /etc/apt/sources.list.d/nginx.list
echo -e "Package: *\nPin: origin nginx.org\nPin: release o=nginx\nPin-Priority: 900\n" | sudo tee /etc/apt/preferences.d/99nginx
sudo apt install nginx
```

安装完成后，可以通过以下命令验证：

```bash
nginx -v
```

设置开机自启动：

```bash
sudo systemctl enable nginx   # 启用开机自启动
sudo systemctl disable nginx  # 禁止开机自启动
```

防火墙允许 Nginx（80）：

```bash
sudo ufw allow 'Nginx Full'
```

打开浏览器访问 `192.168.1.3`（你的 Ubuntu IP 地址），若可以看到 Nginx 的页面，则说明安装好了：

> ### Welcome to nginx!
>
> If you see this page, the nginx web server is successfully installed and working. Further configuration is required.
>
> For online documentation and support please refer to [nginx.org](http://nginx.org/).
>
> Commercial support is available at [nginx.com](http://nginx.com/).
>
> *Thank you for using nginx.*

### 配置 Nginx：

```
# Nginx
配置文件: /etc/nginx/
服务文件: /etc/nginx/nginx.conf
网站文件: /var/www/html/
日志文件: /var/log/nginx/
```

#### 修改主页：

主页的文件通常位于 `/var/www/html` 下，为 `index.nginx-debian.html`：

```html
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<style>
html { color-scheme: light dark; }
body { width: 35em; margin: 0 auto;
font-family: Tahoma, Verdana, Arial, sans-serif; }
</style>
</head>
<body>
<h1>Welcome to nginx!</h1>
<p>If you see this page, the nginx web server is successfully installed and
working. Further configuration is required.</p>

<p>For online documentation and support please refer to
<a href="http://nginx.org/">nginx.org</a>.<br/>
Commercial support is available at
<a href="http://nginx.com/">nginx.com</a>.</p>

<p><em>Thank you for using nginx.</em></p>
</body>
</html>
```

在该目录下新建你的网页 `index.html`，填充内容，然后检查 Nginx 的配置文件语法是否正确。这可以避免因配置错误导致服务无法启动：

```bash
sudo nginx -t
```

如果显示  `syntax is ok`  和  `test is successful`，就可以安全地重启 Nginx 以使更改生效：

```bash
sudo systemctl reload nginx
或
sudo systemctl restart nginx
```

通常推荐使用  `reload`  来进行平滑重载，避免中断现有连接。

#### 关于 HTTPS 配置：

这里连带加上 SSL 证书一并处理，关于 SSL 证书获取，详见下一章节。

配置文件的核心逻辑如下：

> 将 HTTP 和 HTTPS 分开成两个 `server` 块：
>
> 1. 第一个 `server` 块监听 80 端口，将 HTTP 请求重定向到 HTTPS。
> 2. 第二个 `server` 块监听 443 端口，处理 HTTPS 请求，并配置 SSL。

```ini
#nginx.conf    
    # HTTP 服务器 - 重定向到 HTTPS
    server {
        listen 80;
        server_name *.com;	#你的域名
        
        # 将 HTTP 重定向到 HTTPS
        return 301 https://$server_name$request_uri;
    }

    # HTTPS 服务器
    server {
        listen 443 ssl;
        server_name *.com;
        
        # SSL 证书配置 - 使用完整路径
        ssl_certificate /etc/nginx/ssl/*.com_bundle.crt;
        ssl_certificate_key /etc/nginx/ssl/*.com.key;
        
        ssl_session_timeout 5m;
        ssl_protocols TLSv1.2 TLSv1.3;
        ssl_ciphers ECDHE-RSA-AES128-GCM-SHA256:HIGH:!aNULL:!MD5:!RC4:!DHE;
        ssl_prefer_server_ciphers on;

        # 网站根目录
        root /var/www/html;
        index index.html index.htm;

        location / {
            try_files $uri $uri/ /index.html;
        }

        error_page 500 502 503 504 /50x.html;
        location = /50x.html {
            root /usr/share/nginx/html;
        }
    }
```

> 注意：以上配置只展示了 `server` 块，并非完整 `nginx.conf`。

#### 配置 Nginx 以支持 PHP：

现在，需要让 Nginx 知道如何处理 PHP 文件。

##### 编辑 Nginx 站点配置文件：

在 `server` 块中，需要进行以下几处修改，以确保 Nginx 能正确识别和处理 PHP 请求，并优化 WordPress 的固定链接和静态文件处理：

> 设置索引：确保 index 指令中包含 index.php。
>
> 配置 PHP 处理：取消注释或添加处理 .php 文件的 location 块。
>
> 优化 WordPress：在 location / 块中修改 try_files 指令，以支持 WordPress 的固定链接；并添加一些缓存静态文件的规则。

以下是修改后的 `server` 块配置示例：

```ini
server {
    location / {
        try_files $uri $uri/ /index.php?$args; # 此项对WordPress固定链接很重要
    }

    location ~ \.php$ {
        include snippets/fastcgi-php.conf;
        fastcgi_pass unix:/var/run/php/php8.3-fpm.sock; # 确保此路径与您安装的PHP版本匹配
    }

    location ~* \.(js|css|png|jpg|jpeg|gif|ico|svg)$ {
        expires max;
        log_not_found off; # 可选：对静态文件进行缓存
    }

    location ~ /\.ht {
        deny all; # 禁止访问.htaccess文件
    }
}
```

##### 测试并重启 Nginx：

配置修改完成后，使用以下命令检查配置语法是否正确，若无误则重启 Nginx：

```bash
sudo nginx -t
sudo systemctl restart nginx
```

确保 PHP-FPM 服务正在运行：

```bash
sudo systemctl start php8.3-fpm  # 请将 '8.3' 替换为你的实际版本号
sudo systemctl enable php8.3-fpm
```

###### 测试 PHP：

在继续安装 WordPress 之前，最好确认 Nginx 已经能够正确处理 PHP 文件。

创建 PHP 信息测试文件：

在 Nginx 的根目录下创建一个 `info.php` 文件：

```bash
sudo nano /var/www/html/info.php
```

写入：

```php
<?php

phpinfo();
```

随后打开浏览器，访问 http://您的服务器 IP 或域名/info.php。如果能看到详细的 PHP 配置信息页面，说明 Nginx 和 PHP-FPM 的协作是正常的。

测试成功后，务必删除这个测试文件，以防止泄露服务器信息：

```bash
sudo rm /var/www/html/info.php
```

有时 `error.log` 中会报错：`(13: Permission denied)`

也就是说权限不足，查看此文件属性发现属于不同用户的，改为 `nginx` 用户即可。

然而推荐将 Nginx 工作进程用户改为 `www-data` 以匹配 PHP-FPM，因为 `www-data` 是 Ubuntu 上更标准的 Web 服务用户。

- 以下将 PHP-FPM 用户改为 `nginx` 以匹配 Nginx（不推荐，可能导致 Wordpress 安装困难）：

```bash
sudo nano /etc/php/8.3/fpm/pool.d/www.conf
```

修改：

```ini
listen.owner=nginx
listen.group=nginx
```

然后修改 `php-fpm.sock` 文件的属性：

进入此文件所在目录：

```bash
cd /var/run/php
```

```bash
chown nginx php-fpm.sock
chgrp nginx php-fpm.sock
```

重启 nginx 和 PHP：

```bash
sudo systemctl reload nginx
sudo systemctl restart php8.3-fpm
```

[connect() to unix:/var/run/php-fpm/php-fpm.sock failed (13: Permission denied)-CSDN博客](https://blog.csdn.net/chesterblue/article/details/100081797)

- 将 Nginx 工作进程用户改为 `www-data` 以匹配 PHP-FPM（推荐）：

```bash
# 1. 编辑nginx配置文件
sudo nano /etc/nginx/nginx.conf

# 2. 找到 user 指令，修改为：
user www-data;

# 3. 测试配置并重启nginx
sudo nginx -t
sudo systemctl restart nginx

# 4. 设置WordPress文件权限
sudo chown -R www-data:www-data /var/www/html/wordpress

# 5. 启动nginx并验证
sudo systemctl start nginx
ps aux | grep nginx
```

###### 用户权限关系图解：

```txt
Nginx (www-data) → 读取静态文件
    ↓
PHP-FPM (www-data) → 执行PHP代码 → MySQL (wordpressuser)
    ↓
WordPress文件 (www-data拥有)
```

查看运行用户：

```bash
ps aux | grep 服务
```

> 完成调整后，理想的用户配置应该是：
>
> - Nginx: `www-data` (工作进程)
> - PHP-FPM: `www-data` (工作进程)
> - MySQL: `mysql` (保持不变)
> - WordPress 文件所有者: `www-data:www-data`

------

## 基于 Nginx 部署 WordPress

### 下载并解压 WordPress：

进入 Web 根目录（通常是 `/var/www/html`），下载并解压最新版的 WordPress。

```bash
cd /var/www/html
sudo wget https://wordpress.org/latest.tar.gz
sudo tar -xzvf latest.tar.gz
```

解压后会生成一个 `wordpress` 文件夹。你可以根据需要调整文件夹名称或位置。

### 设置文件权限：

将 WordPress 文件的所有权改为 Nginx 运行的用户（通常是 `www-data`）。

```bash
sudo chown -R www-data:www-data /var/www/html/wordpress
```

### 通过 Web 界面完成安装：

在浏览器中访问你的服务器地址，后跟 WordPress 所在目录（如 http://你的服务器IP/wordpress）。页面将跳转至 WordPress 著名的五分钟安装界面，你需要填写之前创建的数据库信息：

> - **数据库名**: `wordpress`
> - **用户名**: `wordpressuser`
> - **密码**: `你设置的密码`
> - **数据库主机**: `localhost`
> - **表前缀**: `wp_`（默认即可）

------

## SSL 证书

### 1）Certbot

可以使用 **Certbot** 进行 SSL 免费证书申请。

[SSL 免费证书申请 – Certbot](https://www.runoob.com/http/ssl-certbot.html)

> Certbot 是一个开源的自动化工具，用于获取和续订由 Let's Encrypt 提供的免费 SSL/TLS 证书。

**Snap** 是 Certbot 官方推荐的安装方式，尤其是针对长期支持的 Ubuntu 版本：

```bash
sudo snap install core
sudo snap refresh core
sudo snap install --classic certbot

sudo ln -s /snap/bin/certbot /usr/bin/certbot  # 这一步是为了确保 certbot 命令能全局使用
```

安装完成后，使用以下命令查看 Certbot 安装的版本：

```bash
certbot --version
```

安装好 Certbot  后就可以使用以下命令来申请证书了，注意 `*.com` 需要修改为你自己的域名：

```bash
sudo certbot certonly  -d *.com --manual --preferred-challenges dns --server https://acme-v02.api.letsencrypt.org/directory 
```

执行以上命令后，填写信息。

### 2）腾讯云 SSL 证书

但是，我使用了腾讯云自带免费证书申请。

[SSL 证书 DNS 验证_腾讯云](https://cloud.tencent.com/document/product/400/54500?from_cn_redirect=1)。随后，[SSL 证书 Nginx 服务器 SSL 证书安装部署](https://cloud.tencent.com/document/product/400/35244)。

[DNSPod 域名检测工具](https://tool.dnspod.cn/)

------

## PaperMC（可选）

> Paper 是一个基于 Spigot 的 Minecraft 游戏服务器，旨在大幅提高性能并提供更多高级功能和 API。
>
> 其仅限于原版，即无法在 Paper 服务器中使用 Mod 等。
>
> 玩家可以通过原版客户端直接连接至服务器。

> 以下是搭建《我的世界》服务器推荐满足的要求：
>
> 一个稳定的运行在 Ubuntu 或基于 Ubuntu 的发行版上的服务器（x64 架构会更好）。
>
> 2GB 及以上内存、2 个及以上 CPU 核心，以及 1GB 及以上的硬盘存储空间。
>
> 良好的互联网连接。

[树莓派5 搭建MC服务器 全过程记录](https://coast23.github.io/2025/02/21/超详细-树莓派5-搭建MC服务器-全过程记录/)

### 1）安装或更新 Java：

[安装或更新 Java | PaperMC 文档](https://paper.8aka.org/misc/java-install)

| Paper 版本     | 推荐的 Java 版本 |
| -------------- | ---------------- |
| 1.8 到 1.11    | Java 8           |
| 1.12 到 1.16.4 | Java 11          |
| 1.16.5         | Java 16          |
| 1.17.1-1.18.1+ | Java 21          |

#### Ubuntu：

安装 Java 所需的工具：

```bash
sudo apt-get update
sudo apt-get upgrade
sudo apt-get install ca-certificates apt-transport-https gnupg wget
```

导入 Amazon Corretto 公钥和 apt 仓库：

```bash
wget -O - https://apt.corretto.aws/corretto.key | sudo gpg --dearmor -o /usr/share/keyrings/corretto-keyring.gpg && \
echo "deb [signed-by=/usr/share/keyrings/corretto-keyring.gpg] https://apt.corretto.aws stable main" | sudo tee /etc/apt/sources.list.d/corretto.list
```

然后，使用以下命令安装 Java 21 和其他依赖项：

```bash
sudo apt-get update
sudo apt-get install -y java-21-amazon-corretto-jdk libxi6 libxtst6 libxrender1
```

##### 验证安装：

运行此命令以确保安装成功：

```bash
java -version
```

### 2）安装 PaperMC：

这里使用复制下载链接，然后用 `wget` 下载的方法。

在下载前先创建服务器的目录文件夹，如：

```bash
mkdir ~/papermc
cd ~/papermc
```

然后下载 PaperMC：

```bash
wget https://api.papermc.io/v2/projects/paper/versions/1.21.1/builds/132/downloads/paper-1.21.1-132.jar
```

下载后会在目录中得到一个 `paper-1.21.1-132.jar` 文件。

输入以下命令以同意 `eula.txt` 里的条款：

```bash
sed -i '$s/false/true/' eula.txt
或
使用 nano 把 eula.txt 最后一行改为 eula=true
```

输入以下命令来第一次启动服务端：

```bash
java -jar paper-1.21.1-132.jar
```

一旦你的《我的世界》服务器启动，它会打开 `25565` 端口，配置防火墙：

```bash
sudo ufw allow from any to any port 25565
sudo ufw reload
```

关于配置文件：[Minecraft Paper Server指南 | RavelloH's Blog](https://ravelloh.top/posts/minecraft-paper-server)

#### 插件：

下载与安装非常简单，直接把 `.jar` 文件放到 `/plugins` 文件夹下，之后重启服务器即可。

> 别忘了 `cd ~/papermc/plugins`

##### 在客户端加载模组：

[Minecraft Java版模组下载安装及游玩保姆级教程](https://www.bilibili.com/opus/1031865244334424105)

以下是 Forge 的安装地址：

[Forgue](https://adfoc.us/serve/sitelinks/?id=271228&url=https://maven.minecraftforge.net/net/minecraftforge/forge/1.21.8-58.1.0/forge-1.21.8-58.1.0-installer.jar)

##### 语言聊天：

[简单的语音聊天](https://www.mcmod.cn/class/3693.html)

我使用了 [SimpleVoiceChat - Paper Plugin | Hangar](https://hangar.papermc.io/henkelmax/SimpleVoiceChat)：

```bash
wget https://cdn.modrinth.com/data/9eGKb6K1/versions/sJNyRF9H/voicechat-bukkit-2.6.1.jar
```

该插件会占用 `24454` 端口，配置防火墙：

```bash
sudo ufw allow from any to any port 24454
sudo ufw reload
```

在游戏中，你可以通过按下 `V` 键并点击设置按钮来访问语音聊天图形用户界面。要打开群聊界面，可以按 `G` 键。

你可以通过输入命令邀请玩家加入你的群聊：

```
/voicechat invite <playername>
```

内网穿透时，要使用语音通话功能，应在服务端上开启 UDP 端口。

##### MOTD 美化：

[MiniMOTD - Paper Plugin | Hangar](https://hangar.papermc.io/jmp/MiniMOTD)

```bash
wget https://hangarcdn.papermc.io/plugins/jmp/MiniMOTD/versions/2.2.0/PAPER/minimotd-paper-2.2.0.jar
```

##### EssentialX：

[EssentialsX - 我的世界 插件](https://bbsmc.net/plugin/essentialsx)

```bash
wget https://cdn.bbsmc.net/bbsmc/data/dUMU64lM/versions/105Y0mMj/EssentialsX-2.21.2.jar
```

#### 性能：

Java 版本：建议使用较新版本的 OpenJDK JRE（如 Java 21）。

内存分配：对于 8GB 内存的树莓派，一个常见的建议是将大约 75% 到 85% 的物理内存分给 Java 堆。然而，我在这个系统上还运行了 Nginx 等服务，所以我选择了 5GB。如此，启动命令可能看起来像这样（2025.09）：

```bash
java -Xms5G -Xmx5G -XX:+UseG1GC -jar paper-1.21.8-60.jar --nogui
```

别忘了先：

```bash
cd ~/papermc
```

> 其中：
>
> - `-Xms5G` 设置内存的初始大小为 5GB。
> - `-Xmx5G` 设置内存的最大大小为 5GB。（与初始值一致可减少运行时动态调整的开销）。
> - `-XX:+UseG1GC` 启用 G1 垃圾回收器，通常在现代 Java 版本上有更好的性能表现。

[启动脚本生成器 | PaperMC 文档](https://paper.8aka.org/misc/tools/start-script-gen)

#### 其他：

如果在 Minecraft 中连接服务器时显示：`Outdated Server`，那就是服务器上 `paper-1.21.x-xxx.jar` 版本过低，请更新。若显示：`Outdated Client`，则 Minecraft 版本过低。

白名单 `whitelist.json`，在服务器中可以直接使用 `/whitelist add <name>` 的方式将玩家写入到白名单中，之后还需要 `/whitelist reload` 即可生效。

### 3）定时备份：

创建备份脚本文件：

```bash
# 创建脚本文件
nano ~/backup_papermc.sh
# 给脚本执行权限
chmod +x ~/backup_papermc.sh
```

脚本内容如下：

```bash
#!/bin/bash

# PaperMC 备份脚本
# 作者：自动生成
# 功能：备份PaperMC服务器文件和配置

# 源目录和目标目录
SOURCE_DIR="$HOME/PaperMc"
BACKUP_BASE_DIR="$HOME/Backup/PaperMc"

# 要备份的文件夹和文件
BACKUP_ITEMS=("world" "world_nether" "world_the_end" "server.properties")

# 创建日期格式的文件夹名（格式：YYYY-MM-DD）
BACKUP_DATE=$(date +%Y-%m-%d)
BACKUP_DIR="$BACKUP_BASE_DIR/$BACKUP_DATE"

# 日志文件
LOG_FILE="$HOME/backup_log.txt"

# 创建日志函数
log_message() {
    echo "[$(date '+%Y-%m-%d %H:%M:%S')] $1" >> "$LOG_FILE"
}

# 检查源目录是否存在
if [ ! -d "$SOURCE_DIR" ]; then
    log_message "错误：源目录 $SOURCE_DIR 不存在"
    exit 1
fi

# 创建备份基础目录
mkdir -p "$BACKUP_BASE_DIR"
if [ $? -ne 0 ]; then
    log_message "错误：无法创建备份基础目录 $BACKUP_BASE_DIR"
    exit 1
fi

# 检查备份目录是否已存在（避免重复备份）
if [ -d "$BACKUP_DIR" ]; then
    log_message "警告：备份目录 $BACKUP_DIR 已存在，跳过本次备份"
    exit 0
fi

# 创建日期备份目录
mkdir -p "$BACKUP_DIR"
if [ $? -ne 0 ]; then
    log_message "错误：无法创建备份目录 $BACKUP_DIR"
    exit 1
fi

log_message "开始备份 PaperMC 到 $BACKUP_DIR"

# 备份每个项目
backup_success=true
for item in "${BACKUP_ITEMS[@]}"; do
    source_path="$SOURCE_DIR/$item"
    destination_path="$BACKUP_DIR/$item"
    
    if [ -e "$source_path" ]; then
        if [ -d "$source_path" ]; then
            # 备份文件夹
            log_message "正在备份文件夹: $item"
            cp -rp "$source_path" "$destination_path"
            if [ $? -eq 0 ]; then
                log_message "✓ 成功备份文件夹: $item"
            else
                log_message "✗ 备份文件夹失败: $item"
                backup_success=false
            fi
        else
            # 备份文件
            log_message "正在备份文件: $item"
            cp -p "$source_path" "$destination_path"
            if [ $? -eq 0 ]; then
                log_message "✓ 成功备份文件: $item"
            else
                log_message "✗ 备份文件失败: $item"
                backup_success=false
            fi
        fi
    else
        log_message "警告：$source_path 不存在，跳过"
    fi
done

# 创建备份信息文件
backup_info_file="$BACKUP_DIR/backup_info.txt"
{
    echo "备份时间: $(date)"
    echo "源目录: $SOURCE_DIR"
    echo "备份目录: $BACKUP_DIR"
    echo "备份项目: ${BACKUP_ITEMS[*]}"
    echo "备份状态: $([ $backup_success = true ] && echo "成功" || echo "部分失败")"
} > "$backup_info_file"

if [ $backup_success = true ]; then
    log_message "✓ PaperMC 备份完成：$BACKUP_DIR"
    
    # 显示备份大小
    backup_size=$(du -sh "$BACKUP_DIR" | cut -f1)
    log_message "备份大小: $backup_size"
else
    log_message "✗ PaperMC 备份部分失败，请检查日志"
fi
```

设置定时任务（Cron），使用 `crontab` 设置每日凌晨 4 点执行备份：

```bash
crontab -e	# 编辑当前用户的 crontab
```

在文件末尾添加以下行：

```bash
0 4 * * * /bin/bash ~/backup_papermc.sh		# PaperMC 每日凌晨4点备份
```

在设置定时任务前，先手动测试脚本：

```bash
~/backup_papermc.sh			# 手动运行备份脚本
ls -la ~/Backup/PaperMc/	# 检查备份是否成功
tail -f ~/backup_log.txt	# 查看日志
```

管理定时任务，常用 `cron` 管理命令：

```bash
crontab -l	# 查看当前定时任务
crontab -e	# 编辑定时任务
crontab -r	# 删除所有定时任务
```

------
## 文件共享服务

### 基本知识：

在 Ubuntu 中挂载U盘：

- 插入 U 盘，Ubuntu 通常会自动挂载到 `/media/用户名/U盘标签` 目录下。
- 如果没有自动挂载，可以手动挂载。

#### 自动挂载：

查看所有存储设备：

```bash
lsblk
```

查看挂载点：

```bash
df -h
```

进入设备目录（根据实际挂载路径）

```bash
cd /media/$(whoami)/名称/
```

#### 手动挂载：

创建挂载点：

```bash
sudo mkdir /mnt/usb
```

挂载U盘（假设U盘是/dev/sdb1）：

```bash
sudo mount /dev/sdb1 /mnt/usb
```

如果遇到文件系统问题，可以指定文件系统类型：

```bash
sudo mount -t vfat /dev/sdb1 /mnt/usb
sudo mount -t ntfs /dev/sdb1 /mnt/usb
sudo mount -t exfat /dev/sdb1 /mnt/usb
```

访问：

```bash
cd /mnt/usb
ls -la
```

#### 权限问题解决：

如果遇到权限被拒绝的错误。

查看挂载点的权限：

```bash
ls -la /mnt/usb/
```

如果需要，修改权限：

```bash
sudo chmod 755 /mnt/usb/
sudo chown $USER:$USER /mnt/usb/ -R
```

#### 安全卸载：

使用完成后，务必正确卸载设备。

卸载设备：

```bash
sudo umount /mnt/usb
```

如果自动挂载：

```bash
sudo umount /media/你的用户名/设备名称
```

#### 自定义卷标与挂载规则:

##### 1）首先修改卷标：

```bash
sudo umount /dev/sdb1
sudo ntfslabel /dev/sdb1 "MyData"
```

> ###### 扩展：
>
> 对于 `NTFS` 文件系统：
>
> ```bash
> sudo apt install ntfs-3g					# 安装ntfs-3g工具（通常已安装）
> sudo umount /dev/sdb1						# 卸载硬盘
> sudo ntfslabel /dev/sdb1 MYLABEL			# 修改卷标
> sudo mount /dev/sdb1 /media/usr/MYLABEL		# 重新挂载
> ```
>
> 对于 `FAT32/exFAT` 文件系统：
>
> ```bash
> sudo umount /dev/sdb1
> 
> # 修改卷标（FAT32最大11字符，exFAT最大15字符）
> sudo fatlabel /dev/sdb1 MYLABEL
> # 或者对于exFAT
> sudo exfatlabel /dev/sdb1 MYLABEL
> ```
>
> 对于 `ext4` 文件系统：
>
> ```bash
> sudo umount /dev/sdb1
> 
> # 修改卷标（最大16字符）
> sudo e2label /dev/sdb1 MYLABEL
> ```

##### 2）然后设置 fstab 自动挂载：

```bash
sudo mkdir -p /media/MyData
sudo nano /etc/fstab
```

添加：

```bash
UUID=XXXXXXXXXXXXXXXX /media/MyData ntfs-3g defaults,uid=1000,gid=1000,umask=022 0 0
```

> 参数说明：
>
> - `uid=1000,gid=1000`：设置所有者为第一个用户（通常是1000）
> - `umask=022`：设置文件权限
> - 对于 ext4 文件系统，使用 `defaults` 即可
> - 对于 FAT32，使用 `defaults,uid=1000,gid=1000,umask=022,dmask=027,fmask=137`
>
> ```bash
> # 获取设备的UUID
> sudo blkid | grep sdb1
> 
> # 输出示例：/dev/sdb1: UUID="XXXXXXXXXXXXXXXX" TYPE="ntfs"
> ```

测试配置：

```bash
sudo mount -a
ls -la /media/MyData
```

### Samba：

[基于Ubuntu22.04的Samba服务器搭建教程（新手保姆级教程）_ubuntu samba-CSDN博客](https://blog.csdn.net/qq_42417071/article/details/136328807)

#### 安装：

输入下列命令安装 Samba 服务器：

```bash
sudo apt install samba -y
```

`cd`  到你想要共享的文件夹，然后进入文件夹，输入 `pwd` 命令获取文件夹的绝对路径。

回到上级目录，使用 `chmod` 命令给这个文件夹开放一些读写的权限。

```bash
chmod 0777 你想要共享的文件夹
```

#### 配置 Samba 文件以及用户密码：

为保险起见，先备份一下原来的 Samba 配置文件：

```bash
sudo cp /etc/samba/smb.conf /etc/samba/smb.conf.bak
```

编辑配置 `smb.conf` 配置文件，添加共享目录：

```bash
sudo nano /etc/samba/smb.conf
```

进来之后，在结尾处把下面的文本添加进去，要注意根据自己的情况修改。

```ini
[Ubuntu]
	comment = Samba
	path = /home/……
	public = yes
	writable = yes
	available = yes
	browseable = yes
	valid users = usr
```

接着给 Samba 用户设置密码：

```bash
sudo smbpasswd -a usr
```

#### 重启 Samba 服务：

可以直接输入下列命令：

```bash
systemctl restart smbd.service
systemctl enable smbd.service
```

期间需多次输入该用户在 Ubuntu 的密码。

#### 公网访问：

内网穿透见下方 `内网穿透` 章节，穿透本地 `445` 端口，`TCP` 协议。

通过文件资源管理器连接：

[在Windows上使用其他端口连接SMB服务 | heStudio](https://www.hestudio.net/posts/use-smb-another-port-on-windows.html)

由于安全限制，公网通常无法直接访问 445 端口，因此需要修改 SMB 服务的端口。然而，Windows 默认只能连接 445 端口的 SMB 服务，且该端口是硬编码的。为了解决这个问题，我们可以通过端口映射的方式来实现连接。

##### 停止并禁用本机的 SMB 服务：

为了通过本地环回地址映射到非标准端口的 SMB 服务，首先需要停止并禁用本机的 SMB 服务。

按下 `Win + R`，输入 `services.msc`，然后按回车。在服务列表中找到 `Server` 服务，禁用。

##### 设置端口转发：

通过命令提示符设置端口转发，将本地的 445 端口映射到远程 SMB 服务器的非标准端口。

以管理员身份打开命令提示符：

```CMD
netsh interface portproxy add v4tov4 listenport=445 listenaddress=127.0.0.1 connectaddress=<服务器IP或域名> connectport=<服务器端口>
```

> 其中：
>
> - `listenport=445` 表示本地监听的端口。
> - `listenaddress=127.0.0.1` 表示本地监听的 IP 地址。
> - `connectaddress=<服务器IP或域名>` 表示远程 SMB 服务器的 IP 地址或域名。
> - `connectport=<服务器端口>` 表示远程 SMB 服务器的端口号。

##### 步骤 3：连接 SMB 服务：

完成端口转发设置后，在文件资源管理器的地址栏中输入 `\\127.0.0.1`，然后按回车。

此时，Windows 将通过本地环回地址连接到远程 SMB 服务器，绕过默认的 445 端口限制。

如果需要恢复本机的 SMB 服务，可以重新启用 `Server` 服务并删除端口转发规则。

查看端口转发规则：

```CMD
netsh interface portproxy show all
```

删除规则：

```CMD
netsh interface portproxy delete v4tov4 listenport=445 listenaddress=127.0.0.1
```

##### 其他：

可以设置除 `127.0.0.1` 外的地址。在 `netsh interface portproxy add v4tov4` 命令中，`listenaddress` 参数指定本地监听的 IP 地址，你可以根据实际需求进行修改 。

比如，如果你的计算机有多块网卡，具有多个 IP 地址，且你希望从其他网络中的设备也能访问到这个端口转发的服务，就可以将 `listenaddress` 设置为计算机在局域网中的 IP 地址（例如 `192.168.1.100` ，具体根据实际网络环境而定）。这样，同一局域网内的其他设备就可以通过这个 IP 地址来访问经过端口转发的服务。

CMD 打开运行窗口，输入 `net use` 会显示已访问的连接。如果需要删除所有已访问连接，可以使用命令 `net use * /delete`。只是删除其中一个访问连接，可以使用 `net use 访问地址 /delete` 。

查看端口占用可以使用 `netstat -ano`。

###### 脚本：

有时，客户端重启后会报错：

```err
本地设备名已使用中，此连接尚未还原
```

随后注意到删除端口映射，再重设后可以解决该问题。故编写脚本自动完成该操作：

```vbscript
' Samba映射脚本 - 不影响其他网络驱动器
Set WshShell = CreateObject("WScript.Shell")
Set fso = CreateObject("Scripting.FileSystemObject")

' 检查是否已有实例运行
If AppAlreadyRunning() Then
    WScript.Quit
End If

' 等待系统启动
Wscript.Sleep 15000

' 以管理员权限设置端口转发
RunAsAdmin "netsh interface portproxy delete v4tov4 listenport=445 listenaddress=127.0.0.1"
RunAsAdmin "netsh interface portproxy add v4tov4 listenport=445 listenaddress=127.0.0.1 connectaddress=*.com connectport=00000"

' 智能重新映射：只处理特定的Samba驱动器
SmartRemap "Z:", "\\127.0.0.1\share"

Function RunAsAdmin(command)
    On Error Resume Next
    Set ShellApp = CreateObject("Shell.Application")
    ShellApp.ShellExecute "cmd.exe", "/c " & command, "", "runas", 0
    Wscript.Sleep 3000
End Function

Function AppAlreadyRunning()
    Dim processName
    processName = Mid(WScript.ScriptName, 1, Len(WScript.ScriptName) - 4)
    AppAlreadyRunning = (WshShell.AppActivate(processName))
End Function

Sub SmartRemap(driveLetter, networkPath)
    On Error Resume Next
    
    ' 检查目标驱动器是否已映射
    Dim currentMapping
    Set oExec = WshShell.Exec("net use " & driveLetter)
    Do While oExec.Status = 0
        Wscript.Sleep 100
    Loop
    
    Dim output : output = oExec.StdOut.ReadAll()
    
    ' 如果驱动器已映射但不是我们要的路径，则跳过
    If InStr(output, networkPath) > 0 Then
        ' 已经是正确的映射，无需操作
        Exit Sub
    ElseIf InStr(output, "\\") > 0 Then
        ' 驱动器被其他网络路径占用，可以选择不同驱动器或强制替换
        ' 这里我们选择跳过，避免影响其他映射
        Exit Sub
    End If
    
    ' 检查网络路径是否已被其他驱动器映射
    Set oExec2 = WshShell.Exec("net use")
    Do While oExec2.Status = 0
        Wscript.Sleep 100
    Loop
    
    Dim allMappings : allMappings = oExec2.StdOut.ReadAll()
    If InStr(allMappings, networkPath) > 0 Then
        ' 网络路径已被其他驱动器映射，先删除该映射
        WshShell.Run "cmd /c net use /delete " & networkPath & " >nul 2>&1", 0, True
        Wscript.Sleep 1000
    End If
    
    ' 创建新的映射
    WshShell.Run "cmd /c net use " & driveLetter & " " & networkPath & " /persistent:yes >nul 2>&1", 0, True
End Sub
```

该脚本手动或放置自启动目录执行时，需要手动赋予管理员权限，这里通过 Windows 任务计划程序进行配置。

> 打开任务计划程序：
>
> - 按下 `Win + R`，输入 `taskschd.msc` 并回车。
>
> 创建基本任务：
>
> - 在右侧面板点击**创建基本任务**。
> - 输入任务名称（如 `Samba 映射`）和描述，点击下一步。
>
> 设置触发器：
>
> - 选择**计算机启动时**，点击下一步。
>
> 设置操作：
>
> - 选择**启动程序**，点击下一步。
> - 在**程序或脚本**中浏览选择你的 VBS 脚本。
> - 可以在**起始于**中填写脚本所在文件夹路径。
>
> 完成设置：
>
> - 勾选**当单击完成时，打开此任务的属性对话框**。
> - 点击**完成**。
>
> 配置高级属性：
>
> - 在属性对话框中，切换到**常规**选项卡。
> - 勾选**不管用户是否登录都要运行**。
> - 勾选**使用最高权限运行**。
> - 切换到**条件**选项卡，根据需要调整电源管理选项。
> - 点击**确定**，输入管理员密码。

### Very Secure FTP Daemon：

vsftpd (Very Secure FTP Daemon) 是 Ubuntu 上最常用的 FTP 服务器，以安全性和稳定性著称。

#### FTP：

##### 安装 vsftpd：

执行以下命令来安装 vsftpd：

```bash
sudo apt install vsftpd
```

安装完成后，vsftpd 服务会自动启动。你可以通过以下命令验证其状态：

```bash
sudo systemctl status vsftpd
```

##### 创建 FTP 专用账户：

如果已有一个用户名，可以跳过。但为了安全，建议为 FTP 创建一个专用账户：

```bash
# 创建系统用户，并指定主目录为你想要的共享目录
sudo useradd -d /…… -s /bin/bash username
# 设置密码
sudo passwd username
```

##### 准备共享目录并设置权限：

设置你指定的共享目录：

```bash
# 确保目录存在
sudo mkdir -p /……
# 更改目录所有者给新创建的FTP用户
sudo chown username:username /……
# 设置目录权限（可根据需要调整）
sudo chmod 755 /……
```

##### 详解与修改配置文件：

vsftpd 的主要配置文件是 `/etc/vsftpd.conf`。在修改前，建议先备份：

```bash
sudo cp /etc/vsftpd.conf /etc/vsftpd.conf.bak
```

打开配置文件：

```bash
sudo nano /etc/vsftpd.conf
```

主要配置：

```ini
# 监听IPv4端口
listen=YES
# 禁用IPv6监听
listen_ipv6=NO

# 禁止匿名登录:
anonymous_enable=NO
# 允许本地系统用户登录
local_enable=YES
# 启用写入权限（上传/创建目录等）
write_enable=YES

# 将本地用户限制在其主目录内
chroot_local_user=YES
# 允许在chroot环境下进行写操作
allow_writeable_chroot=YES

# 使用本地时间
use_localtime=YES
# 启用详细日志
dual_log_enable=YES
xferlog_enable=YES

# 用户配置文件设置（可选，用于更精细的控制）
user_config_dir=/etc/vsftpd/user_conf
```

##### 配置防火墙与重启服务：

需要开放 FTP 端口：

```bash
sudo ufw allow 21/tcp
sudo ufw reload
```

现在重启 vsftpd 服务以应用配置：

```bash
sudo systemctl restart vsftpd
```

要让 vsftpd 在系统启动时自动运行，可以执行：

```bash
sudo systemctl enable vsftpd
```

至此，在文件资源管理器中右键，`添加一个网络地址`，一步步配置即可访问。

------


## 内网穿透

我使用了 [SakuraFrp](https://doc.natfrp.com/)：

以下是一些端口示例：

| 服务         | 本地端口 | 协议    | 远程端口 | 域名（可选）     | 备注            |
| ------------ | -------- | ------- | -------- | ---------------- | --------------- |
| PaperMC      | 25565    | TCP+UDP | 25565    | mc.example.com   | 必须同时开 UDP  |
| Web（Nginx） | 80/443   | TCP     | 80/443   | blog.example.com | 已含 HTTPS      |
| MySQL        | 3306     | TCP     | 3306     | —                | 仅允许白名单 IP |
| SSH          | 22       | TCP     | XXX      | —                | 远程改端口      |

教程：[frpc 基本使用指南](https://doc.natfrp.com/frpc/usage.html)

## 上线

1. 域名注册：[概览 - 域名注册 - 控制台](https://console.cloud.tencent.com/domain)

2. ICP 备案：[ICP 备案 备案概述_腾讯云](https://cloud.tencent.com/document/product/243/18907)

3. 公安联网备案：[ICP 备案 公安联网备案流程指引_腾讯云](https://cloud.tencent.com/document/product/243/19142)

以上三步将耗时约 20 天。当第二步 ICP 备案通过，可查询到域名时，可先暂时跳过第三步而配置 HTTP(S) 内网穿透服务：

内网穿透服务方面：[Web 应用穿透指南 | SakuraFrp 帮助文档](https://doc.natfrp.com/app/http.html)

| 记录类型   | 适用场景                            | 主机记录 (域名前缀)                              | 记录值 (指向目的地)        |
| :--------- | :---------------------------------- | :----------------------------------------------- | :------------------------- |
| A 记录     | 将域名指向一个 IPv4 地址            | @ (表示主域名，如 *.com) 或 www (表示 www.*.com) | 你的服务器公网IP地址       |
| CNAME 记录 | 将域名指向另一个域名(如 CDN)        | @ 或 www 等                                      | 服务商提供的域名地址       |
| MX 记录    | 设置邮箱，让邮件能正确收发          | 通常设置为 @                                     | 邮箱服务商提供的服务器地址 |
| TXT 记录   | 常用于域名所有权验证(如申请SSL证书) | @ 或 服务商指定的名称 (如 _acme-challenge)       | 服务商提供的一串特定文本   |

腾讯云方面，登录腾讯云 DNSPod 权威解析，创建解析，使用节点域名即可。

可以用如下命令查看 DNS 解析：

```CMD
nslookup *.com
```

### 其他：

#### 挂载备案号：

在网页源代码中添加如下代码：

```html
<center>
    <a href="https://beian.miit.gov.cn/" target="_blank">你的ICP备案号</a>
    
    <img src="/image/head-logo.png" width="20"/>
    <a href="https://beian.mps.gov.cn/#/query/webSearch?code=你的公安联网备案号码" target="_blank">你的公安联网备案号</a>
</center>
```

其中，`head-logo.png` 是公安联网备案 logo，如下所示：

<img src="Images\head-logo.png" alt="head-logo" style="zoom:100%;" />

将其放在 `index.html` 同目录下的 `image` 文件夹内即可。

#### 网站 logo：

在网页源代码中添加如下代码：

```html
<!-- 指定favicon -->
    <link rel="icon" href="/image/logo.png" type="image/png">
```

这里我使用了 `png` 类型的 `favicon`，你可以指定其他类型。

同样，将你的 `logo.png` 放在 `index.html` 同目录下的 `image` 文件夹内即可。

## 开机自启

通过 systemd 服务实现开机时自动启动这两个 screen 会话。

首先，创建一个启动脚本：

```bash
#!/bin/bash

# 等待系统网络服务启动，根据实际情况调整
sleep 10

# 启动frpc服务
if ! screen -list | grep -q "frpc"; then
    screen -dmS frpc
    screen -S frpc -X stuff "frpc -f xxxxxxxxxxxxxxxxxxx:00000000\n"
fi

# 启动Minecraft服务器
if ! screen -list | grep -q "mc"; then
    screen -dmS mc
    screen -S mc -X stuff "cd /home/username/papermc\n"
    screen -S mc -X stuff "java -Xms5G -Xmx5G -XX:+UseG1GC -jar paper-1.21.8-60.jar --nogui\n"
fi
```

将其保存为 `/usr/local/bin/start_services.sh`

赋予执行权限：

```bash
sudo chmod +x /usr/local/bin/start_services.sh
```

然后，创建 systemd 服务文件使其开机自启：

```ini
[Unit]
Description=Start frpc and Minecraft server on boot
After=network.target

[Service]
Type=oneshot
ExecStart=/usr/local/bin/start_services.sh
User=root
RemainAfterExit=yes

[Install]
WantedBy=multi-user.target
```

保存为 `/etc/systemd/system/startup-services.service`

启用服务：

```bash
sudo systemctl enable startup-services.service
```

可以立即测试启动：

```bash
sudo systemctl start startup-services.service
```

- 看 frpc 会话：`screen -r frpc`
- 查看 mc 会话：`screen -r mc`

> 注意：需要处于 `root` 用户才能查看。
