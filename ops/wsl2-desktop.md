---
title: How to Launch a Desktop Environment on WSL 2
description: 
published: true
date: 2020-08-01T02:57:06.680Z
tags: linux, wsl, ubuntu, windows
editor: markdown
---

WSL 2 是 WSL 中体系结构的新版本，它改变了 Linux 发行版与 Windows 交互的方式。
WSL 2 的主要目标是提高文件系统性能并增加系统调用的完全兼容性。 每个 Linux 发行版都可以作为 WSL 1 或 WSL 2 发行版运行，并可随时进行切换。
WSL 2 是底层体系结构的主要功能，它使用虚拟化技术和 Linux 内核来实现其新功能。

## 准备
为了使用 WSL 2，你需要 **Windows 10 内部版本 18917 或更高版本**。要想了解目前使用的版本是什么，可以到设置页面查看。
![](https://bobby285271.coding.net/p/img/d/img/git/raw/master/wsl2/0.PNG)

首先，右击任务栏 Windows 图标，选择 Windows Powershell（管理员），输入以下命令：
```
Enable-WindowsOptionalFeature -Online -FeatureName VirtualMachinePlatform
Enable-WindowsOptionalFeature -Online -FeatureName Microsoft-Windows-Subsystem-Linux
```

或者前往控制面板 - 程序 - 启用或关闭 Windows 功能，然后勾选**虚拟机平台**和**执行 Linux 程序的 Windows 子系统**。
![](https://bobby285271.coding.net/p/img/d/img/git/raw/master/wsl2/捕获.PNG)

完成设置后需要重启计算机！

## 安装
前往 Microsoft Store 下载并安装 Ubuntu。
![](https://bobby285271.coding.net/p/img/d/img/git/raw/master/wsl2/2.PNG)

接下来使 WSL 2 成为你的默认体系结构。打开 Windows Powershell 或命令提示符：
```
wsl --set-default-version 2
```

接下来在开始菜单找到 Ubuntu，点击后稍作等候。当一切准备好后，Ubuntu 会提示你新建一个账户和密码，注意输入密码时是不会有回显的。
![](https://bobby285271.coding.net/p/img/d/img/git/raw/master/wsl2/5.PNG)

此时你可以验证刚刚安装好的 Ubuntu 所使用的 WSL 版本：
```
wsl --list --verbose
```

![](https://bobby285271.coding.net/p/img/d/img/git/raw/master/wsl2/6.PNG)

如果你在 Windows Powershell 或命令提示符下运行 `wsl`，同样也是可以进入 WSL 环境的。

## 软件
切换到 Root：
```
$ sudo -i
```

编辑 APT 软件源（`nano` 大法好，不接受反驳） ：
```
# nano /etc/apt/sources.list
```

```
deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ bionic main restricted universe multiverse
deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ bionic-updates main restricted universe multiverse
deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ bionic-security main restricted universe multiverse
deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ bionic-backports main restricted universe multiverse
```

更新可用软件包列表，随后升级系统：
```
# apt update && apt dist-upgrade -y
```

安装基本开发环境、Xubuntu 桌面和远程桌面支持：
```
# apt install build-essential xubuntu-desktop xrdp
```

![](https://bobby285271.coding.net/p/img/d/img/git/raw/master/wsl2/11.PNG)

安装的软件包比较多，可能要等上一段时间。这里的 Xubuntu 桌面实际为 XFCE 桌面环境，换成别的问题不大但还是建议选择轻量级桌面。

保存 Xsession 配置：
```
# exit #切换回普通用户。
$ echo xfce4-session > ~/.xsession
```

## 远程桌面
> 需要使用远程桌面时均要执行本节内容。

启用 Xrdp 服务（这里不能使用 `systemctl`，因为 SystemD 不是默认的 Init 系统），记录端口信息：
```
$ sudo service xrdp start
```

在 Windows 开始菜单栏中找到 Windows 附件 - 远程桌面连接，输入 `localhost:端口`，如 `localhost:3389`。

应该会出现如下警告，选择是即可：
![](https://bobby285271.coding.net/p/img/d/img/git/raw/master/wsl2/16.PNG)

如果你没有看到这个警告，可能需要查找虚拟机的 IP 地址并替代 `localhost` 再试一次：
```
$ ip address
```

![](https://bobby285271.coding.net/p/img/d/img/git/raw/master/wsl2/13.PNG)

在这里，IP 地址查找到是 `172.18.221.157`，端口是 `3389`。注意每次重启 WSL 实例后这个 IP 地址都会换。

输入 `IP 地址:端口`，如 `172.18.221.157:3389`。

接下来输入自己的账户密码：
![](https://bobby285271.coding.net/p/img/d/img/git/raw/master/wsl2/17.PNG)

然后就是很熟悉的界面了：
![](https://bobby285271.coding.net/p/img/d/img/git/raw/master/wsl2/18.PNG)

## 本地化
直接在终端下就可以完成设置了（以下命令在 Root 下运行）：
```
# dpkg-reconfigure locales
```

确认 `en_US.UTF-8 UTF-8` 和 `zh_CN.UTF-8 UTF-8` 被勾选，然后将 `zh_CN.UTF-8 UTF-8` 设定为默认值。
![](https://bobby285271.coding.net/p/img/d/img/git/raw/master/wsl2/19.PNG)

然后安装语言包和输入法（按理来说装完就能用）：
```
# apt install language-pack-zh-hans language-pack-gnome-zh-hans fcitx fcitx-table
```

## 彩蛋
Windows 应用能访问 Linux 根文件系统啦，包括文件资源管理器！尝试在 WSL 中使用普通用户运行 `explorer.exe .`（在远程桌面会话中这样做是没有用滴！），看看会发生什么。

另外内部错误大法又怎能缺席呢（逃）。
![](https://bobby285271.coding.net/p/img/d/img/git/raw/master/wsl2/20.PNG)