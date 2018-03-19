---
layout:     post
title:      "Apache Shiro 介绍"
subtitle:   ""
date:       2017-03-13
author:     ""
header-img: "img/home-bg-o.jpg"
tags: 
    - shiro
---

Apache Shiro 是一个灵活的安全框架，主要处理四个问题：验证（authentication），授权（authorization），session管理（enterprise session management），加密（cryptography）。

### Shiro 可以做什么：
* 验证用户身份
* 访问控制：确认用户是否有权限去做一件事
* 在任何环境使用 Session API ，在非web和非EJB容器下一样可使用 
* 在验证，访问控制，session生命周期中执行动作
* 整合多个数据源的用户安全数据，展现在一个用户视图中
* 单点登录 （SSO，Single Sign On）
* 用户登录时 “ 记住我 ” 选项

Shiro 目的是在所有的应用环境中起作用，从简单的命令行应用到最大的企业应用，并且不依赖框架，容器，应用服务三部分，

### Shiro是一个综合的应用安全框架：

![ShiroFeatures](/img/shiro/ShiroFeatures.png) 

Shiro 的目标就是 Shiro 团队提出的 “ 应用安全的四个基础 ” ：验证（authentication），授权（authorization），session管理（session management），加密（cryptography）
* 验证（authentication）：通常表现为登录动作，证明一个用户是否是他自己
* 授权（authorization）：访问控制的过程，表明谁被允许做什么
* session管理（session management）：管理用户 sessions ，包括在非web和非EJB容器中
* 加密（cryptography）：使用加密规则保证数据安全性，方便使用
* Web 支持：Shiro 的 Web 支持APIs 帮助 Web 应用更安全
* 缓存：缓存是在 Shiro 的 API 中保证安全命令快且有效率的第一要素
* 并发：Shiro 使用自身并发特点支持多线程应用
* Testing：帮助你编写单元测试和集成测试，确保你的代码可以正常运行
* “Run As”：允许用户使用其他权限运行，经常用在管理场景下
* 记住当前用户：通过session记住当前的用户，当用户在需要登陆的场景下直接登录进入

---



图片来源：[http://shiro.apache.org/](http://shiro.apache.org/)