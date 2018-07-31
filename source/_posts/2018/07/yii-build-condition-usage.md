---
title: Yii框架buildCondition的使用
date: 2018-07-31 18:32
tags:
- php
- web
- develop
categories: 前端开发
---

业余项目中遇到一个bug，google好久终于在stackOverflow上面找到了解决方法，用到了Yii框架的buildCondition方法。网上资料不多，分享一下。

### 问题描述

项目中有一张tags的表用来存给content打的标签，建表的时候默认没有设置mysql区分大小写。维护标签的时候无法区分大小写导致数据出错，网上搜索的结果是使用`BINARY`关键字，实际操作时发现卡住了。

具体情况如下：
- yii框架默认的`IN`语法无法直接添加`BINARY`关键字
- 使用参数绑定时无法正确识别数组

示例代码：
```php
// 原标签数组
$oldTagArray = ['abc', 'def'];
// 默认in语法无法添加BINARY关键字
//$tags = Tag::find()
//    ->where(['IN', 'title', $oldTagArray])
//    ->all()
// 使用参数绑定时无法正确识别数组
$tags = Tag::find()
    ->where('BINARY title IN (:tags)', ['tags' => $oldTagArray])
    ->all()
```

### 解决方法

最直接的方法是手动来拼接此处的sql，但是要手动处理字符串转义和sql防注入，比较繁琐。在google上搜索到一段buildCondition的用法。

最终代码：

```php
// 原标签数组
$oldTagArray = ['abc', 'def'];
$oldTagParam = [];
$whereSql = Yii::$app->db->getQueryBuilder()
    ->buildCondition(['IN', 'title', $oldTagArray], $oldTagParam);
$tags = Tag::find()
    ->where("BINARY $whereSql", $oldTagParam)
    ->all();
```

说明：
- 定义$oldTagParam用来存放参数
- 使用buildCondition来生成参数($oldTagParam的为引用类型，buildCondition方法直接修改值)
- 生成后的$whereSql类似title IN (:qp0, :qp1)
- 修改后的$oldTagParam类似[':qp0'=>'abc', ':qp1'=>'def']
- 参过where方法拼接BINARY关键字和sql，并传入绑定参数


### 链接
stackOverflow相关问题链接 https://stackoverflow.com/questions/30000412/correct-way-to-bind-parameters-using-mysql-in-syntax-in-yii2

buildCondition参考 https://www.yiiframework.com/doc/api/2.0/yii-db-querybuilder#buildCondition()-detail