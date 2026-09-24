# Linux Basics for Hackers
## Module 8 - Bash 脚本编写(Bash Scripting)

> 本文档为 Module_08_Bash_Scripting.md 的简体中文翻译版本

---

## 概述(Overview)

一个不断重复敲同样命令的黑客是在浪费时间。脚本就是自动化的方式——你把命令写在一个文件里一次,然后只需要运行这个文件即可。本模块介绍如何编写 Bash 脚本,从最基本的"hello world"到一个真正能接受输入并完成某些有用工作的脚本。

---

## 什么是脚本(What a script is)

脚本就是一个纯文本文件,里面包含一系列命令。当你运行它时,Bash 会按顺序逐行读取并执行这些命令,效果就和你自己在终端里逐条敲入完全一样。区别在于它只需要几秒就能完成,而且你再也不用重复敲一遍。

你可以在脚本里放入任何在终端中能运行的命令。`ls`、`ping`、`nmap`、`grep`——这些通通都可以。你还可以用逻辑将它们组合起来:如果输出中包含某个字符串,就执行另一条命令。脚本的强大之处正源于此。

---

## Shebang 行(The shebang line)

每个 Bash 脚本的第一行都要写上这样一行:

```bash
#!/bin/bash
```

这叫做 **shebang**(也读作 hash-bang)。它用来告诉系统应该使用哪个程序来解释这个文件。如果没有它,Linux 可能不知道该如何处理这个文件——或者用了错误的 shell,导致脚本以莫名其妙的方式报错。

一定要把它放在第一行。它前面不能有空行,也不能有任何其他内容。

---

## 创建你的第一个脚本(Creating your first script)

打开文本编辑器,创建一个名为 `myscript.sh` 的文件:

```bash
#!/bin/bash
echo "Hello, world"
```

保存文件。运行它之前,需要先让它变成可执行的:

```bash
ahegazy0@kali:~$ chmod 755 myscript.sh
```

这一步是授予你以程序方式运行它的权限。没有这一步,Linux 会拒绝执行,并提示"Permission denied(权限被拒绝)"。

现在运行它:

```bash
ahegazy0@kali:~$ ./myscript.sh
Hello, world
```

开头的 `./` 意思是"从当前目录运行这个文件"。之所以需要它,是因为当前目录通常不在你的 PATH 里,所以 Linux 找不到这个脚本。

---

## echo — 在屏幕上打印

`echo` 会把你传给它的内容打印到终端上。它是你的脚本向你回显信息的方式。

```bash
ahegazy0@kali:~$ echo "Scan starting..."
ahegazy0@kali:~$ echo "Done."
```

你也可以输出变量的值:

```bash
ahegazy0@kali:~$ echo "Your username is: $USER"
Your username is: kali
```

---

## 脚本中的变量(Variables in scripts)

变量用于存储数据并复用它。赋值时等号两边不能有空格:

```bash
ahegazy0@kali:~$ name="Bob"
ahegazy0@kali:~$ echo "Hello, $name"
Hello, Bob
```

当你想要使用变量里存储的值时,在变量名前面加上 `$`。赋值时则不加 `$`。

---

## read — 从用户获取输入

`read` 会暂停脚本,等待用户输入内容,然后将用户输入的内容存入一个变量。

```bash
#!/bin/bash
echo "What is your name?"
read name
echo "Hello, $name"
```

运行后脚本会等待。你输入 "Alice" 并回车:

```
What is your name?
Alice
Hello, Alice
```

你也可以用 `-p` 参数(prompt,提示)把提示和读取合并为一行:

```bash
ahegazy0@kali:~$ read -p "Enter your name: " name
```

这样更简洁,效果完全一样。

---

## 一个实用的例子 — ping 扫描器(A practical example - ping scanner)

下面是一个简单的脚本,它会要求你输入一个 IP 地址,然后对它执行 ping。这是你会真正用得上的那种东西:

```bash
#!/bin/bash
read -p "Enter IP address to ping: " target
echo "Pinging $target..."
ping -c 4 $target
```

`-c 4` 告诉 ping 恰好发送 4 个包然后停止。如果不加它,ping 会一直跑下去。

把它保存为 `pinger.sh`,然后 `chmod 755 pinger.sh`,再通过 `./pinger.sh` 运行。

---

## 注释(Comments)

任何以 `#` 开头的行都是注释。Bash 会完全忽略它。用注释来解释你的脚本在做些什么,尤其是那些不直观的部分。

```bash
#!/bin/bash
# This script pings a target IP address
# Written for practice - Module 8

read -p "Enter target IP: " target
ping -c 4 $target   # send 4 packets only
```

注释是写给你自己(以及日后可能阅读这个脚本的其他人)的。就像两周后你一定会忘记这些代码的作用一样去写注释——因为事实确实如此。

---

## 条件逻辑 — if/else(Conditional logic - if/else)

一旦脚本里有了判断能力,它就真正能派上用场了。基本语法:

```bash
#!/bin/bash
read -p "Enter a number: " num

if [ $num -gt 10 ]; then
echo "That number is greater than 10"
else
echo "That number is 10 or less"
fi
```

`fi` 是用来闭合 `if` 块的(就是 `if` 反过来写)。`-gt` 的意思是"大于(greater than)"。常用的比较运算符如下:

| 运算符 | 含义 |
|---|---|
| `-eq` | 等于(Equal to) |
| `-ne` | 不等于(Not equal to) |
| `-gt` | 大于(Greater than) |
| `-lt` | 小于(Less than) |
| `-ge` | 大于或等于(Greater than or equal) |
| `-le` | 小于或等于(Less than or equal) |

如果要比较字符串,在方括号里使用 `=` 和 `!=`。

---

## 循环 — 重复执行(Loops - doing something repeatedly)

`for` 循环可以多次执行一段命令块:

```bash
#!/bin/bash
for i in 1 2 3 4 5; do
echo "Pinging 192.168.1.$i"
ping -c 1 192.168.1.$i
done
```

这段代码会依次 ping 五个不同的 IP 地址。这其实就是一个原始的网络扫描器。真正的扫描器(如 nmap)会把这个操作放大上千次并在其周围加上更多逻辑,但核心概念是相同的。

---

## 让脚本真正可用 — 推荐的结构

一个值得保留的脚本通常具有以下结构:

```bash
#!/bin/bash
# Script name and what it does
# Your name, date

# --- Variables ---
target=""

# --- Input ---
read -p "Enter target: " target

# --- Main logic ---
echo "Running scan on $target..."
nmap -sV $target

# --- Done ---
echo "Scan complete."
```

分节清晰、需要解释的地方加注释、逻辑自上而下流畅执行。你不必完全照搬这种结构,但当你写超过 20 行时,有一定的结构会让脚本不至于变得难以阅读。

---

## 文件权限回顾 — chmod(File permissions recap - chmod)

当你创建一个脚本时,它默认只是一个普通的文本文件。要运行它,需要设置执行权限。

```bash
ahegazy0@kali:~$ chmod 755 myscript.sh
```

`755` 的含义如下:
- `7` — 属主可读、可写、可执行
- `5` — 属组可读、可执行
- `5` — 其他用户可读、可执行

对于只需要你自己运行的个人脚本,使用 `chmod 700` 即可——只有你自己能对它进行任何操作。

---

## 命令参考(Command Reference)

| 命令 / 概念 | 作用 |
|---|---|
| `#!/bin/bash` | Shebang —— 必须是每个脚本的第一行 |
| `echo "text"` | 把文本打印到屏幕上 |
| `read var` | 获取用户输入并存入变量 |
| `read -p "prompt" var` | 同上,只是带一个内联提示 |
| `name="value"` | 给变量赋值 |
| `$name` | 使用变量的值 |
| `chmod 755 file.sh` | 让脚本变得可执行 |
| `./script.sh` | 运行当前目录下的脚本 |
| `# comment` | Bash 会忽略的一行 —— 写给人类的备注 |
| `if [ ] then / fi` | 条件逻辑 |
| `for x in ... do / done` | 遍历一个列表的循环 |

---

## 练习(Practice)

- [ ] 编写一个脚本,询问你的名字并打印 "Hello, [名字]"
- [ ] 用 `chmod 755` 让它可执行,并通过 `./` 运行它
- [ ] 再写一个脚本,要求输入一个 IP 地址,并对其执行 `ping -c 4`
- [ ] 上面能跑通后,改造它,使它在 `192.168.1.1` 到 `192.168.1.5` 的范围内循环 ping 每个地址

ping 循环的练习值得认真做。它是一项真实的技术,弄懂它的工作原理,后面看 nmap 的输出时也会更有感觉。

> 💡 *想要更深入的练习,我还建议完成官方书籍 **Linux Basics for Hackers** 中本章末尾的习题。*
---

[下一篇:Module 9 - 归档与压缩(Archiving & Compression)](Module_09_Archiving_Compression.zh_CN.md)
