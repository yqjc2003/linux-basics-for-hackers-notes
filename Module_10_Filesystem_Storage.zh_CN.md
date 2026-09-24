# Linux Basics for Hackers
## Module 10 - 文件系统与存储(Filesystem & Storage)

> 本文档为 Module_10_Filesystem_Storage.md 的简体中文翻译版本

---

## 概述(Overview)

Linux 与存储设备的关系,和 Windows 完全不同。没有盘符,插入设备时也不会自动弹窗。一切皆文件,每个设备都位于文件系统树中的某个位置,而如果你想用一块硬盘,就必须显式地把它挂载进来。本模块会解释这一切是如何运作的。

---

## 一切皆文件(Everything is a file)

在 Linux 中,"一切皆文件(Everything is a file)"这句话绝不只是口号——它正是系统构建的底层逻辑。你的硬盘并不是文件系统之外的独立存在,而是以文件的形式出现在 `/dev` 中。你的键盘是一个文件。网卡也是一个文件。设备不过是内核知道如何与之通信的特殊文件。

存储设备位于 `/dev`,命名方式如下:

| 设备 | 它是什么 |
|---|---|
| `/dev/sda` | 第一块硬盘 |
| `/dev/sdb` | 第二块硬盘 |
| `/dev/sdc` | 第三块硬盘(或 USB 设备) |
| `/dev/sda1` | 第一块硬盘上的第一个分区 |
| `/dev/sda2` | 第一块硬盘上的第二个分区 |
| `/dev/sr0` | CD/DVD 驱动器 |

`sd` 是 SCSI disk 的缩写——这种命名约定来自一个较老的接口标准,但现在也被用于现代 SATA 和 USB 设备。`sd` 后面的字母表示硬盘的次序(`a` = 第一块,`b` = 第二块),再后面的数字是分区编号。

所以 `/dev/sdb3` 意思是:第二块硬盘上的第三个分区。

---

## 挂载 — Linux 把硬盘接入系统的方式(Mounting - how Linux attaches drives)

在 Windows 里,插上 USB,系统会自动分配一个像 `D:` 这样的盘符。Linux 不会这样做,你必须自己**挂载(mount)** 这块硬盘。

挂载的意思是告诉 Linux:"把这个设备挂到树中的这个文件夹下。"一旦挂载成功,你通过进入那个文件夹来访问这块硬盘的内容。设备所挂入的那个文件夹就叫**挂载点(mount point)**。

```
/
└── mnt/
    └── usb/    ← 挂载之后,你的 USB 盘就出现在这里
```

挂载点的标准位置是 `/mnt`(用于临时的手动挂载)和 `/media`(在桌面环境下系统自动挂载 USB 之类设备时常使用)。

**挂载一块硬盘:**

```bash
ahegazy0@kali:~$ mount /dev/sdb1 /mnt/usb
```

这条命令把第二块硬盘的第一个分区挂载到 `/mnt/usb` 这个文件夹下。之后,进入 `/mnt/usb` 看到的就是那块硬盘上的文件。

挂载之前这个文件夹必须已经存在:

```bash
ahegazy0@kali:~$ mkdir /mnt/usb
ahegazy0@kali:~$ mount /dev/sdb1 /mnt/usb
```

**卸载(取消挂载):**

```bash
ahegazy0@kali:~$ umount /mnt/usb
```

注意拼写——是 `umount` 而不是 `unmount`。在物理拔出硬盘之前,一定要先卸载。如果系统在写入数据时你直接拔掉,可能会损坏文件系统。

---

## fdisk — 看看你有哪些硬盘

`fdisk -l` 会列出当前连接到系统的所有硬盘和分区。它需要 root 权限。

```bash
ahegazy0@kali:~$ fdisk -l
```

输出大致如下:

```
Disk /dev/sda: 500 GB, 500107862016 bytes
Device     Boot    Start      End  Sectors  Size  Type
/dev/sda1  *        2048  1026047  1024000  500M  EFI System
/dev/sda2        1026048 97654784 96628737 46.1G  Linux filesystem

Disk /dev/sdb: 16 GB, 16013942784 bytes
Device     Boot  Start     End  Sectors  Size  Type
/dev/sdb1  *      2048 31252479 31250432 14.9G  Microsoft basic data
```

从这里你可以看到:连接了两块硬盘,每个硬盘各有哪些分区,每个分区的大小,以及文件系统类型。这是处理一台未知机器时的第一步——你需要了解连接了哪些存储设备。

---

## df — 还有多少剩余空间

`df` 是 "disk free(可用磁盘)"的缩写。它会显示每个已挂载文件系统的已用空间和可用空间。

```bash
ahegazy0@kali:~$ df -h
```

`-h` 参数表示 human-readable(人类可读)——用 GB、MB 等单位显示,而非原始字节数。

输出:

```
Filesystem      Size  Used Avail Use%  Mounted on
/dev/sda2        46G   12G   32G  27%  /
/dev/sdb1        15G  8.1G  6.9G  54%  /mnt/usb
tmpfs           2.0G  1.2M  2.0G   1%  /run
```

`Use%` 那一列让你一眼就能看出哪些分区快满了。`tmpfs` 是位于内存中的虚拟文件系统——并不是真实的磁盘空间。

---

## /etc/fstab — 永久挂载配置(/etc/fstab - the persistent mount config)

使用 `mount` 命令挂载只是临时的——重启之后,这块硬盘就不会再保持挂载了。要让挂载永久生效——让它在开机时自动挂载——你就需要在 `/etc/fstab` 中添加一条记录。

```bash
ahegazy0@kali:~$ cat /etc/fstab
```

fstab 中的每一行描述一个要挂载的文件系统:

```
# device          mountpoint    fstype   options    dump  pass
/dev/sda1         /             ext4     defaults    0     1
/dev/sda2         /home         ext4     defaults    0     2
UUID=1234-ABCD    /mnt/data     ntfs     defaults    0     0
```

从侦察的角度来看,`/etc/fstab` 透露了目标系统的很多信息。它显示了开机时要挂载的每块硬盘——包括网络共享(NFS、SMB)、加密卷和外接磁盘。这里隐藏或反常的条目可能指向值得调查的地方。

---

## fsck — 检查并修复硬盘

`fsck` 的全称是 "filesystem check(文件系统检查)"。它会扫描硬盘上的错误,并尝试修复。

```bash
ahegazy0@kali:~$ fsck /dev/sdb1
```

**关键规则:绝对不要在已挂载的文件系统上运行 fsck。** 如果硬盘处于挂载且正在使用状态,fsck 可能让损坏变得比之前更严重,而不是更好。一定要先卸载:

```bash
ahegazy0@kali:~$ umount /dev/sdb1
ahegazy0@kali:~$ fsck /dev/sdb1
```

如果要检查你的根分区(`/`),系统运行中是没法把它卸载的——你需要从 live USB 启动,再从那里运行 fsck。

---

## 文件系统类型(Filesystem types)

在格式化一个分区时,你要选择一种文件系统类型。不同的操作系统使用不同的格式。当你挂载来自其他系统的硬盘时,了解这些格式很重要。

| 文件系统 | 出现的位置 |
|---|---|
| ext4 | Linux 标准文件系统 |
| ext3 / ext2 | 较老的 Linux 文件系统 |
| NTFS | Windows 硬盘 |
| FAT32 / exFAT | USB 盘、SD 卡(跨平台) |
| XFS | 高性能 Linux,常见于服务器 |

挂载非 Linux 的硬盘时,你可能需要指定类型:

```bash
ahegazy0@kali:~$ mount -t ntfs /dev/sdb1 /mnt/windows
```

Linux 可以很好地读取 NTFS 硬盘。写入也行,但历史上存在一些瑕疵——这一点值得知道。

---

## 挂载 USB 盘 — 完整流程

这是从插入到能够访问文件的完整过程:

```bash
# 1. 看看刚连接上的是什么设备
ahegazy0@kali:~$ fdisk -l

# 2. 如果挂载点还不存在,就先创建它
ahegazy0@kali:~$ mkdir /mnt/usb

# 3. 挂载这个分区
ahegazy0@kali:~$ mount /dev/sdb1 /mnt/usb

# 4. 检查是否成功
ahegazy0@kali:~$ ls /mnt/usb

# 5. 做你需要做的事

# 6. 完成后卸载
ahegazy0@kali:~$ umount /mnt/usb
```

---

## 命令参考(Command Reference)

| 命令 | 作用 |
|---|---|
| `fdisk -l` | 列出所有硬盘和分区 |
| `mount /dev/sdb1 /mnt/point` | 把一个分区挂载到某个文件夹 |
| `mount -t ntfs /dev/sdb1 /mnt/point` | 以特定文件系统类型挂载 |
| `umount /mnt/point` | 安全地卸载一块硬盘 |
| `df -h` | 显示磁盘空间使用情况(人类可读) |
| `fsck /dev/sdb1` | 检查并修复文件系统(先卸载) |
| `cat /etc/fstab` | 显示永久挂载配置 |
| `mkdir /mnt/point` | 创建一个挂载点目录 |
| `lsblk` | 以清晰的树状视图展示硬盘和分区 |

另外一个值得了解的命令:`lsblk` 比起 `fdisk -l` 提供了一种更清爽的视图,用来查看硬盘布局:

```bash
ahegazy0@kali:~$ lsblk
NAME   MAJ:MIN  SIZE  TYPE  MOUNTPOINT
sda    8:0      500G  disk
├─sda1 8:1      500M  part  /boot
└─sda2 8:2      499G  part  /
sdb    8:16      16G  disk
└─sdb1 8:17     16G  part  /mnt/usb
```

比原始的 fdisk 输出要直观得多。

---

## 练习(Practice)

- [ ] 运行 `fdisk -l`,识别你的虚拟机上连接了哪些硬盘和分区
- [ ] 运行 `df -h`,找出最接近装满的那个分区
- [ ] 用 `mkdir` 创建 `/mnt/test`,插上一个 USB 盘(或者在 VirtualBox 里添加一块虚拟硬盘),用 `fdisk -l` 找到它的设备名,然后把它挂载到 `/mnt/test`
- [ ] 挂载之后,运行 `ls /mnt/test` 确认它真的成功了,完成后用 `umount /mnt/test` 卸载
- [ ] 通读一遍 `/etc/fstab`,弄懂每一行的作用

> 💡 *想要更深入的练习,我还建议完成官方书籍 **Linux Basics for Hackers** 中本章末尾的习题。*
---

*下一篇:Module 11 - 日志与日志文件(Logging & Log Files)*
