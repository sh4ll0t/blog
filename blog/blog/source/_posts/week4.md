---
title: hgameweek4
tags: misc web
description: week4wp
---
# week4
## web 
### Reverse and Escalation
![](./img/Snipaste_2024-02-24_19-49-39.png)
打开网站需要登录
![](./img/Snipaste_2024-02-24_19-50-06.png)
抓个包，感觉`activemq`有点眼生，查一下发现是一个`CVE-2023-46606`,默认`admin`和`admin`登录一下
![](./img/Snipaste_2024-02-22_21-35-44.png)
github上有专用的工具
`https://github.com/SaumyajeetDas/CVE-2023-46604-RCE-Reverse-Shell-Apache-ActiveMQ/`
按照步骤执行一下
![](./img/Snipaste_2024-02-23_17-06-33.png)
![](./img/Snipaste_2024-02-23_17-06-18.png)
成功反弹shell
`cat /flag`一下发现` Permission denied`
查找资料，原来是权限不够，涉及到提权的知识
匹配一下可以用`find`命令提权
`find / -perm -g=s -type f 2>/dev/null
find . -exec /bin/sh -p \; -quit
`
反弹一个有root权限的shell就可以啦
![](./img/Snipaste_2024-02-23_17-06-00.png)
### everse and Escalation.II
find 命令会出现一个很奇怪的东西
怀疑把find命令内部改过了,把文件base64复制下来，粘贴到010editor,用ida打开
查看main函数
![](./img/Snipaste_2024-02-24_19-05-05.png)
观察函数，会以时间为种子生成伪随机数，需要我们一次性输入伪随机数正确相加的结果38次以上会以root执行ls
```C
#include<stdio.h>
#include<time.h>
#include<stdlib.h>
int main(){
        int seed=time(0ll)+60;
        srand(seed);
        int a,b;
        char str[38];
        for(int i=0;i<40;++i){
                a=rand()%23333;
                b=rand()%23333;
                int c=a+b;
                printf("%d ",c);
        }
}
```
把时间预定到1分钟的时候预测伪随机数，计算出相加的结果，但是成功后执行的是ls 命令，因为system函数有继承环境变量的性质，我们伪造一个ls 的可执行文件，得到flag
![](./img/Snipaste_2024-02-24_20-17-06.png)
### Whose Home?
打开网页是一个QB登录页面，查到了默认用户名`admin`和密码`adminadmin`
登录进去，根据提示，后台是可以rce的,找了找，发现在上传文件的时候可以执行外部程序，
出网＋执行外部程序，猜测反弹shell,找了常见的弹shell方法，由于不知道这个后台Linux装了什么，所以先考虑`bash`
![](./img/Snipaste_2024-02-26_13-38-16.png)
cat /flag 一下发现需要提权
查找了有suid权限的命令
`find / -user root -perm -4000 -print 2>/dev/null`
![](./img/Snipaste_2024-02-26_14-14-23.png)
理论上一个一个搜索看每个命令是什么功能和是否可以利用就可以查看到flag
去网上查了一下命令的每个参数的用法，修改一下就可以啦
![](./img/Snipaste_2024-02-26_14-04-23.png)
111