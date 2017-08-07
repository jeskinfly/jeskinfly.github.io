---
title: Hexo + Github + MarkDown + Subdomain 搭建博客
date: 2017-07-07 17:07:04
tags:
- Hexo
- Blog
- Github
- 静态站点
categories:
- GitHub
- 个人博客
---
### 安装NodeJs
下载及安装 [菜鸟教程 -- Node.js 安装配置](http://www.runoob.com/nodejs/nodejs-install-setup.html)

### 安装Git
下载及安装 [Git for Windows](https://git-scm.com/download/win)
#### 初次运行 Git 前的配置

<!-- more -->

``` bash
// 全局配置
$  git config --global user.name "USER NAME"
$  git config --global user.email "USER EMAIL"

// 配置具体项目，不带参数“--global”
$  cd project
$  git config user.name "USER NAME"
$  git config user.email "USER EMAIL"
```
### SSH配置(公钥登录)
　　**公钥登录**：用户将自己的公钥储存在远程主机上。登录的时候，远程主机会向用户发送一段随机字符串，用户用自己的私钥加密后，再发回来。远程主机用事先储存的公钥进行解密，如果成功，就证明用户是可信的，直接允许登录`shell`，不再要求密码。
　　**首先，你需要确认自己是否已经拥有密钥。**默认情况下，用户的 `SSH密钥` 存储在其 `~/.ssh` 目录下。其中一个带有 `.pub` 扩展名。 `.pub`文件是你的`公钥`，另一个则是`私钥`。如果找不到这样的文件（或者根本没有 `.ssh` 目录），你可以通过运行 `ssh-keygen` 程序来创建它们.在 Linux/Mac 系统中，`ssh-keygen` 随 SSH 软件包(`openssh`)提供；在 `Windows` 上，该程序包含于 `MSysGit` 软件包中。
#### 确认是否有密钥
``` bash
$ cd ~/.ssh
$ ls
authorized_keys2  id_dsa       known_hosts
config            id_dsa.pub
```
#### 生成SSH key: ssh-keygen
``` bash
$ ssh-keygen -t rsa -b 4096 -C "your_email@example.com"
// -C : comment 注释
// -t : type  加密方式，包括 dsa | ecdsa | ed25519 | rsa | rsa1
// -b : 加密字符串的长度
Generating public/private rsa key pair.
Enter file in which to save the key (/home/schacon/.ssh/id_rsa):
Created directory '/home/schacon/.ssh'.
Enter passphrase (empty for no passphrase):
Enter same passphrase again:
Your identification has been saved in /home/schacon/.ssh/id_rsa.
Your public key has been saved in /home/schacon/.ssh/id_rsa.pub.
The key fingerprint is:
d0:82:24:8e:d7:f1:bb:9b:33:53:96:93:49:da:9b:e3 schacon@mylaptop.local
```
　　首先 `ssh-keygen` 会确认密钥的存储位置（默认是 `.ssh/id_rsa`），然后它会要求你输入两次密钥口令。如果你不想在使用密钥时输入口令，将其留空即可。
#### 发送密钥到服务器
　　现在，需要将公钥`~/.ssh/id_rsa.pub`里面的内容发送给服务器。
　　如果是使用 `github` 或 `gitlab` ，可以通过网页在设置页面中添加，例如，`github`是点击个人头像右边三角形，进入 **Settings->SSH and GPG keys** 中添加`SSH key`。
　　如果是自己通过`openssh`搭建的服务器，可把公钥传送到远程主机host上面：
``` bash
ssh-copy-id user@host
```
　　远程主机将用户的公钥，保存在登录后的用户主目录的 `~/.ssh/authorized_keys` 文件中
#### 测试连接
```
ssh -T git@github.com
//成功
Hi jeskinfly! You've successfully authenticated, but GitHub does not provide shell access.
//失败
The authenticity of host 'github.com (192.30.252.1)' can't be established.RSA key fingerprint is 16:27:ac:a5:76:28:2d:36:63:1b:56:4d:eb:df:a6:48.Are you sure you want to continue connecting (yes/no)?
```
### Github
项目名称格式: username.github.io (username是你的账户名)

### 安装Hexo
``` bash
	$  npm install hexo-cli -g
	$  hexo init blog
	$  cd blog
	$  npm install
	$  hexo server
```
　　Hexo配置及使用参考官方 [Hexo中文站点](https://hexo.io/zh-cn/)。

### 配置自定义域名
假设域名: `blog.mealive.com`
1、在域名服务商添加CHNME记录,假设域名
![](</images/20170731144153.png)
2、在github中的*.github.io仓库的设置界面设置域名
![](</images/20170731144358.png)
3、在项目根目录新建文件CNAME,内容如下：
`blog.mealive.cn`
注意:不要 `http`,`https`,`www` 等前缀
	


