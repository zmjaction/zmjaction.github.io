---
title: 字符编码-ASCII-Unicode-UTF8
date: 2020-03-21 18:11:32
categories:
    - 字符编码
tags:
    - ASCII
    - Unicode
---



#### ASCII

![](//tvax2.sinaimg.cn/large/d479ba17ly1gfytwuvab6j20oo0kljsx.jpg)







#### **扩展ASCII码**

```
1000 0000

...

1111 1111
```

当欧洲国家开始使用计算机的时候，发现原有的ASCII根本不够用，所在在原有的ASCII进行了扩展,把原来的第1位0变成了1





#### GB2312

当电脑来到中国时，256个字符也是太少了，就不能用8位表示，用16位表示一个字符

设计字符集:

使用分区管理，共计94个区，每个区含94位，共8836个码位

* 01-09区 收录除汉字外的682个字符                                                                                                 

* 10-15区为空白区，没有使用                                                                                                             

* 15-55区收录3755个一级汉字,按照拼音排序                                                                                      

* 56-87区收录3008个二级汉字，按照部首、笔画排序                                                                         

* 88-94区为空白区，没有使用                                                                                                             

![](//tva2.sinaimg.cn/large/d479ba17ly1gfytps1jvwj2082072gmj.jpg)

![](//tva4.sinaimg.cn/large/d479ba17ly1gfytqbjzk7j208206w0u4.jpg)

![](//tva2.sinaimg.cn/large/d479ba17ly1gfytqpnxfej207u06jab7.jpg)





![](//tva2.sinaimg.cn/large/d479ba17ly1gfytr5i0e7j20ai07b76a.jpg)

![](//tva1.sinaimg.cn/large/d479ba17ly1gfytrg9z1gj20au0730un.jpg)

![](//tva2.sinaimg.cn/large/d479ba17ly1gfytsgq70bj21870pk4qq.jpg)

![](//tvax1.sinaimg.cn/large/d479ba17ly1gfytrspti7j21930pj4qq.jpg)

参考：http://www.ruanyifeng.com/blog/2007/10/ascii_unicode_and_utf-8.html

