# Linux Basics for Hackers
## 模块 2 - 文本处理

> 本文档为 Module_02_Text_Manipulation.md 的简体中文翻译版本

---

## 概述

在 Linux 里,几乎一切都是文本文件。系统设置、日志、配置、用户数据——都是文本。一旦你懂得如何搜索和处理文本,就能在浩如烟海的输出中翻阅数千行内容,瞬间得到你所需的信息。这正是本模块的全部要点。

---

## 这对黑客有什么意义

当扫描工具运行并产生输出时,它不会给你一份简洁的总结,而是把所有内容一股脑倒出来——成千上万行。你不会手动去读。你需要借助文本工具,只过滤出你关心的内容。举个真实例子:扫描完网络后,你会用 `grep` 抓取仅含 "open" 的行,从而找到开放端口,其余一概忽略。

此外,几乎每个系统配置都以纯文本的形式存放在 `/etc` 下的某处。如果你能读写这些文件,几乎就能重新配置系统的任何部分。

---

## 必备命令

### cat

"concatenate"(连接)的缩写。这是读取文件最简单的方式——它只是把整个内容倾倒到屏幕上。

```bash
ahegazy0@kali:~$ cat /etc/passwd
```

> **注意:** 切勿对已编译的二进制文件使用 `cat`(例如 `cat /bin/ls`)。它会把原始字节码直接倒到屏幕上,把终端字体搅成一堆乱码。如果真的发生了,闭着眼输入 `reset` 然后回车即可。

它还可以用来快速创建小文件:

```bash
ahegazy0@kali:~$ cat > targets.txt
```

运行之后,逐行输入内容。输入完毕后按 `Ctrl+D` 保存并退出。你输入的内容就保存到 `targets.txt` 中了。

若想在不覆盖现有文件的情况下追加内容,使用 `>>`:

```bash
ahegazy0@kali:~$ cat >> targets.txt
```

`>` 与 `>>` 的区别很重要。`>` 表示覆盖,`>>` 表示追加。

---

### grep

这是你最常用的命令。`grep` 在文件中搜索,只返回包含特定单词或模式的行。

```bash
ahegazy0@kali:~$ grep "password" logs.txt
```

返回 `logs.txt` 中所有含有 "password" 的行。

常用选项:

| 选项 | 作用 |
|---|---|
| `-i` | 大小写不敏感搜索(可匹配 "Password"、"PASSWORD" 等) |
| `-r` | 递归地搜索目录中的所有文件 |
| `-n` | 在结果中同时显示行号 |
| `-v` | 反向 - 显示不包含该词的所有行 |

示例:
```bash
ahegazy0@kali:~$ grep -i "admin" access.log
```

可以匹配 "admin"、"Admin"、"ADMIN"——全部命中。

---

### head 与 tail

当文件很大、你只想看一小部分时:

```bash
ahegazy0@kali:~$ head /etc/snort/snort.conf
```

默认显示前 10 行。

```bash
ahegazy0@kali:~$ tail /var/log/syslog
```

默认显示最后 10 行。

若要控制行数:

```bash
ahegazy0@kali:~$ head -n 20 file.txt     前 20 行
ahegazy0@kali:~$ tail -n 20 file.txt     后 20 行
```

`tail` 有一个尤为实用的技巧——`-f` 参数,可以实时跟踪文件:

```bash
ahegazy0@kali:~$ tail -f /var/log/syslog
```

它会保持打开状态,文件新增的内容会即时显示出来。在某程序运行时实时观察日志非常有用。

---

### nl

为文件的输出添加行号。当你需要引用具体行号时非常方便。

```bash
ahegazy0@kali:~$ nl /etc/snort/snort.conf
```

---

### less

当文件过长,用 `cat` 无法阅读时(它会飞速刷过屏幕),改用 `less`。它允许你一次滚动一页来阅读文件。

```bash
ahegazy0@kali:~$ less /etc/snort/snort.conf
```

操作方式:
- `Space` - 下一页
- `b` - 上一页
- `/word` - 搜索单词
- `q` - 退出

---

### sed

"stream editor"(流编辑器)的缩写。它在文件中查找某个单词或模式,并将其替换为其他内容。

```bash
ahegazy0@kali:~$ sed 's/mysql/MySQL/g' config.txt
```

拆解来看:
- `s/` - 替换(substitute)
- `mysql` - 查找这个
- `/MySQL/` - 替换为这个
- `g` - 全局替换(所有出现之处,不仅是第一处)

默认情况下,`sed` 仅打印修改后的输出——它并不会真正修改文件。若想把改动保存回文件,请加 `-i`:

```bash
ahegazy0@kali:~$ sed -i 's/mysql/MySQL/g' config.txt
```

> 使用 `-i` 前请先备份。`sed -i` 会永久修改文件,而且无法撤销。

```bash
ahegazy0@kali:~$ cp config.txt config.txt.bak
ahegazy0@kali:~$ sed -i 's/old/new/g' config.txt
```

---

## 管道 - 将命令串联起来

管道 `|` 将一个命令的输出直接送入下一个命令。这是威力倍增的关键。

![Linux 管道示意图](assets/linux_pipe_diagram_1789213616381.jpg)

```bash
ahegazy0@kali:~$ cat access.log | grep "failed"
```

你无需阅读整个日志,只会看到包含 "failed" 的那些行。

你可以串联任意数量的管道:

```bash
ahegazy0@kali:~$ cat access.log | grep "failed" | tail -n 20
```

它先读取日志,过滤出 "failed",再显示这些结果的最后 20 行。三条命令,一行完成。

---

## 命令速查

| 命令 | 作用 | 示例 |
|---|---|---|
| `cat file` | 将整个文件打印到屏幕 | `cat /etc/passwd` |
| `cat > file` | 创建文件并输入内容 | `cat > notes.txt` |
| `grep "word" file` | 查找包含某词的所有行 | `grep "root" passwd` |
| `grep -i` | 大小写不敏感搜索 | `grep -i "admin" log` |
| `head -n 20 file` | 前 20 行 | `head -n 20 file.txt` |
| `tail -n 20 file` | 后 20 行 | `tail -n 20 file.txt` |
| `tail -f file` | 实时跟踪文件 | `tail -f syslog` |
| `nl file` | 显示文件并带行号 | `nl config.conf` |
| `less file` | 滚动浏览文件 | `less bigfile.txt` |
| `sed 's/a/b/g' file` | 将所有 "a" 替换为 "b" | `sed 's/old/new/g' f` |
| `cmd1 \| cmd2` | 将 cmd1 的输出送入 cmd2 | `cat log \| grep fail` |

---

## 练习

- [ ] 进入 `/etc/snort/` 目录,用 `less` 打开 `snort.conf` —— 滚动浏览一下,感受配置文件的样子
- [ ] 执行 `grep "output" /etc/snort/snort.conf`,看看哪些行被返回
- [ ] 执行 `cat > targets.txt`,输入三个 IP 地址(每行一个),然后按 `Ctrl+D` —— 之后用 `cat targets.txt` 读回来
- [ ] 尝试串联:`cat /etc/snort/snort.conf | grep "output" | nl`

> 提示:*如需更深入的练习,建议同步完成官方 **Linux Basics for Hackers** 一书中各章末尾的练习题。*
---

*下一章:模块 3 - 网络管理*