---
layout: post
title: Hexo + gitcafe配置及部署教程
category: web
tags: [git,web,hexo]
---
> hexo安装

```
brew install npm
npm install -g hexo
```

> gitcafe 配置

- 进入gitcafe,新建一个项目，注意项目名与拥有者（即用户名）相同
- 进入项目设置-> DeployKeys，添加`git clone`要用到的ssh key
- 将仓库git clone到本地，并添加一个`gitcafe-pages`分支push到gitcafe

```
git clone git@gitcafe.com:pythell/pythell.git
git checkout -b gitcafe-pages
git push origin gitcafe-pages
```

> hexo 配置

- 编辑hexo的配置文件`_config.yml`，修改deploy段如下：

```
deploy:
  type: git
  repository: git@gitcafe.com:pythell/pythell.git
  branch: gitcafe-pages
```
- 上面代码中type必须为git（不论是github还是gitcafe），repository为你的git仓库路径，branch需要为gitcafe-pages
- 配置好后需要安装hexo-deployer-git, 否则到了`hexo deploy`会出现`Deployer not found: git`的报错

```
npm install hexo-deployer-git --save
```

> 完成

- 以上就是全部步骤，接下来只需要在hexo中新建，生成和部署即可

```
# 添加文章
hexo n "title"  
或
hexo new "title"

# 生成结构
hexo g 
或
hexo generate

# 部署
hexo d
或
hexo deploy
```
