---
title: 程序如何支持emoji表情？ 
date: 2017-10-09 11:18:25
tags: 
- emoji
---
### Unicode简述
　　Unicode（中文：万国码、国际码、统一码、单一码）：把世界上所有语言的文字和所有的符号（如高音谱号、麻将、emoji）用同一套编码表示出来。

### Unicode中的平面
　　Unicode字符分为17组编排，0x0000至0xFFFF，每组称为平面（Plane），而每平面拥有65536个码位，共1114112个。
　　在表示一个Unicode的字符时，通常会用“U+”然后紧接着一组十六进制的数字来表示这一个字符。

![](/images/20171009161417.png)

　　
　　`第０平面`也叫做BMP（Basic Multilingual Plane，基本多文种平面），编码从U+0000至U+FFFF，最为重要，存放着世界上各种语言与标记中最常用的字符。
　　在基本多文种平面（第０平面）里的所有字符，要用`4`个字（即两个char,16bit,例如U+4AE0，共支持六万多个字符）；在零号平面以外的字符则需要使用`5-6`个字。
　　`第１平面`也叫做SMP（Supplementary Multilingual Plane，多文种补充平面），编码从U+10000到U+1FFFD，放着拼音文字、绘文字（[emoji](https://zh.wikipedia.org/wiki/%E7%B9%AA%E6%96%87%E5%AD%97)）、字母与数学符号、音符、交通和地图符号、麻将牌、扑克牌、多米诺骨牌、象形文字、楔形文字和其他图形符号。
　　`第２平面`也叫做SIP（Supplementary Ideographic Plane，表意文字补充平面），编码从U+20000到U+2FFFD，用于存放汉字中的罕用的汉字或地区的方言用字（如粤语用字及越南语的字喃）。

### Unicode中的统一汉字
　　BMP存放着最常用字与扩展A区的汉字。扩展B区到即将到来的扩展E区都放在SIP中。
　　在这些区中，除了独立字源的字，还有同一个字源或部首不同的变体或写法。如“ɑ／a”、“強／强”、“戶／户／戸”。这些差异也会在Unicode中用三个不同的编码去表示。所以B区到E区有不少此种字体。

### 实现方式
　　Unicode的实现方式不同于编码方式。一个字符的Unicode编码是确定的。但是在实际传输过程中，由于不同系统平台的设计不一定一致，以及出于节省空间的目的，对Unicode编码的实现方式有所不同。Unicode的实现方式称为Unicode转换格式（Unicode Transformation Format，简称为UTF）。UTF-8、UTF-16、UTF-32都是将数字转换到程序数据的编码方案。

### 例如UTF-8
　　如果一个仅包含基本7位ASCII字符的Unicode文件，如果每个字符都使用2字节的原Unicode编码传输，其第一字节的8位始终为0。这就造成了比较大的浪费。对于这种情况，可以使用UTF-8编码，这是一种变长编码，它将基本7位ASCII字符仍用7位编码表示，占用一个字节（首位补0）。而遇到与其他Unicode字符混合的情况，将按一定算法转换，每个字符使用1-3个字节编码，并利用首位为0或1进行识别。这样对以7位ASCII字符为主的西文文档就大幅节省了编码长度（具体方案参见[UTF-8](https://zh.wikipedia.org/wiki/UTF-8)）。类似的，对未来会出现的需要4个字节的辅助平面字符和其他UCS-4扩充字符，2字节编码的UTF-16也需要通过一定的算法进行转换。

**UTF-8使用一至六个字节为每个字符编码：**
- 128个US-ASCII字符只需一个字节编码（Unicode范围由U+0000至U+007F）。
- 带有附加符号的拉丁文、希腊文、西里尔字母、亚美尼亚语、希伯来文、阿拉伯文、叙利亚文及它拿字母则需要两个字节编码（Unicode范围由U+0080至U+07FF）。
- 其他基本多文种平面（BMP）中的字符（这包含了大部分常用字，如大部分的汉字）使用三个字节编码（Unicode范围由U+0800至U+FFFF）。
- 其他极少使用的Unicode 辅助平面的字符使用四至六字节编码（Unicode范围由U+10000至U+1FFFFF使用四字节，Unicode范围由U+200000至U+3FFFFFF使用五字节，Unicode范围由U+4000000至U+7FFFFFFF使用六字节）。

### 解决方式
　　emoji表情在Unicode中称为绘文字,处于第１平面,编码范围为U+10000~U+1FFFD，使用UTF-8编码需要4个字节。同样的有粤语用词，也是占用4个字节。
　　utf8_unicode_ci最多支持3个字节的Unicode编码，而utf8mb4_unicode_ci则最多支持4个字节的编码。所以utf8mb4系列覆盖了整个Unicode编码,utf8mb4_unicode_ci是utf8_unicode_ci的超集。但是由于支持的字符集更大，utf8mb4的性能要比utf8系列的字符集低。
　　MySQL从 5.5 之后开始支持utf8mb4系列的字符集。

### 浏览器显示表情问题
#### 无法显示
　　在手机、IE11下，该emoji均可正常显示，但是Windows的Chrome下显示为“口"所以",解决方法是在CSS中font-family中加上”Segoe UI Emoji”，这样根据font-family的回退规则，浏览器会自动以Segoe UI Emoji字体显示表情，Win7开始就支持该字体。


### 参考
[Unicode](https://zh.wikipedia.org/wiki/Unicode)
[Unicode字符平面映射](https://zh.wikipedia.org/wiki/Unicode%E5%AD%97%E7%AC%A6%E5%B9%B3%E9%9D%A2%E6%98%A0%E5%B0%84)
[UTF-8](https://zh.wikipedia.org/wiki/UTF-8)
[绘文字](https://zh.wikipedia.org/wiki/%E7%B9%AA%E6%96%87%E5%AD%97)
