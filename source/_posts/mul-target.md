---
title: 多 Target 来进行 版本or环境 区分
date: 2018-11-26 18:13:57
tags:
---

有时候项目对于环境的切换或者是同一套代码不同的app的切换，每次手动改代码很不方便，而且容易出错。
xcode 可以添加 Target 来非常方便的进行切换

### 添加 Target 步骤
- 在项目设置页面，在 TARGETS 下列表中，选择需要复制的target 右键，选Duplicate，再选 Duplicate only
- 然后Targets 下会新增一个 **'xxx copy'** 的target，选中后 单击可改名
- 并且会多了一个  **'xxx copy-info.plist'** 文件
- 可以点击target 配置不通的app name、bundleID、icon 等等其他相关配置

### Target 区分配置
- 在对应的 target 的 buildsetting 下找到 Preprocessor Macros, 该选择下面有个值 Debug 和 Release。在其中 Debug or Release 里面添加 某个参数=1，例如：在DEBUG 里面 添加 DEBUG=1。那么在scheme里面设置 build configuration 设置为 debug时，DEBUG 的值即为true，这个也可以用来切换线上线下环境
```Swift
#if DEBUG
// 测试环境
#else
// 线上环境
#endif
```
- 在buildsetting 下的Other Swift flags 下也可添加标识符 ，在debug 和release 下添加即为true。
例如 如果在新target下的 debug下添加 -DDEBUG， 则DEBUG为true，debug release均添加 -DNEW，则不论debug还是release模式下， NEW均为true，这样就可以区分不同的target的配置一些
```Swift
#if NEW
// NEW target
#else
// 非 NEW target
#endif
```

