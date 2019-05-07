---
layout: post
title: elasticsearch快照恢复
category: elasticsearch
tags: [__beanhe,elasticsearch,恢复]
---

#### 通过SNAPSHOT方式备份和恢复elasticsearch

##### 思路：通过ElasticSearch自带的snapshot功能创建快照，再使用快照进行index的恢复
##### 测试版本：ElasticSearch 2.4.4
##### 结果：可在同一个cluster中进行备份和恢复，也可以在ClusterA进行snapshot备份，然后将数据恢复到ClusterB
##### 步骤（以两个不同cluster节点为例）：
 - 检查clusterA和clusterB两个节点的`elasticsearch.yml`配置文件，将快照存放目录加入`path.repo:`列表中，如`path.repo: ["/mnt/deploy/elasticdump"]`
 - 在源节点注册一个快照仓库用以存放快照（一个仓库可以用以存放一个或多个snapshot），仓库类型为共享文件系统`fs`，命令如下：
 ```
 $ curl -XPUT 'http://localhost:9200/_snapshot/my_backup' -d '{
    "type": "fs",
    "settings": {
        "location": "/mnt/deploy/elasticdump",
        "compress": true
    }
}'
 ```
 命令中`my_backup`为仓库名，`fs`表示仓库为一个共享文件系统，`location`用以指定文件系统的实际路径，若这个路径没有添加入上一步的列表配置中，则会出现类似如下报错：
```
{
  "error" : {
    "root_cause" : [ {
      "type" : "repository_exception",
      "reason" : "[my_backup] failed to create repository"
    } ],
    "type" : "repository_exception",
    "reason" : "[my_backup] failed to create repository",
    "caused_by" : {
      "type" : "creation_exception",
      "reason" : "Guice creation errors:\n\n1) Error injecting constructor, RepositoryException[[my_backup] location [/mnt/deploy/elasticdump] doesn't match any of the locations specified by path.repo]\n  at org.elasticsearch.repositories.fs.FsRepository.<init>(Unknown Source)\n  while locating org.elasticsearch.repositories.fs.FsRepository\n  while locating org.elasticsearch.repositories.Repository\n\n1 error",
      "caused_by" : {
        "type" : "repository_exception",
        "reason" : "[my_backup] location [/mnt/deploy/elasticdump] doesn't match any of the locations specified by path.repo"
      }
    }
  },
  "status" : 500
}
```
仓库创建成功后就可以使用`$ curl -XGET 'http://localhost:9200/_snapshot/my_backup?pretty'`命令查看仓库的状态信息，也可以使用`$ curl -XGET 'http://localhost:9200/_snapshot/_all?pretty'`来显示当前系统中所有的仓库信息

 - 有了仓库就可以创建库按照了，可以简单使用`$ curl -XPUT "localhost:9200/_snapshot/my_backup/snapshot_1?wait_for_completion=true"`命令对所有所有做快照，也可以使用如下命令对指定index做快照：
 ```
 $ curl -XPUT "localhost:9200/_snapshot/my_backup/snapshot_1" -d '{
    "indices": "filebeat-2017.02.27,filebeat-2017.02.28",
    "ignore_unavailable": "true",
    "include_global_state": false
    "partial": "false"
}'
 ```
 命令中`indices`可以指定多个index名，`ignore_unavailable`为`true`会自动忽略不存在的索引，`include_global_state`为`false`可以防止将全局状态存储到snapshot中。默认情况下，如果快照中的1个或多个索引不是全部主分片都可用会导致整个创建快照的过 程失败。 通过设置 partial 为 true 可以改变这个行为。
 索引创建快照的过程是增量的。在给索引创建快照的过程中，Elasticsearch会分析存储在仓库中的索引文件并且只会复制那些自从上次快照 之后新建或有所更新的文件。这使得多个快照以一种紧凑的方式存储在同一个仓库里。创建快照的过程是以非阻塞方式执行的。一个索引在创 建快照的同时能够被检索和查询。尽管如此，快照保存的是在开始进行创建快照的那个时间点的索引的视图。所以，在开始创建快照之后的记 录不会出现在这个快照里。在主分片启动之后创建快照的过程就会立即开始，并且之后不会改变位置。在1.2.0版本之前如果集群重新定位或者 新加入快照的索引初始化主分片会导致快照操作失败。从1.2.0版本开始，Elasticsearch会等待重新定位和初始化分片然后再创建快照。
快照创建好后，可以使用`$ curl -XGET "localhost:9200/_snapshot/my_backup/snapshot_1"`查看快照的信息，通过`$ curl -XGET "localhost:9200/_snapshot/my_backup/_all"`命令列出所有快照信息，`$ curl -XDELETE "localhost:9200/_snapshot/my_backup/snapshot_1"`则用来删除快照
 - 将以上备份的目录`/mnt/deploy/elasticdump`打包（如elasticdump.tar.gz)拷贝并解压到目标节点clusterB的对应目录（注：此目录页一定要在`elasticsearch.yml`的`path.repo: `列表中），然后使用如下命令进行恢复：
```
$ curl -XPOST "localhost:9200/_snapshot/my_backup/snapshot_1/_restore" -d '{
    "indices": "filebeat-2017.02.27,filebeat-2017.02.28",
    "ignore_unavailable": "true",
    "include_global_state": false,
    "rename_pattern": "index_(.+)",
    "rename_replacement": "restored_index_$1"
}'
```
命令中可以看出新的index可以通过`rename_pattern`和`rename_replacement`进行重命名，至此恢复完成

- 注意：所有创建的动作都要保证对对应的文件系统目录有可写权限否则会有类似如下报错：
```
{
  "error" : {
    "root_cause" : [ {
      "type" : "snapshot_creation_exception",
      "reason" : "[my_backup:snapshot_1] failed to create snapshot"
    } ],
    "type" : "snapshot_creation_exception",
    "reason" : "[my_backup:snapshot_1] failed to create snapshot",
    "caused_by" : {
      "type" : "access_denied_exception",
      "reason" : "/mnt/deploy/elasticdump/meta-snapshot_1.dat"
    }
  },
  "status" : 500
}
```

参考：[elasticsearch快照和恢复](http://blog.csdn.net/crazyhacking/article/details/45726683)