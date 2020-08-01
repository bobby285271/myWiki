---
title: NixOS Installation and Configuration Guide
description: 
published: true
date: 2020-08-01T02:33:45.570Z
tags: guide, linux, nixos, installation, configuration
editor: markdown
---

本文基于 NixOS 20.03 Beta 和 [NixOS Manual 20.03](https://nixos.org/nixos/manual/) 撰写。本文的写作缘由是 NixOS Manual 的编排设计很容易导致习惯了其它发行版安装和配置流程的新人入坑失败。本篇指南给出的解决方案基本能满足**大一软工专业课程学习**和**日常使用**的需要。

**本文将假定你已经独立安装和配置过 Arch Linux、Gentoo 或任一官方不提供 GUI 安装程序的 Linux 发行版并掌握安装过程中涉及的大部分命令。**

了解 NixOS
--------

[可以先从 DistroWatch 开始了解 NixOS 这个发行版](https://distrowatch.com/table.php?distribution=nixos)，[接下来建议阅读 NixOS 官方的介绍](https://nixos.org/nixos/about.html)。有一个不太恰当的比喻，NixOS 相对于其它发行版有点像使用 Git + Markdown + Hexo 相对于无版本管理 + 手撸 HTML 文件，[NixOS Wiki 也给出了 NixOS 和其它发行版的最大区别所在](https://nixos.wiki/wiki/NixOS#Comparison_with_traditional_Linux_Distributions)。而这一切实现的基础是 **Nix**，事实上 NixOS 中所有软件包（**Nixpkgs**）和系统配置都由 Nix 管理。要想了解 Nix，[建议阅读 dram 大佬在知乎相关问题下的回答](https://www.zhihu.com/question/279855101/answer/475896416)。dram 是 TUNA 镜像站镜像 NixOS / Nix / Nixpkgs 的主要推动者之一。至于 NixOS 是否适合你，[可以阅读这篇文章自行判断](https://zhuanlan.zhihu.com/p/50623869)。

准备工作
----

准备好安装 NixOS 了？让我们开始吧。前期的准备工作包括：数据备份、下载镜像、将镜像写入 U 盘、引导安装介质、连接到网络、建立分区、格式化分区、挂载分区，**这一部分和其它（不带 GUI 安装程序的）Linux 发行版的安装几乎是一样的**，这里不会细讲。

2020 年 3 月后清华镜像站就有 Nix / NixOS / Nixpkgs 的镜像了，[NixOS 的安装介质可在这里下载](https://mirrors.tuna.tsinghua.edu.cn/nixos-images/)，建议选带桌面环境的镜像（可以通过文件名判断）。将写入 U 盘可以使用 `dd` 命令，具体用法（`path-to-image` 是 NixOS 安装介质的路径，`/dev/sdX` 则是写入目标）：

    sudo dd if=path-to-image of=/dev/sdX

引导安装介质后，按照 `/etc/issue` 的提示操作即可进入桌面环境（启动登录管理器），注意默认搭载的账户 `nixos` 和 `root` 都没有密码：

    sudo systemctl start display-manager

因为在线安装，联网是必须的。通常情况使用 NetworkManager 联网即可，[当然这里还有其它的联网方式](https://nixos.org/nixos/manual/index.html#sec-installation-booting-networking)。

可选的分区工具除了熟悉的 GDisk 和 GNU Parted 这些 CLI 的分区工具之外，还有 GUI 的分区工具 GParted。[NixOS Manual 对在命令行下分区做了详细的介绍](https://nixos.org/nixos/manual/index.html#sec-installation-partitioning)，[还配有不少的例子](https://nixos.org/nixos/manual/index.html#sec-installation-summary)，[可以配合 ArchWiki 的相关文章进行阅读](https://wiki.archlinux.org/index.php/Partitioning)。因为 Nix 并不会自动删除历史的软件包安装，**根目录尽可能分大一些**，否则随着时间推移 `/nix` 可能会占用较大空间。**除了 Home 和 EFI 分区之外不建议（也没必要）设置独立分区**。**另外 NixOS 的回滚功能是由 Nix 实现的，和 Btrfs 无关**，如果你选择 Btrfs 仅仅是因为它的快照特性，那么没有必要给根分区使用 Btrfs。

格式化分区之后就可以挂载分区了。分区应该挂载到 `/mnt` 下，例如根分区就挂载到 `/mnt`，Home 分区则 `/mnt/home`。**特别留意 NixOS 下 EFI 分区应该挂载到 `/mnt/boot`。**Swap 分区使用 `swapon` 启用即可。

### 初始化 NixOS 配置文件

接下来就是后期的准备工作了。事实上，如果你点开了前面的几个链接，你会发现 `/etc/nixos/configuration.nix` 是整个系统的灵魂所在。你的系统**全局搭载**的软件包和系统配置都取决于这个文件。[因此我们说这个发行版是「先配置后安装」的](https://linux.cn/article-8976-1.html)。

**所以接下来的主要任务，就是配置 `configuration.nix`，这个文件会一直伴随着你的系统。**一旦配置完成，你会使用到 `nixos-install` 脚本，这个脚本会读取这个文件，并调用 Nix 让其根据这个文件「造出对应的系统来」。而在安装好 NixOS 之后，你可以随时改动 `configuration.nix` 这个文件，然后使用 `nixos-rebuild` 脚本，这个脚本同样也是调用 Nix，让 Nix **处理所有的变化**，以「造出对应的系统来」。「造出来的系统」会**和当前系统的状态无关** 。这两个脚本笔者在后面会介绍其用法。

当然了，如果在 Live 环境里面直接编辑 `/etc/nixos/configuration.nix`，打开的是 Live 环境的配置文件。而现在我们需要为新的系统创建一个默认的 NixOS 的配置文件，在进入新系统之前，应该通过 `/mnt` 访问其文件**（本文后续的命令，除非特别注明，否则一律需要在 `root` 账户下执行，可以使用 `su` 命令从普通用户切换到 `root` 账户）**：

    nixos-generate-config --root /mnt

运行这个命令之后，`/mnt/etc/nixos/` 目录会被创建，里面会放置自动生成的两个文件：`configuration.nix` 和 `hardware-configuration.nix` 。我们待会再看 `hardware-configuration.nix`，先看看前面已经提到无数次的 `configuration.nix`，[这个文件在 NixOS 20.03 下默认长这样](https://paste.ubuntu.com/p/JHsyTczk3m/)。当然了，在你安装系统之后，可以使用 `nixos-generate-config --force` 重新生成一个 `configuration.nix`。

### 初识 **NixOS 模块**

让我们来打开 `configuration.nix` 这个文件：

    nano /mnt/etc/nixos/configuration.nix

以 `#` 开头的都是注释，我们先不要管注释，看看没有注释的内容。实际上，这是一个简化的 **NixOS 模块**（NixOS Module）。我们尝试分析一下它的结构：

    { config, pkgs, ... }:
    
    {
        imports = [
            ./hardware-configuration.nix
        ];
        boot.loader.systemd-boot.enable = true;
        boot.loader.efi.canTouchEfiVariables = true;
        networking.useDHCP = false;
        networking.interfaces.docker0.useDHCP = true;
        networking.interfaces.wlo1.useDHCP = true;
        system.stateVersion = "20.03";
    }

第一行的 `{ config, pkgs, ... }:` 表示这是一个至少接受两个参数 `config` 和 `pkgs` 的函数。`config` 主要用于在多个 NixOS 模块的情形下**引用**其它模块定义的配置值，[这里有相关的例子](https://nixos.org/nixos/manual/index.html#sec-modularity)，可以说是引入了完整的系统配置。`pkgs` 主要用于引用 Nixpkgs 中的软件包，例如 `pkgs.fcitx`。**当然我们现在简单地认为第一行就是要这样写的好了**，即使在本文中不会使用到 `config`。说实话在第一次接触 C++ 时对着 `using namespace std;` 自闭了半天，其实到后面需要到名称空间的时候自然就懂了。

`imports` 导入了其它的 NixOS 模块，这里它导入了 `./hardware-configuration.nix`。另外 `modules/module-list.nix` 里面定义了一系列的模块，**这些模块会被自动导入**。`modules/module-list.nix` 的绝对路径是啥呢，可以看回 [NixOS 官方的介绍](https://nixos.org/nixos/about.html)，阅读 `How does NixOS work?`，你会发现 `/nix/store/` 下的目录都带上了 Hash 值，这是为了避免新的安装覆盖旧的安装（便于需要的时候回滚啊！事实上这也是开始时我说根目录尽可能分大一些的原因，后面会提供旧安装清理的方法）。所以要想找到这个 `modules/module-list.nix`，不妨用点小技巧：

    find / | grep "modules/module-list.nix"

`cat` 查看相应的文件就可以看到这个列表了，事实上还真导入了不少的 NixOS 模块。当然我们现在先打开 `hardware-configuration.nix` 看看（要注意上面的 `./hardware-configuration.nix` 是相对于 `/mnt/etc/nixos/configuration.nix` 的路径）：

    cat /mnt/etc/nixos/hardware-configuration.nix

你会发现上面记录了你的分区信息，**如果不是那就是你分区过程中出了差错**，可以调整分区后按照上面的步骤重新 `nixos-generate-config`。这个文件将用于生成 Fstab 等文件。这个文件被单独分出来原因也是为了避免错误地编辑了这个文件。**事实上 NixOS 官方只建议使用 `nixos-generate-config` 更新这个文件。**

接下来我们来看 `configuration.nix` 剩余的部分。这些行的格式都是 `name = value`，我们称这是定义（**_define_**）操作。我们称 `configuration.nix` 为简化的 NixOS 模块，**原因是它只有定义操作**。事实上还有声明（**_declare_**）操作，这一个一个的 `name` 其实在其它文件声明过，这样才可以被定义。笔者接下来会详细介绍定义操作和声明操作，如果想要深入了解 NixOS 模块，[可以查看这里](https://nixos.org/nixos/manual/index.html#sec-writing-modules)。

### 定义操作

我们来继续来研究上面 `configuration.nix` 涉及到的 Nix 表达式：

    boot.loader.systemd-boot.enable = true;
    boot.loader.efi.canTouchEfiVariables = true;

我们已经知道它们格式都是 `name = value`，进一步分析它的结构，这两个表达式中我们把 `boot`、`loader`、`systemd-boot` 还有 `efi` 称之为集合（_**set**_），把 `enable` 和 `canTouchEfiVariables` 称为选项（_**option**_），把 `true` 称为值（**_value_**）。**一个集合里面可能有子集合，也有可能有选项，我们给选项定义值。**集合与子集合、集合与选项之间可以像上面一样，使用 `.` 连接。同时也可以使用下面的使用花括号嵌套的方式（**注意分号出现的位置**）：

    boot = {
        loader = {
            systemd-boot = {
                enable = true;
            };
            efi = {
                canTouchEfiVariables = true;
            };
        };
    };

当然了可以两种方式混合使用：

    boot = {
        loader.systemd-boot = {
            enable = true;
        };
        loader = {
            efi.canTouchEfiVariables = true;
        };
    };

如果是还有疑问的，[NixOS Manual 给出了更多的相关例子](https://nixos.org/nixos/manual/index.html#sec-configuration-syntax)。至于是使用花括号还是半角句号，我认为不重要。但在我看来，**如果有两个选项它们属于同一个集合，它们应该尽量在一起被定义**，而不是一个在文件开头定义，一个在文件末尾定义。**同一集合的两个选项会有一定的关联性，放在一起可以更好地维护。**

当然了，当配置文件足够复杂之后，还可以像 `hardware-configuration.nix` 一样将一部分拆分出来，在 `imports` 部分导入相关文件，这里就不细讲了。

至于 `value`，上面的给出的 `configuration.nix` 中涉及到的值有 `true`、`false`、`"20.03"`。和其它编程语言的数据类型类似，我们称 `true` 和 `false` 是**布尔值**，称 `”20.03"` 是**字符串**（注意两个引号），每一个 `name` 都有对应的可接受的类型。是事实上，对于其它的选项，可以接受的值的类型也有可能是列表、浮点数、软件包等等。[这里详细地介绍了 `value` 的各种类型](https://nixos.org/nixos/manual/index.html#sec-configuration-file)，并给出了相关的例子。

这里问题就来了，怎样知道在一个 NixOS 系统中我有哪些 `name` 可以用呢？这些 `name` 又接受那些 `value` 呢？让我们继续来了解「声明操作」。

### 声明操作

和其它编程语言类似，**声明就是要告诉人家我叫啥、我接受什么样的 `value`。**怎样查看一个 `name` 的声明呢？第一个方法是**使用命令 `nixos-option`**，例如我要查看 `system.stateVersion` 的声明：

    nixos-option system.stateVersion

这个是在我系统上的输出，当然了在 Live 环境下输出不一定相同：

    Value:
    "20.03"
    
    Default:
    "20.03"
    
    Type:
    "string"
    
    Description:
    ''
        Every once in a while, a new NixOS release may change
        configuration defaults in a way incompatible with stateful
        data...（后面的省略）
    ''
    
    Declared by:
    [ "/nix/var/nix/profiles/per-user/root/channels/nixos/nixos/modules/misc/version.nix" ]
    
    Defined by:
    [ "/etc/nixos/configuration.nix" ]
    

我们会看到 `system.stateVersion` 在 `/nix/var/nix/profiles/per-user/root/channels/nixos/nixos/modules/misc/version.nix` 声明过，而笔者刚开始的时候说过 `configuration.nix` 会自动导入 `modules/module-list.nix` 里包含的所有文件，里面就有 `version.nix`，因此我们才可以在 `/etc/nixos/configuration.nix` 定义它。另外我们也能得到其它有用的信息，例如它接受一个字符串作为它的值，`system.stateVersion` 默认定义的值是 `"20.03"`，事实上这些信息就是在声明它的时候提供的。

我们打开这个 `version.nix`，这就是一个声明的真面目了：

    options.system = {
        stateVersion = mkOption {
            type = types.str;
            default = cfg.release;
            description = ''
                Every once in a while, a new NixOS release may change
                configuration defaults in a way incompatible with stateful
                data...（后面的省略）
            '';
        };
    };

当然笔者在本文中并不会要求用户创建一个声明。但我们还是会继续探究这个文件，我们发现 `version.nix` 给多个选项提供了声明，例如 `system.nixos.release`、`system.nixos.codeName` 等等。

我们接着再往下翻一点：

    config = {
        system.nixos = {
            version = mkDefault (cfg.release + cfg.versionSuffix);
        };
    
        # Generate /etc/os-release.
        environment.etc.os-release.text = ''
            NAME=NixOS
            ID=nixos
            VERSION="${cfg.version} (${cfg.codeName})"
            VERSION_CODENAME=${toLower cfg.codeName}
            VERSION_ID="${cfg.version}"
            PRETTY_NAME="NixOS ${cfg.release} (${cfg.codeName})"
            LOGO="nix-snowflake"
            HOME_URL="https://nixos.org/"
            DOCUMENTATION_URL="https://nixos.org/nixos/manual/index.html"
            SUPPORT_URL="https://nixos.org/nixos/support.html"
            BUG_REPORT_URL="https://github.com/NixOS/nixpkgs/issues"
        '';
    };

你会在里面找到 `${cfg.version}` 和 `${cfg.codeName}`。`cfg` 是什么呢，我们回到文件开头：

    { config, lib, pkgs, ... }:
    
    with lib;
    
    let
        cfg = config.system.nixos;
    in
    {
        ...
    }

还记得 `config` 是啥吗？我们说它引入了完整的系统配置。现在，即使我没有告诉你 `environment.etc.os-release.text` 是个啥，相信你也能猜出来，Nix 使用了 `system.nixos.release`、`system.nixos.codeName` 创建了 `/etc/os-release` 这个文件。至于前面提到的 `system.stateVersion` 呢，则被用于其它的地方了（毕竟有了 `config`，我们在任何地方都可以引用它）。**事实上，就是这样声明和定义一个又一个的选项，组成了最终的系统。**

关于怎样查看一个 `name` 的声明，[第二个方法是**使用 NixOS Options 网页**](https://nixos.org/nixos/options.html#)。输入 `name` 同样可以查看到使用 `nixos-options` 能看到的内容。**关键是它还非常适合给我们模糊搜素用。**另外点击 `Declared in` 旁边的超链接，会跳转到 GitHub 的 NixOS/nixpkgs 仓库。例如说我们查找 `system.stateVersion`，[我们会被带到这里](https://github.com/NixOS/nixpkgs/blob/master/nixos/modules/misc/version.nix)。同样也可以查看 `version.nix` 文件，还可以切换 Git 分支查看不同 NixOS 版本下这个文件的历史状态。我们在后面会经常使用到 NixOS Options 网页和 `nixos-option` 命令。

开始编辑 `configuration.nix`
------------------------

讲了这么多，让我们尝试把刚刚学到的东西应用到实际当中吧！接下来我们要写很多很多的 `name = value` 表达式。我们已经知道，当我们有一个需求，我们可以尝试去 NixOS Options 进行模糊搜索，看看有没有对应的选项。此外，我们多次提到的 NixOS Manual 提供了很多的常见需求对应的解决方案。这里还会另外推荐一些寻找解决方案的地方：

*   [NixOS Manual](https://nixos.org/nixos/manual) \- NixOS 官方文档值得阅读值得信赖。
*   [NixOS Options](https://nixos.org/nixos/options.html) \- 列出了所有可用的选项、其声明、可以接受的值。
*   [NixOS Discourse Forum](https://discourse.nixos.org/) \- 用户论坛，有官方开发者答疑。
*   [NixOS Packages](https://nixos.org/nixos/packages.html) \- 列出了所有可用的 Nixpkgs 软件包。
*   [NixOS Wiki](https://nixos.wiki/) \- 社区维护的维基。
*   [GitHub](https://github.com/) \- 查看其它用户的配置，或查看官方的 NixOS/nixpkgs 仓库。
*   [Stack Overflow](https://stackoverflow.com/questions/tagged/nixos) \- 查看打有 `nixos` 标签的问题和回答。
*   各大搜索引擎 \- 这个也不用我多说吧……

当然了，如果实在找不到，可以去 [NixOS Discourse Forum](https://discourse.nixos.org/) 提问，因为官方开发者在这个论坛非常活跃，很多问题都能在短时间内得到答案。

要注意的是，因为 `configuration.nix` 会一直伴随着你的 NixOS 安装，所以这个章节既适用于安装时进行初始化配置，也适用于安装后的日常维护进行配置。下面结合两个自认为比较实际的需求来展开介绍，我会着重说明我是**如何找到**解决方案的，希望能给到大家一些帮助。

### 桌面环境

会发现 NixOS Manual 有多个对应的章节，[有讲 X11 的](https://nixos.org/nixos/manual/index.html#sec-x11)，[有专门讲 Pantheon 桌面环境的](https://nixos.org/nixos/manual/index.html#chap-pantheon)，[有专门讲 XFCE 桌面环境的](https://nixos.org/nixos/manual/index.html#sec-xfce)。个人非常喜欢 Pantheon，我们也知道 X11 是一切桌面环境的基础（假设我们把 Wayland 忽略掉）。我们来看看 X11。首先看到的是这样一句：

The X Window System (X11) provides the basis of NixOS’ graphical user interface. It can be enabled as follows:

    services.xserver.enable = true;

看上去是不是很熟悉？事实上默认的 `configuration.nix` 里面已经有了这样的一行，只是被注释掉了。上面还配有注释 `Enable the X11 windowing system.` 让我们取消注释 `services.xserver.enable = true;`。

搞定！就这么简单。先继续看看 Pantheon 桌面对应的章节。根据 NixOS Manual，要想使用 Pantheon 桌面，需要在 `configuration.nix` 加入：

    services.xserver.desktopManager.pantheon.enable = true;

这也很简单，复制粘贴过去就完成了，搞定！在 `configuration.nix` 加入这个 Nix 是怎样处理的呢？还记得前面提到的 `system.nixos.release`、`system.nixos.codeName`和 `/etc/os-release` 吗？可以按照当时我所给的思路一探究竟，我们在后面揭晓。

### 软件包

[会发现 NixOS Manual 有对应的章节](https://nixos.org/nixos/manual/index.html#sec-declarative-package-mgmt)。它的要求是定义 `environment.systemPackages` 这个选项。它给出的例子是这样的：

    environment.systemPackages = [ pkgs.thunderbird ];

感觉有点迷？那不妨前往 [NixOS Options](https://nixos.org/nixos/options.html#environment.systempackages) 查找 `environment.systemPackages`，这时就应该知道它要求你提供 `list of packages`。我们回到 [NixOS Manual](https://nixos.org/nixos/manual/index.html#sec-configuration-file)，查看 Package 小节。这时就可以大致推断出这个 `name` 怎么填了。

[接下来我们去 GitHub 参考一下别人的配置文件](https://github.com/search?l=Nix&q=environment.systemPackages&type=Code)，注意语言要选择 Nix，关键字将 Option 填入即可。查看搜索结果的时候只看定义操作相关的代码（而不是声明操作），这时你会发现另一个用法：

    environment.systemPackages = with pkgs; [
        vscode
        freemind
    ];

事实上这样写，在后面就不用每个软件包前面加上 `pkgs.` 了，`pkgs.vscode` 可以直接写成 `vscode`。

胜利在望！接下来去 [NixOS Packages](https://nixos.org/nixos/packages.html)，查找自己需要的软件包填入。你会发现有 Package name 和 Attribute name，填哪个呢？不妨回到 GitHub 继续翻配置文件，当你发现所有配置填的都是 Attribute name，我们就可以开始了，下面的就是我的配置：

    environment.systemPackages = with pkgs; 
    [
        dos2unix chromium fcitx fcitx-configtool firefox gcc git gptfdisk libreoffice-fresh neofetch pciutils python transmission vim wget vscode-with-extensions
    ];

你会发现这里面没有 X11 和 Pantheon 相关的包，**事实上， `services.xserver.desktopManager.pantheon.enable = true;` 这一行已经为你安装了相应的包。**[可以查看一下包含这个选项声明的文件](https://github.com/NixOS/nixpkgs/blob/master/nixos/modules/services/x11/desktop-managers/pantheon.nix)，我们定位到 `config =` 往下的内容。虽然有些选项我们没有讨论过，例如 `fonts.fontconfig.defaultFonts` 等等，但是你可以通过搜索 NixOS Options 或者直接按照它的字面意思知道这些选项是干什么的。有安装软件包的、设置系统服务的、设置字体的等等。

有了这么一行，我们也就没有必要在我们自己的 `configuration.nix` 中设置它了。所以说 NixOS Options 的优先级是会高于 NixOS Packages 的，笔者才会用很大的篇幅介绍前者。**当你需要一个软件或功能，首先考虑的应该是有没有对应的 NixOS Option，而不是直接安装一个 Nixpkg。**

* * *

下面是另外两个比较玄学的案例，都是我安装完成之后才开始折腾的问题，**当然了安装的时候也没有必要纠结这些问题**，因为安装完后还可以随时编辑 `configuration.nix`，后面还会再介绍。同样，我会着重说明我是**如何找到**解决方案的，希望能给到大家一些帮助。**介绍这些案例，不是为了劝退，而是为了说明，只要合理利用我在本章介绍的各类资源，善用搜索，必要时勇敢提问、开 Issue，甚至发 PR**（[这里有笔者本人的例子](https://discourse.nixos.org/t/language-packs-for-libreoffice-and-firefox/698/)）**，大部分的问题都能很快地得到解决。**

### 禁用 N 卡

单纯出于想要省电的原因，打算禁用 N 卡。网上给出的禁用 N 卡的方案是都是使用 BBSwitch，四处搜索之。最后在 [Discourse Forum](https://discourse.nixos.org/t/external-monitors-not-working-dell-xps/1799) 里面找到了一个有一定关联的。提到的选项包括 `extraModprobeConfig`、`blacklistedKernelModules`、`extraModulePackages`。

我们知道在其它发行版下要使用 BBSwitch 禁用 N 卡是这样操作的：

    options bbswitch load_state=0 unload_state=1

和帖子中 `extraModprobeConfig` 的值高度类似。而 `extraModulePackages` 在帖子里的值是 `config.boot.kernelPackages.nvidia_x11`，我们搜索到 BBSwitch 的 Attribute name 是 `linuxPackages.bbswitch`，相信也有一定关联。

接下来我们去 NixOS Options 继续搜索，发现上面几个选项都在 `boot` 集合下，查找 `boot module` 关键词，又发现了一个 `boot.kernelModules`，我认为这个选项和我的需求也是高度关联的。于是我写下了以下的配置：

    boot = {
        extraModprobeConfig = ''
            options bbswitch load_state=0 unload_state=1
        '';
        extraModulePackages = [ pkgs.linuxPackages.bbswitch ];
        kernelModules = [ "bbswitch" ];
        blacklistedKernelModules = [
            "nouveau"
            "rivafb"
            "nvidiafb"
            "rivatv"
            "nv"
            "uvcvideo"
        ];
    };

确实是可行的，因为我后面运行 `lspci` 的时候显示已经是 `rev ff` 了：

    01:00.0 3D controller: NVIDIA Corporation GP107M [GeForce GTX 1050 3 GB Max-Q] (rev ff)

### Visual Studio Code 插件

你可能会问这也要提及？情况是这样的，安装了 VS Code（`pkgs.vscode-with-extensions`）之后我安装了几个插件，发现 C++ 插件不工作。我对这个插件提供的补全功能有较大的依赖，第一时间我想到的是这是 NixOS 目录结构和其它发行版不一致的问题。于是去搜索 Discourse Forum，[找到了这个帖子](https://discourse.nixos.org/t/vscode-extensions-setup/1801)。下面有官方开发者给出了一个 Dirty Hack，尝试之。

结果发现 `fetchTarball https://github.com/nixos/nixpkgs-channels/archive/nixpkgs-unstable.tar.gz` 会消耗不少时间。一个解决方案是将整个 GitHub 仓库同步到其它 Git 托管平台，但我们还是希望有更高效的解决方案。留意到了楼主的解决方案，虽然它会覆盖原有的自己在 Marketplace 安装的 VS Code 插件，但是按照楼主的描述它确实是可行的：

    self: super:
    {
        mycode = super.vscode-with-extensions.override {
            # When the extension is already available in the default extensions set.
            vscodeExtensions = with super.vscode-extensions; [
                ms-vscode.cpptools
            ];
        };
    }

但是你并不知道第一行是啥，也不知道如何添加这个方案，[于是去 GitHub 搜索相关的内容](https://github.com/search?l=Nix&q=vscode-with-extensions.override&type=Code)。结果搜出来的结果比上面这个还要再复杂一点，而且这样写的不只一个：

    self: super: {
        vscode-with-extensions = super.vscode-with-extensions.override {
            vscodeExtensions = with super.vscode-extensions;
            [ 
                bbenoist.Nix
                ms-vscode.cpptools
            ]
            ++ super.vscode-utils.extensionsFromVscodeMarketplace [ {
                    name = "vim";
                    publisher = "vscodevim";
                    version = "1.0.8";
                    sha256 = "0yqfn8b2jfrijzf731sggyvik2immlx9hfgmsgp1mx01hpyisd9r";
                } {
                    name = "doxdocgen";
                    publisher = "cschlosser";
                    version = "0.4.1";
                    sha256 = "06f4nxjd5ph66bhlyjim87haams286sjhrw7vmiv2rckzinygh1h";
                } {
                    ...
                }
            ];
        };
    }

这时可以猜出来官方应该有相应的文档，于是就去搜索 NixOS Manual、NixOS Options 还有 NixOS Packages，发现都找不到。终于，[笔者开始查找 GitHub 的 NixOS/nixpkgs 仓库](https://github.com/NixOS/nixpkgs)。[找到了这样一个文件](https://github.com/NixOS/nixpkgs/blob/master/pkgs/applications/editors/vscode/with-extensions.nix)。你会发现第一行没了，后面的内容还是类似的。现在我们可以把关注点放在这个 `override` 上了。我们重新发起一次搜索，[在 NixOS Manual 找到了有关 `override` 的说明](https://nixos.org/nixos/manual/#sec-customising-packages)。

太妙了！但是接下来又有一个问题，这里的 `name`、`publisher`、`version` 和 `sha256` 又该怎么填呢？为什么 `ms-vscode.cpptools` 不需要填 `version` 和 `sha256` 呢？阅读还是刚才的文件，阅读上面的注释：

This expression should fetch:

*   the `nix` vscode extension from whatever source defined in the default nixpkgs extensions set `vscodeExtensions`.
*   the `code-runner` vscode extension from the marketplace using the following url: [(link)](https://bbenoist.gallery.vsassets.io/_apis/public/gallery/publisher/bbenoist/extension/nix/1.0.1/assetbyname/Microsoft.VisualStudio.Services.VSIXPackage).

The original `code` executable will be wrapped so that it uses the set of pre-installed / unpacked extensions as its `--extensions-dir`.

然而我们却找不到 `vscodeExtensions`，但是我们可以搜索 `ms-vscode.cpptools` 啊！[终于笔者找到了这个文件](https://github.com/NixOS/nixpkgs/blob/master/pkgs/misc/vscode-extensions/default.nix)。事实上在这个文件中涉及到的所有插件都无需我们再配置 `publisher`、`version` 和 `sha256` 了。**这和我们使用了 `services.xserver.desktopManager.pantheon.enable = true;` 就不需要在 `environment.systemPackages` 添加相关的包是一个道理。**

至于这个文件里没有的插件呢？例如说 `ms-vscode.cpptools` 的 `name` 就是 `cpptools`，`publisher` 就是 `ms-vscode`。我们可以去 [Marketplace](https://marketplace.visualstudio.com/vscode) 下载这个插件（找到 Download Extension 按钮即可），用 `sha256sum` 命令计算下载下来的文件的 `sha256`，而版本号在文件名有。好了，你已经准备好了！让我们来继续完善 `environment.systemPackages`：

    environment.systemPackages = with pkgs; 
    [
        dos2unix chromium fcitx fcitx-configtool firefox gcc git gptfdisk libreoffice-fresh neofetch pciutils python transmission vim wget
        ( vscode-with-extensions.override {
            vscodeExtensions = with vscode-extensions; [
                bbenoist.Nix
                ms-vscode.cpptools
                ms-azuretools.vscode-docker
            ]
            ++ vscode-utils.extensionsFromVscodeMarketplace [ {
                    name = "bracket-pair-colorizer-2";
                    publisher = "CoenraadS";
                    version = "0.0.29";
                    sha256 = "cadb50a21944e6e0293e3872d2fe23b5d2fd2b603ed2bf4a0675fd29bcfb130c";
                } {
                    name = "python";
                    publisher = "ms-python";
                    version = "2020.3.71113";
                    sha256 = "1d8a98a1eed7588dd3b57e4bbe518fcc88f883e066f1b3342453e9bc1a283fdb";
                } {
                    name = "php-intellisense";
                    publisher = "felixfbecker";
                    version = "2.3.14";
                    sha256 = "3798a5de1172b5803877357d0057e3e129d8d82b8fbe0b53ae28c777a0075ca6";
                } {
                    name = "vscode-language-pack-zh-hans";
                    publisher = "MS-CEINTL";
                    version = "1.44.2";
                    sha256 = "4f6ee18ada0e71dd1545ef49f8810f52fe872d2074612faa908d8bf2687400a0";
                }
            ];
        } )
    ];

* * *

相信对于其它的需求大家也能举一反三。[这里是我的 `configuration.nix`](https://github.com/bobby285271/nixos-configuration/blob/master/configuration.nix)，供大家参考。

执行安装
----

前面已经提及过 `nixos-install` 这个命令了，让我们使用它执行安装，不需要带上任何参数：

    nixos-install

如果 `configuration.nix` 存在任何错误，安装会在一开始的时候就中断，修改好后重新运行上面的命令即可。**网络出现问题同理。**接下来安装程序会提示你设置一个密码，这个密码是给管理员用的（**密码当然是不能在 `configuration.nix` 上设置的**），盲打两次即可。重启并进入新系统：

    reboot

当然，重启后别忘了为你的普通用户设置密码（如 `passwd bobby285271`）。

系统维护
----

笔者之前讲过 `configuration.nix` 是可以随时修改的。当然了现在你应该处于你的新系统中了，所以你不应该编辑 `/mnt/etc/nixos/configuration.nix` 了，而是应该编辑 `/etc/nixos/configuration.nix`。

    nano /etc/nixos/configuration.nix

要使这个配置生效，应该使用 `nixos-rebuild switch` 命令，如果你希望在构建这个配置的时候顺便升级自己的系统（当然如果你不改动 `configuration.nix` 单纯想升级系统，同样可以执行这个命令） ：

    nixos-rebuild switch --upgrade

关于这个，[可以查看 NixOS Manual 了解详情](https://nixos.org/nixos/manual/index.html#sec-changing-config)。