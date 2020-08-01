---
title: Introduction to Django - Illustrated by the Example of SCNUSE SoCoding 1024 Website
description: 
published: true
date: 2020-08-01T02:09:43.780Z
tags: web
editor: markdown
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

## 注册页的实现
现在访问注册页，显示的只有五个字「这是注册页」，但我们实际需要的是：前端一个表单供用户输入账户密码，后端处理用户输入与数据库交互。接下来，我们会根据实际的需求进一步改造这个注册页，使其达到我们所要的效果。

### 模板
首先我们改造注册页的前端部分，刚刚学完 Web 前端，应该不是什么难点。现在的问题讲通俗点就是，怎样做到我访问这个地址就给我渲染指定的 HTML 文件呢？

一个方法是，我把这个 HTML 文件的内容压成一行，换掉 `<p>这是注册页</p>` 。可以是可以，但是一个很最突出的问题是维护起来很难受。事实上这样子其实是将数据与视图混合在一起，我们说是不符合 Django 的 MVC 思想（Model View Controller，具体请百度）。于是我们有了另外一个东西叫做**模板**。我们可以在 [官方文档 How-to 部分](https://docs.djangoproject.com/en/3.0/howto/overriding-templates/) 找到相关的文档。

目前而言，你可以简单地把模板跟 HTML 文件划上等号，我们后面会介绍 Django 标签，届时你将会进一步了解模板的神奇之处。模板是有专门地方存放的，在 `myapp/` 下面建立一个 `templates/` 文件夹。接下来往里面放你的 HTML 文件。

例如我把注册页命名为 `register.html`，也就是说文件的相对路径为 `myapp/templates/register.html`。就和初学 Web 前端一样，先不要管样式，也不要管按钮链接能不能工作，这些在后面会提及。

为了节省时间，这里就直接开抄了。以下的代码为去年的代码改动而来：



```html
<!DOCTYPE html>
<html>

<head>
    <meta charset="utf-8">
    <title>Register</title>
</head>

<body>
    <div class="lg_content">
        <table cellspacing="30px" align="center">
            <form method="post" action="#">
                <tr>
                    <td colspan="2">
                        <input type="text" name="userid" class="reg_ip" placeholder="学号">
                    </td>
                </tr>
                <tr>
                    <td colspan="2">
                        <input type="text" name="username" class="reg_ip" placeholder="昵称">
                    </td>
                </tr>
                <tr>
                    <td colspan="2">
                        <input type="password" name="passwd" class="reg_ip" placeholder="密码">
                    </td>
                </tr>
                <tr>
                    <td colspan="2">
                        <input type="password" name="repasswd" class="reg_ip" placeholder="确认密码">
                    </td>
                </tr>

                <tr>
                    <td align="left">

                        <input type="submit" value="注册" style="position:relative;left:40px;" class="button">

                    </td>
                </tr>
            </form>
            <td align="center" style="position:relative;left:-50px">
                <span class="font1">已经注册？</span>
                <input type="submit" class="button" style="height:25px;width:80px;font-size:12px" value="直接登录"
                    onclick="location.href='#'">
            </td>
        </table>
    </div>
</body>

</html>
```



接下来我们把「注册页」三个字换掉，这里我们使用的是 `render` 方法，它可接收三个参数，一是 `request` 参数，二是待渲染的 HTML 模板文件，三是保存具体数据的字典参数。它的作用就是将数据填充进模板文件，最后把结果返回给浏览器。编辑 `myapp/views.py`：

```python
def register(request):
    return render(request, 'register.html')
```

我们再访问 [http://127.0.0.1:8000/register/](http://127.0.0.1:8000/register/)，发现已经有一个注册页面的样子了。

### 静态资源
现在我们来做注册页的样式吧！你们可能尝试过在 `myapp/templates/` 直接放一个 CSS 文件，然后在 `myapp/templates/register.html` 直接加 `<link href="style.css" rel="stylesheet">`，你会发现是行不通的。调试的时候，你会发现它实际请求的是 [http://127.0.0.1:8000/register/style.css](http://127.0.0.1:8000/register/style.css)，直接 404。

注意到 `mysite/settings.py` 有这么一行：

```python
# Static files (CSS, JavaScript, Images)
# https://docs.djangoproject.com/en/3.0/howto/static-files/

STATIC_URL = '/static/'
```

事实上，关于静态资源的存放，[官方文档 How-to 部分](https://docs.djangoproject.com/zh-hans/3.0/howto/static-files/) 依然给出了具体的说明。和模板类似，我们要单独建立 `static/` 目录来存放这些静态资源。通常我们把这个目录放在应用目录下，我们 `myapp/` 下新建 `static/` 目录，里面再建一个 `css/` 目录。

接下来，我们尝试往 `css/` 目录里面添加我们的样式。这个样式我们会在后面多次使用，编辑 `myapp/static/css/style.css`，这里我们继续偷懒开抄：



```css
body {
    /* background: url("../img/bg_img.jpg"); */
    /* background-size: cover; */
    margin: 0px;
    padding: 0px;
    text-align: center;
}

.containerDiv {
    margin-top: 0;
    margin-bottom: 0;
    background-size: 100% 100%;
    width: 100%;
    text-align: center;
}

.lg_content {
    margin-top: 100px;
}

.titleDiv {
    width: 100%;
    height: 150px;
}

.problemDiv {
    width: 80%;
    height: 360px;
    margin-left: 10%;
    margin-right: 10%;
    position: relative;
}

.leftDiv {
    width: 50%;
    height: 360px;
    text-align: center;
    float: left;
}

.rightDiv {
    width: 40%;
    margin-left: 10%;
    height: 360px;
    text-align: center;
    float: left;
}

.answerDiv {
    width: 100%;
    height: 60px;
    position: relative;
}

.background {
    background-size: 100% 100%;
    width: 100%;
    height: 100%;
}

.button {
    width: 68px;
    height: 34px;
    font-size: 18px;
    color: rgb(255, 255, 255);
    background-color: rgb(76, 129, 179);
    box-shadow: none;
    border: none;
    border-style: inline;
}

.input {
    height: 40px;
    width: 250px;
    font-size: 14px;
    color: rgba(155, 152, 151, 0.774);
    border: none;
    box-shadow: none;
    border-style: none;
    border-width: 0px;
}

.font1 {
    font-family: "Microsoft YaHei";
    font-size: 12px;
    font-weight: bold;
    color: azure;
}

.font2 {
    font-family: "Microsoft YaHei";
    font-size: 22px;
    width: 400px;
    color: rgb(88, 99, 192);
}

#game_div,
p {
    width: 400px;
    margin: auto;
    margin-top: 0px;
}
```



根据 `STATIC_URL = '/static/'` 这一线索，我们尝试访问 [http://127.0.0.1:8000/static/css/style.css](http://127.0.0.1:8000/static/css/style.css)，发现确实是有这个文件的。

事实上，只要将 `<link href="style.css" rel="stylesheet">` 改成 `<link href="/static/css/style.css" rel="stylesheet">` 就可以往页面加载样式了。

**但是**，这样后续维护是很困难的。

一个问题是如果我们后面突然又需要创建应用，需要的是不同的样式呢？想象一下，你有两个应用 `myapp` 和 `yourapp`，然后你在 `myapp/static/css/style.css` 和 `yourapp/static/css/style.css` 放了不同的样式文件，你访问 [http://127.0.0.1:8000/static/css/style.css](http://127.0.0.1:8000/static/css/style.css) 得到的是什么呢？

对这个问题，官方的解决方案是，在 `myapp/static/` 下面再建立一个目录 `myapp`（就是应用本身的名称），然后把静态资源放里面，我们尝试将 `myapp/static/css` 目录移动到 `myapp/static/myapp/css`。现在我们在 [http://127.0.0.1:8000/static/myapp/css/style.css](http://127.0.0.1:8000/static/myapp/css/style.css) 可以看到我们的样式文件了。

另一个问题是如果我突然想更换一下映射到静态资源的 URL 呢？也就是说我把 `STATIC_URL = '/static/'` 改成别的，是不是所有的 HTML 文件我要逐个修改呢？

所以我们是不建议按照前面的方式去写的，我们有一个船新的方法。编辑 `myapp/templates/register.html`：

```html
<head>
    <meta charset="utf-8">
    <title>register</title>
    {% load static %}
    <link href="{%static 'myapp/css/style.css'%}" rel="stylesheet">
</head>
```

要注意的是 `{% load static %}` 和 `{%static 'myapp/css/style.css'%}` 的 `static` 和 `STATIC_URL = '/static/'` 中的路径 `static` 是没有任何联系的，就是静态资源的意思。你可以尝试将 `STATIC_URL` 改成别的再来试试。

现在访问 [http://127.0.0.1:8000/register/](http://127.0.0.1:8000/register/)，F12 看一下，会发现上面 `{% %}` 标签的内容都已经被替换掉了。

接下来是背景图片。同样地，我们将 `bg_img.jpg` 放置在 `myapp/static/myapp/img/` 目录中。这回没那么复杂，在 CSS 文件里引用时直接写相对路径即可，因为无论怎么修改 `STATIC_URL` 都不会对其产生影响：

```css
body {
    background: url("../img/bg_img.jpg");
    background-size: cover;
    margin: 0px;
    padding: 0px;
    text-align: center;
}
```

好了，现在注册页看上去跟去年是一个样子了。

### 数据库与模型
来到后端部分了！后端部分是离不开数据库的，我们首先把数据库配置好。这里我们使用的是 SQLite 数据库，为什么在这个项目使用 SQLite 呢？

* 轻量，无客户端和服务器端之分，跨平台，单文件。
* 我们现在做的网站也只是供我们自娱自乐的而已，没有高并发。
* Python 内置 SQLite，所以你无需安装额外东西来使用它。
* 不需要事先创建数据库，数据库会在需要的时候自动创建。
* Django 默认的数据库就是 SQLite，我们查看 `mysite/settings.py`：

```python
DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.sqlite3',
        'NAME': os.path.join(BASE_DIR, 'db.sqlite3'),
    }
}
```

让我们开始吧！首先我们对设定的数据库进行初始化：

```
$ python manage.py migrate
```

所谓的初始化实际上是**迁移**操作，为在 `mysite/settings.py` 涉及到的 `INSTALLED_APPS` 创建一些数据表。这里先不需要去纠结具体是做什么的，有兴趣的可以查看 [官方文档](https://docs.djangoproject.com/zh-hans/3.0/intro/tutorial02/#database-setup)。我们在后面还会使用到这个命令，到时再做说明。

接下来我们进行数据表字段的规划。在 Django，我们称数据库结构设计为**模型**。你可以把模型类比为数据库里的数据表，在这里我们只需要为学生这个对象创建一个模型，不妨将它命名为 `STUDB`。

下面我们就来实现这个模型，博主认为整个流程和 Git 提交代码的流程非常类似。在 Git 我们并不会在版本库直接改动代码，而是：

* 第一步，你在工作区对文件作出改动，你的改动并不会影响到版本库。
* 第二步，`git add`：你的改动从工作区被转移到了暂存区。
* 第三步，`git commit`：你的改动从暂存区被转移到了版本库。

在这个项目，我们的第一步就是编辑 `myapp/models.py` 文件，在文件记录我们需要的模型。

作为我们需要建立的唯一一个模型，我们要为其添加的字段有：

| 字段名 | 数据类型 | 作用 | 备注 |
|---|---|---|---|
| `userid` | 文本 `TextField` | 学号 | 最大长度为 11 `max_length=11` |
| `username`  | 文本 `TextField` | 昵称 | 最大长度为 10 `max_length=10` |
| `passwd` | 文本 `TextField` | 密码 | 最大长度为 20 `max_length=20` |
| `firstflag`  | 时间 `DateTimeField` | 普通关卡开始时间 | 默认是当前时间 |
| `lastflag`   | 时间 `DateTimeField` | 普通关卡通关时间 | 默认是当前时间 |
| `superflag`  | 时间 `DateTimeField` | 最后一关通关时间 | 默认是当前时间 |
| `specialflag` | 时间 `DateTimeField` | 彩蛋通关时间 | 默认是当前时间 |
| `rank`  | 整数 `IntegerField` | 通关记录 | 默认是 1  |
| `timesubtract` | 整数 `IntegerField`  | 普通关卡时间差  | 默认是 0 |
| `timesubtract_last` | 整数 `IntegerField`  | 最后一关时间差 | 默认是 0 |
| `timesubtract_suprise` | 整数 `IntegerField`  | 彩蛋时间差 | 默认是 0 |

我们可以在 [官方文档](https://docs.djangoproject.com/en/3.0/ref/models/fields/#textfield) 找到可能需要用到的数据类型。

接下来我们将表格转为代码。在 `myapp/models.py` 在这个文件中，每个模型被表示为一个**类**。每个模型有许多类变量，它们都表示模型里的一个数据库字段。

我们可以在 [官方文档](https://docs.djangoproject.com/zh-hans/3.0/intro/tutorial02/#creating-models) 找到很多代码实现的例子。



```python
from django.db import models
import django.utils.timezone as timezone

class STUDB(models.Model):
    userid = models.TextField(max_length=11)
    username = models.TextField(max_length=10)
    passwd = models.TextField(max_length=20)
    firstflag = models.DateTimeField(default=timezone.now)
    lastflag = models.DateTimeField(default=timezone.now)
    superflag = models.DateTimeField(default=timezone.now)
    specialflag = models.DateTimeField(default=timezone.now)
    rank = models.IntegerField(default=1)
    timesubtract = models.IntegerField(default=0)
    timesubtract_last = models.IntegerField(default=0)
    timesubtract_suprise = models.IntegerField(default=0)
```



接下来是第二步，我们让 Django 检测你对模型文件的修改，并且把修改的部分储存为一次迁移。

```python
$ python manage.py makemigrations
```

最后是第三步，在数据库里创建新定义的模型的数据表。

```
$ python manage.py migrate
```

### 表单处理与数据库交互
既然数据库已经配置完毕，接下来我们要做的是读取用户的输入并写入数据库。

怎样获取和处理用户的输入呢？Web 前端曾经讲过怎样用 Javascript 处理 GET 请求和 POST 请求，这里也是类似的，甚至还简单一些。

首先还是要回到前端，我们已经写了一个 form 表单，目前我们的 `form` 标签是这样的：

```html
<form method="post" action="#"></form>
```

我们需要将 `action` 属性补充完整。为此我们需要另外制作一个「页面」用来接收表单的信息，这个页面不一定要给用户看到些什么。我们在这个页面处理完用户的信息后就会跳转到相应的页面。我们把这个页面命名为 `register_to_db`。

我们来写一下视图函数，这回我们需要进行一些逻辑上的处理了，所以这个函数注定要写的又臭又长，我们拆分一下我们要实现的逻辑。

首先，我们要接收表单发过来的 POST 请求。在 Web 前端课程上，我们已经知道前端表单的 `name` 属性会被用于对提交到服务器后的表单数据进行标识。所以我们现在要接收的，就是 `userid`、`username`、`passwd` 和 `repasswd`。

```python
userid = request.POST.get('userid', None)
username = request.POST.get('username', None)
passwd = request.POST.get('passwd', None)
repasswd = request.POST.get('repasswd', None)
```

接下来，我们看看学号是否已经被注册，如果被注册了，我们就跳转回注册页。从这里开始，我们就要用到数据库的增删改查操作了，然而我们并不需要写 SQL 语句，因为 Django 模型使用自带的对象关系映射（ORM）连接业务逻辑层和数据库层。简单来说，ORM 会将 Python 代码转成为 SQL 语句。

[菜鸟教程](https://www.runoob.com/django/django-model.html) 提供了数据库增删改查实现的不少例子，可供参考。

我们也来照葫芦画瓢，首先导入我们的模板，接下来我们使用 `filter` 方法查询学号信息，返回一个列表：

```python
from .models import STUDB
users = STUDB.objects.filter(userid=userid)
```

如果这个列表不为空，意味着这个学号已经被注册，我们就跳转回注册页。怎样实现跳转呢，可以想到的第一个方法是直接使用之前讲过的 `render` 方法：

```python
if len(users) == 1:
    return render(request, 'register.html')
```

会发现确实是重新「显示」注册页了，但是我们看 URL 并没有变，依然是 [http://127.0.0.1:8000/register_to_db/](http://127.0.0.1:8000/register_to_db/)。我们不要忘了 `register.html` 只是一个模板而已。但是我们希望的是跳转到 [http://127.0.0.1:8000/register/](http://127.0.0.1:8000/register/)。这里我们就要介绍到我们的 `redirect` 方法了，它专为重定向而生，我们看看它最基本的使用方法：

```python
if len(users) == 1:
    return redirect("http://127.0.0.1:8000/register/")
```

和我们学 Web 前端接触的 `a` 标签的 `href` 属性几乎一样，填上 URL 就行。当然，上面这样填写绝对路径是不利于后续维护的。哪怕改成相对路径 `redirect("../register/")`，由于我们可以随时修改 `myapp/urls.py` 中的映射关系，事实上也不是最优方法。事实上对于所有的模板，我们都应该避免硬编码链接硬编码。

还记得我们刚开始编辑 `myapp/urls.py` 时提到的 `name` 吗？

```python
path('register/', views.register, name='register')
```

现在是将它派上用场的好机会。我们使用 `reverse` 方法，填上注册页的 `name` 参数：

```python
from django.shortcuts import render, reverse, redirect
if len(users) == 1:
    return redirect(reverse('register'))
```

接下来我们再进一步，我们希望跳转过去之后弹出一条提示。我们知道 GET 的参数就是在 URL 后面跟一个 `?`，然后是参数列表。不妨就利用 GET 请求通知注册页弹窗，我们这里就用一个参数 `code`，当然参数名可以自定。具体的弹窗实现我们稍后再提及：

```python
from django.shortcuts import render, reverse, redirect
if len(users) == 1:
    return redirect(reverse('register')+"?code=-1")
```

我们尝试实现另外的几个逻辑，首先是判断输入是否齐全，然后是判断学号是否为 11 位且开头为 `201`，然后是判断密码是否是 6-18 位且必须是英文字母或数字，最后判断两次输入的密码是否一致。这里需要使用到正则表达式。



```python
import re

# 判断是否为空
if not userid or not passwd or not username or not repasswd:
    return redirect(reverse('register')+"?code=-5")

# 判断学号格式
if not re.match(r'^(201).*\d{8}$', userid):
    return redirect(reverse('register')+"?code=-4")

# 判断密码是否为 6-18 位且为字母或数字
if not re.match(r'^[A-Za-z0-9]{6,18}$', passwd):
    return redirect(reverse('register')+"?code=-3")

# 判断两次密码是否一致
if passwd != repasswd:
    return redirect(reverse('register')+"?code=-2")
```



如果没啥问题，我们就可以写入数据库了，然后跳转到登录页。现在我们用到的是 `save` 函数。这里博主再次给出 [菜鸟教程](https://www.runoob.com/django/django-model.html) 的链接。

```python
STUDBdent = STUDB.objects.create(
    userid=userid, username=username, passwd=passwd)
STUDBdent.save()
return redirect(reverse('login')+"?code=1")
```

视图函数就写好了，我们再来检查一下我们刚才写的东西：



```python
from django.shortcuts import render, reverse, redirect
from django.http import HttpResponse
import re
from .models import STUDB

def register_to_db(request):
    userid = request.POST.get('userid', None)
    username = request.POST.get('username', None)
    passwd = request.POST.get('passwd', None)
    repasswd = request.POST.get('repasswd', None)
    if not userid or not passwd or not username or not repasswd:
        return redirect(reverse('register')+"?code=-5")
    if not re.match(r'^(201).*\d{8}$', userid):
        return redirect(reverse('register')+"?code=-4")
    if not re.match(r'^[A-Za-z0-9]{6,18}$', passwd):
        return redirect(reverse('register')+"?code=-3")
    if passwd != repasswd:
        return redirect(reverse('register')+"?code=-2")
    users = STUDB.objects.filter(userid=userid)
    if len(users) == 1:
        return redirect(reverse('register')+"?code=-1")
    STUDBdent = STUDB.objects.create(
        userid=userid, username=username, passwd=passwd)
    STUDBdent.save()
    return redirect(reverse('login')+"?code=1")
```



接下来别忘了把 `register_to_db` 页的映射关系写好：



编辑 `myapp/urls.py`：

```
urlpatterns = [
    path('', views.index, name='index'),
    path('register/', views.register, name='register'),
    path('login/', views.login, name='login'),
    path('play/', views.play, name='play'),
    path('win/', views.win, name='win'),
    path('register_to_db/', views.register_to_db, name='register_to_db'),
]
```



接下来，我们回到前端的表单，我们把这个 `form` 标签给鸽了很久了。

```html
<form method="post" action="#"></form>
```

首先是 `action` 属性。在后端，我们使用的是 `reverse` 方法，回忆一下？

```python
return redirect(reverse('login')+"?code=1")
```

在模板这里，我们可以使用 `{% url %}` 标签。我们曾经讲过 `{% static %}` 标签，回忆一下？

```html
{% load static %}
<link href="{%static 'myapp/css/style.css'%}" rel="stylesheet">
```

下面我们就来看看 `{% url %}` 最简单的用法，就是带上页面的 `name` 参数：

```html
<form method="post" action="{% url 'register_to_db' %}"></form>
```

我们看看另一个例子，知道注册页底部有一个按钮是链接到登录页的：

```html
<input type="submit" class="button" style="height:25px;width:80px;font-size:12px" value="直接登录" onclick="location.href='#'">
```

我们来实现一下它的跳转功能吧，使用 `{% url %}` 标签：



```html
<input type="submit" class="button" style="height:25px;width:80px;font-size:12px" value="直接登录" onclick="location.href='{% url 'login' %}'">
```



由于我们创建一个 POST 表单（它具有修改数据的作用），所以我们需要小心跨站点请求伪造。Django 自带了一个非常有用的防御系统。所有针对内部 URL 的 POST 表单都应该使用 `{% csrf_token %}` 模板标签，这里就没有什么用法了，在 `form` 标签内添加一行 `{% csrf_token %}` 即可。

我们来看看现在的 `myapp/templates/register.html` 是什么样子的：



```html
<!DOCTYPE html>
<html>

<head>
    <meta charset="utf-8">
    <title>Register</title>
    {% load static %}
    <link href="{% static 'myapp/css/style.css' %}" rel="stylesheet">
</head>

<body>
    <div class="lg_content">
        <table cellspacing="30px" align="center">
            <form method="post" action="{% url 'register_to_db' %}">
                {% csrf_token %}
                <tr>
                    <td colspan="2">
                        <input type="text" name="userid" class="reg_ip" placeholder="学号">
                    </td>
                </tr>
                <tr>
                    <td colspan="2">
                        <input type="text" name="username" class="reg_ip" placeholder="昵称">
                    </td>
                </tr>
                <tr>
                    <td colspan="2">
                        <input type="password" name="passwd" class="reg_ip" placeholder="密码">
                    </td>
                </tr>
                <tr>
                    <td colspan="2">
                        <input type="password" name="repasswd" class="reg_ip" placeholder="确认密码">
                    </td>
                </tr>

                <tr>
                    <td align="left">

                        <input type="submit" value="注册" style="position:relative;left:40px;" class="button">

                    </td>
                </tr>
            </form>
            <td align="center" style="position:relative;left:-50px">
                <span class="font1">已经注册？</span>
                <input type="submit" class="button" style="height:25px;width:80px;font-size:12px" value="直接登录"
                    onclick="location.href='{% url 'login' %}'">
            </td>
        </table>
    </div>
</body>

</html>

```



### 消息框架
现在我们来实现注册页的最后一部分内容，也就是弹窗提示。由于是根据 GET 请求中的 `code` 参数的值弹出响应的内容，我们在 Web 前端已经学过怎样用 Javascript 来做这件事了。但是 Django 提供了另外一个实现的思路，也就是消息框架。

具体实现分为两步，一步是添加消息，另一步是显示消息。

首先是添加信息，我们使用到的是 `add_message` 方法，例如我们要添加一条 Hello World 信息：

```python
from django.contrib import messages
messages.add_message(request, messages.INFO, 'Hello World')
```

但是我们通常不按照上面的方式写，因为有另外的更简单的写法：

```python
from django.contrib import messages
messages.info(request, 'Hello World')
```

在这里 `Hello World` 就是通知的内容了，`info` 指的是通知的级别。你可以将它换成 `success`、`warning`、`error`、`debug` 中的一种。至于通知级别的用途，简单来说就是用于添加 HTML 标签的类，[官方文档](https://docs.djangoproject.com/en/3.0/ref/contrib/messages/#using-messages-in-views-and-templates) 有相关的例子，当然我们这里还不需要到。

回到项目，我们看看怎样根据 `code` 参数添加响应的信息，编辑 `myapp/views.py`，我们开始修改我们的 `register` 视图函数。

首先是获取 `code` 参数，回忆一下我们是怎样获取用户的表单信息的，注意这里使用的是 GET 请求。



```python
code = request.GET.get('code', None)
```



接下来我们来添加信息，如果不记得 `code` 值对应的含义的话，可以回看一下 `register_to_db` 视图函数。



```python
if code == '-5':
    messages.error(request, "不要没填完就注册啊喂！")
elif code == '-4':
    messages.error(request, "学 号 格 式 有 误")
elif code == '-3':
    messages.error(request, "密码需要 6-18 位且必须是英文字母或数字")
elif code == '-2':
    messages.error(request, "两次密码输入不一致的说")
elif code == '-1':
    messages.error(request, "这个学号已被注册！请不要试图顶替别人(｡•ˇ‸ˇ•｡)")
```



现在我们的 `register` 函数也被改得又臭又长了，看看你的函数现在是个什么样子？



别忘了导入 `messages`。

```python
from django.contrib import messages

def register(request):
    code = request.GET.get('code', None)
    if code == '-5':
        messages.error(request, "不要没填完就注册啊喂！")
    elif code == '-4':
        messages.error(request, "学 号 格 式 有 误")
    elif code == '-3':
        messages.error(request, "密码需要 6-18 位且必须是英文字母或数字")
    elif code == '-2':
        messages.error(request, "两次密码输入不一致的说")
    elif code == '-1':
        messages.error(request, "这个学号已被注册！请不要试图顶替别人(｡•ˇ‸ˇ•｡)")
    return render(request, 'register.html')
```



接下来是第二步，就是显示消息。

之前讲过 `{% url %}` 和 `{% static %}`，我们接下来要使用更多的 Django 标签，包括 `{% if %}`、`{% endif %}` 和 `{% for %}`、`{% endfor %}` 标签。我们可以很快地猜出它们的作用，前两个用于判断，后两个用于循环。

我们考虑我们现在要实现的逻辑，就是如果有消息 `messages`（不能改为别的名称），我们就遍历这些消息，对于每一个消息 `msg`（可以改为别的名称），我们执行 `alert()` 方法。

我们来看一下代码实现，编辑 `myapp/templates/register.html`。

```html
{% if messages %}
<script>
    {% for msg in messages %}
    alert('{{msg.message}}')
    {% endfor %}
</script>
{% endif %}
```

其实是很套路化的，在其它需要到消息的场合代码几乎是一样的，无非是把 `script` 标签换掉，把 `alert` 方法换掉。

到这里注册页的实现已经全部完成啦！

### Admin 管理工具
让我们来看看刚才写的这么多东西到底起作用了没有。简单来说，就是模拟一下注册流程，看看是不是真的有东西写进数据表里面。我们就需要一个可视化的数据库管理工具。MySQL 下有~~死掉了的~~ PHPMyAdmin 和它的继任者 Adminer，那么 SQLite 呢？

没错我们终于解锁了讲解「URL 的映射」提到过的神秘的管理工具 [http://127.0.0.1:8000/admin/](http://127.0.0.1:8000/admin/)。然而直接访问这个页面需要账户密码。

我们就创建一个超级用户：

```
$ python manage.py createsuperuser
```

输入你希望设定的账户、邮箱和密码。然后用这个帐号登录这个管理工具。似乎并没有显示我们的模型 `STUDB`。我们把将这个模型添加进管理工具里面，编辑 `myapp/admin.py`：

```python
from myapp.models import STUDB
admin.site.register(STUDB),
```

现在管理页面应该能看到我们的模型，但是当我们去尝试注册几个帐号后，在管理页点击我们的模型查看具体的数据的时候（应该是 [http://127.0.0.1:8000/admin/myapp/studb/](http://127.0.0.1:8000/admin/myapp/studb/) 这个页面），显示的是一个个的 `STUDB object`，我们希望它能显示学号。

事实上是有解决方法的，我们编辑 `myapp/models.py`，在我们的模型类 `STUDB` 末尾加入这两行：

```python
def __str__(self):
    return self.userid
```

方法名如果是 `__xxxx__()` 的，那么就有特殊的功能，因此叫做「魔法」方法。当使用 `print` 输出对象的时候，只要自己定义了 `__str__(self)` 方法，那么就会打印从在这个方法中 `return` 的数据，我们通常让它返回一个对象的描述信息。

刷新页面，这时显示的就是学号了。你还可以尝试将 `self.userid` 换成其它内容。接下来，让我们愉快地开始注册吧（大雾）。

## 登录页的实现
可以说接下来我们要做的就是另一个注册页，几乎是一样的。由于基本的套路都在刚才涉及到了，在阅读前面的部分时候，你也已经做了一部分了，这一部分我们节奏会稍微加快一些。

### 前端部分
我们首先实现的依然是模板。在实现注册页时，我们在讲「模板」、「表单处理」和「消息框架」时都涉及到了模板的编辑。博主认为你读到这里的时候应该把前面讲的忘的差不多了，所以给出几个提示：

* 模板就是 `myapp/templates/` 里的那些 HTML 文件。
* 样式在制作注册页的时候就已经写好了，所以在这里你只需要引用它就好了。
* 尽量使用 `{% url %}` 标签替代实际的链接，减少硬编码。
* POST 表单都应该使用 `{% csrf_token %}` 模板标签。
* 别忘了如果有信息 `messages` 要 `alert` 弹窗显示。



新建并编辑 `myapp/templates/login.html`：

```html
<!DOCTYPE html>
<html>

<head>
    <meta charset="utf-8">
    <title>Login</title>
    {% load static %}
    <link href="{%static 'myapp/css/style.css'%}" rel="stylesheet">
</head>

<body>
    <div class="lg_content">
        <table cellspacing="30px" style="width: 250px" align="center">
            <tr>
                <td colspan="2" align="left">
                    <span class="font2" style="font-size:30px;">WELCOME TO 1024</span>
                </td>
            </tr>
            <form action="{% url 'check_user' %}" method="post">
                {% csrf_token %}
                <tr>
                    <td colspan="2">
                        <input type="text" name="id" class="lg_ip" placeholder="学号">
                    </td>
                </tr>
                <tr>
                    <td colspan="2">
                        <input type="password" name="passwd" class="lg_ip" placeholder="密码">
                    </td>
                </tr>
                <tr>
                    <td align="right">
                        <input type="submit" value="提交" class="button" style="position:relative;left:-50px">
                    </td>
                </tr>
            </form>
            <tr>
                <td align="left">
                    <span class="font1">没有账号？</span>
                    <input type="submit" class="button" style="height:25px;width:80px;font-size:12px" value="注册"
                        onclick="location.href='{% url 'register' %}'">
                </td>
            </tr>
        </table>

    </div>
</body>

</html>

{% if messages %}
<script>
    {% for msg in messages %}
    alert('{{msg.message}}')
    {% endfor %}
</script>
{% endif %}
```



### 后端部分
接下来，我们要实现两个视图函数，一个是 `login`（用于渲染模板）一个是 `check_user`（用于处理表单）。

值得留意的地方有三处，首先在注册页我们查询账户是否已经注册是这样实现的：

```python
users = STUDB.objects.filter(userid=userid)
if len(users) == 1:
    return redirect(reverse('register')+"?code=-1")
```

想想看「验证用户填写的账户密码是否正确」是否能用类似的方式实现呢？



```python
users = STUDB.objects.filter(userid=id, passwd=passwd)
if len(users) == 1:
    return redirect(reverse('play')+"?code=1")
else:
    return redirect(reverse('login')+"?code=-1")
```



第二，我们希望在跳转到闯关页之后，后端可以知道用户是否已经登录。具体的实现需要的是「会话控制」，而 Session 对象转为存储特定用户会话所需的属性及配置信息而生。我们看看相应的代码实现，想想这两行语句应该放置在什么位置：

```python
request.session['userid'] = id
request.session['is_login'] = '1'
```

最后，别忘了在注册成功后，需要我们在**登录页**弹窗提示注册成功。这一部分是在视图函数 `register_to_db` 实现的，可以回看一下。同样地，考虑一下是否其它页面可能会跳转到登录页的且需要弹窗？

接下来，我们尝试实现这两个视图函数吧！



编辑 `myapp/views.py`：

```python
def login(request):
    code = request.GET.get('code', None)
    if code == '-3':
        messages.error(request, "请 先 登 录")
    if code == '-2':
        messages.error(request, "不要没填完就登录啊喂！")
    if code == '-1':
        messages.error(request, "啊哦~登录失败了←_←")
    if code == '1':
        messages.success(request, "注 册 成 功")
    return render(request, 'login.html')

def check_user(request):
    id = request.POST.get('id', None)
    passwd = request.POST.get('passwd', None)
    if not id or not passwd:
        return redirect(reverse('login')+"?code=-2")
    users = STUDB.objects.filter(userid=id, passwd=passwd)
    if len(users) == 1:
        request.session['userid'] = id
        request.session['is_login'] = '1'
        return redirect(reverse('play')+"?code=1")
    else:
        return redirect(reverse('login')+"?code=-1")
```



接下来别忘了把 `check_user` 页的映射关系写好：



编辑 `myapp/urls.py`：

```python
urlpatterns = [
    path('', views.index, name='index'),
    path('register/', views.register, name='register'),
    path('login/', views.login, name='login'),
    path('play/', views.play, name='play'),
    path('win/', views.win, name='win'),
    path('register_to_db/', views.register_to_db, name='register_to_db'),
    path('check_user/', views.check_user, name='check_user'),
]
```



## 闯关页的实现
去年有七个关卡，不知道你还记不记得它们分别是什么。但是 [题解依然是挂着的](https://bobby285271.github.io/SoCoding2019-1024/)，应该能唤起你的一些回忆。

由于关卡具体的 idea 和实现应该是甩锅甩给新一届师弟师妹做的，而从站点制作的角度来说，每个关卡实现起来其实差不多。正如开头所说，我们这里只完成其中的三个关卡，分别是去年第三、六、七关，当然在这里我们就重新将他们编排为第一、二、三关。

### 前端部分
博主认为闯关页部分就做一个页面好了，不然还要做防跳关处理。这意味着每个关卡要做一个模板，在用户访问闯关页的时候就渲染用户当前所在的关卡对应的模板。

我们从第一关开始（去年是第三关），是过气游戏 Flappy Bird，挂上个十次或者直接来个 F12 就通关了。这里我并不会讲解这个游戏怎样用 Web 前端学到的知识实现出来，而是会告诉你网上一大堆现成的 Javascript 实现可以随便抄。

但是我们要记得死亡十次通关还是要自己实现的，其实很简单就在重置游戏的地方调用你的计数器函数即可。

我们开始实现这个小游戏吧！既然需要到 Javascript，还记得 Javascript 这种静态文件要放在什么地方吗？



新建并编辑 `myapp/static/myapp/js/flappybird.js`，留意两个注释出现的位置，那就是你添加你的计时器的位置：

```js
var game = new Phaser.Game(400, 450, Phaser.AUTO, 'game_div');
var game_state = {};
var i = 1;

// 死亡十次通关
function timer(x) {
    i = i + x;
    if (i == 10) {
        alert("有时候失败就是成功！我看到你的努力了！继续前进吧！flag：2048");
    }
    if (i == 30) {
        alert("想再看一次 flag？flag：2048");
    }
    if (i == 50)
        alert("你玩很久了，继续闯关吧！");
}

game_state.main = function () { };
game_state.main.prototype = {
    preload: function () {
        this.game.stage.backgroundColor = '#F0FFFF';
        this.game.load.image('bird', '../static/myapp/img/bird.png');
        this.game.load.image('pipe', '../static/myapp/img/pipe.png');
    },
    create: function () {
        this.bird = this.game.add.sprite(100, 245, 'bird');
        this.bird.body.gravity.y = 1000;
        var space_key = this.game.input.keyboard.addKey(Phaser.Keyboard.SPACEBAR);
        space_key.onDown.add(this.jump, this);
        this.pipes = game.add.group();
        this.pipes.createMultiple(20, 'pipe');
        this.timer = this.game.time.events.loop(1500, this.add_row_of_pipes, this);
        this.score = 0;
        var style = { font: "30px Arial", fill: "#ffffff" };
        this.label_score = this.game.add.text(20, 20, "0", style);
    },
    update: function () {
        if (this.bird.inWorld == false)
            this.restart_game();
        this.game.physics.overlap(this.bird, this.pipes, this.restart_game, null, this);
    },

    jump: function () {
        this.bird.body.velocity.y = -350;
    },

    // 重置游戏
    restart_game: function () {
        if (this.score >= 1) timer(1);
        this.game.time.events.remove(this.timer);
        this.game.state.start('main');
    },

    add_one_pipe: function (x, y) {
        var pipe = this.pipes.getFirstDead();
        pipe.reset(x, y);
        pipe.body.velocity.x = -200;
        pipe.outOfBoundsKill = true;
    },

    add_row_of_pipes: function () {
        var hole = Math.floor(Math.random() * 5) + 1;
        for (var i = 0; i < 8; i++)
            if (i != hole && i != hole + 1)
                this.add_one_pipe(400, i * 60 + 10);
        this.score += 1;
        this.label_score.content = this.score;
    },
};

game.state.add('main', game_state.main);
game.state.start('main'); 
```

不需要理解上面代码的实现，但是通过简单的阅读，我们知道我们还需要一张鸟的图片和一张管道的图片：

```js
this.game.load.image('bird', '../static/myapp/img/bird.png');
this.game.load.image('pipe', '../static/myapp/img/pipe.png');
```

去年实际的代码实际上并没有这两张图片，我们补上。在 `myapp/static/myapp/img` 目录里放置好对应的图片。

这里我们使用到了硬编码链接，这确实是不被提倡的。但是 Django 标签只能在 `myapp/templates/` 中起作用，而 Javascript 中的相对路径的编写是要以 HTML 文件所在位置作为基准的。

当然你可以参考消息框架的实现，选择将部分 Javascript 代码迁移到模板中，使用 `<script></script>` 标签括起来，然后使用 `{% static %}` 标签替换掉硬编码链接，有兴趣的可自行尝试。

同样通过简单的阅读，我们知道我们还需要一个 Phaser（一个 HTML 5 游戏开发框架）的 Javascript 文件，从网上下载即可，放置在 `myapp/static/myapp/js/phaser.min.js`。



接下来我们在模板引用前面的 Javascript 文件，在写模板的时候别忘记了答案提交表单和消息框架（用来提示答案错误的）。



新建并编辑 `myapp/templates/level1.html`：

```html
<!DOCTYPE html>
<html>

<head>
    <meta charset="utf-8">
    <title>Level 1</title>
    {% load static %}
    <link href="{%static 'myapp/css/style.css'%}" rel="stylesheet">
</head>

<body>
    <div class="containerDiv">
        <div class="titleDiv">
            <div id="game_div">
                <span style="font-family:Microsoft YaHei;color: azure;">
                    空格键跳跃
                </span>
            </div>
            
            <script type="text/javascript" src="{% static 'myapp/js/phaser.min.js'%}"></script>
     
            <script type="text/javascript" src="{% static 'myapp/js/flappybird.js'%}"></script>
        </div>

        <div class="problemDiv">
            <div style="text-align:left;clear:both">
                <span class="font2">
                    手残党的福音<br>
                    怎样通过这一片障碍呢
                </span>
            </div>
        </div>
        <div class="answerDiv">
            <form method="post" action="{% url 'compare_flag' %}">
                {% csrf_token %}
                <input type="text" class="input" name="flag">
                <input type="submit" class="button" value="提交">
            </form>
        </div>
    </div>
</body>

</html>
{% if messages %}
<script>
    {% for msg in messages %}
    alert('{{msg.message}}')
    {% endfor %}
</script>
{% endif %}
```



接下来是第二关（去年是第六关）这一关的实现比上一关简单一些，把上一关的 Flappy Bird 换成一张图片就行了，当然图片里面是隐藏了通关信息的。



我们把 `huaji.png` 放置到 `myapp/static/myapp/img/` 目录里面，这就是题目提供的图片了。接下来我们写模板，新建并编辑 `myapp/templates/level2.html`：

```html
<!DOCTYPE html>
<html>

<head>
    <meta charset="utf-8">
    <title>Level 2</title>
    {% load static %}
    <link href="{%static 'myapp/css/style.css'%}" rel="stylesheet">
</head>

<body>
    <div class="containerDiv">
        <div class="titleDiv">
            <span class="font1">Level 2</span>
        </div>

        <div class="problemDiv">
            <span class="font2">
                {% load static %}
                <img src="{%static 'myapp/img/huaji.png'%}" />
                <!-- 可以打开这个神奇的网站看看哦~ -->
                <!-- http://www.atool9.com/steganography.php -->
            </span>
        </div>
        <div class="answerDiv">
            <form method="post" action="{% url 'compare_flag' %}">
                {% csrf_token %}
                <input type="text" class="input" name="flag">
                <input type="submit" class="button" value="提交">
            </form>
        </div>
    </div>
</body>

</html>
{% if messages %}
<script>
    {% for msg in messages %}
    alert('{{msg.message}}')
    {% endfor %}
</script>
{% endif %}
```



最后是第三关（去年是第七关），也不知道去年给大家安利 GitHub 大家用上了没有。这回连图片都不需要了，直接开始写模板。



新建并编辑 `myapp/templates/level3.html`：

```html
<!DOCTYPE html>
<html>

<head>
    <meta charset="utf-8">
    <title>Level 3</title>
    {% load static %}
    <link href="{%static 'myapp/css/style.css'%}" rel="stylesheet">
</head>

<body>
    <div class="containerDiv">
        <div class="titleDiv">
            <span class="font1">Level 3</span>
        </div>

        <div class="problemDiv">
            <span class="font2">
                https://github.com/bobby285271/SoCoding2019-1024<br><br>
                <div style="font-size:18px">去里面看看吧 //滑稽</div>
            </span>
        </div>
        <div class="answerDiv">
            <form method="post" action="{% url 'compare_flag' %}">
                {% csrf_token %}
                <input type="text" class="input" name="flag">
                <input type="submit" class="button" value="提交">
            </form>
        </div>
    </div>
</body>

</html>
{% if messages %}
<script>
    {% for msg in messages %}
    alert('{{msg.message}}')
    {% endfor %}
</script>
{% endif %}
```



### 后端部分
同样地，我们要实现两个视图函数，一个是 `play`（用于渲染模板）一个是 `compare_flag`（用于处理表单）。要注意的是虽然我们只做了三关，但是他们在这里实际上都是不同的角色，第一关是普通关卡前序关卡，第二关是普通关卡最后一关，第三关是地狱关。

* 对于普通关前序关卡，我们只需要更新用户进度即可。
* 对于普通关最后一关，除了更新用户进度，我们需要第一次记录用户通关时间。
* 对于地狱关，我们要允许用户输入彩蛋 Flag 之后记录用户彩蛋通关时间，同时还要在用户输入正确 Flag 之后记录用户完整通关时间（用于兑奖），无论先输入哪个都要允许用户继续游玩该关卡以输入另一个 Flag，同时输入正确 Flag 后要显示通关页。

这里有几个要注意的地方：

首先，实现 `play` 视图函数的时候，要判断用户是否登录。

```python
if request.session.get('is_login') == '1':
    # do something
else:
    # do something
```

其次，普通关前序关卡处理的逻辑都是一样的。考虑在关卡数增多的情况如何简化代码。

最后，由于地狱关要始终开放，以供用户输入正确 Flag 和彩蛋 Flag。考虑用户连输多次正确 Flag 的情况，应该如何更新用户进度？



编辑 `myapp/views.py`：

```python
def play(request):
    if request.session.get('is_login') == '1':
        code = request.GET.get('code', None)
        userid = request.session['userid']
        user = STUDB.objects.filter(userid=userid).last()
        if user.rank < 3 and code == '-1':
            messages.error(request, "哎呀，错了orz")
        if user.rank > 2 and code == '-1':
            messages.error(request, "终局之战总是很坎坷的，请再找找flag是什么吧_(:з」∠)_")
        if user.rank > 2 and code == '1':
            messages.success(request, '哦吼！恭喜你发现了不得了的β时间线，这一切都是命运石之门的选择~！')
        if user.rank > 3:
            url = 'level3.html'
        else:
            url = 'level' + str(user.rank) + '.html'
        return render(request, url)
    else:
        return redirect(reverse('login')+"?code=-3")


def compare_flag(request):
    correct_flag = [
        '',
        '2018',
    ]
    flag = request.POST.get('flag', None)
    userid = request.session['userid']
    user = STUDB.objects.filter(userid=userid).last()
    if user.rank < 2:
        if correct_flag[user.rank] == flag:
            user.rank += 1
            user.save()
            return redirect(reverse('play'))
        else:
            return redirect(reverse('play')+"?code=-1")
    elif user.rank == 2:
        if flag == 'sdltql':
            user.lastflag = timezone.now()
            user.timesubtract = (
                user.lastflag - user.firstflag).total_seconds()
            user.rank += 1
            user.save()
            return redirect(reverse('play'))
        else:
            return redirect(reverse('play')+"?code=-1")
    elif user.rank > 2:
        if flag == 'happy':
            user.superflag = timezone.now()
            user.timesubtract_last = (
                user.superflag-user.firstflag).total_seconds()
            user.rank = 4
            user.save()
            return redirect(reverse('win'))
        elif flag == '666':
            user.specialflag = timezone.now()
            user.timesubtract_suprise = (
                user.firstflag - user.specialflag).total_seconds()
            user.save()
            return redirect(reverse('play')+"?code=1")
        else:
            return redirect(reverse('play')+"?code=-1")
    return redirect(reverse('play'))
```





我赌五毛你会忘记写 `compare_flag` 的 URL 映射。

编辑 `myapp/urls.py`：

```python
urlpatterns = [
    path('', views.index, name='index'),
    path('register/', views.register, name='register'),
    path('login/', views.login, name='login'),
    path('play/', views.play, name='play'),
    path('win/', views.win, name='win'),
    path('register_to_db/', views.register_to_db, name='register_to_db'),
    path('check_user/', views.check_user, name='check_user'),
    path('compare_flag/', views.compare_flag, name='compare_flag'),
]
```



## 通关页的实现
~~此部分留作习题供读者思考~~。



模板，编辑 `myapp/templates/win.html`：

```html
<!DOCTYPE html>
<html>

<head>
    <meta charset="utf-8">
    <title>Win!</title>
    {% load static %}
    <link href="{%static 'myapp/css/style.css' %}" rel="stylesheet">
</head>

<body>
    <div class="containerDiv">
        <div class="titleDiv"></div>
        <div class="problemDiv">
            <span class="font1" style="font-size:40px">
                <p>恭喜你通过所有关卡！</p>
                <p>1024 程序猿节快乐！</p>
                <img src="{%static 'myapp/img/end.jpg' %}" style="height:250px" />
                <p>完结撒花！哔哩哔哩-(゜-゜)つロ乾杯~</p>
            </span>
        </div>
    </div>
</body>

</html>
```

模板中使用了一张图片，放 `myapp/static/myapp/img` 目录，命名为 `end.jpg`。

视图函数，编辑 `myapp/templates/win.html`：

```python
def win(request):
    if request.session.get('is_login') == '1':
        userid = request.session['userid']
        user = STUDB.objects.filter(userid=userid).last()
        if user.rank == 4:
            return render(request, 'win.html')
        else:
            return redirect(reverse('play'))
    else:
        return redirect(reverse('login')+"?code=-3")
```



## 效果预览

![](https://bobby285271.coding.net/p/img/d/img/git/raw/master/2019_1024.png)

代码已经上传到 [GitHub](https://github.com/bobby285271/1024-demo)。