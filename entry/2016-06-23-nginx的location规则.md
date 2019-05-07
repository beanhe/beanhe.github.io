---
layout: post
title: Nginx的location规则 
category: web
tags: [nginx]
---
- 直奔主题，nginx的location用来配置路径的匹配规则，语法如下：
    
    - `location [=|~|~*|^~|@] /uri/ {...}`
    - 配置段：server, location

    - 各种规则用法：
        - `=`: 精确匹配，优先级最高
        - `^~`: 表示URI以指定的字符串开头
        - `~`: 表示区分大小写的正则匹配
        - `~*`: 表示不区分大小写的正则匹配
        - `!~`: 表示区分大小写正则`不`匹配
        - `!~*`: 表示不区分大小写正则`不`匹配
        - `/`: 通用匹配，所有请求都会被匹配到

    - 至于优先级，可参考[此文章](http://www.cnblogs.com/lidabo/p/4169396.html)
