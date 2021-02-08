---
title: SCNUOJ Views Dev
description: 
published: true
date: 2021-02-08T08:12:11.828Z
tags: scnuoj, online-judge, oj
editor: markdown
dateCreated: 2021-02-08T08:12:11.828Z
---

大多数情况下就是希望修改用户界面，只需在 `views` 文件夹下找到相关文件来修改。

对视图层作简单修改只需要一些前端基础，对 Bootstrap 4 有所了解，精通面向搜索引擎编程即可。

### 修改主页的标题

感谢 `enablePrettyUrl` 感谢 JNOJ 开发组感谢我自己，在大部分时候根据 URL 对着找文件就行了。

首先找到 `views/site/index.php`，发现以下内容：

```php
<section class="py-5 text-center">
    <div class="row py-lg-5">
        <div class="mx-auto d-none d-md-block">
            <br />
            <h2>South China Normal University Online Judge</h2>
            <p class="lead text-muted"><?= Yii::$app->setting->get('schoolName') ?>在线评测系统</p>
        </div>
        <div class="mx-auto d-md-none">
            <br />
            <h2>SCNU Online Judge</h2>
            <p class="lead text-muted"><?= Yii::$app->setting->get('schoolName') ?>在线评测系统</p>
        </div>
    </div>
</section>
```

很显然是维护者在偷懒没有在后台加相关的设置项，就结合 Bootstrap 4 的了解，知道宽屏和窄屏会显示不同的东西，接下来修改就是了。

### 去除比赛页的 SCNUCPC 按钮

> 才不会告诉你这个按钮存在的意义就是写文档举例方便。

同样，找到 `views/contest/index.php`，但是一时间没有找到 SCNUCPC 相关的字眼。

但是把这个文件当普通的英文读，结合浏览器打开这个页面显示的东西，我们知道这个页面有两个部分，一个是搜索框一个是比赛列表。

然后你发现了这个东西：

```php
<?php echo $this->render('_search', ['model' => $searchModel]); ?>
```

很显然跟搜索有关，SCNUCPC 那个按钮正好跟搜索栏挨一块，然后找到一个 `views/contest/_search.php`，接下来问题就很简单了。

### 修改顶部导航栏中 "问题" 左边的图标

注意到每个页面都有一模一样的顶部导航栏，猜测会在 `views/layouts/` 里面，这个目录里面有几个文件，根据文件名望文生义即可。

接下来找到这样的内容，继续望文生义即可。

```php
$menuItemsLeft = [
    [
        'label' => '<i class="fas fa-fw fa-book-open"></i> ' . Yii::t('app', 'Problems'),
        'url' => ['/problem/index'],
        'active' => Yii::$app->controller->id == 'problem',
    ]
];
```

这里使用的是 Font Awesome 图标集，因为在同一个文件往上翻翻就会发现：

```
use app\assets\AppAsset;
AppAsset::register($this);
```

还是望文生义，发现确实有 `assets/AppAsset.php` 这个文件，这就是引入静态资源的方式了。

再往下就会探索到 `web/css` 和 `web/js` 两个目录了，这里不再介绍。

**接下来呢？**

你可能会用到 [yii2-bootstrap4 的文档](https://www.yiiframework.com/extension/yiisoft/yii2-bootstrap4/doc/guide/2.0/en/usage-widgets)，但是不用专门去啃这个文档，需要的时候自然就会知道看什么怎么看的了。
