---
title: Fedora Silverblue Guide
description: 
published: true
date: 2021-06-04T04:19:58.771Z
tags: guide, fedora, silverblue, ostree, rpm
editor: markdown
dateCreated: 2021-02-13T07:29:38.903Z
---

## 安装

如果是要拿 Silverblue 替换其它发行版，`/boot/efi` 下旧发行版的文件要删干净，否则报错如下。

```
ostree [‘admin’,’–sysroot=/mnt/sysimage’,‘deploy’, ‘–os=fedora’,‘fedora:fedora/33/x86_64/silverblue’] exited with code -6
```

用自动分区然后回收空间，不要试图手动分区。真要手动分区允许的挂载点以 [Fedora 文档](https://docs.fedoraproject.org/en-US/fedora-silverblue/installation/#manual-partition) 为准（Anaconda 是不会报错的）。

自动分区 `/etc/fstab` 参考：

```
UUID=b6c1083d-1508-4fe6-9ef1-30c5974aa5e4 /                       btrfs   subvol=root     0 0
UUID=ba3a2f1e-37a6-46d4-9db7-0098b7a22382 /boot                   ext4    defaults        1 2
UUID=E288-231E          /boot/efi               vfat    umask=0077,shortname=winnt 0 2
UUID=b6c1083d-1508-4fe6-9ef1-30c5974aa5e4 /home                   btrfs   subvol=home     0 0
```

自动分区 `lsblk` 参考：

```
NAME        MAJ:MIN RM   SIZE RO TYPE MOUNTPOINT
zram0       252:0    0   7.5G  0 disk [SWAP]
nvme0n1     259:0    0 238.5G  0 disk 
├─nvme0n1p1 259:1    0   650M  0 part /boot/efi
├─nvme0n1p2 259:2    0   128M  0 part 
├─nvme0n1p3 259:3    0 140.1G  0 part 
├─nvme0n1p4 259:4    0     1G  0 part /boot
├─nvme0n1p5 259:5    0   990M  0 part 
├─nvme0n1p6 259:6    0  15.3G  0 part 
├─nvme0n1p7 259:7    0   1.3G  0 part 
└─nvme0n1p8 259:8    0    79G  0 part /var/home
```

## 换源

`/etc/ostree/remotes.d/fedora.conf`

```
[remote "fedora"]
url=https://mirror.sjtu.edu.cn/fedora-ostree
gpg-verify=true
gpgkeypath=/etc/pki/rpm-gpg/
# contenturl=mirrorlist=https://ostree.fedoraproject.org/mirrorlist
```

## Flathub

```
sudo flatpak remote-add --if-not-exists flathub https://flathub.org/repo/flathub.flatpakrepo
sudo flatpak remote-modify flathub --url=https://mirror.sjtu.edu.cn/flathub
```

## Toolbox

```
toolbox create
toolbox enter
```

## 更新

```
rpm-ostree upgrade
```

## GRUB 选单重复

```
sudo grub2-switch-to-blscfg
sudo grub2-mkconfig -o /boot/efi/EFI/fedora/grub.cfg
```