---
title: robots.txt详解
date: 2017-08-08 11:30:32
tags:
- 搜索引擎
categories:
- 搜索引擎
---

### 什么是robots.txt文件?

　　`robots.txt` 文件是用来禁止搜索引擎收录的方法。**原则上文件中不写禁止指令则为允许访问**。<!-- more -->
　　搜索引擎使用 `spider程序` 自动访问互联网上的网页并获取网页信息。
　　`spider` 在访问一个网站时，首先会检查该网站的根域下是否有一个叫做 `robots.txt` 的纯文本文件，这个文件用于指定 `spider` 在您网站上的抓取范围。您可以在您的网站中创建一个 `robots.txt` ，在文件中声明 该网站中不想被搜索引擎收录的部分或者指定搜索引擎只收录特定的部分。 
　　请注意，仅当您的网站包含不希望被搜索引擎收录的内容时，才需要使用 `robots.txt`文件。如果您希望搜索引擎收录网站上所有内容，请勿建立 `robots.txt` 文件。

### robots.txt文件放在哪里?

　　`robots.txt`文件应该放置在 **`网站根目录`** 下。

### meta标签
#### 简介

meta标签中**没有大小写之分**，`name="robots"` 表示所有的搜索引擎，`content` 部分有四个指令选项：`index`、`noindex`、`follow`、`nofollow`，指令间以 `,` 分隔。

- `index` ：告诉搜索机器人抓取该页面，`noindex` 相反；
- `follow`：表示搜索机器人可以沿着该页面上的链接继续抓取下去，`nofollow` 相反；

缺省值是 `index` 和 `follow` 。

这样，一共有四种组合：

```
<meta name="robots" content="index,follow">
<meta name="robots" content="noindex,follow">
<meta name="robots" content="index,nofollow">
<meta name="robots" content="noindex,nofollow">
```
其中
```
<meta name="robots" content="index,follow">
等同于 <meta name="robots" content="all">
<meta name="robots" content="noindex,nofollow">
等同于<meta name="robots" content="none">
```

　　**注意：绝大多数的搜索引擎机器人都遵守robots.txt的规则，而对于Robots meta标签，目前支持的并不多。**搜索引擎Google完全支持，还增加了一个指令“archive”，可以限制Google是否保留网页快照。

```
<!-- 表示抓取该站点中页面并沿着页面中链接抓取，但是不在Google上保留该页面的网页快照。-->
<meta name="googlebot" content="index,follow,noarchive">
```




#### 如何禁止跟踪网页的链接，而只对网页建索引？

- 如果您不想搜索引擎追踪此网页上的链接，且不传递链接的权重，请将此元标记置入网页的 部分：

```
<meta name="robots" content="index,nofollow">
```

- 如果您不想百度追踪某一条 **特定链接** ，百度还支持更精确的控制，请将此标记直接写在某条链接上：
```
<a href="signin.php" rel="nofollow">sign in</a>
```

- 要允许其他搜索引擎跟踪，但 **仅防止百度** 跟踪您网页的链接，请将此元标记置入网页的 部分：
```
<meta name="Baiduspider" content="index,nofollow">
```

#### 如何禁止在搜索结果中显示网页快照，而只对网页建索引？

- 要防止 **所有搜索引擎** 显示您网站的快照，请将此元标记置入网页的部分： 
```
<meta name="robots" content="noarchive">
```
- 要允许其他搜索引擎显示快照，但 **仅防止百度显示** ，请使用以下标记：
```
<meta name="Baiduspider" content="noarchive">
```
注：此标记只是禁止百度显示该网页的快照，百度会继续为网页建索引，并在搜索结果中显示网页摘要。

### robots.txt文件的格式
#### 注释
在该文件中可以使用 `#` 进行注解，具体使用方法和UNIX中的惯例一样。
#### User-agent:
　　该项的值用于描述搜索引擎 `robot` 的名字。
　　如果有 `多条User-agent记录` 说明有 `多个robot` 会受到 "robots.txt" 的限制，对该文件来说，**至少要有一条 User-agent 记录**。
　　如果该项的值设为 `*` ，则对任何 robot 均有效，
　　如果在 "robots.txt" 文件中，加入 "User-agent:SomeBot" 和若干 Disallow 、 Allow 行，那么名为 "SomeBot" 只受到 "User-agent:SomeBot" 后面的 Disallow 和 Allow 行的限制。
#### Disallow:
　　该项的值用于描述不希望被访问的一组URL，这个值可以是一条完整的路径，也可以是路径的非空前缀，以Disallow项的值开头的URL不会被 robot 访问。
例如：
1、`"Disallow:/help"`：禁止robot访问/help.html、/helpabc.html、/help /index.html。
2、`"Disallow:/help/"`：则允许robot访问/help.html、/helpabc.html，不能访问 /help/index.html。
3、`"Disallow:"`：说明允许robot访问该网站的所有url。
　　在"/robots.txt"文件中，至少要有一 条Disallow记录。如果"/robots.txt"不存在或者为空文件，则对于所有的搜索引擎robot，该网站都是开放的。
#### Allow:
　　该项的值用于描述希望被访问的一组URL，与Disallow项相似，这个值可以是一条完整的路径，也可以是路径的前缀，以Allow项的值开头的URL是允许robot访问的。
例如：
`"Allow:/hibaidu"`：允许robot访问/hibaidu.htm、/hibaiducom.html、 /hibaidu/com.html。
　　一个网站的所有URL默认是Allow的，所以Allow通常与Disallow搭配使用，实现允许访问一部分网页 同时禁止访问其它所有URL的功能。
#### 使用 "\*" 和 "$" ：
Baiduspider支持使用通配符"\*"和"$"来模糊匹配url。
`"$"`： 匹配行结束符。
`"*"`： 匹配0或多个任意字符。

** 请注意：区分您不想被抓取或收录的目录的大小写 **

### 用法举例


- 禁止所有
  ```
  User-agent: * 
  Disallow: /
  ```
- 允许所有
  ```
  User-agent: *
  Disallow: 
  或者
  User-agent: *
  Allow: /
  ```
- 禁止访问特定目录
  ```
  User-agent: *
  Disallow: /cgi-bin/
  Disallow: /tmp/
  Disallow: /~joe/
  ```
  　　在这个例子中，该网站有三个目录对搜索引擎的访问做了限制，即robot不会访问这三个目录。需要注意的是对每一个目录必须分开声明，而不能写成 "Disallow: /cgi-bin/ /tmp/"。
- 允许访问特定目录
  ```
  User-agent: *
  Allow: /cgi-bin/see
  Allow: /tmp/hi
  Allow: /~joe/look
  ```
- 使用正则
  1、使用"*"限制访问url
  禁止访问 `/cgi-bin/` 目录下的所有以`".htm"`为后缀的URL(包含子目录)。
  ```
  User-agent: *
  Disallow: /cgi-bin/*.htm
  ```
  2、使用`"$"`限制访问url
  仅允许访问以`".htm"`为后缀的URL。
  ```
  User-agent: *
  Allow: .htm$
  Disallow: /
  ```
  3、禁止访问网站中所有的动态页面
  ```
  User-agent: *
  Disallow: /*?*
  ```
  4、禁止抓取图片
  仅允许抓取网页，禁止抓取任何图片。
  ```
  User-agent: *
  Disallow: .jpg$
  Disallow: .jpeg$
  Disallow: .gif$
  Disallow: .png$
  Disallow: .bmp$
  ```
  5、允许抓取特定格式的图片
  允许抓取网页和gif格式图片，不允许抓取其他格式图片
  ```
  User-agent: Baiduspider
  Allow: .gif$
  Disallow: .jpg$
  Disallow: .jpeg$ 
  Disallow: .png$
  Disallow: .bmp$
  ```
  6、禁止抓取特定格式的图片
  ```
  User-agent: Baiduspider
  Disallow: .jpg$
  ```

### 网站管理员工具
Google提供了 [robots.txt分析工具](http://www.google.cn/support/webmasters/bin/answer.py?hlrm=en&answer=35237)。它可以按照 Googlebot 读取 robots.txt 文件的相同方式读取该文件，并且可为 Google user-agents（如 Googlebot）提供结果。我们强烈建议您使用它。



### 误区

*误区一：我的网站上的所有文件都需要蜘蛛抓取，那我就没必要在添加robots.txt文件了。反正如果该文件不存在，所有的搜索蜘蛛将默认能够访问网站上所有没有被口令保护的页面。*

　　每当用户试图访问某个不存在的URL时，服务器都会在日志中记录404错误（无法找到文件）。每当搜索蜘蛛来寻找并不存在的robots.txt文件时，服务器也将在日志中记录一条404错误，所以你应该做网站中添加一个robots.txt。*

*误区二：在robots.txt文件中设置所有的文件都可以被搜索蜘蛛抓取，这样可以增加网站的收录率。*

　　网站中的程序脚本、样式表等文件即使被蜘蛛收录，也不会增加网站的收录率，还只会浪费服务器资源。因此必须在robots.txt文件里设置不要让搜索蜘蛛索引这些文件。

*误区三：搜索蜘蛛抓取网页太浪费服务器资源，在robots.txt文件设置所有的搜索蜘蛛都不能抓取全部的网页。*

　　如果这样的话，会导致整个网站不能被搜索引擎收录。



### 使用技巧

1. 每当用户试图访问某个不存在的URL时，服务器都会在日志中记录404错误（无法找到文件）。每当搜索蜘蛛来寻找并不存在的robots.txt文件时，服务器也将在日志中记录一条404错误，所以你应该在网站中添加一个robots.txt。

2. 网站管理员必须使蜘蛛程序远离某些服务器上的目录 —— 保证服务器性能。比如：大多数网站服务器都有程序储存在“cgi-bin”目录下，因此在robots.txt文件中加入“Disallow: /cgi-bin”是个好主意，这样能够避免将所有程序文件被蜘蛛索引，可以节省服务器资源。一般网站中不需要蜘蛛抓取的文件有：后台管理文件、程序脚本、附件、数据库文件、编码文件、样式表文件、模板文件、导航图片和背景图片等等。
3. 如果你的网站是动态网页，并且你为这些动态网页创建了静态副本，以供搜索蜘蛛更容易抓取。那么你需要在robots.txt文件里设置避免动态网页被蜘蛛索引，以保证这些网页不会被视为含重复内容。

4. robots.txt文件里还可以直接包括在sitemap文件的链接。就像这样：
      Sitemap: sitemap.xml
      　　目前对此表示支持的搜索引擎公司有Google, Yahoo, Ask and MSN。而中文搜索引擎公司，显然不在这个圈子内。这样做的好处就是，站长不用到每个搜索引擎的站长工具或者相似的站长部分，去提交自己的sitemap文件，搜索引擎的蜘蛛自己就会抓取robots.txt文件，读取其中的sitemap路径，接着抓取其中相链接的网页。
5. 合理使用robots.txt文件还能避免访问时出错。比如，不能让搜索者直接进入购物车页面。因为没有理由使购物车被收录，所以你可以在robots.txt文件里设置来阻止搜索者直接进入购物车页面。


