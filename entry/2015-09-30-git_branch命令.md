---
layout: post
title: git_branch命令
category: git
tags: [git]
---
#### 名称
- git branch：用以显示，创建和删除分支
#### 概述
- git branch [--color[=<when>] | --no-color] [-r | -a]
	- [-v [--abbrev=<length> | --no-abbrev]]
	- [(--merged | --no-merged | --contains) [<commit>]]
- git branch [--set-upstream | --track | --no-track] [-l] [-f] <branchname> [<start-point>]
- git branch (-m | -M) [<oldbranch>] <newbranch>
- git branch (-d | -D) [-r] <branchname>...
#### 描述
- 不带任何参数时候，命令结果会显示本地存在的所有分支且当前所在分支会高亮显示。`-r`参数会列出远端分支，`-a`参数会列出所有分支（远端和本地分支以颜色区分）

#### 参数
- `-d`：删除分支。被删除的分支必须已经完全合并到它的上游分支（upstream branch），如果没有上游分支，则分支需要在`HEAD`提交点
- `-D`：无视分支状态强制删除分支
- `-f`或`--force`：强制进行针对分支的操作
- `-m`：重命名分支
- `-M`：重命名分支即使新分支名已存在
- `--color`：以不同的颜色高亮区别不同来源的分支，其值可以是`always`,`never`或`auto`，默认为`--color=always`
- `--no-color`：无视git配置文件，不以颜色区分分支，相当于`--color=never`
