---
layout: post
title: inotifywait + rsync配置实现实时同步
category: 同步,监控 
tags: [inotify,rsync,同步]
---
> rsync可以实现文件或目录的同步，若对于同步实时性要求高，则可以使用inotify对同步的源文件或目录进行监控，此时就可以实现一旦发现源出现某些操作则触发rsync的同步操作，从而实现同步的实时性，以下是一个典型的实时同步脚本：

```
#!/bin/bash

inotifywait -mr /source/path -e modify,delete,create --exclude '((.*\.git.*)|(\.swpx?|~|swx)$)' --timefmt '%d/%m/%y %H:%M' --format '%T %w %f' | while read DATE TIME DIR FILE;
do
       FILECHANGE=${DIR}${FILE}
       rsync --exclude-from=/home/ops/rsync.ignore -rlpgoDvz --delete --update --password-file=/etc/rsync.pass /source/path/ deploy@192.168.1.225::deploy &>/dev/null && \
       echo "At ${TIME} on ${DATE}, file $FILECHANGE was backed up via rsync" >> /var/log/rsync-to-dest.log
done
```
- 脚本中,`/home/ops/rsync.ignore`是一个正则表达式匹配文件，rsync会逐行匹配，匹配的就不会被同步(`--exclude-from`)，另外对于rsync命令中，源路径写法结尾的'/'很有讲究，可参考[rsync用法总结](http://www.pythell.club/2015/09/18/rsync%E7%94%A8%E6%B3%95%E6%80%BB%E7%BB%93/)
