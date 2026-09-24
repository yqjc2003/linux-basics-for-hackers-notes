# Linux Basics for Hackers
## Module 15 - 内核与可加载内核模块(The Kernel & Loadable Kernel Modules)

> 本文档为 Module_15_Kernel_LKMs.md 的简体中文翻译版本

---

## 概述(Overview)

内核是操作系统的核心。其他所有东西 —— 你的终端、你的程序、你的文件系统 —— 都建立在内核之上。本模块讲的是内核实际做什么、如何检查和调整它的设置,以及模块(LKM,Loadable Kernel Module,可在系统运行时动态加载到内核中的代码片段)是如何作为在不需要重启的前提下扩展内核功能的方式工作的。

---

## 内核实际做了什么

内核处于你的软件和硬件之间。当一个程序想要读取文件、通过网络发送数据,或者在屏幕上画点什么时,它并不会直接跟硬件打交道 —— 它向内核请求,然后由内核来处理。

这让内核成为整个系统上权限最高的软件。它运行在所谓的 **内核空间(Kernel Space)** 中,与你的程序运行所在的 **用户空间(User Space)** 完全隔离开。用户空间里的进程只能做内核允许的事情。内核本身则没有这种限制 —— 它可以访问任何内存、任何硬件、任何东西。

这也是为什么它是个攻击目标的原因。运行在内核里的代码在机器上拥有绝对的权限。

![Linux Kernel Architecture](assets/linux_kernel_architecture_1789213639316.jpg)

---

## 检查你的内核版本

```bash
ahegazy0@kali:~$ uname -a
Linux kali 6.1.0-kali9-amd64 #1 SMP PREEMPT_DYNAMIC Debian 6.1.27-1kali1 x86_64 GNU/Linux
```

从左到右解读这行:
- `Linux` —— 操作系统
- `kali` —— 主机名
- `6.1.0-kali9-amd64` —— 内核版本和架构
- `x86_64` —— 确认这是一个 64 位内核

架构部分(`x86_64` 对 `i686`)是判断 64 位还是 32 位的方法。

---

## 可加载内核模块(LKM)

内核不能每次需要支持一个新硬件就被重新编译替换。取而代之的是,它支持 **可加载内核模块(LKM,Loadable Kernel Module)** —— 可以在系统运行期间插入或移除的内核代码块,不需要重启。

驱动程序是最常见的例子。当插入一个 USB Wi-Fi 网卡时,内核会加载相应的驱动模块来与它通信。当你拔掉它,模块就可以被移除。内核本身没有变化 —— 模块只是临时扩展了它。

因为模块运行在内核空间中,它们拥有和内核本身一样的绝对权限。这也是为什么它们也被用来隐藏 rootkit(深度隐藏的恶意程序) —— 以模块形式插入的恶意代码可以从操作系统本身都难以检查的层面拦截系统调用、隐藏进程、隐藏文件。

---

## 列出已加载的模块

```bash
ahegazy0@kali:~$ lsmod
```

显示当前加载到内核的每个模块:

```
Module                  Size  Used by
bluetooth             663552  2 btusb
snd_hda_intel         57344  3
usbcore              286720  5 btusb,xhci_hcd
```

各列含义:
- `Module` —— 模块名
- `Size` —— 使用的内存量
- `Used by` —— 依赖它的其他模块或进程

通常会看到加载了几十个模块。大多数是硬件驱动 —— 音频、USB、显示、网络。

---

## 获取模块的信息

```bash
ahegazy0@kali:~$ modinfo bluetooth
```

输出包括:
- `filename` —— 模块文件在磁盘上的位置
- `description` —— 这个模块做什么
- `author` —— 作者
- `depends` —— 它的功能需要的其他模块
- `vermagic` —— 它是为哪个内核版本编译的

`depends` 字段特别有用。模块之间经常有依赖关系 —— 必须先加载的其他模块。`modinfo` 会把这个依赖链展示给你。

---

## 加载和移除模块

**添加一个模块:**

```bash
ahegazy0@kali:~$ modprobe bluetooth
```

`modprobe` 是做这件事的正确工具。与底层的替代命令 `insmod` 不同,modprobe 会自动解析并加载模块所需的依赖。如果 `bluetooth` 需要另外两个模块先加载,modprobe 会自动处理。

**移除一个模块:**

```bash
ahegazy0@kali:~$ modprobe -r bluetooth
```

`-r` 标志表示移除。同样,modprobe 会处理依赖 —— 它不会移除仍然有其他东西依赖的模块。

---

## sysctl —— 运行时调整内核设置

内核通过位于 `/proc/sys` 的虚拟文件系统公开大量可配置的设置。`sysctl` 命令让你能读取和修改这些设置,无需重启。

**查看所有当前的内核设置:**

```bash
ahegazy0@kali:~$ sysctl -a
```

那是个很长的列表。要查看特定的一个:

```bash
ahegazy0@kali:~$ sysctl net.ipv4.ip_forward
net.ipv4.ip_forward = 0
```

这个特殊的设置 —— IP 转发 —— 控制本系统是否会从一个接口向另一个接口转发网络数据包。默认是关闭的。当它关闭时,你的机器收到一个数据包,如果判断这个包不是发给自己的,就把它丢弃。

**开启 IP 转发:**

```bash
ahegazy0@kali:~$ sysctl -w net.ipv4.ip_forward=1
```

`-w` 标志意思是"写入(write)"。现在内核会在接口之间转发数据包。这在中间人(Man-in-the-Middle)配置中是必需的 —— 你的机器需要位于流量路径的中间,接收发给别人的数据包再转发出去。

**把 sysctl 修改永久化:**

和环境变量一样,`sysctl -w` 只持续到下一次重启。要使其永久,把它加到 `/etc/sysctl.conf`:

```
net.ipv4.ip_forward = 1
```

然后无需重启就应用:

```bash
ahegazy0@kali:~$ sysctl -p
```

**使用 sysctl 时要小心。** 有些设置控制着基础的内核行为。修改错误的值可能造成网络问题、系统不稳定,甚至更糟。在修改之前,一定要搞清楚这个设置是做什么的。

---

## /proc 文件系统

值得与 sysctl 一起了解 —— `/proc` 是一个只存在于内存中的虚拟文件系统。这是内核以文件的形式公开关于自身和运行中进程的信息。

```bash
ahegazy0@kali:~$ ls /proc
```

你会看到带编号的目录 —— 每个运行中的进程对应一个(编号就是 PID,进程标识符)。你也会看到像 `cpuinfo`、`meminfo` 和 `version` 这样的文件,它们公开硬件和内核数据。

```bash
ahegazy0@kali:~$ cat /proc/cpuinfo
ahegazy0@kali:~$ cat /proc/meminfo
ahegazy0@kali:~$ cat /proc/version
```

这些不是磁盘上的真实文件。它们是由内核在你每次读取时动态生成的。这就是"一切皆文件"哲学的逻辑终点 —— 即使是动态内核数据,也以文件形式呈现。

---

## 命令参考(Command Reference)

| 命令 | 说明 |
|---|---|
| `uname -a` | 显示完整的内核版本和架构 |
| `lsmod` | 列出所有当前加载的内核模块 |
| `modinfo [name]` | 显示某个模块的详细信息 |
| `modprobe [name]` | 加载一个模块(及其依赖) |
| `modprobe -r [name]` | 移除一个模块 |
| `sysctl -a` | 显示所有内核参数 |
| `sysctl [parameter]` | 显示特定的内核参数 |
| `sysctl -w [param=value]` | 运行时修改一个内核参数 |
| `sysctl -p` | 应用 /etc/sysctl.conf 中的修改 |
| `cat /proc/cpuinfo` | 来自内核的 CPU 信息 |
| `cat /proc/meminfo` | 来自内核的内存信息 |

---

## 练习(Practice)

- [ ] 运行 `uname -a`,找出内核版本以及它是 32 位还是 64 位
- [ ] 运行 `lsmod` 翻看一下 —— 大致数一下加载了多少个模块
- [ ] 运行 `modinfo bluetooth`,找出 `depends` 字段 —— 看它依赖哪些模块
- [ ] 运行 `sysctl net.ipv4.ip_forward`,查看当前值 —— 想想改变它的含义

> 💡 *如需进行更深入的练习,我推荐同时完成官方 **Linux Basics for Hackers** 一书中本章末尾的习题。*
---

[下一篇:Module 16 - 自动化与定时任务(Automation & Scheduled Jobs)](Module_16_Automation_Jobs.zh_CN.md)
