---
layout: post
title:  "jquery使用on动态给元素添加事件"
date:   2017-2-28
tags: jquery

---

这两天在做项目时使用jquery遇到的一个坑：

在使用.append给jsp页面的table添加button后，原本在jquery中的on事件不能监听到新出现在页面中的button。

在网上找到标准答案：[向未来的元素添加事件处理程序](http://www.runoob.com/try/try.php?filename=tryjquery_event_on_newel)

---

上代码：

1.给页面class为btn的button元素添加点击事件。

```javascript
$('.btn').on('click', function(){...});
```

2.使用append方法给table中的tbody添加一个class为btn的元素。

```html
$("tbody").append('<tr><td><button class="btn">取消</button></td></tr>');
```

这时候会发现，1中的on()方法无法监听到页面ready之后append进入button。

然后正确的方法是：

```javascript
$('#simple-table').on('click', '.btn', function(){...});
```

PS：simple-table是table的id，这时候处于该table中的所有class包含btn的元素都能被添加上click事件