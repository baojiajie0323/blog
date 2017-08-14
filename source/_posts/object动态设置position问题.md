---
layout: title
title: object动态设置position问题
date: 2017-08-14 19:35:45
tags:
categories:
- 日常踩坑
---

### 问题描述
在使用浏览器视频插件（NPAPI）时，发现窗口句柄会动态变化导致视频浏览失败。
<!--more-->
### 问题定位
在抓包等一些列问题排查后，发现是由于窗口句柄会动态变化导致，也就是Npapi中的窗口会自动销毁并重新创建。仔细查看代码后发现是由于在Js代码中动态设置了object 的style的position属性导致。
```
plugin.style.position = 'absolute';
```

### 问题解决
把动态设置属性的地方写到css中即可。