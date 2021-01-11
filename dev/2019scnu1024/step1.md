---
title: SCNUSE SoCoding 1024 Website: Step 1
description: 
published: true
date: 2021-01-11T05:23:49.722Z
tags: web, python, guide, html, django, scnu
editor: markdown
dateCreated: 2021-01-11T05:23:49.722Z
---

好久没有出文章了，尝试再造一遍 2019 年 SCNUSE 软协 1024 程序员节活动网站（尽量还原，会有一些差别），算是 Django 框架的一个学习项目吧，~~搞不好把题目换掉就是今年活动网站了~~。博主本人的基本功也不怎么扎实，下文用到了非常多不严谨的表述，如果有任何错误欢迎指出。

## 要实现什么
去年也是参与到这个活动中来了，但是只是做了一个关卡，比较划水。简单来说整个网站做的就是注册登录（这个没啥好说的）和闯关（七个关卡，根据游戏、图片、代码或文字提示要求用户找出 Flag 并输入，要保存用户进度和通关时间），去年最后具体实现用到的是 Django 框架，使用 MySQL 数据库。

由于博主快被 ACM 多校训练和团委项目两座大山压垮了，这里就略过需求分析，把注册登录、其中的三个关卡和通关页给做出来。页面的规划、数据表字段的规划等系统分析部分会在文章中穿插。博主使用的是 SQLite 数据库。

## 铺垫
博主认为有一些技能在开始动手之前还是要先掌握的。

首先是 Python，其实有了程序设计基础之后 Python 的语法其实很简单。完全不会 Python 的推荐看 [Python 100 天教程](https://github.com/jackfrued/Python-100-Days) 前 15 天部分的内容，然后找个 Online Judge 做几道水题，最多两个小时应该都能熟悉。

然后是 Web 前端 HTML、CSS、Javascript 的一些最基本的应用，学校讲的足够用了，这里推荐 [菜鸟教程](https://www.runoob.com/)，可以尝试自己写一些小的静态网站。

最后 Django 本身还是要先做了解的。可以简单浏览 [Python 100 天教程](https://github.com/jackfrued/Python-100-Days) 第 14 天和第 41 天的内容，另外也推荐阅读官方推荐的一些 [设计理念](https://docs.djangoproject.com/zh-hans/3.0/misc/design-philosophies/) 。

## 准备
先来做一些准备工作，包括开发环境的配置、项目和应用的初始化。开发环境可以按照自己的喜好来配置，不一定要按照这里的来；项目和应用的初始化有对应的具体的命令，执行即可。

### 开发环境
要做的事情就是安装一个对 Python 支持良好的 IDE 或者为你的代码编辑器添加 Python 支持，然后安装 Django。

假设你跟我一样已经是 Arch Linux + VS Code 用户。VS Code 下的 Python 支持是需要到 pip 的，pip 是 Python 的包管理器，直接使用 Linux 下的包管理器安装。

```
# pacman -S python-pip
```

pip 既然是包管理器，能条件反射到的是肯定要换源，我们在家目录创建并编辑 `.pip/pip.conf`：

```
$ cd 
$ mkdir .pip
$ nano .pip/pip.conf
```
```
[global]
index-url = https://pypi.tuna.tsinghua.edu.cn/simple
```

启动 VS Code，先装上 Python 扩展和 Visual Studio IntelliCode 扩展，然后打开任意 Python 文件，尝试格式化文档，VS Code 就会提示你安装格式化程序，安装就可。可以观察到实质就是调用 pip 安装 `autopep8`。

接下来安装 Django，这里有两个方法，一个是继续使用 Linux 下的包管理器安装：

```
# pacman -S python-django
```

或者是用上面刚刚讲到的 pip：

```
$ pip install django
```

要注意的是给 Arch Linux 软件包锁版本是不被提倡的，毕竟是滚动更新发行版。所以如果你使用 Pacman 安装 `python-django` 就得一直跟着系统走用最新版，要考虑其利弊。

接下来尝试使用 `django-admin` 命令，测试一下是不是一切都准备妥当了：

```
$ django-admin version
```

理想情况是输出一个版本号。如果提示命令没有找到，那就是在普通用户下使用 pip 按照 Django 导致 `django-admin` 不在 PATH 里。解决方法很多，例如改用 root 安装，或是修改 PATH，可以参考 [官方文档](https://docs.djangoproject.com/en/3.0/faq/troubleshooting/#troubleshooting-django-admin)。

### 创建项目
我们首先使用 `django-admin` 命令为站点创建一个已经初始化的项目目录，不妨叫 `mysite`。

> 如果你起的名字跟我的不一样，只要看到 `mysite` 改过来就行，另外不要用保留字命名，后面也是一样的道理。

```
$ django-admin startproject mysite
$ cd mysite
```

进入项目目录之后，应该能见到另一个 `mysite/` 目录和一个 `manage.py` 文件：

* `mysite/` 包含了 Django 项目实例需要的设置项集合，包括数据库配置、Django 配置和应用配置。
* `manage.py` 就是一个让你用各种方式管理 Django 项目的命令行工具。

我们约定，后面涉及的所有文件和文件夹均以当前所在的项目目录为基准标注**相对路径**。

### 创建应用
在 Django 中，每一个应用都是一个 Python 包。**应用**通常是一个专门做某件事的网络应用程序，比如博客系统、投票程序。**项目**则是一个网站使用的配置和应用的集合。一个项目可以包含很多个应用，但这里我们只创建一个，就叫 `myapp`：

```
$ python manage.py startapp myapp
```

接下来往项目添加这个应用。编辑 `mysite/settings.py`，找到 `INSTALLED_APPS`：

> 这里没有展示整个的完整的文件，只会展示需要改动的代码或是关键的代码，后面也是一样的道理。我们会在文末附上成品的代码，可以结合成品代码理解。

```python
INSTALLED_APPS = [
    'django.contrib.admin',
    'django.contrib.auth',
    'django.contrib.contenttypes',
    'django.contrib.sessions',
    'django.contrib.messages',
    'django.contrib.staticfiles',
    'myapp',
]
```

## 站点调试
我们可以利用 `manage.py` 运行 Django 自带的用于开发的简易 Web 服务器对我们的网站进行预览和调试。

> 不要将这个服务器用于和生产环境相关的任何地方，这个服务器只是为了开发而设计的。

```
$ python manage.py runserver
```

终端应该会输出类似这样的东西：

```
Starting development server at http://127.0.0.1:8000/
Quit the server with CONTROL-C.
```

按照上面的输出，我们打开浏览器访问 [http://127.0.0.1:8000/](http://127.0.0.1:8000/)，在下文中我们假设你的服务器设置和我的一样为监听本机内部 IP 的 `8000` 端口。在一个网页都没有完成的情况下，你应该能看到 Django 自带的测试页面。当我们写好我们的网站之后则应该能看到自己的网站，测试页面就不会再出现了。

现在我们尝试将测试页面改成中文的，其实是为 Django 设置语言。编辑 `mysite/settings.py`：

```python
LANGUAGE_CODE = 'zh-hans'
TIME_ZONE = 'Asia/Shanghai'
```

不必重新启动服务器，直接刷新页面，现在测试页面显示的是中文，你的改动生效了。通常来说服务器会对每一次的访问请求重新载入一遍 Python 代码，但是例外还是有的（如添加新文件）。如果你发现你的改动没有生效，尝试重启服务器就好了。

如果你想把服务器停掉，就按照终端的提示按下 Ctrl + C 组合键即可，和 Linux 杀死其它命令的方式是一样的。

## 第一个页面
几乎所有教程做第一个页面都是 Hello World，虽然这和 1024 站点没啥关联，我们也先实现一下，目标是访问站点首页后显示一行文字 `Hello World`。

博主认为虽然是 Hello World，但可能还是会有点难度。其实不需要纠结太多，关键是记住具体的实现，知道哪些地方是套路，哪些地方要改动。我们在后面会制作更多的页面，自然就会熟悉了。

### 响应与视图函数
**视图函数**（简称视图）是一个简单的 Python 函数，它接受 Web 请求并且返回 Web 响应。响应可以是 HTML、重定向、404、XML、图片等等。无论视图函数本身包含什么逻辑，都要返回响应。我们通常将视图函数放置在 `views.py` 文件中。

我们现在只需要在页面上显示上一行 `Hello World`，所以这个视图函数不需要做什么，直接返回响应就行。既然是首页，不妨将函数命名为 `index`，我们看看具体的实现。编辑 `myapp/views.py`：

```python
from django.shortcuts import render, redirect
from django.http import HttpResponse

def index(request):
    return HttpResponse("<h1>Hello, World</h1>")
```

视图函数就写好啦！这里我们用到了 `HttpResponse` 对象，你可以翻阅 [官方文档](https://docs.djangoproject.com/en/3.0/ref/request-response/#httpresponse-objects) 了解这个对象，并尝试将 `<h1>Hello, World</h1>` 改成其它内容。

### URL 的映射
接下来我们建立 URL（统一资源定位符）与视图函数 `index` 的映射。具体可以分成两步，第一步是在应用下面处理自己的映射关系，第二步是进行路由分发。

博主认为这可以理解为一个相对路径的确定，当然这里的路径指的是我们访问的 URL，和项目文件的放置没有关联。第一步实际上就是将应用根目录的绝对路径作为基准，确定各个网页的相对路径。第二步就是将项目根目录的绝对路径作为基准，确定各个应用根目录的相对路径。

现在我们想在首页显示 Hello World，我们来看看具体的实现。

先来做第一步，在 `myapp` 下面新建一个 `urls.py`：

```python
from django.urls import path
from . import views

urlpatterns = [
    path('', views.index, name='index'),
]
```

`path('', views.index, name='index')` 可以理解为这个网页就位于应用的根目录，响应的是 `views.py` 文件里的 `index` 函数。而最后的 `name='index'` 则是指定 `name` 属性，这里的 `name` 和 HTML 下的 `id` 属性是类似的，必须要唯一。我们会在后面用到它。

再来做第二步，编辑 `mysite/urls.py`：

```python
from django.contrib import admin
from django.urls import include, path

urlpatterns = [
    path('', include('myapp.urls')),
    path('admin/', admin.site.urls),
]
```

`path('', include('myapp.urls'))` 可以理解为 `myapp` 这个应用的根目录就位于项目的根目录，`path('admin/', admin.site.urls)` 为 Django 自带的管理页面，可以暂时不用理会，我们会在后面用到它。

当然了这样的理解自然是不严谨的，还是建议大家深入了解 URL 和路由，详情可以查阅 [官方文档](https://docs.djangoproject.com/en/3.0/topics/http/urls/)。

现在使用 `python manage.py runserver` 调试，访问 [http://127.0.0.1:8000/](http://127.0.0.1:8000/)，显示的是 Hello World 页了。你可以尝试再对 `myapp/urls.py` 和 `mysite/urls.py` 进行编辑，看看 Hello World 页是否能在你期望的位置出现。

### 更多的页面
有了 Hello World 页的铺垫之后，我们就可以愉快地进行页面规划了。

我们需要做的实际上只有四个页面，一个是注册页（`register`），一个是登录页（`login`），一个是闯关页（`play`），还有一个通关页（`win`）。现在我们不需要把他们实际的功能做出来，而是做四个 Hello World 页，这时为了避免因为「挖了坑没填完」而导致的站点调试时报错。

现在我们就来仿照上面 Hello World 页的例子，初始化注册页、登录页、闯关页和通关页。具体来说，我们为他们分别写一个临时的视图函数并建立好 URL 映射。我们后面每一章都会展开说明一个页面具体的实际的实现，届时再对我们的视图函数做进一步的修改，也有可能因为处理表单的需要继续创建页面。



首先我们写视图函数，编辑 `myapp/views.py`：

```python
def register(request):
    return HttpResponse("<p>这是注册页</p>")

def login(request):
    return HttpResponse("<p>这是登录页</p>")

def play(request):
    return HttpResponse("<p>这是闯关页</p>")

def win(request):
    return HttpResponse("<p>这是通关页</p>")
```

接下来我们建立 URL 映射，编辑 `myapp/urls.py`：

```python
urlpatterns = [
    path('', views.index, name='index'),
    path('register/', views.register, name='register'),
    path('login/', views.login, name='login'),
    path('play/', views.play, name='play'),
    path('win/', views.win, name='win'),
]
```

路由分发我们在做 Hello World 页的时候已经做好了，无需再做改动。



现在使用 `python manage.py runserver` 调试，访问 [http://127.0.0.1:8000/register/](http://127.0.0.1:8000/register/) 查看注册页目前的效果。然后尝试访问登录页、关卡页和通关页。