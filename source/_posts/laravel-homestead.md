---
title: laravel安装homestead要注意的几个问题
date: 2017-07-31 18:22:13
tags:
- Laravel
- Vagrant
- VirtualBox
- Homestead
- 自动化部署
categories:
- PHP
- Laravel

---

### 网络问题
国内因为网络的原因，默认的盒子安装命令 `vagrant box add laravel/homestead` 变得不适用。**可以先下载到本地后通过本地安装**。
**注意**，请勿使用迅雷离线下载，亲测下载后的文件会损坏。

<!-- more  -->
**获取下载链接**
执行命令`vagrant box add laravel/homestead`后会出来三个选项，选择`virtualbox`后出现在`Downloading`后的地址为下载链接。
```bash
$ vagrant box add laravel/homestead
==> box: Loading metadata for box 'laravel/homestead'
    box: URL: https://atlas.hashicorp.com/laravel/homestead
This box can work with multiple providers! The providers that it
can work with are listed below. Please review the list and choose
the provider you will be working with.

1) parallels
2) virtualbox
3) vmware_desktop

$ Enter your choice: 2
==> box: Adding box 'laravel/homestead' (v2.0.0) for provider: virtualbox
    box: Downloading: https://vagrantcloud.com/laravel/boxes/homestead/versions/2.0.0/providers/virtualbox.box
    box: Progress: 0% (Rate: 75458/s, Estimated time remaining: 9:56:08)==> box: Waiting for cleanup before exiting...
```
此处下载地址为: 
`https://vagrantcloud.com/laravel/boxes/homestead/versions/2.0.0/providers/virtualbox.box`

**从本地安装**
假设把下载下来的box命名为`homestead-virtualbox-2.0.0.box`，可以使用以下方法导入`.box` 文件：
1、在 .box 的同文件夹下创建一个 `metadata.json` 文件。json文件命名是随便的,此文件在后面的命令行中会用到。内容为以下:
``` json
{
    "name": "laravel/homestead",
    "versions": 
    [
        {
            "version": "2.0.0",
            "providers": [
                {
                  "name": "virtualbox",
                  "url": "homestead-virtualbox-2.0.0.box" // eg.  "url": "file:///e:/boxes/homestead-virtualbox-2.0.0.box"
                }
            ]
        }
    ]
}
```
字段说明:
- version - 可以指定当前盒子导入的版本标示；
- url - 支持 绝对文件路径 和 相对文件路径
	如 "url": "file:///e:/boxes/homestead-virtualbox-2.0.0.box"

2、运行以下命令导入，使用新建的json文件导入盒子：
```bash
vagrant box add metadata.json
```

3、运行 list 命令查看是否添加成功：
```bash
vagrant box list
```

### 执行 `vagrant up` 报错
执行 `vagrant up` 报错 The requested address is not valid
```bash
$ vagrant up
Bringing machine 'homestead-7' up with 'virtualbox' provider...
==> homestead-72: Importing base box 'laravel/homestead2'...
==> homestead-72: Matching MAC address for NAT networking...
==> homestead-72: Checking if box 'laravel/homestead2' is up to date...
==> homestead-72: Setting the name of the VM: homestead-72
==> homestead-72: Destroying VM and associated drives...
C:/HashiCorp/Vagrant/embedded/gems/gems/vagrant-1.9.3/lib/vagrant/util/is_port_open.rb:21:in `initialize': The requested address is not valid in its context. - connect(2) for "0.0.0.0" port 8000 (Errno::EADDRNOTAVAIL)
```
默认会绑定所有IP就是提示里面的 `0.0.0.0` ，改成只是我本地和虚拟机的转发。
修改文件 `./scripts/homestead.rb` 的网络配置
```ruby
        # Use Default Port Forwarding Unless Overridden
        unless settings.has_key?("default_ports") && settings["default_ports"] == false
            default_ports.each do |guest, host|
                unless settings["ports"].any? { |mapping| mapping["guest"] == guest }
                    # 原来的
                    # config.vm.network "forwarded_port", guest: guest, host: host, auto_correct: true, host_ip: '0.0.0.0'
                    # 修改后
                    config.vm.network "forwarded_port", guest: guest, host: host, auto_correct: true, host_ip: '127.0.0.1'
                end
            end
        end
```

### 同步读取慢 
window7 下使用`rsync`，解决`vagrant` 共享目录读取速度慢的问题，采用的是一次性单向同步。
在 `Vagrantfile` 文件中设置同步目录为 `rsync`
``` ruby
config.vm.synced_folder ".", "/vagrant", type: "rsync", rsync__auto: true
```
- 下载安装 `Cygwin` 或 `MinGW` ，然后查找出 `rsync` 安装。
- 把 `rsync.exe` 的路径加入 windows的 `path` 环境。
- 执行 `vagrant reload` ，现在可以享受 windows下的vbox极速共享目录了。


### 自定义虚拟机的名称
```ruby
 config.vm.provider "virtualbox" do |vb|
  # add a line
  # Define this vm's name
      vb.name = "centos_7"   # 自定义在虚拟机中显示的名称
  #   # Display the VirtualBox GUI when booting the machine

  #   vb.gui = true
  #
  #   # Customize the amount of memory on the VM:
  #   vb.memory = "1024"
  end
```