---
layout: post
title: mysqldump用法
category: mysql
tags: [mysql,mysqldump,数据库备份,数据库恢复]
---

- 导出整个数据库-包括数据库结构和数据：
`mysqldump -u USERNAME -p DB_NAME > result.sql`

- 只导出数据库结构：
`mysqldump -u USERNAME -p -d DB_NAME > result.sql`

- 导出数据库某张表-包括表结构和表数据：
`mysqldump -u USERNAME -p DB_NAME TABLE_NAME > result.sql`

- 只导出数据库某张表的表结构：
`mysqldump -u USERNAME -p -d DB_NAME TABLE_NAME > result.sql`
