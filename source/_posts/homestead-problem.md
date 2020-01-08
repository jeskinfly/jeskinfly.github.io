---
title: homestead问题
date: 2020-01-08 14:27:31
tags:
- Laravel
categories:
- PHP
- laravel

---

### 问题描述
homestead启动时,卡在"SSH auth method: private key"的问题
```bash
E:\Work\Vagrant\Homestead (master)
vagrant up
...
homestead: SSH address: 127.0.0.1:2222
homestead: SSH username: vagrant
homestead: SSH auth method: private key
Timed out while waiting for the machine to boot. This means that
Vagrant was unable to communicate with the guest machine within
the configured ("config.vm.boot_timeout" value) time period.
...
```
<!-- more  -->

### 解决办法
#### 检查 SSH Key 填写是否正确
首先，查看homestead.yaml文件中 authorize 选项：
```bash
authorize: ~/.ssh/id_rsa.pub
```
请确定 客户端~/.ssh/id_rsa.pub 文件是否存在

再次，查看服务器中文件/home/vagrant/.ssh/authorized_keys是否保存了客户端公钥。

#### 重新生成 insecure_private_key
在 Homestead 文件夹下执行：
```bash 
vagrant ssh-config
```
会输出：
```bash
Host homestead-7
  HostName 127.0.0.1
  User vagrant
  Port 2222
  UserKnownHostsFile /dev/null
  StrictHostKeyChecking no
  PasswordAuthentication no
  IdentityFile E:/Work/Vagrant/Homestead/.vagrant/machines/homestead-7/virtualbox/private_key
  IdentitiesOnly yes
  LogLevel FATAL
  ForwardAgent yes

```
移除 IdentityFile 选项里的文件 E:/Work/Vagrant/Homestead/.vagrant/machines/homestead-7/virtualbox/private_key
重新运行 vagrant up（vagrant 会自动生成一个新 insecure_private_key 文件）。

#### 在 BIOS 中开启虚拟化技术支持
相关问题问百度

#### 重新生成虚拟机
确认是否开启虚拟化

执行命令
```bash
vagant destroy
vagrant up
```

#### 重置系统设置（Windows）
```bash
netsh advfirewall reset
netsh int ip reset
netsh int ipv6 reset
netsh winsock reset
```