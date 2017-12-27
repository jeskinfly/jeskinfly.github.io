---
title: 阿里云虚拟主机配置子域名指向子目录
date: 2017-12-27 14:59:50
tags:
- 阿里云-虚拟主机
categories:
- 其他
---

### 背景
　　我在阿里云有一个免费赠送的虚拟主机，并有一个域名，想部署多个测试的项目，必要是可以用作项目展示，然后通过独立的子域名去访问。
　　不考虑增加经济支出，如升级虚拟主机为服务器啥的。

### 方案
　　利用web服务器(apache、nginx)的重写功能做重定向。
　　***前提你的空间服务器如果安装的web服务器是apache且rewrite功能，才能继续下一步。默认状态为开启。
　　nginx默认关闭.htaccess的加载，不作介绍。***

<!-- more  -->

### 步骤

#### 确认虚拟主机所使用的服务器
　　访问虚拟主机的任何可用链接，通过响应头判断所使用的服务器，是用的apache，还是nginx。
```html
	HTTP/1.1 304 Not Modified
	Date: Wed, 27 Dec 2017 07:12:08 GMT
	Server: Apache
	Connection: Keep-Alive
	Keep-Alive: timeout=15, max=300
	ETag: "7a04ae-a3a5-553cae160d402"
	Vary: Accept-Encoding,User-Agent
```
　　当前使用的是apache服务器，apache服务器安装时默认支持.htaccess。如果是nginx,默认是不支持的。
　　nginx配置文件nginx.conf默认：
```
    location ~ /\.ht {
        deny all;
    }
```

#### 根据服务器类型编写.htaccess文件，放在主机根目录

```sh
<IfModule mod_rewrite.c>
RewriteEngine On
RewriteBase /

# 绑定example.mealive.cn 到 example 子目录
RewriteCond %{HTTP_HOST} ^example\.mealive\.cn$ [NC]
RewriteCond %{REQUEST_URI} !^/example/
RewriteRule ^(.*)$ example/$1?Rewrite [L,QSA]
#可以绑定多个 只需重复上三行代码并更改一下域名、目录名 就好了

#绑定 example2.mealive.cn 到 example2 子目录
RewriteCond %{HTTP_HOST} ^example2\.mealive\.cn$ [NC]
RewriteCond %{REQUEST_URI} !^/example2/
RewriteRule ^(.*)$ example2/$1?Rewrite [L,QSA]

</IfModule>
```
#### 登录阿里云，在控制台添加域名解析
　　完成。