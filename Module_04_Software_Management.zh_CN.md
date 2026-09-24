# Linux 黑客基础
## 第 4 模块 - 软件管理

> 本文档为 Module_04_Software_Management.md 的简体中文翻译版本

---

## 概述

Kali 自带了许多工具,但你经常会需要添加新工具、更新现有工具,或者从 GitHub 上直接拉取开发者的项目。本模块将介绍 Linux 如何处理软件安装——其工作方式与 Windows 截然不同——以及如何获取标准库中不存在的工具。

---

## Linux 如何处理软件

在 Windows 上,你从网站下载安装程序,双击它,然后按向导一步步点击完成。Linux 不是这样工作的。

Linux 使用**软件仓库(Repositories)**——由发行版维护和验证的大型在线软件库。你无需到处寻找下载链接,只需告诉系统你想要什么名称的软件,它就会自动查找、下载并安装,包括该软件运行所需的所有其他依赖。

在 Kali(以及基于 Debian 的系统)上,管理这一切的工具是 `apt-get`。

系统已知的仓库列表保存在 `/etc/apt/sources.list` 文件中,称为 `sources.list`。除非你知道自己在添加什么,否则一般不要修改此文件——非官方仓库可能包含恶意软件包。

---

## 基础命令

### apt-get update

在安装任何东西之前,先运行此命令。它并不安装或升级任何东西——只是刷新本地缓存中仓库可用的软件列表。如果跳过这一步直接安装,你可能会得到旧版本或报错。

```bash
ahegazy0@kali:~$ apt-get update
```

养成在任何安装之前都先运行此命令的习惯。

---

### apt-get install

下载并安装一个软件包。Kali 会自动处理依赖关系——如果该工具需要另外五个库才能运行,它也会一并获取这些库。

```bash
ahegazy0@kali:~$ apt-get install wireshark
```

下载前它会要求确认。输入 `y` 并按 Enter 键。

要跳过确认提示:

```bash
ahegazy0@kali:~$ apt-get install -y wireshark
```

`-y` 标志会自动回答 yes。在你清楚自己在做什么的时候,这非常有用。

---

### apt-get remove

移除已安装的软件包。

```bash
ahegazy0@kali:~$ apt-get remove wireshark
```

这会移除程序,但会保留其配置文件。如果你想连同配置文件一并删除:

```bash
ahegazy0@kali:~$ apt-get purge wireshark
```

---

### apt-get upgrade

将所有已安装的软件包更新到最新版本。

```bash
ahegazy0@kali:~$ apt-get upgrade
```

建议定期运行以保持工具为最新版本。升级前始终先运行 `apt-get update` 来刷新软件包列表。

---

### apt-cache search

在本地仓库数据库中搜索与关键词匹配的软件包。当你大致知道要找什么,但不确定确切名称时,这非常有用。

```bash
ahegazy0@kali:~$ apt-cache search wifi
```

返回名称或描述中包含 "wifi" 的软件包列表。然后从中挑选看起来合适的,按名称安装即可。

```bash
ahegazy0@kali:~$ apt-cache search wireless
ahegazy0@kali:~$ apt-cache search password crack
```

---

### git clone

许多最好的、最新颖的黑客工具都不在任何仓库中——它们托管在 GitHub 上。`git clone` 可以从 GitHub 直接将整个项目的副本下载到你的机器上。

```bash
ahegazy0@kali:~$ git clone https://github.com/username/toolname
```

这会在当前目录下创建一个以工具名为名的文件夹。大多数工具都附带一个 `README` 文件,其中说明了如何从这里安装和运行它们。

克隆后的通用流程:

```bash
ahegazy0@kali:~$ git clone https://github.com/example/tool
ahegazy0@kali:~$ cd tool
```

然后阅读 README 中的安装步骤——通常是类似 `pip install -r requirements.txt` 的命令,或者直接运行 Python 脚本。

> 许多最新的工具只存在于 GitHub 上。尽早学会使用 `git clone` 意味着你不会被局限于官方仓库中的工具。

---

## 关于 sources.list 的说明

文件 `/etc/apt/sources.list` 包含了系统从中拉取软件的所有仓库 URL。你可以用以下命令打开它:

```bash
ahegazy0@kali:~$ cat /etc/apt/sources.list
```

有时你会在网上看到指示,让你向此文件中添加一行来访问第三方仓库。对此要小心。Kali 官方仓库经过维护和验证;随机的第三方仓库则没有。添加错误的源很容易导致安装非预期的软件。

---

## 命令参考

| 操作 | 命令 |
|---|---|
| 刷新软件包列表 | `apt-get update` |
| 安装软件包 | `apt-get install [name]` |
| 安装时跳过提示 | `apt-get install -y [name]` |
| 移除软件包 | `apt-get remove [name]` |
| 移除包括配置文件 | `apt-get purge [name]` |
| 更新所有软件包 | `apt-get upgrade` |
| 搜索软件包 | `apt-cache search [keyword]` |
| 从 GitHub 克隆 | `git clone [url]` |

---

## 练习

- [ ] 运行 `apt-get update` 刷新你的软件包列表
- [ ] 使用 `apt-cache search wifi` 并查看返回的结果
- [ ] 从结果中挑选一个工具,使用 `apt-get install` 安装它
- [ ] 在 GitHub 上找一个简单的工具,用 `git clone` 将其下载到你的机器上

> 💡 *想要更深入的练习,我还建议完成官方 **Linux Basics for Hackers** 一书中各章节末尾的练习题。*
---

*下一篇:第 5 模块 - 控制文件和目录权限*