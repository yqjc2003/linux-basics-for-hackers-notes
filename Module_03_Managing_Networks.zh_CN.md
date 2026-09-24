# Linux Basics for Hackers
## 模块 3 - 网络管理

> 本文档为 Module_03_Managing_Networks.md 的简体中文翻译版本

---

## 概述

黑客活动几乎都通过网络进行。在你能通过网络做任何事之前,你需要先了解自己在网络上的身份——你的 IP 地址、MAC 地址——以及如何管理它们。本模块涵盖读取网络信息、更改信息以及 DNS 工作原理的基础知识。

![网络路由](assets/network_routing_diagram_1789213191969.jpg)

---

## 每个设备都拥有的两个地址

网络上的每个设备都带有两个标识符:

**IP 地址** - 这是你在网络上的逻辑地址。它由路由器(或你手动)分配,是数据知道送往何处的依据。可以把它想象成街道地址——它告诉网络你身在何处,但它会变。

**MAC 地址** - 这是由制造商烧录进物理网卡中的硬件级标识符。它是 12 个十六进制字符,在全球范围内对每个网络接口都唯一。它形如: `00:1A:2B:3C:4D:5E`。与 IP 地址不同,你无法永久修改它——但你可以临时伪装它。

理解这两者之所以重要,是因为当你在网络中时,正是它们标识了你的身份。掌握它们的读取与更改是一项基础技能。

---

## 必备命令

### ifconfig

这是读取与管理网络接口的主要工具。不带参数运行时,它会显示当前网络配置的全部信息。

```bash
ahegazy0@kali:~$ ifconfig
```

输出会列出机器的每一个网络接口。常见的接口:

| 接口 | 含义 |
|---|---|
| `eth0` | 有线以太网连接 |
| `wlan0` | 无线(Wi-Fi)连接 |
| `lo` | 本地回环——系统与自身的通信,始终为 127.0.0.1 |

在输出中,留意 `inet` —— 那就是你当前的 IP 地址。留意 `ether` —— 那就是你的 MAC 地址。

若只想查看特定接口:

```bash
ahegazy0@kali:~$ ifconfig eth0
```

若要为接口分配新的 IP 地址:

```bash
ahegazy0@kali:~$ ifconfig eth0 192.168.1.100
```

若要同时设置子网掩码:

```bash
ahegazy0@kali:~$ ifconfig eth0 192.168.1.100 netmask 255.255.255.0
```

启用或停用某个接口:

```bash
ahegazy0@kali:~$ ifconfig eth0 up
ahegazy0@kali:~$ ifconfig eth0 down
```

有时在做完更改之后需要先把接口关闭再重新启用。

---

### iwconfig

类似 `ifconfig`,但专用于无线接口。它会显示 Wi-Fi 特有的信息,例如网络名称(ESSID)、信号强度、传输速率。

```bash
ahegazy0@kali:~$ iwconfig
```

大多数时候你只是在读取它的输出,而不是写入。但它在确认你连接到了哪个无线网络、信号状况如何时非常有用。

---

### dhclient

当你用 `ifconfig` 手动设置了 IP 地址,就无法再从路由器自动获取 IP。若想回到从网络自动获取 IP 的状态,使用 `dhclient`。

```bash
ahegazy0@kali:~$ dhclient eth0
```

它会向网络的 DHCP 服务器发出请求,告诉服务器"给我分配一个 IP 地址",服务器就会分配给你一个。如果你手动改过 IP 后上不了网,通常运行这个命令即可恢复。

你也可以用它在认为现有 IP 引发问题时重新申请 IP:

```bash
ahegazy0@kali:~$ dhclient -r eth0     释放当前 IP
ahegazy0@kali:~$ dhclient eth0        请求新的 IP
```

---

### 修改 MAC 地址

MAC 地址本应保持不变,但在软件层面很容易伪装。这种修改只持续到重启——重启之后,真实的硬件 MAC 会恢复。

首先关闭接口:

```bash
ahegazy0@kali:~$ ifconfig eth0 down
```

然后设置新的 MAC:

```bash
ahegazy0@kali:~$ ifconfig eth0 hw ether 00:11:22:33:44:55
```

接着重新启用接口:

```bash
ahegazy0@kali:~$ ifconfig eth0 up
```

运行 `ifconfig eth0` 确认修改已生效。`ether` 一行现在应显示你新设置的 MAC。

> MAC 地址的修改仅存在于内存中,重启后即失效。如果你希望修改长期生效,可以借助 `macchanger` 等工具来更稳妥地处理。

---

### dig

`dig` 用于查询 DNS —— 即把像 `google.com` 这样的域名翻译成 IP 地址的系统。它比单纯地 ping 域名提供更多细节。

```bash
ahegazy0@kali:~$ dig google.com
```

输出中的 `ANSWER SECTION` 会显示 `google.com` 所解析到的 IP 地址。

若要专门查找邮件服务器记录:

```bash
ahegazy0@kali:~$ dig google.com mx
```

`mx` 是 Mail Exchange 的缩写——它会告诉你该域名由哪些服务器处理邮件。侦察阶段很有用。

其他值得了解的记录类型:

| 记录类型 | 含义 |
|---|---|
| `a` | 域名的 IPv4 地址 |
| `aaaa` | IPv6 地址 |
| `mx` | 邮件服务器 |
| `ns` | 域名服务器(授权的 DNS 服务器) |
| `txt` | 文本记录——通常包含验证信息 |

```bash
ahegazy0@kali:~$ dig google.com ns
```

---

## DNS - 它为何重要的简要说明

DNS(域名系统,Domain Name System)是互联网的地址簿。当你在浏览器中输入 `google.com` 时,你的计算机会向 DNS 服务器询问"google.com 的 IP 是多少?",并得到一个地址返回。没有 DNS,你就必须记住每一个网站的 IP 地址。

黑客关心 DNS 有几方面原因。首先,DNS 查询会泄露目标基础设施的许多信息——邮件服务器、子域名、域名服务器。其次,DNS 可以被操纵——如果你能篡改 DNS 响应,你就能把人引导到完全错误的服务器(这称为 DNS 投毒或 DNS 欺骗,后续模块会介绍)。

现在,只要掌握用 `dig` 读取 DNS 记录就足够了。

---

## 命令速查

| 任务 | 命令 |
|---|---|
| 查看所有网络接口 | `ifconfig` |
| 查看某个接口 | `ifconfig eth0` |
| 设置 IP 地址 | `ifconfig eth0 192.168.1.100` |
| 启用 / 停用接口 | `ifconfig eth0 down / up` |
| 查看无线信息 | `iwconfig` |
| 从 DHCP 请求 IP | `dhclient eth0` |
| 释放当前 IP | `dhclient -r eth0` |
| 伪装 MAC 地址 | `ifconfig eth0 hw ether 00:11:22:33:44:55` |
| DNS 查询 | `dig google.com` |
| 查找邮件服务器 | `dig google.com mx` |

---

## 练习

- [ ] 运行 `ifconfig`,在 `eth0` 或 `wlan0` 上找到你的 IP 地址和 MAC 地址
- [ ] 运行 `dig google.com`,阅读答案段——记下返回的 IP 地址
- [ ] 运行 `dig google.com mx`,看看 Google 的邮件由哪些服务器处理
- [ ] 尝试将你的 IP 改为 `192.168.1.100`,用 `ifconfig` 确认,然后运行 `dhclient eth0` 从 DHCP 重新获取一个

> 提示:*如需更深入的练习,建议同步完成官方 **Linux Basics for Hackers** 一书中各章末尾的练习题。*
---

[下一章:模块 4 - 软件管理](Module_04_Software_Management.zh_CN.md)