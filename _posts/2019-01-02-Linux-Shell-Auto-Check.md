---
layout:     post
title:      "linux-shell脚本自动检测tomcat,nginx,mysql并配置邮件报警"
subtitle:   ""
date:       2019-01-02
author:     "Neal Wang"
header-img: "img/tag-bg.jpg"
tags: 
    - Linux
---

# 部署自动检测运行的shell脚本

## 文件说明：

* job.txt：自动执行计划
* test.log：日志存放
* statusCheck.sh：检测主文件

PS：三个文件可以存放在同一目录下比较方便

## 脚本部署

### statusCheck.sh代码说明

这里要说明一下检测原理，是根据设置端口是否有连接查询，通过比对进程数量来确定程序是否运行正常。

如何确认网站是否可以正常访问，发送一个访问请求然后判断返回码状态即可。

``` 
#!/bin/bash

source /etc/profile #加载用户环境变量

tlog=/opt/statustest/test.log #位置日志
DATE=`date +'%F %T'` #日期调用

check=`wget -S --spider --tries=3 --timeout=3 http://www.tianhenet.cn 2>&1 | grep HTTP/1.1|awk '{print $2}'` #检测www.tianhenet.cn的状态码是否为200，不是则开始检测

if [ "$check"x = "200"x ];then
  echo "$DATE Tianhenet Good State">>$tlog
else
  mail -s "www.tianhenet.cn Bad State Now!!! Begin Auto Checking..." jack.wang@shun-lu.cn #邮件通知，需要配置/etc/mail.rc文件
  echo "$DATE Tianhenet Bad State">>$tlog
  echo "$DATE Begin Checking...">>$tlog

  #---------mysql check------------------------------------------------------------------
  mysqlport=`netstat -nlt|grep 3306|wc -l`
  mysqlproc=`ps -ef|grep mysql|grep -v grep|wc -l`
  if [ $mysqlport -eq 1 ] && [ $mysqlproc -eq 2 ]
  then
    echo "$DATE MySQL is running...">>$tlog
  else
    /etc/init.d/mysqld start
    echo "$DATE MySQL Restarted!">>$tlog
  fi

  #---------tomcat check-----------------------------------------------------------------
  if lsof -i :9090 | grep -q tomcat
  then    
                proc=`lsof -i :9090 | grep tomcat | awk '{print $1}'`
    pid=`lsof -i :9090 | grep tomcat | awk '{print $2}'`
    echo "$DATE Exist process ${proc} with id ${pid} occupying port 9090...">>$tlog
    kill -9 ${pid}
    echo "$DATE Killed it!">>$tlog
  else
    echo "$DATE No process occupying port 9090...">>$tlog
  fi
  /opt/thwoctomcat8/bin/shutdown.sh
  sleep 10
  /opt/thwoctomcat8/bin/startup.sh
  echo "$DATE Tomcat Restarted!">>$tlog

  #-------nginx check--------------------------------------------------------------------
  nginxport=`netstat -nlpt|grep 0.0.0.0:80|grep nginx|wc -l`
  nginxproc=`ps -ef|grep nginx|grep -v grep|wc -l`
  if [ $nginxport -eq 1 ] && [ $nginxproc -eq 2 ]
  then
    echo "$DATE nginx is running...">>$tlog
  else
    service nginx restart
    echo "$DATE nginx Restarted!">>$tlog
  fi
fi
```

### 邮件配置

/etc/mail.rc最后面加入如下信息：

```
set from=jack.wang@shun-lu.cn #你要发送邮件的邮箱
set smtp=smtp://smtp.mxhichina.com:25 #你使用邮箱的smtp服务器
set smtp-auth-user=jack.wang@shun-lu.cn #邮箱用户名
set smtp-auth-password= #邮箱密码
```

使用邮箱的smtp服务器可以在网页邮箱进去后设置里查到

### 部署自动执行

1. 执行crontab job.txt
2. 查看添加的计划任务crontab –l 
3. 重启crond服务 service crond restart
4. 自动检测生效
