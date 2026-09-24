# Linux Basics for Hackers
## 第 16 章 - 自动化与计划任务(Cron)

> 本文档为 Module_16_Automation_Jobs.md 的简体中文翻译版本

---

## 概述

每天手动输入相同的命令是一种浪费时间。Linux 内置了一个称为 cron 的调度系统,允许你告诉系统在任意时间、按你定义的任意计划运行任何脚本或命令。本章介绍如何设置它、如何让程序在启动时运行,以及整个自动化系统是如何工作的。

---

## Cron - Linux 调度器

Cron 是一个后台服务,它每分钟唤醒一次,检查是否有任何计划任务需要运行,并执行它们。它已经成为 Unix 系统的一部分几十年了,至今仍是 Linux 上自动化任务的标准方式。

每个用户都有自己的 cron 计划,称为 **crontab**(cron 表)。Root 有自己的 crontab,你也有自己的 crontab。它们是相互独立的,并以所属用户的权限运行。

---

## crontab 语法

crontab 中的每一行代表一个计划任务,遵循以下格式:

```
MIN  HOUR  DAY  MONTH  WEEKDAY  command
```

五个时间字段,然后是要运行的命令。各字段含义:

| 字段 | 范围 | 含义 |
|---|---|---|
| MIN | 0–59 | 一小时中的分钟 |
| HOUR | 0–23 | 一天中的小时(24 小时制) |
| DAY | 1–31 | 一个月中的日期 |
| MONTH | 1–12 | 一年中的月份 |
| WEEKDAY | 0–7 | 一周中的某天(0 和 7 都代表星期日) |

任何字段中的 `*` 表示"每"——每分钟、每小时、每天,以此类推。

下面是一些具体示例:

```bash
# 每分钟、每小时、每天
* * * * *  /path/to/script.sh

# 每天凌晨 3:00
0 3 * * *  /path/to/script.sh

# 每周三凌晨 3:00
0 3 * * 3  /path/to/script.sh

# 每月 15 日中午 12:00
0 12 15 * *  /path/to/script.sh

# 每周一至周五上午 8:00
0 8 * * 1-5  /path/to/script.sh

# 每小时整点
0 * * * *  /path/to/script.sh
```

解读方法:从左到右读取。`0 3 * * 3`——在分钟为 0、小时为 3、月份中任意日期、任意月份时运行,但仅在星期 3(星期三)运行。

---

## 编辑你的 crontab

```bash
ahegazy0@kali:~$ crontab -e
```

这会在文本编辑器中打开你的个人 crontab。每行添加一个任务,保存并关闭即可——cron 会立即获取更改。

**查看当前 crontab:**

```bash
ahegazy0@kali:~$ crontab -l
```

**删除你所有的 cron 任务:**

```bash
ahegazy0@kali:~$ crontab -r
```

使用 `-r` 时要小心——它没有确认提示,也无法撤销。

**编辑 root 的 crontab**(需要 sudo):

```bash
ahegazy0@kali:~$ sudo crontab -e
```

root 的 crontab 中的任务以完整的 root 权限运行。

---

## 简写 - 特殊的 cron 语法

除了五字段格式之外,cron 还支持一些简写:

| 简写 | 等价于 | 运行时机 |
|---|---|---|
| `@reboot` | - | 系统启动时运行一次 |
| `@hourly` | `0 * * * *` | 每小时运行 |
| `@daily` | `0 0 * * *` | 每天午夜运行 |
| `@weekly` | `0 0 * * 0` | 每周日午夜运行 |
| `@monthly` | `0 0 1 * *` | 每月第一天运行 |

`@reboot` 特别有用——它会在每次系统启动时运行你的命令一次,与时间无关。

```bash
@reboot  /home/kali/scripts/startup.sh
```

---

## 一个实际示例

编写一个将当前日期和时间记录到文件的脚本:

```bash
#!/bin/bash
echo "System check: $(date)" >> /home/kali/check.log
```

将其保存为 `check.sh`,并赋予可执行权限:

```bash
ahegazy0@kali:~$ chmod 755 /home/kali/check.sh
```

调度它每分钟运行一次以进行测试:

```bash
ahegazy0@kali:~$ crontab -e
```

添加:

```
* * * * *  /home/kali/check.sh
```

保存后等待几分钟,然后检查:

```bash
ahegazy0@kali:~$ cat /home/kali/check.log
```

你应该会看到新条目不断出现。确认后,将计划更改为你真正需要的频率。

---

## Cron 输出与日志

默认情况下,cron 会通过电子邮件向你发送任何产生输出的任务的输出。在没有配置邮件的本地系统上,该输出通常会丢失。如果你希望自行重定向输出:

```bash
* * * * *  /home/kali/check.sh >> /home/kali/cron.log 2>&1
```

`>> /home/kali/cron.log` 表示将 stdout 追加到日志文件。
`2>&1` 表示将 stderr 重定向到同一位置,因此错误也会显示在日志中。

如果你希望任务静默运行,不产生任何输出:

```bash
* * * * *  /home/kali/check.sh > /dev/null 2>&1
```

`/dev/null` 是一个特殊文件,写入其中的所有内容都会被丢弃。

---

## 让服务在启动时自动运行

对于服务而非脚本,`systemctl enable` 是现代的标准做法:

```bash
ahegazy0@kali:~$ systemctl enable mysql
ahegazy0@kali:~$ systemctl enable apache2
ahegazy0@kali:~$ systemctl enable ssh
```

这会创建必要的符号链接,以便 systemd(系统和服务管理器)在启动时自动启动这些服务。

查看当前已启用的服务:

```bash
ahegazy0@kali:~$ systemctl list-unit-files --type=service | grep enabled
```

**较旧的方法 - update-rc.d:**

在较老的基于 Debian/Ubuntu 的系统上,可能会看到使用 `update-rc.d`:

```bash
ahegazy0@kali:~$ update-rc.d mysql defaults
```

这会在旧的 init.d 启动系统中注册一个服务。在现代系统上,`systemctl enable` 完成同样的工作,这是你应该继续使用的方法。

---

## /etc/cron 目录

除了用户 crontab 之外,系统还有自己的计划任务目录:

```
/etc/cron.hourly/     此处的脚本每小时运行
/etc/cron.daily/      此处的脚本每天运行
/etc/cron.weekly/     此处的脚本每周运行
/etc/cron.monthly/    此处的脚本每月运行
```

将一个可执行脚本放入这些文件夹中的任意一个,它就会按对应计划自动运行——无需编写 crontab 条目。系统维护任务和日志轮转通常就采用这种方式。

---

## 命令参考

| 命令 | 作用 |
|---|---|
| `crontab -e` | 编辑你的个人 cron 计划 |
| `crontab -l` | 查看你当前的 cron 任务 |
| `crontab -r` | 删除你所有的 cron 任务 |
| `sudo crontab -e` | 编辑 root 的 cron 计划 |
| `systemctl enable [service]` | 设置一个服务在启动时自动运行 |
| `systemctl disable [service]` | 从自动启动中移除一个服务 |
| `systemctl list-unit-files --type=service` | 查看所有服务及其启动状态 |

---

## 实践

- [ ] 使用 `crontab -e` 打开你的 crontab,添加一个每分钟将当前日期写入文件的作业——两分钟后验证其正常工作,然后删除该作业
- [ ] 将一个脚本调度为 `@reboot` 运行,并确认它在重启后执行
- [ ] 在 root 上运行 `crontab -l`,查看是否已经存在任何系统任务
- [ ] 使用 `systemctl enable mysql` 让 MySQL 在启动时运行,然后重启并使用 `systemctl status mysql` 检查它是否正在运行
- [ ] 尝试自行编写 cron 语法:每天早上 6:30、仅在工作日运行——在查看答案之前先自己推导

cron 语法需要几次练习才能熟练。从头编写几个不同的计划是将其内化为本能的最快方式。

> 💡 *如需更深入的练习,我还建议完成官方《Linux Basics for Hackers》一书中每章末尾的习题。*
---

[下一章:第 17 章 - Python 脚本编写](Module_17_Python_Scripting.md)
