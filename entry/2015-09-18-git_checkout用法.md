---
layout: post
title: git_checkout用法
category: git
tags: [git,版本控制]
---
> git status信息

```
[05:57:21 PM]root@devops: ~/opstest.devel kevin|REBASE 1/8 # git status
# On branch kevin
# Changed but not updated:
#   (use "git add <file>..." to update what will be committed)
#   (use "git checkout -- <file>..." to discard changes in working directory)
#
#	modified:   test
#
no changes added to commit (use "git add" and/or "git commit -a")
```
- 其实看提示即可，若要保持修改则直接`git add test`添加进行后续操作，若不想修改则直接`git checkout -- test`来放弃，`git checkout`后：

```
[05:57:26 PM]root@devops: ~/opstest.devel kevin|REBASE 1/8 # git checkout test
[05:57:38 PM]root@devops: ~/opstest.devel kevin|REBASE 1/8 # git status
# On branch kevin
nothing to commit (working directory clean)
```
