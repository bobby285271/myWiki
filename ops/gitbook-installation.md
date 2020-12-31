---
title: Gitbook Installation
description: 
published: true
date: 2020-09-14T08:13:35.078Z
tags: gitbook, nodejs, npm
editor: markdown
---

# Gitbook 安装

https://github.com/rcore-os/rCore-Tutorial

```bash
npm install -g gitbook-cli
gitbook install
gitbook serve
```

报错如下：
```
bobby285271@inspiron [ test ] $ gitbook init
Installing GitBook 3.2.3
/usr/lib/node_modules/gitbook-cli/node_modules/_npm@5.1.0@npm/node_modules/graceful-fs/polyfills.js:287
      if (cb) cb.apply(this, arguments)
                 ^

TypeError: cb.apply is not a function
    at /usr/lib/node_modules/gitbook-cli/node_modules/_npm@5.1.0@npm/node_modules/graceful-fs/polyfills.js:287:18
    at FSReqCallbac
```

解决方法：

https://blog.csdn.net/cuk0051/article/details/108319622

编辑 `polyfills.js`，注释以下行：

```js
fs.stat = statFix(fs.stat)
fs.fstat = statFix(fs.fstat)
fs.lstat = statFix(fs.lstat)
```