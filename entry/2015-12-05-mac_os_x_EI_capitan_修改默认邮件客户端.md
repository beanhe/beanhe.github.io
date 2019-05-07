---
layout: post
title: mac_os_x_EI_capitan_修改默认邮件客户端
category: mac
tags: [macosx]
---
- Mac OS X EI Capitan之前的系统只需要打开默认邮件客户端，进入偏好设置就可以选择更改默认的客户端，但是在新版的EI Capitan中，更改后关闭设置仍然会自动更换回系统默认客户端，解决方案是在昨晚之前的步骤后，需要使用下面的命令重置相关数据库：

    ```
    /System/Library/Frameworks/CoreServices.framework/Versions/A/Frameworks/LaunchServices.framework/Versions/A/Support/lsregister -kill -r -all local,system,user
    ```

    参考：[链接](https://forums.developer.apple.com/thread/14502)
