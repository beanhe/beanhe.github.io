---
layout: post
title: Ignore dirty status
category: git
tags: [git,error]
---
### Ignore dirty info in git

- 有时候使用git status命令会有类似如下信息出现：

```
On branch master
Your branch is ahead of 'origin/master' by 1 commit.
  (use "git push" to publish your local commits)
Changes not staged for commit:
  (use "git add <file>..." to update what will be committed)
  (use "git checkout -- <file>..." to discard changes in working directory)
  (commit or discard the untracked or modified content in submodules)

	modified:   themes/next (modified content)
no changes added to commit (use "git add" and/or "git commit -a")
```

- 此时会误以为是有文件更改没有提交，当使用`git add`和`git commit`之后再使用`git status`仍然会有以上的提示信息，此时使用`git diff thems/next`命令会有如下提醒信息：

```
t a/wiki/themes/next b/wiki/themes/next
+++ b/wiki/themes/next
@@ -1 +1 @@
-Subproject commit 74fe43abb2ba7b2c4e4e58fbe267e9c37993c2e9
+Subproject commit 74fe43abb2ba7b2c4e4e58fbe267e9c37993c2e9-dirty
```

- 此时可使用`git status --ignore-submodules=dirty`来避免出现这些提醒 

```
[11时13分59秒]youdao@localhost: ~/pythell/wiki master ⚡ $ git status --ignore-submodules=dirty
On branch master
Your branch is ahead of 'origin/master' by 3 commits.
  (use "git push" to publish your local commits)
nothing to commit, working directory clean
```
