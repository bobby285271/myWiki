---
title: SCNUOJ Backend Dev
description: 
published: true
date: 2021-02-08T08:13:47.569Z
tags: scnuoj, online-judge, oj
editor: markdown
dateCreated: 2021-02-08T08:13:47.569Z
---

那就是 `controllers` 和 `models` 两个目录了。

继续望文生义，公共题库相关的控制器就在 `controllers/ProblemController.php`，比赛相关的控制器就在 `controllers/ContestController.php`，以此类推。

打开控制器文件，发现都是 `action啥啥啥()` 这样的方法，即使没有学 Yii，知道钦定这个命名规则的不是 JNOJ 开发组就是 Yii，那就遵守这样的命名规则就好不会错的，后面遇到一些 `get啥啥啥()` 这样的方法也是一样的处理方式。

### 添加 SCNUCPC 2021 榜单页

> 才不会告诉你 SCNUCPC 2020 这个页面就是拿来举例子用的。不过哪来的 SCNUCPC 2021...

应该不难找到 SCNUCPC 2020 的视图文件在哪里，照葫芦画瓢整亿个，保存到 `views/board/scnucpc2021.php`。

发现还是没法访问，那么就知道控制器的用途是啥了。打开 `controllers/BoardController.php`。

```php
<?php

namespace app\controllers;

use app\components\BaseController;

class BoardController extends BaseController
{
    public $layout = 'board';
    public function actionScnucpc2020()
    {
        return $this->render('scnucpc2020');
    }
}
```

这就是一个最简单的控制器了，发现有一个 `actionScnucpc2020()` 这样的方法。那就照葫芦画瓢整亿个：

```php
public function actionScnucpc2021()
{
    return $this->render('scnucpc2021');
}
```

既然是 Yii 钦定的命名规则，Yii 自然会把剩下的事情处理好，现在就可以访问新加的页面啦。

那么如果想要删掉这个页面呢，那就疯狂撤销就行了呀。

当然，有时这个方法的实现可谓是又臭又长，例如 `controllers/ContestController.php` 下有这样一个方法。

```php
public function actionIndex()
{
    $searchModel = new ContestSearch();
    $dataProvider = $searchModel->search(Yii::$app->request->queryParams);

    $this->layout = 'main';
    return $this->render('index', [
        'searchModel' => $searchModel,
        'dataProvider' => $dataProvider,
    ]);
}
```

可能会完全不知道 `$dataProvider` 又是什么，这时就可以去视图文件看看哪里用到，知道是干啥的之后接下来就要到业务模型那里去找答案啦。

### 更新榜单排名规则

榜单排名肯定是在榜单页显示的啦，你应该会很快找到 `views/contest/_standing_group.php` 和 `views/contest/_standing_oi.php` 两个视图文件，接下来又会在 `controllers/ContestController.php` 下找到一个 `actionStanding($id, $showStandingBeforeEnd = 1)` 的方法，然后发现有这样一句：

```php
$rankResult = $model->getRankData(true);
```

又可以开始望文生义大法了，知道这肯定就是在获取榜单数据了，接下来就去业务模型那里去找答案。直接在项目中搜索 `getRankData` 马上定位出来了，在 `models/Contest.php` 这里。

```php
public function getRankData($lock = true, $endtime = null)
{
    if ($this->type == Contest::TYPE_OI || $this->type == Contest::TYPE_IOI) {
        return $this->getOIRankData($lock, $endtime);
    }
    return $this->getICPCRankData($lock, $endtime);
}
```

马上就能知道有一个 `getOIRankData` 一个 `getICPCRankData`，我防出去了啊防出去以后自然是传统功夫点到为止（错乱）。

接下来就是一些码农大模拟啊，结构体排序这些东西了，这里不再多说。

### 给比赛添加搜索

这个 SCNUOJ 是自带的，您当然不需要再实现一遍，不过也可以试着给 JNOJ 加一个。

我们现在假设你在看 JNOJ 的源码吧，JNOJ `controllers/ContestController.php` 的 `actionIndex()` 是这样的：

```php
public function actionIndex()
{
    $this->layout = 'main';
    $dataProvider = new ActiveDataProvider([
        'query' => Contest::find()->where([
            '<>', 'status', Contest::STATUS_HIDDEN
        ])->andWhere([
            'group_id' => 0
        ])->orderBy(['id' => SORT_DESC]),
    ]);

    return $this->render('index', [
        'dataProvider' => $dataProvider,
    ]);
}
```

`models/ContestSearch.php` 却是一片荒芜，好家伙。

此时你比对一下 `models/ProblemSearch.php`，就发现了可以照葫芦画瓢，顺便整理一下代码。

于是直接把上面的代码挪进去，加上一行查一下 title 就行了。

```php
$query->FilterWhere(['like', 'title', $this->title])
    ->andwhere([
        '<>', 'status', Contest::STATUS_HIDDEN
    ])->andWhere([
        'group_id' => 0
    ])->orderBy(['id' => SORT_DESC]);
```

当然还有一些复杂一些的例子，直接在项目目录搜索 `new Query()` 和 `Yii::$app->db->createCommand()` 就行了。

例如如何给做题量排行添加学号呢？当然 JNOJ 还是没有：

```php
$query = (new Query())->select('u.id, u.nickname, u.rating, s.solved')
    ->from('{{%user}} AS u')
    ->innerJoin('(SELECT COUNT(DISTINCT problem_id) AS solved, created_by FROM {{%solution}} WHERE result=4 AND status=1 GROUP BY created_by ORDER BY solved DESC) as s', 'u.id=s.created_by')
    ->orderBy('solved DESC, id');
```

这时就搜上面两个关键词找一下，很快就会知道 Join 是什么东西了，然后就可以对比一下 `controllers/RatingController.php` 现在的实现。
