---
title: SCNUSE SoCoding 1024 Website: Step 3
description: 
published: true
date: 2021-01-11T05:26:38.366Z
tags: web, python, guide, html, django, scnu
editor: markdown
dateCreated: 2021-01-11T05:26:38.366Z
---

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


