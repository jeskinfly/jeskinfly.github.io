---
title: homestead.yaml配置详解
date: 2020-01-08 14:15:59
tags:
- Laravel
categories:
- PHP
- laravel

---

```
# 指定 IP ，允许多台虚拟机并存，除了指定 IP 以外，可用选项还有：autonetwork 
ip: "192.168.10.10"

# Homestead.yaml 中的 networks 属性为 Homestead 环境配置网络接口。你可以根据需要配置多个接口：
# https://learnku.com/docs/laravel/5.8/homestead/3882#network-interfaces
networks:
    - type: "public_network"
      ip: "192.168.10.20"
      bridge: "en1: Wi-Fi (AirPort)"

<!-- more  -->

# 设置虚拟机内存大小
memory: 2048

# 设置虚拟机多少核 CPU，
cpus: 1

# 虚拟机的「主机名词」，默认是`homestead`，登录虚拟机后
# 会显示 vagrant@homestead
hostname: homestead

# 虚拟机盒子的名称，这是虚拟机软件创建时为虚拟机设置的名词，为 homestead-7
name: homestead 

# 目前支持四个虚拟机类型：virtualbox, vmware_desktop, parallels, hyperv
# 请见 https://app.vagrantup.com/laravel/boxes/homestead
provider: virtualbox

# box 用来作为初始化虚拟机的 vagrant box 名称
# version 指定虚拟机盒子版本，与 box 参数挂钩
box: laravel/homestead
version: 7.0.0  

# 当你的 Vagrant 盒子销毁时，Homestead 能自动备份你的数据库
# 请见文档 https://learnku.com/docs/laravel/5.8/homestead/3882#62d8f8
backup: true

# 虚拟机认证相关，免密码登录
# authorize 填写主机公钥文件路径，虚拟机初始化时，会被复制到虚拟机
# 的 /home/vagrant/.ssh/authorized_keys文件中，从而实现 SSH 免密码登录。
authorize: ~/.ssh/id_rsa.pub

# 填本机的 SSH 私钥文件地址。虚拟机初始化时，私钥文件复制到虚拟机
# 的 /home/vagrant/.ssh/ 文件夹中，使虚拟机能共享主机上的 SSH 私钥文件，
# 使虚拟机具备等同于主机的身份认证。如你主机已经设置好了 GitHub 认证，
# 虚拟机也会同步获取到 GitHub 的认证。
keys:
    - ~/.ssh/id_rsa

# 文件夹映射，将主机中的文件夹映射到虚拟机里，方便在虚
# 拟机读取和操作主机的文件
folders:
    - map: ~/code # 主机里的目录
      # 对应映射到的虚拟机目录，也就是说，在此目录下将映射
      # 与 ~/code 一致的目录结构
      to: /home/vagrant/code 
      # NFS 模式映射文件夹，可以有效提高效率，可选参数是 rsync,smb
      type: "nfs"
    - map: "~Project/myapp"
      to: "/home/vagrant/myapp"
      type: "rsync"
      # Vagrant 提供的文件夹挂载选项，请见
      # https://www.vagrantup.com/docs/synced-folders/basic_usage.html
      options:
          rsync__args: ["--verbose", "--archive", "--delete", "-zz"]
          rsync__exclude: ["node_modules"]

# 添加站点，默认将会生成 Nginx 站点，详见
# https://learnku.com/docs/laravel/5.8/homestead/3882#97ea16
sites:
    - map: learnku.test # 域名地址
      to: "/home/vagrant/code/learnku/public"   # 网站的根目录
      php: '5.6'  # 选择 PHP 版本, 每个站点可以选择不同的 PHP 版本, 此站点项目选择 php 5.6 版本
      # 计划任务，为一个 Homestead 站点的 schedule:run 命令运行起来
      schedule: true 
      # 为 Nginx 添加 fastcgi_param 参数
      params:
          - key: FOO
            value: BAR
      # 站点类型，默认是 Laravel
      # 所有类型请见：https://learnku.com/docs/laravel/5.8/homestead/3882#8681e1
      type: "laravel"
    - map: myapp.test
      to: "/home/vagrant/myapp/public"
      php: '7.1'  # 此站点项目选择 php 7.1 版本

databases:
    - myapp
    - myapp

# 设置 PHP 全局环境变量，
variables:
    - key: 'APP_ENV'
      value: 'local'
    - key: 'APP_DEBUG'
      value: 'true'

# 端口转发
ports:
    - send: 50000   # 主机端口
      to: 5000      # 虚拟机端口
    - send: 7777
      to: 777
      protocol: udp # 端口转发类型
```