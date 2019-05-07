---
layout: post
title: PHP中diplay_errors为OFF仍然会显示报错信息
category: php
tags: [php,error]
---

- 生产环境中希望不显示PHP的错误信息，当`display_errors`不生效时候，可以考虑以下几点：
	- 正常做法是在`/etc/php.ini`中将`display_errors`设置为`false`即可
	- 若已经为`false`但仍然显示错误信息，则需要考虑是否是因为代码中开启了`display_errors`，相应代码为`ini_set(‘display_errors’, false);`，
	- 还有可能是`php-fpm`等程序中也有相应配置，`php-fpm`中对应为`php_flag[display_errors]`配置项
