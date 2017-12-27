---
title: PHP OPcache 性能加速
date: 2017-08-01 16:54:52
tags:
- PHP
categories:
- PHP
- 性能
---

### 简介
　　OPcache 通过将 PHP 脚本预编译的字节码存储到共享内存中来提升 PHP 的性能， 存储预编译字节码的好处就是 省去了每次加载和解析 PHP 脚本的开销。
　　`PHP 5.5.0` 及后续版本中已经绑定了 `OPcache 扩展`。 对于 `PHP 5.2`，`5.3` 和 `5.4` 版本可以使用 » `PECL` 扩展中的 `OPcache` 库。

### 安装
#### PHP 5.5.0 及后续版本
　　编辑　`PHP` 配置文件 `php.ini`, 取消对应的 `zend_extension` 前的注释 ( 删除前面的`;` )
　　在 `非 Windows` 平台使用 `zend_extension=/full/path/to/opcache.so`， 
　　`Windows 平台`使用 `zend_extension=C:\path\to\php_opcache.dll`。
#### PHP 5.2, 5.3 和 5.4 版本
**升级PHP版本到5.5或以上** 或 手动安装此 PECL 扩展。

### 配置
Opcache的推荐配置:
```ini
zend_extension=opcache.so　　　　　 // 非Windows平台开启扩展
opcache.enable_cli=1
opcache.memory_consumption=128      // 共享内存大小, 这个根据你们的需求可调
opcache.interned_strings_buffer=8   // interned string的内存大小, 也可调
opcache.max_accelerated_files=4000  // 最大缓存的文件数目
opcache.revalidate_freq=60          // 60s检查一次文件更新
opcache.fast_shutdown=1             // 打开快速关闭, 打开的时候,
                                       回收内存的速度会提高
opcache.save_comments=0             // 不保存文件/函数的注释
```
### 重启服务器
重启服务器`apache`或`Nginx`即可

### 小提示
　　1、不建议在开发过程中开启 `Opcache`。
　　2、如果在更新代码之后，发现没有执行的还是旧代码，可使用函数 `opcache_reset()` 来清除缓存。该函数将重置整个字节码缓存。 在调用 `opcache_reset()` 之后，所有的脚本将会重新载入并且在下次被点击的时候重新解析。
　　3、通过图像查看各项指标和管理 `OPcache` , 可通过 `Github` 开源项目 [PeeHaa/OpCacheGUI](https://github.com/PeeHaa/OpCacheGUI);