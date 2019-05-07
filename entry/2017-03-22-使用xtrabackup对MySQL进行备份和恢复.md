---
layout: post
title: 使用xtrabackup对MySQL进行备份和恢复
category: mysql
tags: [__beanhe,mysql,xtrabackup]
---

#### 环境：
 - OS: ubuntu 14.04
 - MySQL Server: 5.7.17
 - Xtrabackup:  percona-xtrabackup-24
 - 代码和备份的根目录：`/mnt/deploy`
 

#### 相关脚本：
- 主脚本：`xtrabackup.sh`，内容如下(将`<YOUR_BACKUPUSER>`改为MySQL备份用户名，`<YOUR_BACKUPUSER_PASSWORD>`改为MySQL备份用户密码)：

```
#!/bin/bash
set +x

usage() {
        echo "usage: $(basename $0) [option]"
        echo "option=full: do a full backup of vinnie /var/lib/mysql using innobackupex, aprox time 6 hours."
        echo "option=incremental: do a incremental backup"
        echo "option=restore: this will restore the latest backup to vinnie, BE CAREFUL!"
        echo "option=help: show this help"
}

full_backup() {
        date
        if [ ! -d $BACKUP_DIR ]
        then
                echo "ERROR: the folder $BACKUP_DIR does not exists"
                exit 1
        fi
        echo "doing full backup..."
        echo "cleaning the backup folder..."
        rm -rf $BACKUP_DIR/*
        echo "cleaning done!"
        innobackupex $ARGS $BACKUP_DIR/FULL
        date
        echo "backup done!, now uncompressing the files..."
        for bf in `find $BACKUP_DIR/FULL -iname "*\.qp"`; do qpress -d $bf $(dirname $bf) ;echo "processing" $bf; rm $bf; done
        date
        echo "uncompressing done!, preparing the backup for restore..."
        innobackupex --apply-log --redo-only $BACKUP_DIR/FULL
        date
        echo "preparation done!"
}
incremental_backup()
{
        if [ ! -d $BACKUP_DIR/FULL ]
        then
                echo "ERROR: no full backup has been done before. aborting"
                exit -1
        fi

        #we need the incremental number
        if [ ! -f $BACKUP_DIR/last_incremental_number ]; then
            NUMBER=1
        else
            NUMBER=$(($(cat $BACKUP_DIR/last_incremental_number) + 1))
        fi
        date
        echo "doing incremental number $NUMBER"
        if [ $NUMBER -eq 1 ]
        then
                innobackupex $ARGS --incremental $BACKUP_DIR/inc$NUMBER --incremental-basedir=$BACKUP_DIR/FULL
        else
                innobackupex $ARGS --incremental $BACKUP_DIR/inc$NUMBER --incremental-basedir=$BACKUP_DIR/inc$(($NUMBER - 1))
        fi
        date
        echo $NUMBER > $BACKUP_DIR/last_incremental_number
        echo "incremental $NUMBER done!, now uncompressing the files..."
        for bf in `find $BACKUP_DIR/inc$NUMBER -iname "*\.qp"`; do qpress -d $bf $(dirname $bf) ;echo "processing" $bf; rm $bf; done
        date
        echo "uncompressing done!, the preparation will be made when the restore is needed"

}

restore()
{
        echo "WARNING: are you sure this is what you want to do? (Enter 1 or 2)"
        select yn in "Yes" "No"; do
            case $yn in
                Yes ) break;;
                No ) echo "aborting... that was close."; exit;;
            esac
        done

        date
        echo "doing restore..."
        #innobackupex --apply-log --redo-only $BACKUP_DIR/FULL

        #we append all the increments
        P=1
        while [ -d $BACKUP_DIR/inc$P ] && [ -d $BACKUP_DIR/inc$(($P+1)) ]
        do
              echo "processing incremental $P"
                innobackupex --apply-log --redo-only $BACKUP_DIR/FULL --incremental-dir=$BACKUP_DIR/inc$P
                P=$(($P+1))
        done

        if [ -d $BACKUP_DIR/inc$P ]
        then
                #the last incremental has to be applied without the redo-only flag
                echo "processing last incremental $P"
                innobackupex --apply-log $BACKUP_DIR/FULL --incremental-dir=$BACKUP_DIR/inc$P
        fi

        #we prepare the full
                innobackupex --apply-log $BACKUP_DIR/FULL

        #finally we copy the folder
        cp -r $DATA_DIR $DATA_DIR.back
        rm -rf $DATA_DIR/*
        innobackupex --copy-back $BACKUP_DIR/FULL

        chown -R mysql:mysql $DATA_DIR

}

#######################################
#######################################
#######################################

BACKUP_DIR=/mnt/deploy/backup
DATA_DIR=/var/lib/mysql
USER_ARGS=" --user=<YOUR_BACKUPUSER> --password=<YOUR_BACKUPUSER_PASSWORD>"

ARGS="--rsync $USER_ARGS --no-timestamp --compress --compress-threads=4"

if [ $# -eq 0 ]
then
usage
exit 1
fi

    case $1 in
        "full")
            full_backup
            ;;
        "incremental")
        incremental_backup
            ;;
        "restore")
        restore
            ;;
        "help")
            usage
            #break
            ;;
        *) echo "invalid option";;
    esac
```
- 全量备份脚本：`fullBackupForCrontab.sh`,内容如下：

```
#!/bin/bash

BACKUP_DIR=/mnt/deploy/backup
SCRIPT=/mnt/deploy/xtrabackup.sh
LOG_FILE=/var/log/db_backup.log
TIME_NOW=`date +%Y%m%d%H%M`
if [ -d $BACKUP_DIR ];then
	mv $BACKUP_DIR ${BACKUP_DIR}.${TIME_NOW}
	mkdir $BACKUP_DIR
else
	mkdir $BACKUP_DIR
fi
echo "$(date) Begin full backup"
${SCRIPT} full >> $LOG_FILE 2>&1
if [ $? -eq 0 ];then
   echo "$(date) Full backup succeed "
else
    echo "$(date) Full backup faild "
fi
```

- 增量备份脚本：`incrementalBackupForCrontab.sh`，内容如下

```
#!/bin/bash
BACKUP_DIR=/mnt/deploy/backup
FULL_BACKUP_DIR=${BACKUP_DIR}/FULL
SCRIPT=/mnt/deploy/xtrabackup.sh
LOG_FILE=/var/log/db_backup.log
if [ -d $FULL_BACKUP_DIR ];then
    echo "$(date) Begin incremental backup" 
    $SCRIPT incremental >> $LOG_FILE 2>&1
    if [ $? -eq 0 ];then
	echo "$(date) Incremental backup succeed "
    else
	echo "$(date) Incremental backup faild"
    fi
else
    echo "$(date) Can't find full backup"
fi
```

#### 如何备份：
 假设备份策略为：每天全量和增量各一次，全量时间凌晨`1:00`,增量时间早上`6:30`，则直接在`crontab`中加入（`crontab -e`）如下内容：
 
 ```
0 1 * * * /mnt/deploy/fullBackupForCrontab.sh
30 6 * * * /mnt/deploy/incrementalBackupForCrontab.sh
 ```
 
 添加后重启`cron`进程：`service cron restart`

#### 全量备份恢复：
如果恢复全量备份数据，则需要先将`/mnt/deploy/backup`目录下`除FULL目录以外的文件和目录都移除`，然后直接运行`xtrabackup.sh`脚本然后重启mysql即可，以恢复2017年3月10日的全量备份为例:

```
cd /mnt/deploy
# 备份当前备份目录
mv backup backup.bak
# 将3月10日的（由 fullBackupForCrontab.sh 脚本可知目录时间戳为0311的存储的是3月10日的备份数据）备份目录命名为脚本定义的数据目录
cp -pr backup.201703110100/ backup
# 移除增量备份相关文件和目录
mv backup/inc* last_incremental_number /tmp/
# 恢复
./xtrabackup.sh restore
# 重启mysql
service mysql restart
# 删除目录
rm -rf backup
# 恢复当前备份目录
mv backup.bak backup
```
#### 增量备份恢复
恢复增量备份更简单，直接运行`xtrabackup.sh`脚本恢复重启mysql即可,以恢复2017年3月10日的增量备份为例:

```
cd /mnt/deploy
mv backup backup.bak
cp -pr backup.201703110100/ backup
./xtrabackup.sh restore
service mysql restart
rm -rf backup
mv backup.bak bakcup
```