---
title: hgamemini-Misc总结
tags: 图片隐写
description: hgamemini-图片隐写总结
---
# hgamemini-图片隐写总结
## 复原图片尺寸
![](../img/Snipaste_2023-12-10_17-43-05.png)

是一张看不出来任何文字和图像的乱码文字，可能是修改了图片的长和宽

通过010 Editor载入png模板，查看到图片的长和高
![](../img/Snipaste_2023-12-10_17-39-44.png)
利用Python脚本计算正确的尺寸
```python
import os
import binascii
import struct

crcbp = open("xxx.png", "rb").read()    #打开图片
for i in range(2000):
    for j in range(2000):
        data = crcbp[12:16] + \
            struct.pack('>i', i)+struct.pack('>i', j)+crcbp[24:29]
        crc32 = binascii.crc32(data) & 0xffffffff
        if(crc32 == 0x38162a34):    #图片当前CRC
            print(i, j)
            print('hex:', hex(i), hex(j))
```
计算出图片原本的长和高，并修改，得到原图，注意路径使用转义字符。
![](../img/Snipaste_2023-12-10_18-07-18.png)
## 隐藏文件分离
观察图片，发现`jpg`文件过大，应该图片后面存在非`jpg`的二进制数据，保存在虚拟机里之后执行命令行
![](../img/Snipaste_2023-12-10_18-36-59.png)
得到jpg里的压缩包文件
![](../img/Snipaste_2023-12-10_18-49-55.png)

在第二题的相似思路中我们得到了![](../img/Snipaste_2023-12-13_19-00-56.png)
这并不是正确的flag,通过观察我们发现这串字符只是改变了顺序，考虑可能是栅栏密码。
通过解密得到flag。
![](../img/Snipaste_2023-12-13_19-02-34.png)
### lsb隐写
![](../img/Snipaste_2023-12-10_18-55-56.png)
![](../img/Snipaste_2023-12-10_18-59-18.png)
![](../img/Snipaste_2023-12-10_18-57-04.png)
#### 魔数识别文件格式
![](../img/Snipaste_2023-12-13_19-03-05.png)
打开压缩包，发现有三个文本文件，打开也是乱码，猜测可能是不同格式的文件。打开`010editor`,查看魔数，修改后缀名，得到flag。
![](../img/Snipaste_2023-12-13_19-03-33.png)
![](../img/16进制文件类型.jpg)



