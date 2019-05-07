---
layout: post
title: gitlab上复制已有Project
category: git 
tags: [gitlab,git]
---

> 步骤：

- 在gitlab新建一个项目(如test)
- 将要复制的目标project(如target)克隆到本地

```
git clone git@git.pythell.club:root/target.git
```

- 进入克隆完的target目录删除.git目录后重新初始化

```
cd target
rm -rf .git
git init
```

- 在重新初始化后的target目录中关联新建的project(test)，将其所有文件上传至远端的
新建project中即可

```
git remote add origin git@git.pythell.club:root/test.git
git add .
git commit -m 'init'
git push -u origin master
```
