---
layout: post
title: 使用Python和JS将URL中的普通字符串参数转换为URN编码字符串
category: web
tags: [__beanhe,python,js]
---

#### 使用Python和JS将URL中的普通字符串参数转换为URN编码字符串

以一条含有特殊字符的命令为例，比如要往连接`http://www.example.com/arg?`中传入如下参数：`cd /home && ls -l`，这一串参数中，如果不进行编码，则`&`会被认为是URL中的参数连接符，但是很明显这个的`&&`两个连用是shell中的用法，要正确传递这一串命令必须先对参数进行编码：

在`JS`中可以直接使用`encodeURIComponent`方法：

```
var encodedStr = encodeURIComponent("cd /home && ls -l")
```

在`Python`中可以使用`urllib`库进行编码：

```
import urllib
encodedStr = urllib.quote_plus('cd /home && ls -l')
```

最终`'cd /home && ls -l`被JS编码为`cd%20%2Fhome%20%26%26%20ls%20-l`,被Python编码为`cd+%2Fhome+%26%26+ls+-l`，两者均可正常使用，只不过JS转换的更彻底