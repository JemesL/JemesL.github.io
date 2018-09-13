---
title: iOS约束冲突警告解决办法
date: 2018-09-12 15:49:50
tags:
---

在开发过程中经常会遇到页面显示正常，但依然还有冲突警告。
通常的原因有2个。

### 一、cell 中的约束冲突
在这种情况下，在 xcode 的控制台打印出得信息当中会单独拎出一条约束。
Will attempt to recover by breaking constraint 下面那一条

```Swift
Probably at least one of the constraints in the following list is one you don't want. 
Try this: 
    (1) look at each constraint and try to figure out which you don't expect; 
    (2) find the code that added the unwanted constraint or constraints and fix it. 
(
    "<SnapKit.LayoutConstraint:0x6040002a08a0@ChoiceArticleCell.swift#182 UIImageView:0x7fae99d49240.left == UIView:0x7fae99d55e70.left>",
    "<SnapKit.LayoutConstraint:0x6040002a0900@ChoiceArticleCell.swift#182 UIImageView:0x7fae99d49240.top == UIView:0x7fae99d55e70.top>",
    "<SnapKit.LayoutConstraint:0x6040002a0960@ChoiceArticleCell.swift#182 UIImageView:0x7fae99d49240.right == UIView:0x7fae99d55e70.right>",
    "<SnapKit.LayoutConstraint:0x6040002a09c0@ChoiceArticleCell.swift#182 UIImageView:0x7fae99d49240.bottom == UIView:0x7fae99d55e70.bottom>"
)

Will attempt to recover by breaking constraint 
<SnapKit.LayoutConstraint:0x6040002a0a20@ChoiceArticleCell.swift#183 UIImageView:0x7fae99d49240.height == UIImageView:0x7fae99d49240.width * 0.425531923770905>
```

解决办法就是：将这条约束的 **优先级降低**。约束默认的优先级是1000。
```Swift
make.height.equalTo(cover.snp.width).multipliedBy(1/2.35).priority(999)
```

### 二、普通view里面 会出现 约束冲突
这种情况 通常是view的 **translatesAutoresizingMaskIntoConstraints**属性 导致
将其改为 false 可以解决问题

```Swift
self.translatesAutoresizingMaskIntoConstraints = false

```