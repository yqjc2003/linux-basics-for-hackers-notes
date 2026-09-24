# Linux Basics for Hackers
## Module 12 - 使用与滥用服务(Using & Abusing Services)

> 本文档为 Module_12_Using_Abusing_Services.md 的简体中文翻译版本

---

## 概述(Overview)

服务(daemon,守护进程)是在后台持续运行的程序,等待执行任务——提供网页、响应数据库查询、接受 SSH 连接等。它们是服务器的实际工作组件。理解它们既对搭建自己的工具很重要,也对理解目标机器上运行着什么、以及如何通过这些服务入侵进去至关重要。

---

## 服务的本质

服务(也称为守护进程,daemon)是一个在系统启动时就开始、并持续在后台运行的进程。它没有窗口也没有可见界面。它只是在特定的端口上监听请求,当有连接进来时进行响应。

示例:
- Apache 监听 80 端口(HTTP)和 443 端口(HTTPS) —— 提供 Web 页面
- SSH 监听 22 端口 —— 接受远程终端连接
- MySQL 监听 3306 端口 —— 响应数据库查询
- PostgreSQL 监听 5432 端口 —— 类似作用,不同的数据库

端口号是你与远程机器通信时识别目标服务的方式。这在后面使用 nmap 进行端口扫描时会变得非常重要 —— 每个开放的端口都对应一个服务,每个服务都是一个潜在的入侵入口。

![Background Services on Server](assets/background_services_diagram_1789213348325.jpg)

---

## 启动和停止服务

管理服务有两个工具:`service`(较旧方式)和 `systemctl`(现代标准)。两者在 Kali 上都可以使用。在当前系统上,`systemctl` 是最常见的。

**使用 service:**

```bash
ahegazy0@kali:~$ service apache2 start
ahegazy0@kali:~$ service apache2 stop
ahegazy0@kali:~$ service apache2 restart
ahegazy0@kali:~$ service apache2 status
```

**使用 systemctl:**

```bash
ahegazy0@kali:~$ systemctl start apache2
ahegazy0@kali:~$ systemctl stop apache2
ahegazy0@kali:~$ systemctl restart apache2
ahegazy0@kali:~$ systemctl status apache2
```

它们功能相同。`systemctl` 提供更详细的状态输出,是当前的发展方向,所以值得同时了解两者,但默认使用 `systemctl`。

**将服务设置为开机自启动:**

```bash
ahegazy0@kali:~$ systemctl enable apache2
```

**禁用自启动:**

```bash
ahegazy0@kali:~$ systemctl disable apache2
```

---

## Apache —— Web 服务器

Apache 是全球使用最广泛的 Web 服务器之一。启动它就把你的机器变成了 Web 服务器。

```bash
ahegazy0@kali:~$ service apache2 start
```

运行后,打开浏览器访问 `http://localhost`。你会看到 Apache 的默认页面。这个页面只是一个位于 `/var/www/html/index.html` 的文件。把它替换成你自己的内容后,服务器就会提供你设置的内容。

```bash
ahegazy0@kali:~$ echo "<h1>My custom page</h1>" > /var/www/html/index.html
```

刷新浏览器 —— 你提供的内容现在已经在 Web 服务器上了。

Apache 记录所有东西:每个请求、每个连接的 IP、每个被请求的文件。这些日志位于 `/var/log/apache2/`。访问日志显示谁连接了以及他们请求了什么。错误日志显示哪里出了问题。

**为什么这对黑客很重要:** Apache 历史上出现过很多漏洞。未打补丁或配置不当的 Apache 服务器是常见的入侵入口。当你扫描目标时,如果看到 80 或 443 端口开放,几乎可以肯定背后是 Apache(或 nginx,或其他 Web 服务器)。版本号很重要 —— 旧版本存在已知漏洞可以利用。

---

## SSH —— 远程终端访问

SSH(Secure Shell,安全外壳)允许你通过网络加密地控制另一台计算机的终端。这是管理远程 Linux 系统的标准方式。

```bash
ahegazy0@kali:~$ ssh username@192.168.1.50
```

这会在该 IP 上的机器里打开一个交互式终端会话,以 `username` 身份登录。你输入的所有内容在传输过程中都会被加密。

**在你的机器上启动 SSH 服务**(以便其他人可以连接到你):

```bash
ahegazy0@kali:~$ service ssh start
```

**使用指定端口进行连接**(如果服务器不在默认的 22 端口):

```bash
ahegazy0@kali:~$ ssh -p 2222 username@192.168.1.50
```

**通过 scp 跨 SSH 复制文件:**

```bash
ahegazy0@kali:~$ scp file.txt username@192.168.1.50:/home/username/
```

这会把 `file.txt` 复制到远程机器。双向都可以工作 —— 你也可以用同样的方式从远程机器拉取文件。

**为什么这对黑客很重要:** SSH 无处不在。如果你在运行 SSH 的机器上获得了有效凭据,就拥有了对它的完整远程访问权限。SSH 账户上的弱口令是服务器被入侵的最常见方式之一。如果服务器没有配置速率限制或 fail2ban(自动封锁多次登录失败的 IP 的工具),Hydra(在线密码爆破工具)等工具可以暴力破解 SSH 登录。

Kali 默认对 SSH 服务设置了一个密码。如果要启用 SSH,请立即修改密码。以默认凭据运行 Kali 并开放 SSH 是一个严重的错误。

---

## MySQL —— 数据库

MySQL 是一个关系型数据库服务器。它把数据保存在表中,并通过 SQL 语言响应查询。它是大多数 Web 应用的基础 —— 登录系统、用户数据、产品列表,等等。

**启动 MySQL:**

```bash
ahegazy0@kali:~$ service mysql start
```

**登录 MySQL Shell(命令行交互界面):**

```bash
ahegazy0@kali:~$ mysql -u root -p
```

`-u root` 意思是作为 root 数据库用户登录。`-p` 表示提示输入密码。登录后,你会看到 MySQL 提示符:

```sql
mysql>
```

从这里开始需要了解的基本 SQL:

```sql
SHOW DATABASES;           -- 列出所有数据库
USE database_name;        -- 切换到某个数据库
SHOW TABLES;              -- 列出当前数据库的所有表
SELECT * FROM users;      -- 导出 users 表的所有内容
```

**为什么这对黑客很重要:** SQL 注入是最常见的 Web 漏洞之一。如果一个 Web 应用没有正确地对输入进行过滤(转义),你就可以通过登录表单或 URL 注入 SQL 命令,直接从数据库提取数据。即使没有 SQL 注入,如果 MySQL 服务器使用了弱密码或默认的 root 密码,你就能访问其中存储的所有内容 —— 用户名、密码哈希、邮箱地址、私人数据。

---

## PostgreSQL

PostgreSQL 是另一种数据库服务器,与 MySQL 类似。这里特别提到它的原因是 **Metasploit**(本课程后面会用到的主要漏洞利用框架)使用 PostgreSQL 作为其后端来存储扫描结果和会话数据。

```bash
ahegazy0@kali:~$ service postgresql start
```

你不会太多地直接使用它。只要知道当你启动 Metasploit 时,如果 PostgreSQL 没有先启动,它会告诉你数据库未连接。

---

## 检查正在运行的服务

查看系统上当前所有活动的服务:

```bash
ahegazy0@kali:~$ systemctl list-units --type=service --state=running
```

查看当前哪些端口是开放的以及哪些进程在监听:

```bash
ahegazy0@kali:~$ ss -tlnp
```

或者旧版本的等价命令:

```bash
ahegazy0@kali:~$ netstat -tlnp
```

输出会显示端口号、协议以及哪个进程在监听。这对你自己的机器(了解你在暴露哪些服务)和目标机器(如果你有访问权限,想看内部运行哪些服务)都很有用。

---

## 默认凭据 —— 一个真实存在的问题

Kali Linux 为多个服务设置了默认密码。默认情况下,许多路由器、摄像头、数据库和服务器也都有默认密码。"admin/admin"、"root/root"、"admin/password" —— 这些是安装后从未被修改的凭据。

现实世界中很大一部分数据泄露就是这样发生的。不是通过巧妙的漏洞利用或零日漏洞,而是某人只是尝试了默认的用户名和密码,然后直接进入了系统。

当你配置任何服务时,立即修改默认凭据。当你评估目标时,尝试默认凭据也总是首要步骤之一。

---

## 命令参考(Command Reference)

| 命令 | 说明 |
|---|---|
| `service name start/stop/restart` | 管理服务(旧语法) |
| `systemctl start/stop/restart name` | 管理服务(现代语法) |
| `systemctl status name` | 检查服务是否在运行 |
| `systemctl enable name` | 设置开机自动启动服务 |
| `systemctl disable name` | 禁用开机自动启动 |
| `systemctl list-units --type=service` | 列出所有运行中的服务 |
| `ssh user@ip` | 通过 SSH 连接到远程机器 |
| `scp file user@ip:/path` | 把文件复制到远程机器 |
| `mysql -u root -p` | 登录 MySQL Shell |
| `ss -tlnp` | 显示开放端口和监听的服务 |

---

## 关键服务及其端口

| 服务 | 默认端口 | 说明 |
|---|---|---|
| SSH | 22 | 远程终端访问 |
| HTTP (Apache) | 80 | Web 服务器 |
| HTTPS | 443 | Web 服务器(加密) |
| MySQL | 3306 | 数据库 |
| PostgreSQL | 5432 | 数据库 |
| FTP | 21 | 文件传输 |
| SMTP | 25 | 邮件发送 |

记住这些是值得做的事。当 nmap 返回一个开放端口列表时,知道每个端口通常跑的是什么服务,你就能立刻知道面对的是什么。

---

## 练习(Practice)

- [ ] 用 `service apache2 start` 启动 Apache,然后打开浏览器访问 `http://localhost`
- [ ] 把 `/var/www/html/index.html` 的内容替换成自定义内容并刷新浏览器
- [ ] 启动 MySQL 并用 `mysql -u root -p` 登录,运行 `SHOW DATABASES;`
- [ ] 运行 `ss -tlnp`,看启动这些服务后哪些端口开放了
- [ ] 完成后停止这两个服务 —— 不要让不需要的东西一直运行

> 💡 *如需进行更深入的练习,我推荐同时完成官方 **Linux Basics for Hackers** 一书中本章末尾的习题。*
---

[下一篇:Module 13 - 安全与匿名(Security & Anonymity)](Module_13_Security_Anonymity.zh_CN.md)
