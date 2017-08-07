---
title: 错误的 git reset --hard 操作
date: 2017-07-07 09:33:06
tags:
- Git命令
categories:
- Git
---


### 场景
　　通过 **[vagrant](https://www.vagrantup.com/)** 安装 **[box](https://app.vagrantup.com/boxes/search)**  -- `homestead`，并把里面相应的配置都修改好了，并执行过 `git add` 和 `git commit`，但是本应在另一个目录执行`git reset --hard` 操作，却因没有切换目录，导致所有的修改失效。如何恢复到已修改好的那个状态&nbsp;?

<!-- more -->

### 测试
#### 新增仓库并增加提交记录
``` bash
	$  mkdir test
	$  cd test
	$  git init
	$  touch a.txt
	$  git add a.txt
	$  git commit -m "initial commit"
	$  echo 'new data' >> a.txt
	$  git commit -a -m "add something"
	```
#### 查看提交记录
``` bash
	$ git log -2
	commit 2d22a9f20a7a14021b6b2bafaae6bf57985b8435 (HEAD -> master)
	Author: Zhenliang Zhou <zhouzhenliang@shucong.com>
	Date:   Fri Jul 7 10:14:04 2017 +0800

	    add something

	commit 355382785fca8936eba4f5b1ccfb3f5675d1fc14
	Author: Zhenliang Zhou <zhouzhenliang@shucong.com>
	Date:   Fri Jul 7 10:13:48 2017 +0800

	    initial commit
```
#### 返回到上次提交
``` bash
	$ git reset --hard 355382785fca8936eba4f5b1ccfb3f5675d1fc14
	$ cat a.txt

	$ git log
	commit 355382785fca8936eba4f5b1ccfb3f5675d1fc14 (HEAD -> master)
	Author: Zhenliang Zhou <zhouzhenliang@shucong.com>
	Date:   Fri Jul 7 10:13:48 2017 +0800

	    initial commit
```
现在情况正好模拟了文章开头提到的场景，这时该怎么回到第二次提交。

#### 前进到最后一次提交
`git reflog` 会记录与 `HEAD` 相关操作的记录
``` bash
	$ git reflog
	3553827 (HEAD -> master) HEAD@{0}: reset: moving to 355382785fca8936eba4f5b1ccfb3f5675d1fc14
	2d22a9f HEAD@{1}: commit: add something
	3553827 (HEAD -> master) HEAD@{2}: commit (initial): initial commit
	$ git reset --hard 2d22a9f
	$ git log -2
	commit 2d22a9f20a7a14021b6b2bafaae6bf57985b8435 (HEAD -> master)
	Author: Zhenliang Zhou <zhouzhenliang@shucong.com>
	Date:   Fri Jul 7 10:14:04 2017 +0800

	    add something

	commit 355382785fca8936eba4f5b1ccfb3f5675d1fc14
	Author: Zhenliang Zhou <zhouzhenliang@shucong.com>
	Date:   Fri Jul 7 10:13:48 2017 +0800

	    initial commit
```

### 总结
　　综合使用 `git reflog` 和 `git reset --hard` 可以找回丢到的代码。