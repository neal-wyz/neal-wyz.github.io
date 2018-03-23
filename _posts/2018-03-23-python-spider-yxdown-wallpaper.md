---
layout:     post
title:      "python爬取游迅网【真·无码】你不得不收藏的精品壁纸大合集！系列"
subtitle:   ""
date:       2018-03-23
author:     "Neal Wang"
header-img: "img/home-bg-o.jpg"
tags: 
    - python
    - spider
---

# 游迅网【真·无码】你不得不收藏的精品壁纸大合集！系列：

### 作为强迫症，一定要从第一期开始抓，人工测试到previd=100000时找到了第二期，所以打算写个脚本自动搜寻到第一期。

1.起始地址：http://dy.www.yxdown.com/news/indexmore.json?callback=viewMoreCallback&key=bagua&previd=100000

获得的json格式
```
"writer": "Viper",
"UserName": "PCnetwork",
"image500": "http://i-3.yxdown.com/2015/8/31/20396fbb-827b-4330-ad92-395d4e616af0.jpg",
"catalogid": 39,
"ArticleType": 0,
"title": "【真·无码】你不得不收藏的精品壁纸大合集！(2)",
"adddate": "/Date(1377502883000)/",
"comNum": 10,
"prefix": "201308/",
"content": "……</p>\r\n\r\n<p style=\"text-align:center;\"><a href=\"http://i-1.yxdown.com/2013/8/26/e29bf6d5-3841-4608-ade1-895d36421935.jpg\" target=\"_blank\"><img alt=\"游迅网www.yxdown.com\" src=\"http://i-1.yxdown.com/2013/8/26/KDYwMHgp/e29bf6d5-3841-4608-ade1-895d36421935.jpg\" title=\"【真·无码】你不得不收藏的精品壁纸大合集！(2)\" /></a></p>……",
"ArticleID": 99987,
"path": "/news/",
"images": "",
"jianjie2": "在经历了几天Gamescom游戏新闻的“轰炸”后，我们也应该适时的娱乐一下。一组高清壁纸献上，希望大家喜欢。\r\n"

```


可以看到有个ArticleID，previd就是记录上次看到哪里的文章号，并查出从该文章时间起往前推20篇的文章

记录下最后一个然后继续查，直到发现title为

>【真·无码】你不得不收藏的精品壁纸大合集！(1)

的相关文章即可

```
import urllib.request
import json
import re

# 初始previd
n = 100000

# 初始url
orginurl = "http://dy.www.yxdown.com/news/indexmore.json?callback=viewMoreCallback&key=bagua&previd="

# 循环查找第一期壁纸资源

flag = True

while flag:
    url = orginurl + str(n)
    print(url)

    request = urllib.request.Request(url)
    response = urllib.request.urlopen(request)
    result = response.read().decode('utf-8')
    # print(result)

    # 正则表达式匹配json的data数据并转为python对象
    pattern = re.compile(r'{\"data[\S\s]+}\]}')
    content = pattern.search(result).group()
    
    # 获取dict中data的值
    contentlist = json.loads(content)['data']
    for x in contentlist:
        n = x['ArticleID']
        if x['title'] == '【真·无码】你不得不收藏的精品壁纸大合集！(1)':
            flag = False
            break

    # 强制结束，第一篇文章的Id为20770
    if n == 20770:
        flag = False

```

最终找到第一期的ArticleID：95237

### 接下来就是获取所有壁纸系列的文章图片信息并存储到文件中

```
# -*- coding:utf-8 -*-

import urllib.request
import os
import logging
import json
import re

# create logger
logging.basicConfig(filename='saveimg.log', level=logging.INFO, format='%(asctime)s - %(message)s')
logger = logging.getLogger('nana')

# 如果不存在则创建目录
localpath = r'C:\Users\admin\Pictures\wallpaper\\'
if not os.path.exists(localpath):
    os.makedirs(localpath)
    logger.info(r'已创建' + localpath)

# 初始previd，第一期为95237，从后往前查询，写该脚本时最新一期为392431，以及初始url
n = 392431
orginurl = "http://dy.www.yxdown.com/news/indexmore.json?callback=viewMoreCallback&key=bagua&previd="


# 保存图片到壁纸文件夹
def saveimg(imgurl, filename):
    imgpath = localpath + filename
    if os.path.exists(imgpath):
        pass
    else:
        u = urllib.request.urlopen(imgurl)
        imgdata = u.read()
        f = open(imgpath, 'wb')
        f.write(imgdata)
        f.close()


# 匹配content内容，获取高清地址和文件名并调用saveimg函数保存
def getimg(content):
    pattern = re.compile(r'<a href=.*?jpg')
    result = re.findall(pattern, content)
    logger.info(r'图片数：' + str(len(result)))
    for item in result:
        originalurl = item.split('"')[1]
        newurl = item.split('"')[1] + "?imageView2/2/q/85"
        imgname = originalurl.split('/')[-1]
        saveimg(newurl, imgname)


# 循环查找壁纸资源
flag = True
while flag:
    url = orginurl + str(n)
    logger.info(url)
    request = urllib.request.Request(url)
    response = urllib.request.urlopen(request)
    result = response.read().decode('utf-8')
    # 正则表达式匹配json的data数据并转为python对象
    pattern = re.compile(r'{\"data[\S\s]+}\]}')
    content = pattern.search(result).group()
    # 获取dict中data的值
    contentList = json.loads(content)['data']
    for x in contentList:
        n = x['ArticleID']
        if r'【真·无码】你不得不收藏的精品壁纸大合集！' in x['title']:
            logger.info(x['title'] + r'，ID：' + str(x['ArticleID']) + r'，author：' + x['writer'] + '，username：' + x['UserName'])
            # 执行时会抛出IncompleteRead异常，由于tcp连接异常造成，因为tcp关闭或发送data为空。这里做的处理是忽略该异常，并重新访问当前文章
            try:
                getimg(x['content'])
            except Exception as e:
                pass
            finally:
                n = n + 1
        if x['ArticleID'] == 95237:
            flag = False
    # 强制结束，第一篇文章的Id为20770
    if n == 20770:
        flag = False

logger.info(r'大功告成，撒花！')
```

最后，上述代码首次执行了一个小时左右，然后发现总共爬到了8105张图片，
执行第二遍后为8191张，不知道中间哪里出了bug，猜测应该是try语句块那里。
大神如果找到了原因，跪请指教~~~！

## 感谢
* [游迅网](http://www.yxdown.com/)
* 【真·无码】你不得不收藏的精品壁纸大合集！系列的小编：一封情书