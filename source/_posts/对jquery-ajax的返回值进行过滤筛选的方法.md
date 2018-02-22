---
title: 对jquery-ajax的返回值进行过滤筛选的方法
categories: 编程
tags:
  - jquery
translate_title: the-jqueryajax-return-value-filtering-filter-method
date: 2016-07-15 18:08:56
---
博主在开发的时候，前端通过ajax返回的一个html字符串，但是需要对其进行筛选，找到可操作的dom，结果弄了半天返回的都是undefined，无法筛选出来，苦找了半天，看到一篇[网上的文章](http://blog.163.com/hai_binbin@126/blog/static/121388609201031544727393/)。原理是使用filter(expr)进行
```javascript
var stext = $(data).filter(‘#content').html();
```
data是通过ajax取回的内容，我想进行筛选，只要取回内容里的ID为content的部分。
这样的写法在IE里一切正常，不知道为什么在Firefox里就不行，用Firebug来进行查找错误，提示返回值是undefined

这个问题已经自己解决！
不知道有没有人遇到同样的问题，但我想还是分享一下自己的经验！
用filter 进行筛选的时候，固定的数据如 march.hu 所说的那个（var data = "<p>第一段</p><p id='second'>第二段</p>";），这种没有关系，<font color='red'>但用AJAX取回动态数据进行筛选的时候，必须同时指定标签类型和 ID，才能正常进行筛选，要不然在Firefox和Chrome下会出错</font>。
错误的：
```javascript
var stext = $(data).filter('#content').html();
```
正确的：
```javascript
var stext = $(data).filter('div#content').html();
```
看来JS基础知识还是重要啊