---
layout: post
title: MacOSX中设置sudo免密码
category: mac
tags: [__beanhe,maxosx,sudo]
---
    
- MacOSX中配置无密码的sudo，步骤： 

    ```
    sudo visudo
    ```

- 修改如下行：

    ```
    %admin ALL=(ALL) ALL
    ```

 - 修改为:

    ```
    %admin ALL=(ALL) NOPASSWD: ALL
    ```

- 保存关闭