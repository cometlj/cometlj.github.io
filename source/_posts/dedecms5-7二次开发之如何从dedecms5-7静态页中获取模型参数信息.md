---
title: dedecms5-7二次开发之如何从dedecms5-7静态页中获取模型参数信息
categories: 编程
tags:
  - dedecms
  - sql
  - 二次开发
translate_title: >-
  dedecms57-secondary-development-how-to-get-model-parameter-information-from-static-page
date: 2016-07-15 18:17:04
---
博主在二次开发的时候，发现在dedecms生成静态页后，就无法通过$_GET[‘id’]形式获取参数的值了，比如在文章内容页模版里面通过

```php
{dede:php}
echo $_GET['id'];
{/dede:php}
```
这样动态访问页面里面是能获取的，例如/plus/view.php?id=123这样能获取到，但是静态页生成的话，是不认的。后来在dede官方论坛找到了解决方法，只要在模板中使用这样语句
```php
{dede:php}
$thisid = $refObj->Fields['id'];//同理可以设置不同的key，对应的附加表自定义字段等
$sql ="select * from dede_archives where id=$thisid";
$row = $dsql-&gt;GetOne($sql);
{/dede:php}
```
有兴趣的同学可以把`$refObj`打印出来看看，里面包含很多有用的信息，如此，通过$refObj对象的Fields方法，就能实现获取参数的值了，同理，自定义模型的附加表字段也可以通过这种形式获取，其他的就发挥想象，通过万能的sql解决剩下问题把。