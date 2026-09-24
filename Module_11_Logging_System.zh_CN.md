# Linux Basics for Hackers
## Module 11 - 日志系统(The Logging System)

> 本文档为 Module_11_Logging_System.md 的简体中文翻译版本

---

## 概述(Overview)

Linux 会记下系统上几乎发生过的所有事——登录、登录失败、硬件事件、服务活动、错误。这些日志文件之所以"有用",目的完全取决于你站在键盘的哪一侧。作为防御者,它们会告诉你是否有人正在探查系统;作为攻击者,它们则是你所做一切事情的记录,必须在离开之前处理掉。

---

## 日志存放的位置(Where logs live)

几乎所有日志都存放在 `/var/log` 下。跳转到该目录并列出其内容:

```bash
ahegazy0@kali:~$ ls /var/log
```

你会看到相当多的文件。其中重要的几个:

| 日志文件 | 它记录的内容 |
|---|---|
| `/var/log/auth.log` | 登录尝试、sudo 使用情况、SSH 连接 |
| `/var/log/syslog` | 一般性系统消息 —— 大多数事件的总汇 |
| `/var/log/kern.log` | 内核消息、硬件事件、驱动错误 |
| `/var/log/messages` | 与 syslog 类似,某些发行版用这个替代 syslog |
| `/var/log/mail.log` | 邮件服务器活动 |
| `/var/log/apache2/` | Web 服务器访问日志与错误日志(若 Apache 在运行) |
| `/var/log/mysql/` | 数据库活动(若 MySQL 在运行) |

你最常查看的就是 `auth.log`。它会显示每一次登录尝试——无论成功还是失败——如果是来自网络的连接,还会带上源 IP。

---

## 阅读日志(Reading logs)

日志是纯文本文件,可以用任何文本工具来读取。

**查看最后 10 行:**

```bash
ahegazy0@kali:~$ tail /var/log/auth.log
```

**实时跟踪日志**(有新条目就立即显示出来):

```bash
ahegazy0@kali:~$ tail -f /var/log/auth.log
```

`-f` 表示 "follow(跟踪)"。当你正在等待某个特定事件时——比如有人正在尝试闯入时,观察登录尝试——这个选项非常有用。

**在日志中搜索特定内容:**

```bash
ahegazy0@kali:~$ grep "Failed" /var/log/auth.log
```

这条命令会抽出所有包含 "Failed" 这个词的行——这样你看到的就是登录失败的那些条目,而不会被其他日志噪音干扰。

```bash
ahegazy0@kali:~$ grep "Failed" /var/log/auth.log | grep "192.168.1.50"
```

现在你筛选的是来自某个特定 IP 的登录失败。这就是用来检查是否某台机器正在对你的系统暴力猜密码的方式。

---

## rsyslog — 真正负责写日志的后台服务

负责收集和写入日志数据的后台服务叫做 **rsyslog**。它始终在后台安静地运行,监听来自内核和其他程序的消息,并将它们写到 `/var/log` 中对应的文件里。

可以查看它是否在运行:

```bash
ahegazy0@kali:~$ service rsyslog status
```

以及停止它:

```bash
ahegazy0@kali:~$ service rsyslog stop
```

停止 rsyslog 之后,系统就不再写入新的日志条目,之后发生的事都不会被记录下来。这算是一种手段,但十分粗糙——任何有经验的、正在监控系统的管理员都会发现日志突然停摆。日志出现断档本身就是一种警示信号。

---

## logrotate — 防止日志把磁盘撑满

日志永远都在增长。如果没有任何管理手段,`/var/log` 终将把整个磁盘填满。`logrotate` 就是用来自动处理这个问题的工具。

它按调度运行(通常是每天),做以下这些事情:
- 把当前日志文件改名(例如 `auth.log` 变成 `auth.log.1`)
- 创建一个全新的空 `auth.log` 用于写入新的条目
- 压缩较旧的轮转备份文件(如 `auth.log.2.gz` 等)
- 删除超过配置天数的旧日志

控制以上行为的所有配置位于:

```
/etc/logrotate.conf          ← 主配置
/etc/logrotate.d/            ← 各个服务的单独配置(比如 apache、mysql 等)
```

打开 `/etc/logrotate.conf` 可以看到诸如保留多少份旧日志、轮转多长时间运行一次之类的设置。这些内容回答了你笔记中那道关于"日志轮转是在哪里配置的"的练习题。

---

## shred — 真正彻底地销毁文件

当你用 `rm` 正常删除一个文件时,数据并不会真正消失。文件系统只是把那块空间标记为可重用。在有别的东西覆盖之前,原始数据仍然物理存在于磁盘上,可以通过取证工具恢复。

`shred` 会在删除之前用随机数据对文件进行多次覆写,从而让恢复变得几乎不可能。

```bash
ahegazy0@kali:~$ shred -vzu filename.txt
```

逐一解释这些参数:
- `-v` — verbose,详细模式,显示进度
- `-z` — shred 完成后,再用 0 再覆写一次,从而把"已被 shred"的痕迹掩盖掉
- `-u` — shred 之后移除(delete)该文件

默认情况下 shred 会覆写 3 次。你也可以加大次数:

```bash
ahegazy0@kali:~$ shred -n 10 -vzu filename.txt
```

这条命令会在用 0 覆写并删除之前覆写 10 次。

**应用在一个日志文件上:**

```bash
ahegazy0@kali:~$ shred -vzu /var/log/auth.log
```

执行之后,即便有人试图恢复这个文件,得到的也只会是随机数据。这个日志算是彻底消失了。

---

## 更聪明的办法 — 编辑日志,而非删除日志

把整个日志文件删掉太明显了。突然没有 `auth.log` 的 syslog 会立刻引起怀疑——几乎就像日志彻底静默了一样可疑。

有经验的攻击者会选择另一种做法:打开日志文件,找出包含自己活动(自己的 IP、自己用的用户名、自己会话的时间戳)的那些行,只删掉这几行,然后保存文件。日志文件还在,内容还在,看上去一切正常——只不过你那些特定的条目消失了。

要找出与自己相关的条目:

```bash
ahegazy0@kali:~$ grep "192.168.1.100" /var/log/auth.log
```

然后用 `nano` 或 `vi` 这样的文本编辑器打开这个文件,找到那些行并删除,再保存。日志文件保持完整、不易引起怀疑——只是在你曾经在的位置少了一小块不可见的缺口。

这比 `shred` 要花更多功夫,但被发现的可能性要低得多。

---

## 抹除痕迹 — 全局视角(Covering tracks - the full picture)

站在攻击者的视角,清理日志的检查清单通常长这样:

1. 检查哪些日志记录了你的活动——`auth.log` 用来记录登录,如果你访问过 Web 服务器或数据库,还要查看对应的服务特定日志
2. 决定是编辑(精准,不太可疑)还是 shred(彻底,但只要有人看一眼就会露馅)
3. 清除本次会话的 bash 历史——`export HISTSIZE=0` 和 `history -c`
4. 检查是否有其他工具也记下了你的行为(IDS、Web 服务器访问日志、数据库查询日志等)

日志分散在系统的各个角落。漏掉一处很常见。这就是取证调查员要在多个地方查找、而不只是盯着 `/var/log/auth.log` 的原因。

---

## 站在防御者的视角(From a defensive perspective)

如果你是要保护一台系统而不是去攻击它:

- 如果你怀疑有人正在尝试闯入,`tail -f /var/log/auth.log` 是你应该一直开着的
- 如果同一 IP 出现多次 "Failed password" 条目,那就是暴力破解——可以用 `iptables` 或 `ufw` 把这个 IP 封掉
- 一个完全为空、丢失或最近被 shred 过的日志文件本身,就是被篡改的证据
- 像 `fail2ban` 这样的工具可以在登录失败次数过多时自动封禁 IP——任何暴露在公网上的机器都值得配置一下

---

## 命令参考(Command Reference)

| 命令 | 作用 |
|---|---|
| `ls /var/log` | 查看所有日志文件 |
| `tail /var/log/auth.log` | 查看 auth 日志的最后 10 行 |
| `tail -f /var/log/auth.log` | 实时跟踪 auth 日志 |
| `grep "term" /var/log/file` | 在日志中搜索特定字符串 |
| `service rsyslog status` | 检查日志服务是否在运行 |
| `service rsyslog stop` | 停止日志服务 |
| `shred -vzu filename` | 安全地覆写并删除一个文件 |
| `shred -n 10 -vzu filename` | 同上,但覆写 10 次 |
| `history -c` | 清除你的 bash 命令历史 |
| `cat /etc/logrotate.conf` | 查看日志轮转配置 |

---

## 练习(Practice)

- [ ] 切换到 `/var/log` 并运行 `ls` 查看目录里都有什么
- [ ] 运行 `tail /var/log/auth.log`,看一下最后几条记录 —— 你应该能看到自己最近的那次登录
- [ ] 在一个终端里运行 `tail -f /var/log/auth.log`,然后在另一个终端里尝试 `ssh localhost` —— 实时观察该条目是如何出现的
- [ ] 用 `echo "sensitive data" > test.txt` 创建一个测试文件,然后运行 `shred -vzu test.txt`,之后再尝试找回那个数据
- [ ] 打开 `/etc/logrotate.conf`,找到控制保留几份旧日志的那一行

> 💡 *想要更深入的练习,我还建议完成官方书籍 **Linux Basics for Hackers** 中本章末尾的习题。*
---

[下一篇:Module 12 - 使用与滥用服务(Using & Abusing Services)](Module_12_Using_Abusing_Services.zh_CN.md)
