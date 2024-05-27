---
title: hgamemini-ReadSomethingUseful
categories: hgame-mini wp
description: hgamemini ReadSomethingUseful 题解
tags: 
- PHP 伪协议
- PHP 文件上传

---
# Read Something Useful 题解
首先观察界面

![](img/Snipaste_2023-12-06_14-55-24.png)
发现是通过点击获取不同的表情包，没有任何头绪，于是按F12进入开发者模式
![](img/Snipaste_2023-12-06_17-05-46.png)
发现 
`<!-- try to read index.php first! -->`以及
```php
 $file = $_GET['file'];
if(strpos($file, "emoji") !== false){
    include($file.'.php');
}
```

根据提示要读取`index.php`，而在此处发现文件上传漏洞，可以在此处读取`index.php`

根据代码，要进入`index.php`首先要保证传入的`value`内容中有`emoji`，但是又要保证读入的内容为`index.php`，通过查阅资料发现可以通过伪协议来实现文件读取的功能。
![](img/Snipaste_2023-12-06_17-24-44.png)
通过伪协议构造`payload`构造`write=emoji`来达到不影响读取同时又有`emoji`的效果。
![](img/Snipaste_2023-12-06_17-33-08.png)
得到代码
```php
<html>
    <head>
        <meta charset="utf-8">
        <meta name="author" content="R1esbyfe">
        <meta name="keywords" content="Challenge12">
    </head>
        <body>
        <div style="text-align: center">
            <!-- try to read index.php first! -->
            <h4>Try to click the button to enjoy my emoji!!!</h4>
            <form action="index.php" method="get">
                <input type="hidden" name="file" id="file" value="emoji">
                <button type="submit">Click me</button>
                <!-- You may find it is a little difficult to read other files, right :( -->
                <!-- in index.php:

                    $file = $_GET['file'];
                    if(strpos($file, "emoji") !== false){
                    include($file.'.php');
                    }
                -->
            </form>
        </div>
        </body>
</html>

<?php
include('blacklist.php');

//try to read findme.php to find my secrets :D

$file = $_GET['file'];

if(isset($file)){
    foreach ($blacklist as $blacklistedWord) {
        if (preg_match('/(?i)' . $blacklistedWord . '/', $file)) {
            echo "<center>nonono</center>";
            return;
        }
    }

    if(strpos($file, "emoji") !== false){
        include($file.'.php');
    }else{
        echo "<center>No,You need to find another way to find my secret :(</center>";
        return;
    }
}
?>

```
发现`try to read findme.php to find my secrets :D`
用相同的办法获取代码
![](img/Snipaste_2023-12-06_17-48-06.png)
```php
<?php
    $flag = $_ENV['FLAG'];

    $lucky = $_GET['luckynumber'];
    $context = $_POST['context'];

    if(isset($lucky)&&isset($context)){
        if(is_numeric($lucky)){
            die("Not a number");
        }elseif($lucky == 777){
            if(file_get_contents($context,'r') === "humanoid"){
                    echo "Congratulations, the flag is: ".$flag;
            }else{
                die("Try again");
            }
        }
    }else{
        return;
    }

```
观察代码，发现要发送get请求传入lucky，发送post请求传入context，
首先二者要同时进行，并且`$lucky`不能为数字并且是777，`$context`要有humanoid，才可以查看`flag`

我们发送`get`请求需要用%00绕过，使他不被判定为数字。

发送`post`请求要求文件获取内容为`humanoid`
我们利用伪协议构造数据流，得到flag
![](img/Snipaste_2023-12-06_18-02-14.png)
