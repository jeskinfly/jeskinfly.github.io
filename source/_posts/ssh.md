---
title: SSH登录原理
date: 2020-01-08 12:35:40
tags:
- Linux
categories:
- Linux

---

### 口令登录

假定你要以用户名user，登录远程主机host。

```bash
$ ssh user@host
```

SSH的默认端口是22，也就是说，你的登录请求会送进远程主机的22端口。使用p参数，可以修改这个端口。

```bash
$ ssh -p 2222 user@host
```

如果你是第一次登录对方主机，系统会出现下面的提示：

<!-- more  -->

``` bash
$ ssh user@host

The authenticity of host 'host (12.18.429.21)' can't be established.

RSA key fingerprint is 98:2e:d7:e0:de:9f:ac:67:28:c2:42:2d:37:16:58:4d.

Are you sure you want to continue connecting (yes/no)?
```

这段话的意思是，无法确认host主机的真实性，只知道它的公钥指纹，问你还想继续连接吗？

所谓"公钥指纹"，是指公钥长度较长（这里采用RSA算法，长达1024位），很难比对，所以对其进行MD5计算，将它变成一个128位的指纹。
上例中是`98:2e:d7:e0:de:9f:ac:67:28:c2:42:2d:37:16:58:4d`，再进行比较，就容易多了。

很自然的一个问题就是，用户怎么知道远程主机的公钥指纹应该是多少？回答是没有好办法，远程主机必须在自己的网站上贴出公钥指纹，以便用户自行核对。

假定经过风险衡量以后，用户决定接受这个远程主机的公钥。

```bash
Are you sure you want to continue connecting (yes/no)? yes
```

系统会出现一句提示，表示host主机已经得到认可。

```bash 
Warning: Permanently added 'host,12.18.429.21' (RSA) to the list of known hosts.
```

然后，会要求输入密码。

```bash
Password: (enter password)
```

如果密码正确，就可以登录了。

**当远程主机的公钥被接受以后，它就会被保存在文件`$HOME/.ssh/known_hosts`之中**。下次再连接这台主机，系统就会认出它的公钥已经保存在本地了，从而跳过警告部分，直接提示输入密码。有的终端工具软件有记住密码功能选项，就会跳过输入密码。

每个SSH用户都有自己的`known_hosts`文件，此外系统也有一个这样的文件，通常是`/etc/ssh/ssh_known_hosts`，保存一些对所有用户都可信赖的远程主机的公钥。

### 公钥登录

使用密码登录，每次都必须输入密码，非常麻烦。好在SSH还提供了公钥登录，可以省去输入密码的步骤。

所谓"公钥登录"，原理是客户端将自己的公钥发送到服务端，服务端用客户端的公钥加密一个256位的随机字符串，客户端接收后解密，然后将这个字符串和会话标识符session ID合并在一起，对结果应用MD5散列函数并把散列值返回给服务器，服务器进行相同的MD5散列函数，如果客户端和该值可以匹配，就证明用户是可信的，直接允许登录shell，不再要求密码。

关于会话标识符，客户端与服务器实际上是先有会话加密过程，再进行的客户端验证，所以这个会话标识符就是第一个阶段生成的共享密钥。

这种方法要求用户必须提供自己的公钥。如果没有现成的，可以直接用`ssh-keygen`生成一个：

```bash
$ ssh-keygen
```

运行上面的命令以后，系统会出现一系列提示，可以一路回车。其中有一个问题是，要不要对私钥设置口令（passphrase），如果担心私钥的安全，这里可以设置一个。

运行结束以后，在$HOME/.ssh/目录下，会新生成两个文件：`id_rsa.pub`和`id_rsa`。前者是你的公钥，后者是你的私钥。

这时再输入下面的命令，将公钥传送到远程主机host上面：

```bash
$ ssh-copy-id user@host
```

远程主机将用户的公钥，保存在登录后的用户主目录的$HOME/.ssh/authorized_keys文件中。好了，从此你再登录，就不需要输入密码了。

如果还是不行，就打开远程主机的`/etc/ssh/sshd_config`这个文件，检查下面几行前面"#"注释是否取掉。

```bash
RSAAuthentication yes
PubkeyAuthentication yes
AuthorizedKeysFile .ssh/authorized_keys
```

然后，重启远程主机的ssh服务。

```bash
// ubuntu系统
service ssh restart

// debian系统
/etc/init.d/ssh restart
```

摘自  [阮一峰 - SSH原理与运用（一）：远程登录](http://www.ruanyifeng.com/blog/2011/12/ssh_remote_login.html)