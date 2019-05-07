---
layout: post
title: Ubuntu报错:No space left on device 
category: linux 
tags: [ubuntu,error]
---

- 当linux中磁盘过载时（或曾经过载），系统会在`/tmp`目录挂载一个`overflow`类型的文件系统，此时如果不作处理，即使你的空间已经清理，保证能够充足使用，系统仍然无法正常工作，其中的一个现象是：tab补全失败并报错：`No space left on device`，解决的方法：

    - 清理空间
    - 找出使用`/tmp`的进程，将其关闭
    - 卸载overflow文件系统(直接使用umount命令)，若不做第二部检查，同时有进程在使用/tmp目录则会导致失败并给出设备繁忙（device is busy）的报错提醒：

        ```
        umount overflow
        或
        umount /tmp
        ```
