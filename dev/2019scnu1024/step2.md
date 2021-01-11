---
title: SCNUSE SoCoding 1024 Website: Step 2
description: 
published: true
date: 2021-01-11T05:25:22.294Z
tags: web, python, guide, html, django, scnu
editor: markdown
dateCreated: 2021-01-11T05:25:22.294Z
---

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