---
title: Sound Configuration on AOSC OS (Inspiron 7591)
description: 
published: true
date: 2020-08-06T11:28:09.308Z
tags: guide, linux, aosc, sound
editor: markdown
---

在 Inspiron 7591 下无法播放声音，解决方案：

编辑 `/etc/modprobe.d/audio-fix.conf`：

```
blacklist snd-sof-pci
options snd-intel-dspcfg dsp_driver=1
```

重启。