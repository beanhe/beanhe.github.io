---
layout: post
title: git杂记 
category: git,版本控制
tags: [git,版本控制]
---
- 要想保持树的整洁，使用git push建议步骤如下：

```
git fetch origin master
git rebase origin master
git push
```
