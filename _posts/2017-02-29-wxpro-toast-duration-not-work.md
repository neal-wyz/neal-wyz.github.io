---
layout: post
title:  "微信小程序toast在跳转页面时duration无效"
date:   2017-2-29
tags: weChat

---

直接上代码：

微信小程序使用 :

```javascript
wx.showToast({
	title: '测试',
    icon: 'success',
    duration: 2000, //toast控件在界面显示的时间
    complete: function () {
      wx.navigateBack({
        delta: 1 //这里代表返回上一级页面，微信小程序目前支持层级为5
      })
    }
  })
```
会直接执行complete中的内容，要使用下面的方式执行

```javascript
complete: function () {
	setTimeout(function () { //使用javascript的setTimeout方法
        wx.navigateBack({
          delta: 1
        })
      }, 2000);
    }
```