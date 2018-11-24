---
title: runloop
date: 2018-09-20 14:13:03
tags:
---

### runloop相关的应用
#### 滑动列表的优化
cell中设置图片 会卡顿以下，可以吧设置image的代码放置在runloop default 的mode里运行。
避免滑动的过程中去触发设置image的动作。

#### app Crash 的优化
crash后 可以重新唤醒一个app 做一些提示性的文字 发送报错邮件等
