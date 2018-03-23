---
layout:     post
title:      "Maven 概述"
subtitle:   ""
date:       2017-04-06
author:     "Neal Wang"
header-img: "img/home-bg-o.jpg"
tags: 
    - maven
---

# 概念
Apache Maven 是一套软件工程管理和整合工具。基于工程对象模型（POM）的概念，通过一个中央信息管理模块，Maven 能够管理项目的构建、报告和文档。
> At first glance Maven can appear to be many things, but in a nutshell Maven is an attempt to apply patterns to a project's build infrastructure in order to promote comprehension and productivity by providing a clear path in the use of best practices

maven能够完成的工作：
* 构建
* 文档生成
* 报告
* 依赖
* SCMs
* 发布
* 分发
* 邮件列表

Maven 的主要目的是为开发者提供
* 一个可复用、可维护、更易理解的工程综合模型
* 与这个模型交互的插件或者工具

Maven 工程结构和内容被定义在一个 xml 文件中 － pom.xml，是 Project Object Model (POM) 的简称，此文件是整个 Maven 系统的基础组件。

Maven 能给你的开发过程带来什么好处？
Maven 提供了许多约定好的标准来简化你开发时对项目的构建，打包等等，并且尽量去缩短你的开发周期，并且同时提高你的开发成率。

# 安装 （windows）
安装支持对应版本 Maven 的 JAVA
* Maven 3.3 要求 JDK 1.7 或以上
* Maven 3.2 要求 JDK 1.6 或以上
* Maven 3.0/3.1 要求 JDK 1.5 或以上

安装JDK
* 从 [http://www.oracle.com/technetwork/java/javase/downloads/index-jsp-138363.html#javasejdk]() 下载 JDK 并安装
* 添加系统环境变量 JAVA_HOME = C:\Program Files\Java\jdk1.8.0_121（你的本地 JAVA 文件夹）
* 添加系统环境变量 CLASSPATH = .;%JAVA_HOME%\lib\dt.jar;%JAVA_HOME%\lib\tools.jar;
* 把 %JAVA_HOME%\bin;%JAVA_HOME%\jre\bin; 加到 PATH 变量的末尾
* 在命令行使用 java -version 来验证你的 java 版本

安装 Maven
* 从 [http://maven.apache.org/download.html]() 下载 Maven 并解压到你想要安装Maven的文件夹中
* 添加 Maven bin 目录到系统环境变量 Path 中
* 添加 MAVEN_OPTS 系统变量，值为-Xms256m -Xmx512m
* 在命令行使用 mvn -v 来验证你的 Maven 安装