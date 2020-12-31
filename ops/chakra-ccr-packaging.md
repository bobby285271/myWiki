---
title: Chakra CCR Packaging Guide
description: 
published: true
date: 2020-08-01T02:58:26.831Z
tags: linux, chakra, ccr, pacman, chaser, packaging
editor: markdown
---

Chakra 的成功证明了只围绕 KDE 和 Qt 搭建一个简约的系统是完全可行的。但正是因为「简约」这一特性，一些用户需要的软件可能不被包含在我们的官方软件仓库中。[Chakra 社区软件仓库](https://ccr.chakralinux.org)（CCR）也就因此诞生了，它由社区维护，为所有类型的软件都敞开了大门。

## 搜索、安装、升级 CCR 软件包

> 老用户请注意：`ccr` 工具已经被弃用，请转而使用 `chaser`。

Chakra 提供了一个叫 `chaser` 的工具供用户管理 CCR 软件包。它的一些常见用法如下：
```
[chakra@localhost ~]$ chaser -h
usage: chaser [-h] [-v] [-b BUILD_DIR]
              {get,install,listupdates,update,search,info} ...

Next-generation community package management for Chakra.

positional arguments:
  {get,install,listupdates,update,search,info}
    get                 download source files here
    install             install a package from the CCR
    listupdates         list available updates
    update              search for and install updates for CCR packages
    search              search CCR packages
    info                display package information

optional arguments:
  -h, --help            show this help message and exit
  -v, --version         show version information and exit
  -b BUILD_DIR, --build-dir BUILD_DIR
                        build packages in BUILD_DIR. default = /tmp/chaser
```

### 搜索
搜索 CCR 软件包有两种方式，一种是直接访问 [CCR](https://ccr.chakralinux.org/) 站点并发起搜索，另一种是使用 `chaser search` 命令，比如搜索 `neofetch`：
```
[chakra@localhost ~]$ chaser search neofetch
```

`chaser` 既会扫描官方软件仓库，也会扫描 CCR。两处符合条件的软件包都会被输出。

* 如果您没有在其中找到您所需要的包，请考虑阅读下一章节。
* 如果您找到的包位于 `[core]`、`[desktop]`、`[gtk]`、`[lib32]`、`[testing]` 之一，这意味着您所需的包已经存在于官方软件仓库，请使用 [Pacman](https://community.chakralinux.org/t/chakra-package-manager-simplified-chinese/7341) 进行后续操作。
* 如果您找到的包位于 `[ccr]`，请继续阅读。

下面可以使用 `chaser info` 进一步了解软件包信息。
```
[chakra@localhost ~]$ chaser info neofetch
```

下面是一个示例输出：
```
Name           : neofetch
Version        : 5.0.0-1
URL            : https://github.com/dylanaraps/neofetch
Licenses       : MIT
Category       : utils
Votes          : 2
Maintainer     : bobby285271
OutOfDate      : False
Description    : A CLI system information tool written in BASH that supports displaying images.
```

由于 CCR 活跃用户不多，请务必通过 `Version` 和 `OutOfDate` 了解清楚软件包维护情况，如果这个软件包过旧，考虑帮忙维护它（请阅读下一章节）。

### 安装
如果您决定安装，使用 `chaser install` 命令。**不要**使用 `root` 运行以下命令，当 `chaser` 需要管理员权限的时候会要求您输入密码。
```
[live@localhost ~]$ chaser install neofetch
```

大致的安装流程如下（以安装 `neofetch` 为例，部分输出被省略）。

* 确认构建软件包：请在此处回答 `y`（是）。

```
resolving dependencies...
Downloading neofetch

Targets (1) neofetch

:: Proceed with installation? [Y/n]
```

* 定制软件包：PKGBUILD 是 Arch Linux 的一大特色，也被 Chakra 很好地继承了下来。感兴趣的请阅读 [PKGBUILD 语法](https://www.archlinux.org/pacman/PKGBUILD.5.html)，一般用户直接回答 `n`（否）。

```
Downloading neofetch
Edit neofetch PKGBUILD with $EDITOR? [Y/n]
```

* 构建软件包：此过程全自动，无需用户干预。

```
==> Making package: neofetch 5.0.0-1
==> Checking runtime dependencies...
==> Checking buildtime dependencies...
==> Retrieving sources...
==> Validating source files with sha256sums...
==> Extracting sources...
==> Entering fakeroot environment...
==> Starting package()...
==> Tidying install...
==> Checking for packaging issue...
==> WARNING: backup entry file not in package : etc/neofetch/config.conf
==> Creating package "neofetch"...
==> Leaving fakeroot environment.
==> Finished making: neofetch 5.0.0-1
==> Installing package neofetch with pacman -U...
```

* 确认安装软件包：后续的工作将自动交由 Pacman 完成。请在此处回答 `y`（是）。

```
loading packages...
resolving dependencies...
looking for conflicting packages...

Packages (1) neofetch-5.0.0-1

Total Installed Size:  0.26 MiB

:: Proceed with installation? [Y/n] 
```

* 安装软件包：此过程全自动，无需用户干预。

```
(1/1) checking keys in keyring                                                                                                                                     [#####################################################################################################] 100%
(1/1) checking package integrity                                                                                                                                   [#####################################################################################################] 100%
(1/1) loading package files                                                                                                                                        [#####################################################################################################] 100%
(1/1) checking for file conflicts                                                                                                                                  [#####################################################################################################] 100%
(1/1) checking available disk space                                                                                                                                [#####################################################################################################] 100%
:: Processing package changes...
(1/1) installing neofetch                                                                                                                                          [#####################################################################################################] 100%
:: Running post-transaction hooks...
(1/1) Arming ConditionNeedsUpdate...
```

### 升级
使用 `chaser listupdates` 查看可用的更新，使用 `chaser update` 检查并应用更新 CCR 软件包。

### 移除
要查看您安装的 CCR 软件包，使用 `pacman -Qm` 命令。

> 这个命令可能会输出一些您觉得很陌生的软件包，这很有可能是从官方软件仓库仓库移除且不再受 Chakra 团队支持的包。

使用 Pacman 移除您不再需要的包，详见 [这篇文章](https://community.chakralinux.org/t/chakra-package-manager-simplified-chinese/7341)。

## 提交、维护 CCR 软件包（初阶）
感谢您为 Chakra 做出贡献！接下来，您将了解到如何准备 PKGBUILD，制作源码包并上传到 CCR 上。首先，请先前往 CCR 的 [帐户页面](https://ccr.chakralinux.org/account.php) 注册一个帐号。

### 准备 PKGBUILD：概述
PKGBUILD 是整个 CCR 包的灵魂所在。要知道的是，所谓的「制作源码包」不过是将 PKGBUILD 和几个补丁捆起来制作一个压缩包罢了。用户试图安装一个 CCR 包，本质上来说，其实是将 PKGBUILD 给下载下来，并执行里面的命令，从而编译安装真正的软件包。

#### PKGBUILD 示例
编写 PKGBUILD 并不是一件容易事，这里提供一个示例文件：

> 如果您知道分包是什么，请注意：**CCR 不支持分包，一个 PKGBUILD 只能对应一个软件包**。

```
# This is an example PKGBUILD file. Use this as a start to creating your own,
# and remove these comments. For more information, see 'man PKGBUILD'.
# NOTE: Please fill out the license field for your package! If it is unknown,
# then please put 'unknown'.

# Maintainer: Your Name <youremail@domain.com>
pkgname=NAME
pkgver=VERSION
pkgrel=1
epoch=
pkgdesc=""
arch=()
url=""
license=('GPL')
groups=()
depends=()
makedepends=()
checkdepends=()
optdepends=()
provides=()
conflicts=()
replaces=()
backup=()
options=()
install=
changelog=
source=("$pkgname-$pkgver.tar.gz"
        "$pkgname-$pkgver.patch")
noextract=()
md5sums=()
validpgpkeys=()

prepare() {
        cd "$pkgname-$pkgver"
        patch -p1 -i "$srcdir/$pkgname-$pkgver.patch"
}

build() {
        cd "$pkgname-$pkgver"
        ./configure --prefix=/usr
        make
}

check() {
        cd "$pkgname-$pkgver"
        make -k check
}

package() {
        cd "$pkgname-$pkgver"
        make DESTDIR="$pkgdir/" install
}
```

#### 简要说明
基本信息：

* `pkgbase`： 软件包的名称，名称由小写字母、数字和任意以下字符组成： `@ . _ + -`；
* `pkgver`：软件包的版本号，应该与软件原作者发布的版本号一致；
* `pkgrel`：发布号，一个正整数，用来区分同一版本软件的多次构建；
* `epoch`：当一个软件的版本编号方式改变，使用此数值进行强制升级。

描述：

* `pkgdesc`：软件包描述，合理使用关键字以方便搜索；
* `arch`： PKGBUILD 可以编译和使用的架构（Chakra 仅支持 `x86_64`） ；
* `url`：软件官方站点的网址；
* `license`：软件发布许可证；
* `groups`：软件包所在的组。

依赖关系：

* `depends`：软件的运行时依赖列表；
* `optdepends`：一组不影响软件主要功能，但能提供额外特性的软件包；
* `makedepends`：仅在软件编译时需要的软件包列表；
* `checkdepends`：运行测试组件时需要，而运行时不需要的包列表。

相关性：

* `provides`：这个变量说明当前包能提供的功能（或者像 cron、sh 这样的虚包）；
* `conflicts`：与当前软件包发生冲突的包列表；
* `replaces`：会因安装当前包而取代的包列表。

其它： 

* `backup`：当包被升级或卸载时，应当备份的文件；
* `options`：这个变量允许您重置 `makepkg` 的部分默认行为；
* `install`：`.install` 脚本的名称，应该和 `pkgname` 相同；
* `changelog`：软件包 changelog 的名称。

源码：

* `source`：构建软件包时需要的文件列表；
* `noextract`：一些列举于 `source` 中，但不需要在运行 `makepkg` 时解包的文件；
* `validpgpkeys`：PGP 指纹列表；

完整性：

* `md5sums`：`source` 列表文件中的 128 位 MD5 校验和；
* `sha1sums`：`source` 列表文件中 160 位 SHA-1 校验和；
* `sha224sums`、`sha256sums`、`sha384sums`、`sha512sums`：`source` 列表文件中224、256、384 或 512 位 SHA-2 校验和。

想要了解更多，可以阅读 ArchWiki 上的 [相关页面](https://wiki.archlinux.org/index.php/PKGBUILD)。

### 准备 PKGBUILD：从零开始（进阶）
准备好大干一场？如果您打算从零开始编写一个 PKGBUILD，建议继续阅读以下文章：

* [AUR 打包准则](https://wiki.archlinux.org/index.php/Arch_package_guidelines)（对 CCR 也是同样适用的）；
* [PKGBUILD 官方手册](https://www.archlinux.org/pacman/PKGBUILD.5.html)。

请确认您已经彻底了解清楚您所维护的软件包**依赖什么包**以及**如何编译安装**，否则还是建议先往下阅读。

### 准备 PKGBUILD：从外部导入
要知道现实情况是：CCR 的成熟程度远远落后于 AUR，和 [KCP](https://kaos-community-packages.github.io/)（KaOS 社区软件仓库）差不多。AUR 的生态就摆在那里，**几乎所有原生支持 Linux 的应用程序都能在 AUR 找到对应的软件包**。而 KaOS 的定位和 Chakra 类似，也是围绕 KDE 和 Qt 搭建的发行版。所以说，在大部分情况下是没有必要自己造轮子的。**有些轮子说不定 Chakra 团队已经为您造好了，在 Chakra 官方软件仓库里，不要重复提交这些软件包！**

请从 **Arch Linux 官方软件仓库 & AUR** 和 **KaOS 官方软件仓库 & KCP** 两个来源选择一个，并继续阅读下面的教程。

#### Arch Linux 官方软件仓库 & AUR
前往 [Arch Linux 软件包页面](https://www.archlinux.org/packages/) 和 [AUR 首页](https://aur.archlinux.org/) 发起搜索，直到您找到您打算提交到 CCR 的软件包。点击软件包名进入软件包详情页，定位到页面右侧的「Package Actions」方框：

![](https://bobby285271.coding.net/p/img/d/img/git/raw/master/ccr/1.png)

![](https://bobby285271.coding.net/p/img/d/img/git/raw/master/ccr/2.png)

对于 AUR 软件包页面，选择「View PKGBUILD」，当页面加载完毕后还需要点击「summary」链接前往 Git 仓库概览页面。

对于 Arch Linux 官方软件包，直接选择「Source Files」。

AUR 的软件包是一个软件包对应一个 Git 仓库的，而 Arch Linux 官方软件包则是 [core]、[extra] 仓库中的**所有软件包**放一个 Git 仓库，[community]、[multilib] 仓库中的**所有软件包**放另一个 Git 仓库。为了效率最大化，对于 AUR 包我们会选择 Clone 整个 Git 仓库，而官方软件包我们会选择逐个文件拷贝。

对于 AUR 软件包，找到 Clone 这一栏，下面有一个地址，将它复制下来：

![](https://bobby285271.coding.net/p/img/d/img/git/raw/master/ccr/3.png)

打开终端，输入 `git clone <刚刚复制的 URL 地址> --depth=1`，例如：
```
git clone https://aur.archlinux.org/wps-office.git/ --depth=1
```

至于 Arch Linux 官方软件包，新建一个文件夹（名字随意），然后回到浏览器，**逐个文件**点击文件名旁边的「plain」链接（如图所示），如果是文本文件复制一下，在刚刚新建的文件夹新建一个**同样名称**的文件（千万不要随意改名），将文本粘贴下去。

![](https://bobby285271.coding.net/p/img/d/img/git/raw/master/ccr/4.png)

至于二进制文件，浏览器应该会弹出下载窗口，将文件下载后放入文件夹中。

记得检查文件权限是否正常，如果发现文件权限和浏览器中「Mode」显示的是否一致。如果不一致，使用 `chmod` 修正妥当（[这里](http://linux.vbird.org/linux_basic/0210filepermission.php#chmod) 有这个命令相关的介绍）。

#### KaOS 官方软件仓库 & KCP

和前面 Arch Linux 官方软件仓库 & AUR 大同小异。[KaOS 软件包页面](https://kaosx.us/packages/) 和 [KCP](https://kaos-community-packages.github.io/) 同样提供了搜索框，发起搜索后进入软件包详情页。

同样，官方包建议逐个文件获取，点击右栏的「Source Files」，剩下的操作估计大家很熟悉了，毕竟 GitHub 大家都用过，逐个查看「Raw」文件即可。

![](https://bobby285271.coding.net/p/img/d/img/git/raw/master/ccr/5.png)

KCP 包可以整个仓库 Clone 下来（原因同 AUR），点击右上角的「View on GitHub」，后面的我也不再多说了，依然是 `git clone <刚刚复制的 URL 地址> --depth=1`。

![](https://bobby285271.coding.net/p/img/d/img/git/raw/master/ccr/6.png)

### 准备 PKGBUILD：收尾 & 检查工作
 **这一步很关键，一定要重视！**

上一步完成后，我们得到了一个文件夹，里面有些什么取决于您选择的包，大多数情况下里面可能只有一个 `PKGBUILD` 文件。没关系，**反正一定要有 PKGBUILD**，剩下的无需我们来检查。再次强调，PKGBUILD 非常关键，所以我们一定要进行检查。

这里是一份 `PKGBUILD`：
```
# Maintainer: Jan de Groot <jgc@archlinux.org>

pkgname=xorg-xprop
pkgver=1.2.3
pkgrel=1
pkgdesc="Property displayer for X"
arch=(x86_64)
url="https://xorg.freedesktop.org/"
license=('custom')
depends=('libx11')
makedepends=('xorg-util-macros')
groups=('xorg-apps' 'xorg')
source=(https://xorg.freedesktop.org/archive/individual/app/xprop-${pkgver}.tar.bz2{,.sig})
sha512sums=('ad7987fec11ae19b7adc3b0f683fc393e95155f3b6c753d1d8744aedcfb360452eee5735a4c380152b286905931515f3e1a28676b5531001eb8dd93b7249916a'
            'SKIP')
validpgpkeys=('4A193C06D35E7C670FA4EF0BA2FB9E081F2D130E') # "Alan Coopersmith <alan.coopersmith@oracle.com>"

build() {
  cd xprop-${pkgver}
  ./configure --prefix=/usr
  make
}

package() {
  cd xprop-${pkgver}
  make DESTDIR="${pkgdir}" install
  install -m755 -d "${pkgdir}/usr/share/licenses/${pkgname}"
  install -m644 COPYING "${pkgdir}/usr/share/licenses/${pkgname}/"
}
```

聪明人看完第一行就应该知道这份 `PKGBUILD` 是从 Arch Linux 官方仓库上抓取下来的。如果您刚才「 简要说明」，大概就知道我们要重点检查什么了。

#### 依赖关系
首先是 `depends` 和 `makedepends`，这两项都是针对 Arch Linux 来写的。例如说一个包依赖 `foo`，到了 Chakra 这边，可能会有这几种情况：
* Chakra 官方软件仓库正好有 `foo`，提供的是同样的功能；
* Chakra 官方软件仓库没有 `foo`，但 CCR 有；
* Chakra 官方软件仓库或 CCR 提供了提供同样功能，但包名不同的 `foo1`；
* Chakra 官方软件仓库或 CCR 不提供 `foo`。

第一、二种情况可以直接给过不用管，因为 `chaser` 会自动从官方软件仓库和 CCR 获取相应的依赖。

第三种情况要小心一点，并不是所有情况下简单地将 `foo` 改成 `foo1` 就可以的，看一下下面这个案例：

![](https://bobby285271.coding.net/p/img/d/img/git/raw/master/ccr/7.png)

注意到在 Arch Linux 这里同时存在 `openssl` 和 `openssl-1.0` 两个软件包，观察版本号，我们知道后者是为了兼容性而单独打包的。`openssl` 的版本号（截至 20190201）是 1.1.1.a-1，`openssl-1.0` 则是 1.0.2.q-1。

接下来看看 AUR 里 `wps-office` 这个包：

![](https://bobby285271.coding.net/p/img/d/img/git/raw/master/ccr/8.png)

它依赖的是 `openssl-1.0`。我们可以搜索 Chakra 官方软件仓库（省略部分输出）：
```
[chakra@localhost ~]$ pacman -Ss openssl
lib32/lib32-openssl 1.0.2.q-1
    The Open Source toolkit for Secure Sockets Layer and Transport Layer Security (32-bit)
core/openssl 1.0.2.q-1 [已安装]
    The Open Source toolkit for Secure Sockets Layer and Transport Layer Security
core/perl-crypt-openssl-bignum 0.04-3
    OpenSSL's multiprecision integer arithmetic
core/perl-crypt-openssl-random 0.04-8
    Interface to OpenSSL PRNG methods
core/perl-crypt-openssl-rsa 0.28-2
    Interface to OpenSSL RSA methods
core/perl-net-ssleay 1.63-1
    Perl extension for using OpenSSL
core/pkcs11-helper 1.25.1-1
    A library that simplifies the interaction with PKCS11 providers for end-user applications using a simple API and optional OpenSSL engine
core/python2-characteristic 14.3.0-1
    Service identity verification for pyOpenSSL
core/python2-service-identity 17.0.0-1
    Service identity verification for pyOpenSSL
core/python3-characteristic 14.3.0-1
    Service identity verification for pyOpenSSL
core/python3-service-identity 17.0.0-1
    Service identity verification for pyOpenSSL
core/tls 1.6.7-1
    OpenSSL extension to Tcl
desktop/python2-ndg-httpsclient 0.4.3-1
    Provides enhanced HTTPS support for httplib and urllib2 using PyOpenSSL
desktop/python2-pyopenssl 17.3.0-1
    Python2 wrapper module around the OpenSSL library
desktop/python3-ndg-httpsclient 0.4.3-1
    Provides enhanced HTTPS support for httplib and urllib2 using PyOpenSSL
desktop/python3-pyopenssl 17.3.0-1
    Python3 wrapper module around the OpenSSL library
[chakra@localhost ~]$
```

这就是所有的搜索结果了，有 `openssl` 却没有 `openssl-1.0`，但是要注意 Chakra 提供的 `openssl` 版本是 1.0.2.q-1，和 Arch 中 `openssl-1.0` 是一致的。从中我们可以得出以下信息：

Chakra 是半滚动更新发行版，更新一般来说会滞后于 Arch Linux，Chakra 的 OpenSSL 打包落后于 Arch Linux，所以目前来说我们要将 `PKGBUILD` 文件中的 `openssl-1.0` 修改为 `openssl`，但当 Chakra 将 `openssl` 升级到 1.1+ 之后，我们要实时监测 Chakra 动态，做好两手准备：

* 如果 `openssl-1.0` 可用，我们要重新打包 `wps-office`，将 PKGBUILD 改回 `openssl-1.0`；
* 如果 Chakra 不提供 `openssl-1.0`，我们得为 `openssl-1.0` 打包并提交到 CCR。

注意：在这个案例中，只要 Chakra 的 `openssl` 不更新，用户就不能提前将 `openssl-1.0` 提交到 CCR，因为**它和官方源所提供的重复了**。

第四种情况，您需要前往**您刚刚所选的来源**（**Arch Linux 官方软件仓库 & AUR** 或 **KaOS 官方软件仓库 & KCP**），按照上面提供的步骤，获取这个依赖包的相关文件，在稍后一起进行打包。

#### PGP 密钥
如果您从 Arch Linux 或 KaOS 官方软件仓库获取软件包的要特意留意 `validpgpkeys` 这一项，如果有所填写，要注意的是，Charka 的 `chakra-keyring` 软件包只提供了 Chakra 维护者的签名，没有 Arch Linux 或 KaOS 的。这样会导致打包失败，因为打包工具 `makepkg` 是会检查 PGP 密钥的。

规避检查很简单，将 `validpgpkeys` 所在的那一行删掉。当然也别忘了将相关的文件删掉，还是上面 `xorg-xprop` 的那个例子：
```
...
source=(https://xorg.freedesktop.org/archive/individual/app/xprop-${pkgver}.tar.bz2{,.sig})
sha512sums=('ad7987fec11ae19b7adc3b0f683fc393e95155f3b6c753d1d8744aedcfb360452eee5735a4c380152b286905931515f3e1a28676b5531001eb8dd93b7249916a'
            'SKIP')
validpgpkeys=('4A193C06D35E7C670FA4EF0BA2FB9E081F2D130E') # "Alan Coopersmith <alan.coopersmith@oracle.com>"
...
```

在删除下面这一行之后：
```
validpgpkeys=('4A193C06D35E7C670FA4EF0BA2FB9E081F2D130E') # "Alan Coopersmith <alan.coopersmith@oracle.com>"
```

我们留意到 `source` 那一行，`{,.sig}` 意味着它同时包含了两个文件：`xprop-${pkgver}.tar.bz2` 和 `xprop-${pkgver}.tar.bz2.sig`，后者是用于验证密钥的，删除它。

而 `sha512sums` 那里，所有的检验值都和 `source` 的文件逐一对应，所以我们得把 `xprop-${pkgver}.tar.bz2.sig` 的校验值给相应地删除，即使它是 `SKIP`，意为「跳过校验值检查」。

这个文件最终被修改成了这样：
```
# Maintainer: Jan de Groot <jgc@archlinux.org>

pkgname=xorg-xprop
pkgver=1.2.3
pkgrel=1
pkgdesc="Property displayer for X"
arch=(x86_64)
url="https://xorg.freedesktop.org/"
license=('custom')
depends=('libx11')
makedepends=('xorg-util-macros')
groups=('xorg-apps' 'xorg')
source=(https://xorg.freedesktop.org/archive/individual/app/xprop-${pkgver}.tar.bz2)
sha512sums=('ad7987fec11ae19b7adc3b0f683fc393e95155f3b6c753d1d8744aedcfb360452eee5735a4c380152b286905931515f3e1a28676b5531001eb8dd93b7249916a')

build() {
  cd xprop-${pkgver}
  ./configure --prefix=/usr
  make
}

package() {
  cd xprop-${pkgver}
  make DESTDIR="${pkgdir}" install
  install -m755 -d "${pkgdir}/usr/share/licenses/${pkgname}"
  install -m644 COPYING "${pkgdir}/usr/share/licenses/${pkgname}/"
}
```

#### 其它
如果您是从外部导入的软件包，案例来说您是无需检查校验值的，如果是自己创建的软件包就一定要检查校验值了：打开 Konsole，使用 `cd` 命令进入您的文件夹，然后执行 `updpkgsums` 命令即可。

另外我们还建议您将 Maintainer 后面的姓名和邮箱改成您自己的。

#### 亲自测试（建议）
打开 Konsole，使用 `cd` 命令进入您的文件夹，然后执行 `makepkg` 命令。 如果 `PKGBUILD` 没有错误，将会生成一个包，但是如果 `PKGBUILD` 被破坏或未完成，它将抛出一个错误。

`makepkg` 的用法如下：
```
用法：/usr/bin/makepkg [选项]

选项：
  -A, --ignorearch      忽略不完整的 arch 字段 (位于 PKGBUILD 中)
  -c, --clean           编译后清理工作文件
  -C, --cleanbuild      在编译软件包之前删除 $srcdir/ 文件夹
  -d, --nodeps          跳过所有依赖关系检查
  -e, --noextract       不解压源文件 (使用现存的 $srcdir/ 目录)
  -f, --force           覆盖现存的软件包
  -g, --geninteg        为源码文件生成完整性检查值
  -h, --help            显示本帮助信息并退出
  -i, --install         成功编译后安装软件包
  -L, --log             记录软件包编译过程
  -m, --nocolor         禁止彩色输出信息
  -o, --nobuild         仅下载和解压缩文件
  -p <文件>             使用另外的编译脚本 (而不是 'PKGBUILD' ) 
  -r, --rmdeps          编译成功后删除安装的依赖关系
  -R, --repackage       不编译而重新打包软件包内容
  -s, --syncdeps        使用 pacman 安装缺失的依赖关系
  -S, --source          不下载源文件只生成仅包含源的包
  -V, --version         显示版本信息并退出
  --allsource           只生成源码包 (包括有已下载的源码) 
  --check               运行 check() 函数 (包含于 PKGBUILD 中)
  --config <文件>       使用另外的配置文件 (而不是 '/etc/makepkg.conf')
  --holdver             不升级版本控制系统源
  --key <密匙>          指定签名 gpg 使用的密匙而不用默认密匙
  --noarchive           不生成软件包归档
  --nocheck             不执行 check() 函数在 PKGBUILD 中
  --noprepare           不执行 prepare() 函数在 PKGBUILD 中
  --nosign              不为该软件包创建签名
  --packagelist         只列出将会产生的包，不带PKGEXT
  --printsrcinfo        打印出生成的 SRCINFO 并退出
  --sign                使用 gpg 签名生成的软件包
  --skipchecksums       不验证源文件的检验值
  --skipinteg           不对源文件执行任何验证检查
  --skippgpcheck        不验证有 PGP 签名的源文件
 --verifysource         下载源文件(如果需要)并进行完整性检查

这些选项可以传递给 pacman：
  --asdeps              作为依赖安装
  --needed              不重装已是最新的目标软件包
  --noconfirm           当解决依赖关系时不询问确认
  --noprogressbar       下载文件时不显示进度条

如果没有指定 -p，makepkg 将寻找 'PKGBUILD'
```

如果运行 `makepkg` 成功，在您工作的目录下将会生成一个名为 `$pkgname-$pkgver.pkg.tar.gz` 的新文件。这个文件可以使用 `pacman -U` 或 `pacman -A` 安装。如果包看起来是正确的，那您的工作就完成了。

#### 一个实用的检查工具（建议）
Namcap 是 Arch Linux 推荐的检查工具，它将会做以下工作：

* 检查 PKGBUILD 文件里的一些常见错误；
* 用 `ldd` 扫描包中所有的ELF文件，自动报告缺失或可去除的依赖；
* 启发式搜寻缺失或冗余的依赖。

要养成用 Namcap 检查包的习惯，以避免提交包后再做修复的麻烦。

打开 Konsole，使用 `cd` 命令进入您的文件夹，然后执行：
```
namcap PKGBUILD
```

### 制作源码包
后面的步骤都很简单了！打开 Konsole，使用 `cd` 命令进入您的文件夹，执行以下命令：
```
makepkg --source
```

这个命令实际上就是将文件夹里的文件打成一个包，但为了确认您的 PKGBUILD 中的校验值准确无误，PKGBUILD 中 `source` 一栏所有文件都会被下载下来以校验完整性，耐心等待就好。

输出内容大概是：
```
==> 正在进入 fakeroot 环境...
==> 正在创建源码包...
  -> 正在添加 PKGBUILD...
  -> 正在生成 .SRCINFO 文件...
  -> 正在压缩源码包...
==> 正在离开 fakeroot 环境。
==> 源代码包已创建：foo
```

### 将源码包上传到 CCR
请确认您已经登录 CCR 帐户，接下来访问 [上传页面](https://ccr.chakralinux.org/pkgsubmit.php)，选择软件包分类，接下来选择源码包，源码包应该位于您的文件夹中，包名应该是 `$pkgname-$pkgver.src.tar.gz`。

![](https://bobby285271.coding.net/p/img/d/img/git/raw/master/ccr/9.png)

点击「上传」即可。如果 CCR 里面本身没有这个包或这个包是一个无人维护的包（孤儿包），您将会成为维护者。CCR 里面旧版本的软件包将会被您的新软件包取代。如果这个包已经有维护者，CCR 会拒绝您的提交。如果您发现 CCR 里的包已经被标记为「过期」一段时间而维护者没有采取任何动作，您可以联系 Chakra 团队将该包重置为无人维护状态。