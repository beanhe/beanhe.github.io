---
layout: post
title: Xtrabackup 备份与恢复
category: mysql
tags: [mysql,backup,xtrabackup,数据库备份,数据库恢复]
---
> 备份 - 完整备份：

```
innobackupex --host=127.0.0.1 --defaults-file=/etc/my.cnf --user=root --password=root /data/KEVIN/BKUP/mysql_db/full --no-timestamp 
```

- 以上命令中，若再本地执行则可以省略`--host`参数，若默认主配置文件问`/etc/my.cnf`则`--defaults-file`参数可以省略，若带有`--no-timestamp`参数则不会在执行备份目录以备份时间点创建子目录（默认不带此参数会创建名为$(date +%Y-%m-%d_%H-%M-%S)的备份子目录），执行上面命令后查看`xtrabackup_checkpoints`文件内容如下：

```
root@devops:~# cat /data/KEVIN/BKUP/mysql_db/2015-09-07_13-39-48/xtrabackup_checkpoints
backup_type = full-backuped
from_lsn = 0
to_lsn = 435754006
last_lsn = 435754006  
```



> 备份 - 增量备份

- 增量备份的前提是要有完整备份，命令为

```
innobackupex --user=USERNAME --password=PASSWORD --incremental /backup/data/dir
```
- 同样，执行完毕后会自动在`/backup/data/dir`目录下创建一个`$(date +%Y-%m-%d_%H-%M-%S)`格式的子备份目录，通过查看此目录中得`xtrabackup_checkpoints`文件可以发现，它的`from_lsn`值会与最近一次的备份的`last_lsn`值相等。如下：

```
root@devops:/data/KEVIN/BKUP/mysql_db/aaa# cat 2015-09-07_10-38-06/xtrabackup_checkpoints
backup_type = full-prepared
from_lsn = 0
to_lsn = 441119302
last_lsn = 441119302
root@devops:/data/KEVIN/BKUP/mysql_db/aaa# cat 2015-09-07_10-52-53/xtrabackup_checkpoints
backup_type = incremental
from_lsn = 435752329
to_lsn = 441119302
last_lsn = 441119302
root@devops:/data/KEVIN/BKUP/mysql_db/aaa# cat 2015-09-07_10-59-50/xtrabackup_checkpoints
backup_type = incremental
from_lsn = 441119302
to_lsn = 441123709
last_lsn = 441123709
```

> 恢复 - 完整备份恢复

- 先停止mysql服务，将现有mysql数据目录备份(一般默认为`/var/lib/mysql`)，然后新建新的mysql数据目录，再直接使用`--copy-back`参数即可恢复,恢复后需要将mysql数据目录权限恢复，否则无法启动mysql
```
# 停止mysql服务
service mysql stop

# 备份或删除当前数据文件
mv /var/lib/mysql /var/lib/mysql.bak
mkdir /var/lib/mysql

# 恢复日志文件
innobackupex --user=USERNAME --password=PASSWORD --apply-log /full/backup/data/dir/time-stamp

# 恢复数据文件
innobackupex --user=USERNAME --password=PASSWORD --copy-back /full/backup/data/dir/time-stamp

# 更改数据目录权限
cd /var/lib/
chown mysql.mysql -R /var/lib/mysql

# 重新启动mysql
service mysql start
```

- 上面的步骤中，如果忘记了恢复日志文件，则会出错，可以通过备份目录下的`xtrabackup_checkpoints`文件中的`backup_type`判断，若`backup_type`为`full-backuped`则说明没有执行`--apply-log`，若直接跳过这步进行文件恢复，会出现如下报错：

```
root@devops:/var/lib# innobackupex --user=root --password=root --apply-log /data/KEVIN/BKUP/mysql_db/2015-09-07_11-37-26/ --incremental-dir=/data/KEVIN/BKUP/mysql_db/2015-09-07_11-41-52/

InnoDB Backup Utility v1.5.1-xtrabackup; Copyright 2003, 2009 Innobase Oy
and Percona Inc 2009-2012.  All Rights Reserved.

This software is published under
the GNU GENERAL PUBLIC LICENSE Version 2, June 1991.

IMPORTANT: Please check that the apply-log run completes successfully.
           At the end of a successful apply-log run innobackupex
           prints "completed OK!".



150907 11:53:47  innobackupex: Starting ibbackup with command: xtrabackup_55 --prepare --target-dir=/data/KEVIN/BKUP/mysql_db/2015-09-07_11-37-26 --incremental-dir=/data/KEVIN/BKUP/mysql_db/2015-09-07_11-41-52/

xtrabackup_55 version 2.0.0 for Percona Server 5.5.16 Linux (x86_64) (revision id: undefined)
incremental backup from 443550849 is enabled.
xtrabackup: cd to /data/KEVIN/BKUP/mysql_db/2015-09-07_11-37-26
xtrabackup: This target seems to be not prepared yet.
xtrabackup: error: applying incremental backup needs target prepared.
innobackupex: Error:
innobackupex: ibbackup failed at /usr/bin/innobackupex line 371.
```
- 已执行过`--apply-log`动作的`xtrabackup_checkpoints`文件的`backup_type`字段如下：

```
root@devops:~# cat /data/KEVIN/BKUP/mysql_db/2015-09-07_13-39-48/xtrabackup_checkpoints
backup_type = full-prepared
from_lsn = 0
to_lsn = 435754006
last_lsn = 435754006   
```


> 恢复 - 完整+增量备份恢复

- 先停止mysql服务，备份现有数据目录并新建mysql数据目录，然后按时间先后顺序将增量备份的log apply到完整备份，最后使用`--copy-back`参数恢复完整备份的内容
```
# 停止mysql
service mysql stop

# 备份或删除当前mysql的数据目录
mv /var/lib/mysql /var/lib/mysql.bak
mkdir /var/lib/mysql

# 恢复日志文件
innobackupex --user=USERNAME --password=PASSWORD --apply-log /full/backup/data/dir/timestamp

# 恢复数据文件
innobackupex --user=USERNAME --password=PASSWORD --apply-log /full/backup/data/dir/timestamp --incremental-dir=/first/incremental/backup/dir
innobackupex --user=USERNAME --password=PASSWORD --apply-log /full/backup/data/dir/timestamp --incremental-dir=/second/incremental/backup/dir
...(一直到要恢复的增量备份点结束)

# 恢复数据库
innobackupex --user=USERNAME --password=PASSWORD --copy-back /full/backup/data/dir/timestamp

# 修改mysql数据目录权限
chown mysql.mysql -R /var/lib/mysql

# 重新启动mysql
service mysql start
```
