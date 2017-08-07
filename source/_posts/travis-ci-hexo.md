---
title: Travis CI自动持续化部署博客
date: 2017-08-07 16:34:52
tags:
- Travis CI
- Github
- Hexo
categories:
- Travis CI
- Blog
---
### 创建分支存储源代码
在本地创建分支 `hexo` 来存储源代码，在根目录执行如下命令:

```bash
git init
git config  user.name 'username'
git config user.email 'username@email'
git remote add origin `git@github.com:username/username.github.io.git`
git commit -am "fix"
git checkout -b 'hexo'
```
<!-- more -->

### 登录Travis CI
[Travis CI官网](https://travis-ci.org/)，使用GitHub账户登录。登录完成后点击左侧仓库列表上面的 `+` 号，找到`username.github.io` 这个项目,并点击前面的开关,让其处于打开状态。
![](</images/20170807165045.png)
![](</images/20170807164842.png)

### Github 生成 Personal access tokens
点击个人头像右边的三角符号，选择 `Setting` ，在左侧找到 `Personal access tokens`， 点击按钮 `Generate new token` 生成新的token ，记录下生成的token，命名随意。
![](</images/20170807165518.png)
![](</images/20170807165612.png)

### 设置Travis CI使用token自动部署
#### 设置Travis CI中的项目，在GitHub中使用push命令时自动部署。
![](</images/20170807165353.png)
![](</images/20170807165415.png)
#### 使用Github生成的  Personal access tokens
在项目设置界面创建环境变量，命名为GH_TOKEN，值为在Github 生成 的Personal access token的值。
![](</images/20170807165432.png)

### 创建配置文件 .travis.yml
在根目录创建文件.travis.yml，内容如下:
```
language: node_js  #设置语言

node_js: stable  #设置相应的版本

before_install:
- npm install -g hexo-cli # 安装 hexo

install:
  - npm install  #安装hexo及插件

script:
  - hexo clean  #清除
  - hexo g  #生成

after_script:
  - cd ./public
  - git init
  - git config user.name "jeskinfly"  #修改name
  - git config user.email "jeskinfly@163.com"  #修改email
  - git add .
  - git commit -m "update"
  - git push --force --quiet "https://${GH_TOKEN}@${GH_REF}" master:master  # GH_TOKEN是在Travis中配置token的名称

branches:
  only:
    - hexo  #只监测hexo分支，hexo是我的分支的名称，可根据自己情况设置
env:
 global:
   - GH_REF: github.com/jeskinfly/jeskinfly.github.io.git  #设置GH_REF，注意更改yourname
```