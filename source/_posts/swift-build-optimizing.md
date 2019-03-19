---
title: Swift 编译优化
date: 2019-03-07 17:11:07
tags:
---

之前每次编译, 需要花费太长时间. 甚至改一行代码也是如此.
后面花了一些时间去做优化

### 显示编译耗时过长的警告
target -> buildsetting -> Other Swift Flags -> Debug 
添加
-Onone
-Xfrontend
-warn-long-function-bodies=100 // 检测函数体的 类型检查 100ms 是警告的上限
-Xfrontend
-warn-long-expression-type-checking=100 // 检查表达式

### 多线程build
scheme 的 build 页面
勾上 Parallelize Build

### 单文件 增量编译
target -> buildsetting -> SWIFT_WHOLE_MODULE_OPTIMIZATION
设置为NO

### 编译模式
target -> buildsetting -> compilation mode
设置为 incremental 增量模式