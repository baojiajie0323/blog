---
layout: title
title: zrender transform后拖动bug
date: 2017-08-07 17:31:25
tags:
categories:
- 日常踩坑
---

### 问题描述
使用zrender库时，如果定义的元素设置draggable为ture,并且设置了rotation或者scale,origin不为null时，拖动元素会出现异常。

<!--more-->
### 问题定位
翻看源码,在拖动元素时，会进入到element.js
```
drift: function (dx, dy) {
    var m = this.transform;
    if (!m) {
        m = this.transform = [1, 0, 0, 1, 0, 0];
    }
    m[4] += dx;
    m[5] += dy;

    this.decomposeTransform();
    this.dirty(false);
}
```
这里的dx 和 dy 就是每次拖动的偏移量x,y，虽有matrix的后2位加上偏移量，就是新的矩阵位置。这里是没有问题的，但是这里最后调用了 decomposeTransform 方法，本意是重新计算一遍transform属性，看代码
```
    transformableProto.decomposeTransform = function () {
        if (!this.transform) {
            return;
        }
        var parent = this.parent;
        var m = this.transform;
        if (parent && parent.transform) {
            // Get local transform and decompose them to position, scale, rotation
            matrix.mul(tmpTransform, parent.invTransform, m);
            m = tmpTransform;
        }
        var sx = m[0] * m[0] + m[1] * m[1];
        var sy = m[2] * m[2] + m[3] * m[3];
        var position = this.position;
        var scale = this.scale;
        if (isNotAroundZero(sx - 1)) {
            sx = Math.sqrt(sx);
        }
        if (isNotAroundZero(sy - 1)) {
            sy = Math.sqrt(sy);
        }
        if (m[0] < 0) {
            sx = -sx;
        }
        if (m[3] < 0) {
            sy = -sy;
        }
        position[0] = m[4];
        position[1] = m[5];
        scale[0] = sx;
        scale[1] = sy;
        this.rotation = Math.atan2(-m[1] / sy, m[0] / sx);
    };
```
其中这里的 position[0] = m[4],position[0] = m[5], 这句话就存在问题了。position本指的是元素原始坐标偏移量，而这里的m[4] 和m[5] 指的是元素经过矩阵转换后的坐标偏移量，这2个是不同的概念。当然，如果元素没有经过transform 或者 origin为[0,0] 时，这2个的值是一样的。

### 问题解决
在drift方法中，把坐标偏移的参数传入decomposeTransform方法
```
this.decomposeTransform(dx,dy);
```
同时改写一下decomposeTransform方法
```
position[0] += dx;
position[1] += dy;
```
这样就达到预期效果了。