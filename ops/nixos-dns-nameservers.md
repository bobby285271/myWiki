---
title: Change DNS nameservers in NixOS
description: 
published: true
date: 2021-03-28T11:40:58.382Z
tags: linux, nixos, dns, networking
editor: markdown
dateCreated: 2021-03-28T11:40:58.382Z
---

## NixOS 下修改 DNS 服务器

编辑 `/etc/nixos/configuration.nix`。

一个是腾讯的，一个是阿里的。

```
{ config, pkgs, ... }:

{
  networking = {
    nameservers = [
      "119.29.29.29"
      "223.5.5.5"
    ];
  };
}
```

NetworkManager 下把方法从自动改为自动（仅地址）。

`nixos-rebuild switch` 后看一下 `/etc/resolv.conf`。