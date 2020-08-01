---
title: How to Generate a Hexo Blog and Deploy it on GitHub
description: 
published: true
date: 2020-08-01T03:00:04.887Z
tags: hexo, github, travis-ci
editor: markdown
---

相信我，这是最最最适合懒人的 Hexo 教程！全文内容保证和百度搜到的完全不一样！只需一个浏览器，像玩转 [Jekyll Now](http://www.jekyllnow.com) 那样玩转 Hexo 是完全有可能的！没错，我们将要用到的“黑科技”就是 [Travis CI](https://travis-ci.org)！ 毕竟这么高级的玩具（误）如果不拿来好好利用就浪费了。

**以下为正文部分，请注意：为了降低阅读门槛，我们提供了大量的代码供参考。在实际操作中，请不要一味地复制粘贴，所有代码中带有全角标点“【】”的地方请务必作相应的替换，不要保留中括号。**

## 扫盲
### Hexo
[Hexo](https://hexo.io/zh-cn) 是一个快速、简洁且高效的博客框架。Hexo 使用 [Markdown](https://daringfireball.net/projects/markdown)（或其他渲染引擎）解析文章，在几秒内，即可利用靓丽的主题生成静态网页。

### Git
[Git](https://git-scm.com) 是 Linus Torvalds 为了帮助管理 [Linux](https://www.kernel.org) 内核开发而开发的一个开放源码的版本控制软件，可以有效、高速的处理从很小到非常大的项目版本管理。

### GitHub
[GitHub](https://github.com) 是最大的 Git 版本库托管商，是成千上万的开发者和项目能够合作进行的中心。 大部分 Git 版本库都托管在 GitHub，很多开源项目使用 GitHub 实现 Git 托管、问题追踪、代码审查以及其它事情。 

### Travis CI
[Travis CI](https://travis-ci.org) 是目前新兴的开源持续集成构建项目，它采用 [YAML](http://www.yaml.org) 格式，简洁清新独树一帜。目前不少 GitHub 项目都已经移入到 Travis CI 的构建队列中。

## 铺垫
### GitHub & Git 上手
#### 注册 GitHub 账户并验证邮箱
前往 [这里](https://github.com/join) 注册邮箱，详情请阅读 [官方文档（英文）](https://help.github.com/articles/signing-up-for-a-new-github-account) 和 [百度经验上的教程](http://jingyan.baidu.com/article/455a9950abe0ada167277864.html)；前往[这里](https://github.com/settings/emails) 验证邮箱，详情请阅读 [官方文档（英文）](https://help.github.com/articles/verifying-your-email-address)。

#### GitHub 基本图形化操作
请阅读 [官方文档（英文）](https://guides.github.com/activities/hello-world)，[这里](http://blog.csdn.net/kabulore/article/details/51801337) 提供了中文翻译。这个指南将帮助你了解创建和使用仓库、创建和管理分支、改变一个文件并将它提交到 GitHub 上和发起以及合并请求的方法。

#### 最最最基本的几个 Git 命令
别被百度出来的什么命令大全吓到了，其实无非就几个命令：
* `git init`：在当前目录新建一个 Git 代码库;
* `git clone 【项目 URL】`：下载一个项目和它的整个代码历史；
* `git add -A`：添加当前目录的所有文件到暂存区；
* `git commit -m 【提交信息】`：提交暂存区到仓库区；
* `git push 【远程仓库别称】 【分支】`：上传本地指定分支到远程仓库；
* `git pull 【远程仓库别称】 【分支】`：取回远程仓库的变化，并与本地分支合并；
* `git branch 【分支】`：新建一个分支，但依然停留在当前分支；
* `git checkout 【分支】 `：切换到指定分支，并更新工作区。

若想深入学习可阅读 [常用 Git 命令清单](http://www.ruanyifeng.com/blog/2015/12/git-cheat-sheet.html) 和 [官方文档](https://git-scm.com/book/zh)，实际上等你真的用到这些功能时再百度慢慢学也不迟。当然，如果你连这些基本命令都看不懂，没关系，我们会在后面具体地介绍它们。

### Markdown 语法上手
这是写博客必备的一种语法，而且学起来比你想象中简单很多，这里提供一个 [说明文档](http://www.appinn.com/markdown) 和一个 [交互式入门向导（英文）](http://commonmark.org/help/tutorial)。

## 开始
### 迈出第一步
进入 GitHub 网站，登录你的账户，新建一个名为 `【你的 GitHub 用户名】.github.io` 的仓库，具体操作不再细讲。接下来应该看到的是类似这样的东西：
```
Quick setup — if you’ve done this kind of thing before
or https://github.com/xxx/xxx.github.io.git

We recommend every repository include a README, LICENSE, and .gitignore.
…or create a new repository on the command line

echo "# xxx.github.io" >> README.md
git init
git add README.md
git commit -m "first commit"
git remote add origin https://github.com/xxx/xxx.github.io.git
git push -u origin master

…or push an existing repository from the command line

git remote add origin https://github.com/xxx/xxx.github.io.git
git push -u origin master

…or import code from another repository

You can initialize this repository with code from a Subversion, Mercurial, or TFS project.
```

然而`README`、`LICENSE` 和 `.gitignore`，我一个都不想建，我们要开始建立用于初始化 Hexo 的 Travis CI 配置文件，“Create new file”的按钮我又找不到？很简单，文件都是可以改名的嘛，点击“README”这个链接，把人家帮你填的东西全部删掉，就可以开始你的骚操作啦。

### 初始化 Hexo 的 Travis CI 配置文件
将新建的文件命名为 `.travis.yml`（请勿改名），然后填写以下内容：
```
language: node_js
node_js: stable

install:
  - npm install hexo-cli -g

script:
  - hexo init blog
  - cd blog/

after_script:
  - git init
  - git config user.name "【你的大名，如 Bobby Rong】"
  - git config user.email "【你的邮箱】"
  - git add -A
  - git commit -m "【提交信息，如 Update files】"
  - git push --force --quiet "https://${githubblog}@${GH_REF}" master

branches:
  only:
  - master
    
env:
 global:
  - GH_REF: github.com/【你的 GitHub 用户名】/【你的 GitHub 用户名】.github.io.git
```

作相应替换后点击“Commit new file”即可。下面我来解释一下这是在干什么：这是初始化 Hexo 的 Travis CI 配置文件，待会把这个仓库加入构建队列，Travis 就会依次执行 `install`、`script` 和 `after_script` 这三个部分里面的命令，这些命令前面要加一个 `-` 千万别漏掉了。
* `npm install hexo-cli -g`：安装 Hexo-cli；
* `hexo init blog`：初始化博客；
* `git init`：初始化 Git 目录；
* `git config user.name "【你的大名，如 Bobby Rong】"`：配置 Git，填写提交者姓名（昵称也行）；
* `git config user.email "【你的邮箱】"`：配置 Git，填写提交者邮箱；
* `git add -A`：添加当前目录的所有文件到暂存区；
* `git commit -m "【提交信息，如 Update files】`：提交暂存区到仓库区，提交信息可以随便填写；
* `git push --force --quiet "https://${githubblog}@${GH_REF}" master`：上传本地指定分支到远程仓库的 master 分支，${GH_REF} 为变量，Travis CI 会赋什么值扫一眼下面就知道了：`github.com/【你的 GitHub 用户名】/【你的 GitHub 用户名】.github.io.git`，${githubblog} 在后续配置后，会被 Travis CI 换成你 GitHub 的 Access Token，至于两个变量名为啥画风完全不同，嗯是故意的，就是为了烧死强迫症嘛。不过为了避免改完后到后面复制粘贴太爽有不知道改哪里，忍了吧。

### 配置 Travis CI
#### 新建 Access Token
毕竟 Travis CI 是不可能在提交代码时向你询问 GitHub 账户密码的，所以就有了这个东西的产生。

首先我们来到 GitHub 的设置界面（没错就是点 GitHub 右上角的头像再点“Settings”），点击“Personal access tokens”，点击右上角的“Generate new token”按钮生成一个，点击后它会叫你输入密码，然后来到如下界面：
```
New personal access token

Personal access tokens function like ordinary OAuth access tokens. They can be used instead of a password for Git over HTTPS, or can be used to authenticate to the API over Basic Authentication.

Token description
____________________
What's this token for?

Select scopes
Scopes define the access for personal tokens. Read more about OAuth scopes.

（下面就是一堆东西让你去勾选的）
```

在 Token description 随便填个标题，下面是勾选一些权限，每一项旁边都有解释的，有强迫症的可以想想部署一个 Hexo 它应该需要一些什么权限，然后按需勾选；没有强迫症的就全部勾选了吧，反正 Travis CI 也只是乖乖地跑你的代码而已，真的出了事也只能怪你。

生成完后，把那一串东西拷贝下来找个地方妥善保存，注意这和密码是一回事，不要偷懒把它放到 GitHub 上或别人能看到的地方。注意只有这时候它才显示出来，为了安全它以后就再也不会显示了，如果忘了只能重新生成一个了。

#### 添加环境变量与设定构建条件
这时候就要前往 [Travis CI](https://travis-ci.org) 了。使用你的 GitHub 账户登录，随后前往 Accounts 页面（点击右上角的头像，选择“Accounts”）,找到"【你的 GitHub 用户名】/【你的 GitHub 用户名】.github.io"这一行，点击旁边的小齿轮进入该仓库的设置页面。

几个地方需要设定：
* Gerenal 中启用“Build only if .travis.yml is present”和“Build branch updates”，不启用剩余两个；
* Environment Variables 中新建一个环境变量，在“Name”填写 `githubblog`（如果你手贱真的把前面的变量名改了记得把这里的也给改掉），“Value”填写你复制的那一串 Access Token，旁边的“Display value in build log”千万别启用，前面说过原因了。

其它的如果你知道这是在干什么可以自己设定，不知道的都不用碰了。

### 你的第一次构建
在当前页面，点击“Current”，然后激活这个仓库。理论上你应该能看到 Build #1 正在运行，Job log 应该有东西在滚动输出。如果没有，只需点击“More options”，选择“Trigger build”，什么都不用设定，只需点击“Trigger custom build”就可以开始了，最终的构建日志看起来大概是这样的：
```
Worker information
Build system information

W: http://dl.hhvm.com/ubuntu/dists/trusty/InRelease: Signature by key 36AEF64D0207E7EEE352D4875A16E7281BE7A449 uses weak digest algorithm (SHA1)
W: http://ppa.launchpad.net/couchdb/stable/ubuntu/dists/trusty/Release.gpg: Signature by key 15866BAFD9BCC4F3C1E0DFC7D69548E1C17EAB57 uses weak digest algorithm (SHA1)
$ git clone --depth=50 --branch=test https://github.com/xxx/xxx.github.io.git

Setting environment variables from repository settings
$ export githubblog=[secure]

Setting environment variables from .travis.yml
$ export GH_REF=github.com/bobby285271/blog-test.git

$ export PATH=./node_modules/.bin:$PATH
Updating nvm
$ nvm install stable
$ node --version
v8.6.0
$ npm --version
5.3.0
$ nvm --version
0.33.4
$ npm install hexo-cli -g
$ hexo init blog
INFO  Cloning hexo-starter to ~/build/bobby285271/blog-test/blog
Cloning into '/home/travis/build/bobby285271/blog-test/blog'...
Submodule 'themes/landscape' (https://github.com/hexojs/hexo-theme-landscape.git) registered for path 'themes/landscape'
Cloning into '/home/travis/build/bobby285271/blog-test/blog/themes/landscape'...
Submodule path 'themes/landscape': checked out 'decdc2d9956776cbe95420ae94bac87e22468d38'
INFO  Install dependencies
yarn install v0.27.5
info No lockfile found.
[1/4] Resolving packages...
warning hexo > swig@1.4.2: This package is no longer maintained
[2/4] Fetching packages...
warning fsevents@1.1.2: The platform "linux" is incompatible with this module.
info "fsevents@1.1.2" is an optional dependency and failed compatibility check. Excluding it from installation.
[3/4] Linking dependencies...
[4/4] Building fresh packages...
success Saved lockfile.
Done in 10.64s.
INFO  Start blogging with Hexo!


The command "hexo init blog" exited with 0.
$ cd blog/


The command "cd blog/" exited with 0.
$ git init
$ git config user.name "xxx"
$ git config user.email "xxx@xxx.com"
$ git add -A
$ git commit -m "xxx"
$ git push --force --quiet "https://${githubblog}@${GH_REF}" master

Done. Your build exited with 0.
```

## 站点配置
回去 GitHub，重新进入 `【你的 GitHub 用户名】.github.io` 仓库，是否发现多了很多玩意？有这几个文件：`scaffolds/`、`source/`、`.gitignore`、`_config.yml`、`package.json`、`yarn.lock`，暂时不用管发生了什么，先把站点配置好。

### 新建分支
由于 GitHub 要求最终生成的网页必须在 master 分支，而我们却把源文件放在了这个分支，非常尴尬，把内容迁移走吧。很简单，只需新建一个分支，这里假设叫“blog-source”，同理，如果你不知道后面哪个地方要做相应调整就用这个名字吧。然后把这个新分支设定为默认分支（除非你知道后面要怎么相应作调整否则一定要设），点击那个“2 branches”，点击“Change default branch”，在“Default branch”这一栏有一个下拉框，选择新建的分支再选择“Update”。有些时候会让你输密码，完事。

### 挑选一个主题
终于不用面对一堆代码了，下面是考验你审美能力的时间。百度 Hexo 主题，选择一个你喜欢的又有 GitHub 仓库链接的主题。[知乎](https://www.zhihu.com/question/24422335) 上有不少好的推荐。由于我对 Material Design 非常感兴趣，我选择的是 [hexo-theme-material](https://github.com/viosey/hexo-theme-material)，注意有个 [同名](https://github.com/wayou/hexo-theme-material) 的主题，要区分开来。

如果你也选择了 hexo-theme-material（指 Viosey 创作的主题，下同），我还在本文后面附上配置这个主题的一些注意事项。

选择完了之后，进入其 GitHub 的仓库，点击右上角的“Fork”以拷贝该项目的代码到自己这里。

### 站点配置文件 config.yml
接下来编辑 `config.yml`，这是整个博客站点的编辑文件，要修改的地方其实也不多，数数有多少个中括号就知道了。强调：所有的半角冒号后面都有一个空格别弄丢了。
```
# Hexo Configuration
## Docs: https://hexo.io/docs/configuration.html
## Source: https://github.com/hexojs/hexo/

# Site
title: 【站点的标题】
subtitle: 【站点的副标题】
description: 【站点的描述】
author: 【作者名字】
language: 【这里填写 zh-CN】
timezone: 【这里填写 Asia/Shanghai】

# URL
## If your site is put in a subdirectory, set url as 'http://yoursite.com/child' and root as '/child/'
url: 【这里填写 https://【你的 GitHub 用户名】.github.io，打算在后面绑定自己的域名的填写相应的完整 URL】
root: 【这里填写 /】
permalink: :year/:month/:day/:title/
permalink_defaults:

# Directory
source_dir: source
public_dir: public
tag_dir: tags
archive_dir: archives
category_dir: categories
code_dir: downloads/code
i18n_dir: :lang
skip_render:

# Writing
new_post_name: :title.md # File name of new posts
default_layout: post
titlecase: false # Transform title into titlecase
external_link: true # Open external links in new tab
filename_case: 0
render_drafts: false
post_asset_folder: false
relative_link: false
future: true
highlight:
  enable: true
  line_number: true
  auto_detect: false
  tab_replace:
  
# Search
search:
  path: search.xml
  field: all
  
# Home page setting
# path: Root path for your blogs index page. (default = '')
# per_page: Posts displayed per page. (0 = disable pagination)
# order_by: Posts order. (Order by date descending by default)
index_generator:
  path: ''
  per_page: 10
  order_by: -date
  
# Category & Tag
default_category: uncategorized
category_map:
tag_map:

# Date / Time format
## Hexo uses Moment.js to parse and display date
## You can customize the date format as defined in
## http://momentjs.com/docs/#/displaying/format/
date_format: YYYY-MM-DD
time_format: HH:mm:ss

# Pagination
## Set per_page to 0 to disable pagination
per_page: 10
pagination_dir: page

# Extensions
## Plugins: https://hexo.io/plugins/
## Themes: https://hexo.io/themes/
theme: 【这里填写主题名称，允许起别名，后面我会提醒你在哪里作修改。例如 material】

# Deployment
## Docs: https://hexo.io/docs/deployment.html
deploy:
type:
```

保存退出即可。

### 绑定个人域名（可选）
如果你有一个个人域名需要绑定，进入 `source/` 文件夹（默认情况下只有一个 source/_posts 的入口，很简单，先点进去，再返回上一级目录），新建一个 `CNAME` 的文件，在里面输入：
```
【你打算绑定的域名，如 www.bobby285271.top】
```

保存退出，然后参考 [这篇教程](http://www.cnblogs.com/penglei-it/p/hexo_domain_name.html) 完成后续设置。

### 写一篇博客
进入 `source/_posts` 文件夹，新建一个文档，命名为 `【文档名称，尽量统一只使用英文、数字和“-”】.md`，内容大概是这样的：
```
title: 【文章标题】
date: 【四位年份】-【两位月份】-【两位日期】
layout: post
tags: 
 - 【标签1】
 - 【标签2】
 - 【...】
categories:
 - 【分类父目录】
 - 【分类子目录】
 - 【...】
---

【从这里开始用 Markdown 写作】
```

如果您有过使用 WordPress 的经验，就很容易误解 Hexo 的分类方式。WordPress 支持对一篇文章设置多个分类，而且这些分类可以是同级的，也可以是父子分类。但是 Hexo 不支持指定多个同级分类。下面的指定方法：
```
categories:
- Diary
- Life
```

会使分类“Life”成为“Diary”的子分类，而不是并列分类。因此，有必要为您的文章选择尽可能准确的分类。

### 建立一个独立页面
和写博客差不多，进入 `source/` 文件夹，新建一个文档，命名为 `【填一个关键词，如 about】/index.md`，这个网页就会在 `【你的 GitHub 用户名】.github.io/【你填的关键词】` 处显示，内容大概是这样的：
```
title: 【页面标题】
date: 【可以空着，也可以仿照前面的填写】
layout: 【一般是 page，部分主题还提供其它的 layout，请以主题的文档为准】
---

【从这里开始用 Markdown 写作】
```

## 主题配置
主题太多，请参考主题提供的文档配置在前面 Fork 好的仓库，注意站点配置文件和主题配置文件都叫 `_config.yml`，但两个所在的仓库不同，功能也完全不同。

如果你也选择了 hexo-theme-material，可以阅读 [官方文档](https://material.viosey.com/docs)，注意：
* 如果你想建立友情链接页、图库页、标签云页、时间轴页，请不要按照前面一章的内容做，而是按照 [官方文档](https://material.viosey.com/docs/#/pages) 的指示做，这样子做出来的页面好看很多。但注意，填写内容时第一行的 `---` 请删掉，否则可能会出现各种问题；
* 用个人惨痛的经历提醒各位：编辑站点配置文件时不要画蛇添足，如在不应该敲空格的地方敲空格（半角冒号后面一定要加一个空格），另外尽量避免负责粘贴！

## 胜利的曙光
### 构建博客的 Travis CI 配置文件
这回又是配置 Travis CI，注意上次配置的文件是一次性的，你应该找不到它才对。
返回【你的 GitHub 用户名】.github.io 仓库，检查你所在的分支是否为 blog-source，若不是按照前面所讲修改默认分支，新建文件，命名为 `.travis.yml`（请勿改名），然后填写以下内容：
```
language: node_js
node_js: stable

install:
  - npm install

script:
  - git submodule add https://github.com/bobby285271/【主题在 GitHub 仓库的仓库名，如 hexo-theme-material】.git themes/【改成你在站点配置文件中填写的主题别称，如 material】
  - hexo g

after_script:
  - cd ./public
  - git init
  - git config user.name "【你的大名，如 Bobby Rong】"
  - git config user.email "【你的邮箱】"
  - git add -A
  - git commit -m "【提交信息，如 Update files】"
  - git push --force --quiet "https://${githubblog}@${GH_REF}" master

branches:
  only:
    - blog-source
    
env:
 global:
  - GH_REF: github.com/【你的 GitHub 用户名】/【你的 GitHub 用户名】.github.io.git
```

作相应替换后点击“Commit new file”即可。和之前的配置文件看起来差不多的，解释一下：
* `git submodule add https://github.com/【你的 GitHub 用户名】/【主题在 GitHub 仓库的仓库名，如 hexo-theme-material】.git themes/【改成你在站点配置文件中填写的主题别称，如 material】`：将你所 Fork 的主题仓库添加到站点仓库中；
* `hexo g`：生成网页；
* `cd ./public`：生成的网页都在这里面，我们只要里面的文件。

### 成功
回到 Travis CI，按照前面所讲查看自己的构建进度，只需仓库名旁边的图标变成了 Build passing（底色还是原谅绿），你就可以前往 `https://【你的 GitHub 用户名】.github.io` 或你绑定的域名查看你的网页了。至于后续修改，只需前往【你的 GitHub 用户名】.github.io 仓库，确认所在分支后编辑相应的文件就行了，详情请参考 Hexo 的 [官方文档](https://hexo.io/zh-cn/docs)，一旦仓库有变动，Travis 就会自动构建新的网页。

教程到此结束。