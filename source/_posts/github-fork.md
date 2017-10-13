---
title: github如何同步fork？
date: 2017-10-13 11:25:07
tags:
- Github
categories:
- Github
- fork
---

### 为 `fork` 配置远程 `upstream`仓库

- 列出当前远程仓库配置.

```bash
$ git remote -v
origin  https://github.com/YOUR_USERNAME/YOUR_FORK.git (fetch)
origin  https://github.com/YOUR_USERNAME/YOUR_FORK.git (push)
```

<!-- more -->

- 新增要同步的 远程 `upstream` 仓库.

```bash
$ git remote add upstream https://github.com/ORIGINAL_OWNER/ORIGINAL_REPOSITORY.git
```

- 验证，为`fork`添加 `upstream`仓库是否成功.

```bash
$ git remote -v
origin    https://github.com/YOUR_USERNAME/YOUR_FORK.git (fetch)
origin    https://github.com/YOUR_USERNAME/YOUR_FORK.git (push)
upstream  https://github.com/ORIGINAL_OWNER/ORIGINAL_REPOSITORY.git (fetch)
upstream  https://github.com/ORIGINAL_OWNER/ORIGINAL_REPOSITORY.git (push)
```



###  同步更新你的`fork`

- 从远程`upstream` 仓库获取所有分支的更新文件和提交历史信息到本地目录

```bash
$ git fetch upstream
remote: Counting objects: 75, done.
remote: Compressing objects: 100% (53/53), done.
remote: Total 62 (delta 27), reused 44 (delta 9)
Unpacking objects: 100% (62/62), done.
From https://github.com/ORIGINAL_OWNER/ORIGINAL_REPOSITORY
 * [new branch]      master     -> upstream/master
```

- 确保你在本地的 `master` 分支

```bash
$ git checkout master
Switched to branch 'master'
```

- 分支合并

```bash
$ git merge upstream/master
Updating a422352..5fdff0f
Fast-forward
 README                    |    9 -------
 README.md                 |    7 ++++++
 2 files changed, 7 insertions(+), 9 deletions(-)
 delete mode 100644 README
 create mode 100644 README.md
```