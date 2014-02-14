---
layout: post
title: "Linux下Android SDK设置代理服务器"
description: ""
category: ""
tags: []
---
{% include JB/setup %}

vim tools/android

在exec里面直接添加java的环境变量

```
    -Dhttps.proxyHost=10.19.110.32
    -Dhttp.proxyHost=10.19.110.32
    -Dhttp.proxyPort=8080
    -Dhttps.proxyPort=8080
```

