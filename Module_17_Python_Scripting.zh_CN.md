# Linux Basics for Hackers
## 第 17 章 - Python 脚本编写

> 本文档为 Module_17_Python_Scripting.md 的简体中文翻译版本

---

## 概述

对于简单的自动化和快捷的单行命令,Bash 脚本可以应对得很远。当事情变得更加复杂时——当你需要解析数据、处理网络、构建能够做出决策的工具,或是理解别人编写的漏洞利用代码时——Python 就是你会求助于的工具。本章涵盖作为安全工作的基础所必需的 Python 基本知识。

---

## 为何选择 Python 从事安全工作

Python 在安全和黑客领域长期以来都是主流语言。原因并不复杂:

- 语法简洁、可读性强——你可以快速理解脚本的功能
- 标准库涵盖了网络、文件 I/O、密码学等领域,无需额外安装任何东西
- 存在成千上万个用于特定安全任务的第三方库
- 你在网上找到的大多数漏洞利用、工具和概念验证代码都是用 Python 编写的
- 它能在所有运行 Linux 的平台上运行

即便在你尚未开始编写自己的工具之前,能够阅读 Python 实际上也是真正有用的。你遇到的大多数漏洞利用都是 Python 脚本。如果读不懂它们,你就是在盲目工作。

---

## Python 2 vs Python 3

Python 2 已经消亡——自 2020 年起已正式停止生命周期(EOL)。你编写的所有代码都应是 Python 3。Kali 默认自带 Python 3。当你看到老教程使用 `print "hello"` 而没有括号时,那是 Python 2 语法。在 Python 3 中,它是 `print("hello")`。

检查你的版本:

```bash
ahegazy0@kali:~$ python3 --version
```

启动 Python 3 解释器:

```bash
ahegazy0@kali:~$ python3
```

这会进入一个交互式 Shell,你可以在其中直接输入 Python 代码,适合测试小程序。使用 `exit()` 或 `Ctrl+D` 退出。

---

## 你的第一个脚本

创建一个名为 `hello.py` 的文件:

```python
#!/usr/bin/env python3

print("Hello, world")
```

shebang 行与 Bash 略有不同——`/usr/bin/env python3` 会在系统中任何位置找到 Python 3 解释器,这比硬编码路径更具可移植性。

运行它:

```bash
ahegazy0@kali:~$ python3 hello.py
```

或者赋予可执行权限后直接运行:

```bash
ahegazy0@kali:~$ chmod 755 hello.py
ahegazy0@kali:~$ ./hello.py
```

---

## 变量与数据类型

Python 是动态类型的——你不需要声明类型,只需赋值,Python 会自行判断。

```python
name = "Alice"           # 字符串
port = 80                # 整数
pi = 3.14                # 浮点数
active = True            # 布尔值
```

**字符串(String)** - 文本,始终用引号括起来:

```python
target = "192.168.1.1"
print("Scanning: " + target)
print(f"Scanning: {target}")    # f-string,更整洁的变量内嵌方式
```

**整数(Integer)** - 整数,常用于端口、计数、索引:

```python
port = 22
print(port + 1)    # 23
```

**列表(List)** - 有序集合,类似于数组:

```python
ports = [22, 80, 443, 3306]
print(ports[0])      # 22 - 索引从 0 开始
print(ports[-1])     # 3306 - 负索引从末尾开始计数
```

**字典(Dictionary)** - 键值对,类似于查找表:

```python
user = {"username": "admin", "password": "password123", "role": "root"}
print(user["username"])    # admin
```

字典在安全脚本中非常常见——你会看到它们用于存储解析后的数据、HTTP 头、配置值等。

---

## 获取用户输入

```python
target = input("Enter target IP: ")
print("Scanning " + target)
```

`input()` 会暂停并等待,然后将用户输入的内容以字符串形式存储。如果你需要将其作为数字使用:

```python
port = int(input("Enter port: "))
```

`int()` 将字符串转换为整数。不进行此转换,`"80" + 1` 会抛出错误——Python 不会默默地混合类型。

---

## 条件判断

```python
password = input("Enter password: ")

if password == "secretpass":
    print("Access granted")
elif password == "admin":
    print("Admin access")
else:
    print("Wrong password")
```

Python 使用缩进来定义代码块——没有花括号。标准是 4 个空格。如果缩进不一致,Python 将抛出错误。这是几乎每一个来自其他语言的初学者都会遇到的问题。

常用比较运算符:

| 运算符 | 含义 |
|---|---|
| `==` | 等于 |
| `!=` | 不等于 |
| `>` `<` | 大于 / 小于 |
| `>=` `<=` | 大于等于 / 小于等于 |
| `in` | 检查某个值是否存在于列表或字符串中 |

---

## 循环

**For 循环** - 遍历列表或范围:

```python
ports = [22, 80, 443]
for port in ports:
    print(f"Checking port {port}")
```

```python
for i in range(1, 256):
    print(f"192.168.1.{i}")
```

`range(1, 256)` 生成从 1 到 255 的数字。这便是构建一个用于扫描的基本 IP 范围的方法。

**While 循环** - 在条件为真时持续运行:

```python
attempts = 0
while attempts < 3:
    password = input("Password: ")
    if password == "secret":
        print("Access granted")
        break
    attempts += 1
print("Too many attempts")
```

`break` 立即退出循环。`continue` 跳过本轮,进入下一次迭代。

---

## 函数

函数允许你编写一次代码块,然后在需要时通过名称调用它:

```python
def scan_port(ip, port):
    print(f"Scanning {ip}:{port}")

scan_port("192.168.1.1", 80)
scan_port("192.168.1.1", 443)
```

函数可以返回值:

```python
def add(a, b):
    return a + b

result = add(3, 4)
print(result)    # 7
```

结构良好的脚本会将逻辑放在函数中,并在文件底部调用它们。这让代码更具可读性和可复用性。

---

## 导入库

Python 真正的强大之处在于它的库。通过 `import` 引入它们:

```python
import socket
import os
import sys
```

**socket** - 用于网络连接,是网络工具的基础:

```python
import socket

s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
s.connect(("google.com", 80))
print("Connected")
s.close()
```

这会打开一个到 google.com 的 80 端口 TCP 连接——与浏览器通过 HTTP 访问一个站点时所做的操作相同。

**os** - 用于与操作系统交互:

```python
import os

os.system("ls -la")           # 运行 shell 命令
cwd = os.getcwd()             # 获取当前目录
files = os.listdir(".")       # 列出当前目录下的文件
```

**sys** - 用于命令行参数等系统级内容:

```python
import sys

print(sys.argv)               # 传给脚本的参数列表
# python3 script.py 192.168.1.1 80
# sys.argv = ['script.py', '192.168.1.1', '80']
```

这允许你在运行脚本时向其传递参数,而不是硬编码值。

---

## 安装第三方库

标准库涵盖了大量内容,但 Python 生态系统中还有数十万个用于专门任务的第三方包:

```bash
ahegazy0@kali:~$ pip3 install requests
ahegazy0@kali:~$ pip3 install scapy
ahegazy0@kali:~$ pip3 install paramiko
```

- `requests` - 比直接使用 socket 更简洁的 HTTP 请求
- `scapy` - 强大的数据包构造与分析
- `paramiko` - 在 Python 中进行 SSH 连接

安装后再导入:

```python
import requests

response = requests.get("http://example.com")
print(response.status_code)
print(response.text)
```

---

## 一个实际示例 - 基本端口扫描器

下面的示例将上述大部分内容整合成一个真正有用的工具:

```python
#!/usr/bin/env python3
import socket
import sys

def scan_port(ip, port):
    s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
    s.settimeout(1)
    result = s.connect_ex((ip, port))   # 如果连接成功则返回 0
    s.close()
    return result == 0

target = input("Enter target IP: ")
print(f"\nScanning {target}...\n")

for port in range(1, 1025):
    if scan_port(target, port):
        print(f"Port {port} is OPEN")

print("\nScan complete.")
```

`connect_ex` 尝试连接并返回错误码,而不是抛出异常——0 表示成功(端口开放)。`settimeout(1)` 意味着每个端口等待时间不超过 1 秒,然后继续。

这是 nmap 所做工作的一个简化版本。理解它会让 nmap 的输出更易懂。

---

## 错误处理

事情总会出错。网络连接失败、文件不存在、用户输入了错误的内容。Python 使用 `try/except` 来优雅地处理这些情况:

```python
try:
    s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
    s.connect(("192.168.1.1", 80))
    print("Connected")
except socket.error as e:
    print(f"Connection failed: {e}")
finally:
    s.close()
```

`try` - 尝试执行这部分
`except` - 如果失败,改执行这部分
`finally` - 无论如何都执行这部分(清理代码放在这里)

没有错误处理,一次失败的连接就会使整个脚本崩溃。有了它,你的脚本会继续运行,并告诉你发生了什么错误。

---

## 命令参考

| 命令 | 作用 |
|---|---|
| `python3 script.py` | 运行一个 Python 脚本 |
| `python3` | 打开交互式 Python Shell |
| `pip3 install [package]` | 安装一个第三方库 |
| `pip3 list` | 显示已安装的包 |
| `pip3 show [package]` | 显示某个已安装包的详细信息 |

---

## Python 速查表

| 概念 | 语法 |
|---|---|
| 打印输出 | `print("text")` |
| 变量 | `name = "value"` |
| 用户输入 | `x = input("prompt: ")` |
| 字符串格式化 | `f"Hello {name}"` |
| If/else | `if x == y:` / `else:` |
| For 循环 | `for item in list:` |
| While 循环 | `while condition:` |
| 函数 | `def name(params):` |
| 导入 | `import socket` |
| 列表 | `items = [1, 2, 3]` |
| 字典 | `d = {"key": "value"}` |
| Try/except | `try:` / `except Error as e:` |

---

## 实践

- [ ] 编写一个询问你名字并使用 f-string 打印问候语的脚本
- [ ] 编写一个遍历端口 20–25 并逐一打印的脚本
- [ ] 构建上面的端口扫描器,并对 `127.0.0.1` (你自己的机器) 进行测试——看看自己机器上哪些端口是开放的
- [ ] 编写一个脚本,询问密码,根据硬编码值打印 "correct" 或 "incorrect"
- [ ] 当你熟悉以上内容后:为端口扫描器添加错误处理,以使网络错误不会使整个扫描崩溃

> 💡 *如需更深入的练习,我还建议完成官方《Linux Basics for Hackers》一书中每章末尾的习题。*
---

## 下一步往何处去

本章只是一个基础。书中内容到此结束,但用于安全工作的 Python 学习还有更广阔的空间:

- **Scapy** - 构造并发送自定义数据包,在数据包级别构建你自己的扫描器和嗅探器
- **Paramiko** - 自动化 SSH 连接,用于编写访问远程系统的脚本
- **Requests + BeautifulSoup** - 网页抓取与 HTTP 交互
- **Subprocess** - 从 Python 调用系统命令并捕获其输出
- 阅读已有的漏洞利用代码 - GitHub 上大多数 CVE 概念验证代码都是 Python。能够阅读并修改它们,是这个领域最实用的技能之一。

此后的模式与整个课程中一直采用的一样:理解概念、运行命令、在你的 VM 中搞坏东西、当无法正常工作时查阅资料。这才是真正学会这些东西的方式。

---

*Linux Basics for Hackers 至此结束 - 完成全部 17 章。*
