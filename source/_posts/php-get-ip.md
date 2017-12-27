---
title: PHP获取真实IP
date: 2017-12-27 17:34:24
tags:
- PHP
categories:
- PHP
- IP
---

### 变量
- $_SERVER['HTTP_CLIENT_IP'] 通过请求头传递，可伪造。
- $_SERVER['HTTP_X_FORWARDED_FOR'] 格式：`clientip,proxy1,proxy2`。通过请求头传递，可伪造。
- $_SERVER['REMOTE_ADDR'] 可信的。它是与服务器握手的最后一个ip，可能是用户的真是ip，也可能是用户的代理服务器或自己的反向代理服务器ip地址。

### 反向代理服务器传递ip

```sh
location /{
	proxy_set_header client-real-ip $remote_addr
}
```
