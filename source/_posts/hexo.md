---
title: Hexo + Github + MarkDown + Subdomain 搭建博客
date: 2017-07-07 17:07:04
tags:
- Hexo
- Blog
- Github
- 静态站点
categories:
- Github
- blog
---
### 安装NodeJs
下载及安装 [菜鸟教程 -- Node.js 安装配置](http://www.runoob.com/nodejs/nodejs-install-setup.html)

### 安装Git
下载及安装 [Git for Windows : git-scm](https://git-scm.com/download/win) 或 [ Git for Windows : git-github ](https://git-for-windows.github.io/)。
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
GitHub Pages包含两种类型: `User/Organization Pages` and `Project Pages`。自 2016.6.16 起，支持 `HTTPS` 协议。
User Pages site，默认用来构建和发布静态页面的分支是 `master` 分支 。
在Github中创建User Pages项目, 名称格式: `username.github.io` ( `username` 是你的账户名)。

### 安装Hexo
``` bash
  $  npm install hexo-cli -g
  $  hexo init blog
  $  cd blog
  $  npm install
  $  hexo server
```
　　Hexo配置及使用参考官方 [Hexo中文站点](https://hexo.io/zh-cn/)。

#### 配置
配置根目录 **站点配置文件** : `_config.yml`。
```yml
# Site
title: Jeskinfly's blog
subtitle: Quick notes
description: Jeskinfly's blog
author: Jeskinfly
language: zh-Hans
timezone: UTC

# URL
## If your site is put in a subdirectory, set url as 'http://yoursite.com/child' and root as '/child/'
url: http://blog.mealive.cn/

...

# Extensions
## Plugins: https://hexo.io/plugins/
## Themes: https://hexo.io/themes/
theme: next  // 使用的主题

# Deployment
## Docs: https://hexo.io/docs/deployment.html
deploy:
  type: git  //使用git部署
  repo: git@github.com:jeskinfly/jeskinfly.github.io.git
branch: master

search:   // 站内搜索
  path: search.xml
  field: post
  format: html
  limit: 10000

```
以上列出部分重点内容，其他可使用默认值，具体参考[官方文档](https://hexo.io/zh-cn/docs/configuration.html)。

#### 部署
- Git
安装 `hexo-deployer-git`。
```
$ npm install hexo-deployer-git --save
```

**注**: --save参数可追加依赖关系到 `package.json` 文件中，这样在Travis CI中配置自动化部署时，只需在配置文件 `.travis.yml` 的`install:`部分为:
```
install:
  - npm install  #安装hexo及插件
```
否则要使用:
```
install:
  - npm install  #安装hexo依赖
  - npm install hexo-deployer-git 安装部署插件
  - npm install hexo-generator-searchdb  安装站点搜索插件
```

#### 主题
创建 `Hexo主题` 非常容易，您只要在 `themes` 文件夹内，新增一个任意名称的文件夹，并修改 `站点配置文件` 内的 theme 设定，即可切换主题。

#### NexT主题
安装 `NexT主题` ,点击下载 [**NexT主题**](https://github.com/iissnan/hexo-theme-next) 。
下载后，放在根目录下的 `themes` 文件夹下,并命名文件为 `next`。

- 修改默认的样式
在主题下的 `/themes/next/source/_custom/custom.styl` 文件中增加样式覆盖默认样式。

- NexT主题配置(未安装此主题请忽略)
**修改 **主题配置文件** --> `/themes/next/_config.yml` 。具体修改请点击查看 [NexT主题官方文档](http://theme-next.iissnan.com/getting-started.html#sidebar-settings)**

- 添加「标签」页面
在 `_posts` 同级新增目录并命名为 `tags` ，新建 `index.md` 文件,里面内容如下:
```
---
title: 标签
date: 2017-07-07 12:22:11
type: "tags"
---
```

- 添加「分类」页面
在 `_posts` 同级新增目录并命名为 `categories` ,新建 `index.md` 文件,里面内容如下:
```
---
title: 分类
date: 2017-07-07 12:23:36
type : "categories"
---
```

- 腾讯公益404页面
新建 404.html 页面，放到主题的 source 目录下，内容如下：
```html
<!DOCTYPE HTML>
<html>
<head>
  <meta http-equiv="content-type" content="text/html;charset=utf-8;"/>
  <meta http-equiv="X-UA-Compatible" content="IE=edge,chrome=1" />
  <meta name="robots" content="all" />
  <meta name="robots" content="index,follow"/>
  <link rel="stylesheet" type="text/css" href="https://qzone.qq.com/gy/404/style/404style.css">
</head>
<body>
  <script type="text/plain" src="http://www.qq.com/404/search_children.js"
          charset="utf-8" homePageUrl="/"
          homePageName="回到我的主页">
  </script>
  <script src="https://qzone.qq.com/gy/404/data.js" charset="utf-8"></script>
  <script src="https://qzone.qq.com/gy/404/page.js" charset="utf-8"></script>
</body>
</html>
```
#### NexT主题第三方服务
1、阅读次数统计（LeanCloud） 由 Doublemine 贡献
请查看 [为NexT主题添加文章阅读量统计功能](https://notes.wanghao.work/2015-10-21-%E4%B8%BANexT%E4%B8%BB%E9%A2%98%E6%B7%BB%E5%8A%A0%E6%96%87%E7%AB%A0%E9%98%85%E8%AF%BB%E9%87%8F%E7%BB%9F%E8%AE%A1%E5%8A%9F%E8%83%BD.html#%E9%85%8D%E7%BD%AELeanCloud)
2、Local Search 由 flashlab 贡献
- 安装 hexo-generator-searchdb，在站点的根目录下执行以下命令：
```
$ npm install hexo-generator-searchdb --save
```
- 编辑 **站点配置文件**，新增以下内容到任意位置：
```
search:
  path: search.xml
  field: post
  format: html
  limit: 10000
```
- 编辑 **主题配置文件**，启用本地搜索功能：
``` 
# Local search
local_search:
  enable: true
```

### 配置自定义域名
假设域名: `blog.mealive.com`
1、在域名服务商添加 `CHNME记录`，假设域名
![](</images/20170731144153.png)
2、在 `github` 中的 `*.github.io` 仓库的设置界面设置域名
![](</images/20170731144358.png)
3、在项目根目录新建文件CNAME,内容如下：
`blog.mealive.cn`

**注意**: 
1、不要 `http`,`https`,`www` 等前缀。
2、可用在 `_posts` 目录同级下创建文件 `CNAME`，每次执行 `hexo d` 命令, 会自动复制一份到 `public` 目录中。

您可能还会关注: 使用 [**Travis CI自动持续化部署博客**](/2017/08/08/travis-ci-hexo/) 。