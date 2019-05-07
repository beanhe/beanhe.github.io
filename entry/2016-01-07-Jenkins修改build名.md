---
layout: post
title: Jenkins修改build名
category: jenkins
tags: [jenkins]
---

### 如何自定义Jenkins Build名


- 背景：默认情况下Jenkins的build名由『数字+#』的方式构成依次递增，但是在敏捷开发过程中，通常希望这个名称更有意义甚至直接作为软件的Build版本号来显示，要实现这个功能，可以使用*Build Name Setter Plugin*来实现

- 使用插件：Build Name Setter Plugin

- 使用方法：

    - 安装build name setter plugin插件，重启Jenkins使其生效
    - 进入对应Project的配置，发现多了*Build Environment*配置区域
    - 点选Set Build Name则会出现*Build Name*输入框，在输入框右侧帮助信息中列出了所支持的变量，我们可以使用这些变量来自由拼接build name的形式，比如使用git branch name + build number + git revision（12位）的方式来显示可以使用在其中设置：

    ```
    ${GIT_BRANCH,fullName="false"}-${BUILD_NUMBER}-${GIT_REVISION,length=12}
    ```
    - 设置完成后保存即可生效，以后build的名称就不是生硬的数字了，而是类似`master-33-b1689337f1da`这样更有意义的表示方法，是不是很给力，😄
    - 另在后续的SHELL脚本中可以使用${BUILD_DISPLAY_NAME}来调用这个新的build名
