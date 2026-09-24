# Linux Basics for Hackers
## Module 9 - 归档与压缩(Archiving & Compression)

> 本文档为 Module_09_Archiving_Compression.md 的简体中文翻译版本

---

## 概述(Overview)

有时你需要把一堆文件打包成一个。有时要先把一个大文件压缩变小再发送。还有些时候——比如在做取证或正经的侦察时——你需要一块磁盘的逐比特精确副本。本模块会覆盖这三种场景:使用 `tar` 进行归档,使用 `gzip` 和 `bzip2` 进行压缩,以及使用 `dd` 进行底层的磁盘镜像制作。

---

## 归档 vs 压缩 — 不是一回事

这两个词经常被混用,但它们的含义不同。

**归档(Archiving)** 的意思是把多个文件合并成单独一个文件。它并不会让文件变小——它只是把它们打包到一起。Linux 中负责归档的工具是 `tar`。

**压缩(Compression)** 的意思是把文件编码得更紧凑,从而让文件变小。负责压缩的工具是 `gzip` 和 `bzip2`,它们都作用于单个文件。

实际中你通常会一起做这两件事——先归档,再把结果压缩。这就是 `.tar.gz` 这种文件的来源。其中 `.tar` 表示它被归档过,`.gz` 表示随后用 gzip 进行了压缩。

---

## tar — 把文件打包到一起

`tar` 的全称是 "Tape Archive"(磁带归档)——这个名字来自当年用磁带做备份的时代。名字虽然古老,但这个工具如今仍无处不在。

**创建一个归档:**

```bash
ahegazy0@kali:~$ tar -cvf archive.tar file1.txt file2.txt file3.txt
```

参数解释:
- `-c` — 创建一个新的归档
- `-v` — 详细模式,显示正在加入的文件(可选但很有用)
- `-f` — 后面紧跟归档文件的文件名

生成的结果是 `archive.tar`——一个包含这三个文件的单一文件。

**解包一个归档:**

```bash
ahegazy0@kali:~$ tar -xvf archive.tar
```

- `-x` — extract,解包
- `-v` — 详细模式
- `-f` — 从哪个文件解包

这条命令会把文件还原到你当前的目录里。

**不解包,只查看归档内容:**

```bash
ahegazy0@kali:~$ tar -tvf archive.tar
```

- `-t` — list,列出内容

当你收到一个 tar 文件、想先看看里面是什么再决定是否解包时,这个命令就很有用。

---

## gzip 和 gunzip — 让文件变小

`gzip` 会就地压缩单个文件——原文件消失,被替换为一个 `.gz` 版本。

```bash
ahegazy0@kali:~$ gzip archive.tar
```

结果:`archive.tar` 消失了,被 `archive.tar.gz` 取代。

解压缩:

```bash
ahegazy0@kali:~$ gunzip archive.tar.gz
```

或者等价的写法:

```bash
ahegazy0@kali:~$ gzip -d archive.tar.gz
```

两条命令作用相同——还原原文件。

---

## 一步搞定 — tar.gz

你不必分别运行 `tar` 和 `gzip`,而是可以通过加上 `-z` 参数一条命令搞定:

**创建一个压缩归档:**

```bash
ahegazy0@kali:~$ tar -cvzf archive.tar.gz file1.txt file2.txt file3.txt
```

在参数里加上的 `z` 表示"顺便用 gzip 压缩"。

**解包一个压缩归档:**

```bash
ahegazy0@kali:~$ tar -xvzf archive.tar.gz
```

思路一致——`x` 表示解包,`z` 表示处理 gzip 这一层。

这是你最常遇到的格式。当有人说 "tar-ball(打包文件)"时,他们通常指的就是 `.tar.gz` 文件。

---

## bzip2 — gzip 的替代选择

`bzip2` 是另一种压缩工具。它通常能产生比 gzip 更小的文件,但压缩和解压都要更慢。在大多数场景下两者的差异并不显著,但了解这两个工具的存在是有必要的。

| | gzip | bzip2 |
|---|---|---|
| 速度 | 更快 | 更慢 |
| 压缩率 | 良好 | 更好 |
| 文件扩展名 | `.gz` | `.bz2` |
| tar 参数 | `-z` | `-j` |

配合 tar 使用:

```bash
ahegazy0@kali:~$ tar -cvjf archive.tar.bz2 files/
```

解包:

```bash
ahegazy0@kali:~$ tar -xvjf archive.tar.bz2
```

唯一的区别是把 `-z` 换成了 `-j`。

---

## dd — 底层磁盘复制

`dd` 和这里的其他工具完全不属于同一类。`tar` 和 `gzip` 处理文件和文件夹,而 `dd` 在裸磁盘级别上工作。它以块、字节为单位进行复制,完全不在意文件系统结构。

```bash
ahegazy0@kali:~$ dd if=/dev/sda of=/dev/sdb
```

- `if` — input file,输入文件(源)
- `of` — output file,输出文件(目标)

这条命令会把 `/dev/sda`(你的第一块硬盘)上的每一个比特都复制到 `/dev/sdb`(另一块硬盘)上。复制结果是完美一致的——包括删除过的文件、文件系统元数据、所有的内容。目标盘会成为源盘的精确克隆。

**制作磁盘镜像文件:**

```bash
ahegazy0@kali:~$ dd if=/dev/sda of=disk_image.img
```

这条命令是把整块磁盘保存为一个文件,而不是复制到另一块盘。取证调查员之所以这样做,是为了在不接触原始证据的前提下对镜像展开工作。

**为什么它叫做"数据终结者(Data Destroyer)":**

如果你不小心把 `if` 和 `of` 搞反了——或者瞄错了目标盘——你会用零或垃圾数据覆盖真实的磁盘。它不会要求确认,也没有撤销的办法。

```bash
ahegazy0@kali:~$ dd if=/dev/sdb of=/dev/sda    ← 用 sdb 上的内容覆盖你的主盘
```

运行 dd 之前一定要反复确认 `if` 和 `of` 的值。如果其中之一是真实的磁盘,那就要检查三遍。

**添加进度显示:**

默认情况下 `dd` 会安静地运行,你完全不知道它进行到了哪里。加上 `status=progress` 可以看到进度输出:

```bash
ahegazy0@kali:~$ dd if=/dev/sda of=disk_image.img status=progress
```

---

## 取证视角(The forensics angle)

`dd` 是数字取证中的标配工具。当调查人员扣押一台电脑时,他们不会直接对原始硬盘动手——而是先用 `dd` 做一个镜像,然后再分析这个镜像。这样原始证据就始终没有被触碰或改动过。

由于 `dd` 在块级别进行复制,它会捕获一切:文件、已删除的文件、文件碎片、文件系统结构、磁盘剩余空间(slack space)。像 Autopsy 和 Sleuth Kit 这样的工具就可以对镜像展开分析,并恢复出普通文件管理器看不到的数据。

---

## 命令参考(Command Reference)

| 命令 | 作用 |
|---|---|
| `tar -cvf archive.tar files` | 创建一个 tar 归档 |
| `tar -xvf archive.tar` | 解包一个 tar 归档 |
| `tar -tvf archive.tar` | 列出 tar 归档的内容 |
| `tar -cvzf archive.tar.gz files` | 创建一个 gzip 压缩的归档 |
| `tar -xvzf archive.tar.gz` | 解包一个 gzip 压缩的归档 |
| `tar -cvjf archive.tar.bz2 files` | 创建一个 bzip2 压缩的归档 |
| `tar -xvjf archive.tar.bz2` | 解包一个 bzip2 压缩的归档 |
| `gzip filename` | 用 gzip 压缩文件 |
| `gunzip filename.gz` | 解压一个 .gz 文件 |
| `dd if=source of=destination` | 将原始块从源复制到目标 |
| `dd if=/dev/sda of=image.img status=progress` | 创建带进度输出的磁盘镜像 |

---

## tar 参数速查(tar flags at a glance)

| 参数 | 含义 |
|---|---|
| `-c` | 创建新归档(Create) |
| `-x` | 解包归档(Extract) |
| `-t` | 列出归档内容 |
| `-v` | 详细输出(Verbose) |
| `-f` | 指定文件名 |
| `-z` | 使用 gzip 压缩 |
| `-j` | 使用 bzip2 压缩 |

---

## 练习(Practice)

- [ ] 用 `touch file1.txt file2.txt file3.txt` 创建三个空文本文件
- [ ] 把它们打包成一个 tar 归档:`tar -cvf bundle.tar file1.txt file2.txt file3.txt`
- [ ] 压缩它:`gzip bundle.tar` —— 然后运行 `ls -lh` 看看大小的差异
- [ ] 用 `tar -xvzf bundle.tar.gz` 把它解包回来,并确认这些文件都在
- [ ] 用 `bzip2` 做同样的事情,并比较 `.tar.gz` 和 `.tar.bz2` 的最终文件大小

> 💡 *想要更深入的练习,我还建议完成官方书籍 **Linux Basics for Hackers** 中本章末尾的习题。*
---

*下一篇:Module 10 - 文件系统与存储设备(Filesystem & Storage Devices)*
