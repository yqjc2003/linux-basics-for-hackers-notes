# Linux Basics for Hackers
## Module 14 - 无线网络(Wireless Networking)

> 本文档为 Module_14_Wireless_Networking.md 的简体中文翻译版本

---

## 概述(Overview)

Wi-Fi 是现实世界中最常见的攻击面之一,因为它无处不在,而且常常配置不佳。本模块介绍无线网络实际如何工作、用于检查和审计的工具,以及蓝牙(Bluetooth)在这幅图景中的位置。这里所有的内容都应当只在你拥有或获得明确书面授权测试的网络和设备上使用。

---

## Wi-Fi 实际是如何工作的(对这部分很重要的部分)

当你的笔记本连接到路由器时,它们在某个特定频率上交换无线电信号。正常情况下,你的 Wi-Fi 网卡只关注发给自己的数据包 —— 它会忽略空气中漂浮的所有其他数据包。这叫 **管理模式(Managed Mode)**。

要进行任何严肃的无线分析,你需要让你的网卡能够监听所有内容 —— 范围内所有网络、所有设备的所有数据包,而不只是发给你的那些。这叫做 **监控模式(Monitor Mode)**。这就好比从私人对话切换成能听到整个房间里所有声音。

不是每个 Wi-Fi 网卡都支持监控模式或数据包注入(发出精心构造的无线信号,用来触发特定行为或扰乱网络)。大多数笔记本内置的网卡都不支持。你需要一个明确支持这些功能的外接 USB 网卡 —— Alfa AWUS036ACH 和类似的 Alfa 网卡是这类工作的标准选择。

---

## 无线工具集 —— aircrack-ng

`aircrack-ng` 是一个工具套件,而不是单个程序。你主要会用到:

| 工具 | 说明 |
|---|---|
| `airmon-ng` | 在你的网卡上启用和禁用监控模式 |
| `airodump-ng` | 抓取数据包并显示所有附近的网络 |
| `aireplay-ng` | 向网络中注入数据包(用来强制重新连接等) |
| `aircrack-ng` | 尝试破解捕获到的 WPA 握手包(设备之间为建立加密连接而交换的一组数据包) |

它们作为一个流水线协同工作。很少单独使用某一个。

---

## 把网卡置于监控模式

首先,找到你的无线接口名:

```bash
ahegazy0@kali:~$ iwconfig
```

输出会显示你的接口。一般是 `wlan0` 或类似。

在启用监控模式之前,杀掉任何可能干扰的进程:

```bash
ahegazy0@kali:~$ airmon-ng check kill
```

这会停止 NetworkManager 和其他试图管理接口的后台工具。如果跳过这一步,监控模式经常会出问题,或者接口会不断掉线。

现在启用监控模式:

```bash
ahegazy0@kali:~$ airmon-ng start wlan0
```

你的接口很可能会被重命名为 `wlan0mon`,以表明现在处于监控模式。再用 `iwconfig` 确认一下。

要回到正常的管理模式:

```bash
ahegazy0@kali:~$ airmon-ng stop wlan0mon
```

---

## 用 airodump-ng 扫描附近的网络

网卡处于监控模式后,你就可以捕获空气中的一切:

```bash
ahegazy0@kali:~$ airodump-ng wlan0mon
```

输出会显示范围内的每个网络:

```
BSSID              PWR  Beacons  #Data  CH   MB   ENC   ESSID
AA:BB:CC:DD:EE:FF  -45      120     34   6  130   WPA2  HomeNetwork
11:22:33:44:55:66  -72       80     12  11   54   WPA2  CoffeeShop_WiFi
```

各列含义:

| 列名 | 含义 |
|---|---|
| BSSID | 路由器的 MAC 地址(网卡出厂时烙印的唯一硬件标识) |
| PWR | 信号强度 —— 越负越弱 |
| CH | 网络广播使用的信道(无线传输的频率通道) |
| ENC | 加密类型(WPA2、WPA3、WEP、OPN) |
| ESSID | 你连接时看到的网络名(SSID,即 Wi-Fi 名称) |

要聚焦于某个特定网络并同时看到连接的客户端:

```bash
ahegazy0@kali:~$ airodump-ng --bssid AA:BB:CC:DD:EE:FF --channel 6 -w capture wlan0mon
```

- `--bssid` 过滤到一台具体的路由器
- `--channel` 锁定到该信道,这样在跳频时就不会漏掉数据包
- `-w capture` 把所有捕获的内容写入以 "capture" 开头的文件

---

## 需要了解的 Wi-Fi 术语

**SSID** —— 网络名。你在连接时看到的列表里那个名字。

**BSSID** —— 路由器的 MAC 地址。和 SSID(网管可以随意设置)不同,BSSID 与硬件绑定。

**Channel(信道)** —— Wi-Fi 在 2.4GHz 或 5GHz 频段的特定信道上广播。2.4GHz 频段有 1–14 信道(1、6 和 11 是标准上互不重叠的三个)。5GHz 频段信道更多。

**WPA2** —— 当前 Wi-Fi 网络的加密标准。使用 AES 加密。取代了 WEP(已经完全被攻破)和 WPA。WPA3 是更新的标准,正开始在现代路由器上出现。

**WEP** —— 很老,而且已经完全被破解。使用 WEP 的网络可以在几分钟内被破解。如果你在现实中看到一个,那是台很久没更新的传统设备。

**Handshake(握手包)** —— 当设备连接到 WPA2 网络时,路由器和设备之间交换一个 4 次握手来验证密码。捕获这个握手包就是 WPA2 破解的基础 —— 捕获它,然后离线尝试破解。

---

## 用 iwlist 扫描

如果只是想查看附近网络而不进入监控模式:

```bash
ahegazy0@kali:~$ iwlist wlan0 scan
```

这给你一个可见网络的基础扫描 —— SSID、BSSID、信道、信号强度、加密类型。没有 airodump-ng 那么详细,但不需要监控模式,对侦察阶段足够用,而且不会实际触碰什么。

---

## 使用 BlueZ 处理蓝牙

蓝牙是一种短距离无线协议,被手机、耳机、键盘、音箱、智能手表以及大量其他设备使用。Linux 上用于处理蓝牙的工具来自 **BlueZ** 软件包。

**扫描附近的蓝牙设备:**

```bash
ahegazy0@kali:~$ hcitool scan
Scanning...
    AA:BB:CC:DD:EE:FF    John's iPhone
    11:22:33:44:55:66    Sony WH-1000XM4
```

这会显示处于可发现模式的任何设备。处于可发现模式的设备正在主动广播自身的存在 —— 这就是新设备配对的方式。许多人在没意识到的情况下让设备永久处于可发现模式。

**检查一个设备是否可达:**

```bash
ahegazy0@kali:~$ l2ping AA:BB:CC:DD:EE:FF
```

这就像用 IP 地址 `ping` 那样 ping 一个蓝牙设备 —— 确认设备在范围内并且有响应。

**获取设备信息:**

```bash
ahegazy0@kali:~$ hcitool info AA:BB:CC:DD:EE:FF
```

返回设备名称、制造商、支持的功能以及其他元数据(关于设备本身属性的描述信息,如型号、版本等)。

**更详细的扫描:**

```bash
ahegazy0@kali:~$ hcitool lescan
```

`lescan` 是针对 **低功耗蓝牙(Bluetooth Low Energy, BLE)** 设备的 —— 健身追踪器、智能家居传感器、物联网(IoT)设备。这些设备经常持续广播,正在成为一个不断增长的攻击面。

---

## 无线侦察中你要找什么

在已获得授权的网络上做无线侦察时,关键要找这些东西:

- **开放的网络(OPN)** —— 没有加密,所有传输都可读
- **WEP 网络** —— 已经破掉的加密,几分钟内可破
- **使用弱密码的 WPA2 网络** —— 握手包可捕获,离线用字典文件即可尝试破解
- **隐藏的 SSID** —— 不广播名称的网络,在 airodump-ng 输出中显示为空白 ESSID 字段,但 BSSID 仍然存在
- **处于可发现模式的蓝牙设备** —— 尤其是那些本不应处于可发现模式的设备(键盘、企业笔记本)

---

## 命令参考(Command Reference)

| 命令 | 说明 |
|---|---|
| `iwconfig` | 显示无线接口及其当前模式 |
| `iwlist wlan0 scan` | 扫描附近的 Wi-Fi 网络 |
| `airmon-ng check kill` | 杀掉会干扰监控模式的进程 |
| `airmon-ng start wlan0` | 启用监控模式 |
| `airmon-ng stop wlan0mon` | 禁用监控模式 |
| `airodump-ng wlan0mon` | 抓包并显示所有附近网络 |
| `airodump-ng --bssid [MAC] --channel [CH] -w out wlan0mon` | 抓取某个特定网络 |
| `hcitool scan` | 扫描处于可发现模式的蓝牙设备 |
| `hcitool lescan` | 扫描低功耗蓝牙设备 |
| `l2ping [MAC]` | ping 一个蓝牙设备 |
| `hcitool info [MAC]` | 获取蓝牙设备的详细信息 |

---

## 练习(Practice)

- [ ] 运行 `iwlist wlan0 scan` 看输出 —— 找到你自己的家庭网络,记下信道和加密类型
- [ ] 在你的房间里运行 `hcitool scan`,看哪些蓝牙设备出现 —— 检查有多少处于可发现模式
- [ ] 如果你有兼容的网卡:用 `airmon-ng` 把它设置为监控模式,运行 `airodump-ng` 30 秒,在输出中找到你的家用路由器,记下它的 BSSID 和信道,然后停止抓取并把网卡退出监控模式

只在你自己的家庭网络上做 airodump-ng 的练习。在大多数国家,捕获你不拥有的网络的流量是违法的,无论你是否对数据做了什么。

> 💡 *如需进行更深入的练习,我推荐同时完成官方 **Linux Basics for Hackers** 一书中本章末尾的习题。*
---

*下一篇:Module 15 - 管理 Linux 内核与可加载内核模块(Managing the Linux Kernel & Loadable Kernel Modules)*
