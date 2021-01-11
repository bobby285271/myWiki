---
title: SCNUSE SoCoding 1024 Website: Step 3
description: 
published: true
date: 2021-01-11T05:27:11.465Z
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

