---
title: hgame
tags: misc web
description: week1wp
---
# hgame week1 
## web
### ezhttp
题目要求从`vidar.club`进入
![](./img/Snipaste_2024-01-29_21-08-07.png)
修改请求头之后,根据题目修改`use-agent`并从本地访问,这里普通的 X-Forwarded-For: 127.0.0.1，或client-ip:127.0.0.1都不可以，查阅资料，我们发现了`X-Real-IP:127.0.0.1`可以实现功能
![](./img/Snipaste_2024-01-29_21-14-15.png)
刚开始以为是需要抓包，尝试了半天结果是上方给的密文，试了一下`base64`解码，得到flag
### bypass it
题目要求我们登录，但是首先要注册，点击注册会根据弹窗返回登陆页面，根据提示，我们禁用`js`，成功登录
![](./img/Snipaste_2024-01-29_23-19-24.png)
![](./img/Snipaste_2024-01-29_23-18-44.png)
![](./img/Snipaste_2024-01-29_23-19-14.png)
### Select Courses
这题模拟的是选课系统，随机会有课放出来，因此我们搞个爬虫判断课程是否有余量并不断发送请求
```python
import json
import requests
for i in range(5):
    while(1):
        a=requests.get('http://47.100.137.175:30125/api/courses')
        b=json.loads(a.content)
        c=b['message']
        d=c[i]
        check=d['is_full']
        choice=d['status']
        if(check==False):
            headers ={
            "Content-Type": "application/json;charset=utf-8"
            }
            payload={"id":i+1}
            url="http://47.100.137.175:30125/api/courses"
            e=requests.post(url,data=json.dumps((payload)),headers=headers)
            print(e)
            a=requests.get('http://47.100.137.175:30125/api/courses')
            b=json.loads(a.content)
            c=b['message']
            d=c[i]
            check=d['is_full']
            choice=d['status']
            if(choice==True):
                print(i)
                break

```
### 2048*16
游戏要求我们玩到`2048*16`的分数即可获得`flag`,网页源代码是混淆过的`js`代码，我们想要分析出在游戏胜利的时候会输出什么内容，我们查找关键字`game`
```js
g[h(432)][h(469)] = function(x) {
    var n = h
      , e = x ? "game-won" : n(443)
      , t = x ? s0(n(439), "V+g5LpoEej/fy0nPNivz9SswHIhGaDOmU8CuXb72dB1xYMrZFRAl=QcTq6JkWK4t3") : n(453);
    this[n(438)][n(437)].add(e),
    this[n(438)][n(435)]("p")[-1257 * -5 + 9 * 1094 + -5377 * 3].textContent = t
}
```
通过分析，猜测输出的可能是`s0(n(439), "V+g5LpoEej/fy0nPNivz9SswHIhGaDOmU8CuXb72dB1xYMrZFRAl=QcTq6JkWK4t3")`,但是未知的变量和函数还需要逐步查找
我们拼接出这一行的关键信息的相关代码
```js

function $() {
    var x = ["debu", "charAt", "game-over", "push", "tile", "3218200jObBXv", "gger", "bestContainer", "firstChild", "chain", "4992592cfFfKg", "updateBestScore", "Game over!", "add", "score-addition", ".best-container", "over", ".tile-container", "scoreContainer", "counter", "clearMessage", "tile-", "tile-merged", "appendChild", "remove", "1457704JdCGrI", "apply", "clearContainer", "message", "1135845OAckHq", "init", "requestAnimationFrame", "addTile", "applyClasses", "\\+\\+ *(?:[a-zA-Z_$][0-9a-zA-Z_$]*)", "value", "while (true) {}", "call", "length", "querySelector", "indexOf", "string", "div", "tile-new", "function *\\( *\\)", "setInterval", "2589jWZTtI", "updateScore", "class", "createElement", "score", '{}.constructor("return this")( )', "4321134sPxlgc", "stateObject", "positionClass", "action", "terminated", "won", "tile-position-", "constructor", "join", "fromCharCode", "forEach", "textContent", "normalizePosition", "continueGame", "previousPosition", "bestScore", "3224mBKYMJ", "1522395ywebnW", "prototype", ".score-container", "actuate", "getElementsByTagName", "tile-super", "classList", "messageContainer", "I7R8ITMCnzbCn5eFIC=6yliXfzN=I5NMnz0XIC==yzycysi70ci7y7iK", "tileContainer"];
    return $ = function() {
        return x
    }
    ,
    $()
}
function F(x, n) {
    var e = $();
    return F = function(t, r) {
        t = t - (-4073 * 1 + 84 * -39 + 7766);
        var a = e[t];
        return a
    }
    ,
    F(x, n)
}
var h=F;
console.log(h(442))
function s0(x, n) {
    for (var e = h, t = 0, r, a, o = 0, c = ""; a = x[e(442)](o++); ~a && (r = t % (-1 * 445 + -324 + -1 * -773) ? r * (-64 * 33 + -6548 + 8724) + a : a,
    t++ % (-268 * -25 + 166 * -37 + -277 * 2)) ? c += String[e(423)](7397 + 173 * 13 + 1 * -9391 & r >> (-2 * t & 1573 + -2423 * 1 + -856 * -1)) : 3978 + -26 * 153)
        a = n[e(481)](a);
    return c
}
var n = h;
console.log(s0(n(439), "V+g5LpoEej/fy0nPNivz9SswHIhGaDOmU8CuXb72dB1xYMrZFRAl=QcTq6JkWK4t3") )

```
但是并没有输出答案，查找猜测可能是源代码把x打乱了
```js
(function(x, n) {
    for (var e = F, t = x(); ; )
        try {
            var r = -parseInt(e(470)) / 1 + -parseInt(e(466)) / 2 + parseInt(e(487)) / 3 * (parseInt(e(430)) / 4) + parseInt(e(446)) / 5 + parseInt(e(493)) / 6 + -parseInt(e(431)) / 7 + parseInt(e(451)) / 8;
            if (r === n)
                break;
            t.push(t.shift())
        } catch {
            t.push(t.shift())
        }
}
)($, -1 * -639371 + -997 * 937 + 896117 * 1);
```
加入到测试代码后成功输出flag
### jhat
`java`小白对界面可以说是一无所知，只能根据现有的hint，进行查阅资料
`jhat`
```
jhat是jdk内置的工具之一。 主要是用来分析java堆的命令，可以将堆中的对象以html的形式显示出来，包括对象的数量，大小等等，并支持对象查询语言。 使用jmap等方法生成java的堆文件后，使用其进行分析。
```
又粗查了一些简单的oql语法,长得和sql有点像
![](./img/Snipaste_2024-02-03_22-47-05.png)
查到了相关用法，根据hint,查到资料可以使用`exec()`函数，查询`flag`但是由于输出会调用`ToString`方法，导致无法输出内容，查阅资料得到`payload`。
![](./img/Snipaste_2024-02-03_22-51-27.png)

## misc
### simple_attack
下载附件之后发现是一个压缩包和一个图片的组合，上网查找了一下有关压缩包的Misc知识，联想可能是明文爆破，
![](./img/Snipaste_2024-01-31_23-22-44.png)
但是出现了报错，询问了学长，发现明文需要用`src`，爆破成功。
打开`photo.txt`，发现该文本文件是一个以`image/png;base64`开头的乱码，在查阅资料后，查到需要我们把文本内容复制到浏览器里面，得到flag。
### 希尔希尔希尔
打开图片是乱码，猜测是修复图片长宽，载入crc32脚本，查验出图片原来的数据，把图片拖到`010editor`,修改数据，得到原本的图片。
![](./img/Snipaste_2024-02-01_12-53-01.png)
```python
import os
import binascii
import struct

crcbp = open("secret.png", "rb").read()    #打开图片
for i in range(2000):
    for j in range(2000):
        data = crcbp[12:16] + \
            struct.pack('>i', i)+struct.pack('>i', j)+crcbp[24:29]
        crc32 = binascii.crc32(data) & 0xffffffff
        if(crc32 == 0x121b804d):    #图片当前CRC
            print(i, j)
            print('hex:', hex(i), hex(j))
```
看文件大小，直觉先`binwalk`一下，确实有隐藏文件,给了好几个文件，折腾了半天，原来就一个`secret.txt`,根据题目提示，可能是希尔密码，但是缺少密钥，但是还原后的图片还没有用上，原来是还有一层`lsb`隐写，哇咔咔，密钥到手了。
![](./img/Snipaste_2024-02-02_15-54-59.png)
载入解密网站，拿到`flag`。
### 来自星辰的问候
根据题目提示的6位弱加密，用了`setghide`找到了隐藏文件，分别是一个图片和一个网页源代码文件，随后用浏览器打开，是一个可以上传文件的网站
随后我们打开图片，还好出题人给了提示，原来官网有这种奇妙文字，按`F12`找到字体的ttf文字,想要通过给的网站转换一下，但是没用明白，最后手动对位的，得到flag