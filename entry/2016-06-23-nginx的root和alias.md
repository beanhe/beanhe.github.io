---
layout: post
title: Nginx的root和alias 
category: web
tags: [nginx]
---

- Nginx中指定路径的方式有两种：root, alias,具体用法如下：

    - root:
        - 语法：root PATH
        - 默认值：root html
        - 配置段： http server location if
    - alias:
        - 语法：alias PATH
        - 配置段：location

    - 区别：
        - root根据完整的URI进行映射，比如配置为：
        
        ```
        location /img {
            root /var/www/html/img;
        }
        ```

        访问URI为`/img/test.gif`,则真实的访问路径为：`/var/www/html/img/img/test.gif`

        - 对于alias，则会丢掉location后面的路径进行映射，比如配置为：

        ```
        location /img {
            root /var/www/html/files/;
        }
        ```
        访问URI为`/img/test.gif`,则真实的访问路径为：`/var/www/html/files/test.gif`,
