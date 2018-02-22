---
title: javascript中浅拷贝和深拷贝的区别
categories: 编程
tags:
  - javascript
translate_title: the-difference-between-shallow-copy-and-deep-in-javascript
date: 2017-03-02 12:19:03
---

今天在看阮一峰大神的[ECMAScript 6入门的电子版](http://es6.ruanyifeng.com/)，其中在第9节`对象的扩展`讲到了`对象的扩展运算符` 和 `Object.getOwnPropertyDescriptors`方法，里面有关于Object对象的属性拷贝一栏，非常感兴趣，百度google后发现，在前端工程师的面试题中也经常会问到这个知识点，于是在网上搜集整理了些摘要，数据来源为`知乎`和`SegmentFault`，对应的作者都是`JerryZou`，传送门：http://jerryzou.com/posts/dive-into-deep-clone-in-javascript/

------

## 深拷贝和浅拷贝区别
> 浅拷贝只复制一层对象的属性，而深拷贝则递归复制了所有层级
> 深拷贝必须用到递归

首先深复制和浅复制只针对像 Object, Array 这样的复杂对象的。简单来说，**浅复制只复制一层对象的属性，而深复制则递归复制了所有层级**。
抛开jQuery，上代码例子。下面是一个简单的浅复制实现：
```javascript
var obj = { a:1, arr: [2,3] };
var shadowObj = shadowCopy(obj);

function shadowCopy(src) {
  var dst = {};
  for (var prop in src) {
    if (src.hasOwnProperty(prop)) {
      dst[prop] = src[prop];
    }
  }
  return dst;
}
```
因为浅复制只会将对象的各个属性进行依次复制，并不会进行递归复制，而 JavaScript 存储对象都是存地址的，所以浅复制会导致 `obj.arr` 和 `shadowObj.arr`指向同一块内存地址，大概的示意图如下。
![](https://ww2.sinaimg.cn/large/006tNc79gy1fd9ju10nipj30go068gnm.jpg)

导致的结果就是：
```javascript
shadowObj.arr[1] = 5;
obj.arr[1]   // = 5
```

而深复制则不同，它不仅将原对象的各个属性逐个复制出去，而且将原对象各个属性所包含的对象也依次采用深复制的方法递归复制到新对象上。这就不会存在上面 obj 和 shadowObj 的 arr 属性指向同一个对象的问题。
```javascript
var obj = { a:1, arr: [1,2] };
var obj2 = deepCopy(obj);
```
结果如下面的示意图所示：
![](https://ww3.sinaimg.cn/large/006tNc79gy1fd9jvm22gyj30go069acr.jpg)

需要注意的是，如果对象比较大，层级也比较多，深复制会带来性能上的问题。在遇到需要采用深复制的场景时，可以考虑有没有其他替代的方案。在实际的应用场景中，也是浅复制更为常用。

## 如何实现深拷贝

要实现深复制有很多办法，比如最简单的办法有：
```jascript
var cloneObj = JSON.parse(JSON.stringify(obj));
```

上面这种方法好处是非常简单易用，但是坏处也显而易见，这会抛弃对象的constructor，也就是深复制之后，无论这个对象原本的构造函数是什么，在深复制之后都会变成`Object`。另外诸如`RegExp`对象是无法通过这种方式深复制的。

所以这里我将介绍一种，我自认为很优美的深复制方法，当然可能也存在问题。如果你发现了我的实现方法有什么问题，请及时让我知道~

> 先决条件：
1. 对于任何对象，它可能的类型有Boolean, Number, Date, String, RegExp, Array 以及 Object（所有自定义的对象全都继承于Object）
2. 我们必须保留对象的构造函数信息（从而使新对象可以使用定义在prototype上的函数）

最重要的一个函数：
```javascript
Object.prototype.clone = function () {
    var Constructor = this.constructor;
    var obj = new Constructor();
    for (var attr in this) {
        if (this.hasOwnProperty(attr)) {
            if (typeof(this[attr]) !== "function") {
                if (this[attr] === null) {
                    obj[attr] = null;
                }
                else {
                    obj[attr] = this[attr].clone();
                }
            }
        }
    }
    return obj;
};
```
定义在`Object.prototype`上的`clone()`函数是整个方法的核心，对于任意一个非js预定义的对象，都会调用这个函数。而对于所有js预定义的对象，如Number,Array等，我们就要实现一个辅助clone()函数来实现完整的克隆过程：
```javascript
/* Method of Array*/
Array.prototype.clone = function () {
    var thisArr = this.valueOf();
    var newArr = [];
    for (var i=0; i<thisArr.length; i++) {
        newArr.push(thisArr[i].clone());
    }
    return newArr;
};
/* Method of Boolean, Number, String*/
Boolean.prototype.clone = function() { return this.valueOf(); };
Number.prototype.clone = function() { return this.valueOf(); };
String.prototype.clone = function() { return this.valueOf(); };
/* Method of Date*/
Date.prototype.clone = function() { return new Date(this.valueOf()); };
/* Method of RegExp*/
RegExp.prototype.clone = function() {
    var pattern = this.valueOf();
    var flags = '';
    flags += pattern.global ? 'g' : '';
    flags += pattern.ignoreCase ? 'i' : '';
    flags += pattern.multiline ? 'm' : '';
    return new RegExp(pattern.source, flags);
};
```
可能直接定义在预定义对象的方法上，让人感觉会有些问题。但在我看来这是一种优美的实现方式。

以上内容有删改