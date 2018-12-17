---
title: iOS提高页面书写效率
date: 2018-11-24 21:54:15
tags:
---

之前的项目用了的RXSwift 和 RxCocoa，现在结合RxCocoa 做一些动态布局的优化。

### 一、实现类似H5的流式布局
如果了解H5的应该都知道它的流式布局非常方便，不需要考虑上下元素的关系。
iOS的布局其实类似H5的绝对布局，但每每要写清楚上下元素之间的关系，而且每当页面有元素增删的时候又要去修改相关的约束。

现在一个替代思路就是写一个通用的函数，给定元素的数组，父view等参数，函数内部执行添加相关约束。避免每次都要写约束以及改动

### 二、点击功能的按钮的状态切换
在写某些按钮的时候，选择状态 和 非选择状态 样式差异比较大，btn自带的那些状态 不能够满足条件。
有时候只能根据上层传来的方法再去做相应的修改。
但是这样也写代码就不是舒服，可以结合rxswift ，继承btn,添加几个属性，如下：
```Swift
lazy var disposeBag = DisposeBag()
typealias Block = (STATButton) -> ()
var stat = BehaviorRelay<Bool>(value: false)
var trueSB: Block?
var falseSB: Block?
```

然后再init 里面订阅 stat，后续会对stat
```Swift
stat.skip(1).asObservable().subscribe(onNext: { [weak self] status in
  let block = status ? self?.trueSB : self?.falseSB
  if let b = block {
    b(self!)
  }
}).disposed(by: disposeBag)
```

在写样式的时候可以吧选择和非选择状态一起写了
```Swift
likeBtn.trueSB = { btn in
    btn.setImage(UIImage(named: "zan_full.png"), for: .normal)
    btn.setTitle("已赞", for: .normal)
}
likeBtn.falseSB = { btn in
    btn.setImage(UIImage(named: "zan.png"), for: .normal)
    btn.setTitle("赞", for: .normal)
}
```

再赋值的时候就不用再关心样式的问题，只需要给stat属性赋一个true or false
```Swift
likeBtn.stat.accept(true)
```

同样的 UIView UIImageView 都可以类似操作

### 三、部分元素根据数据是否显示对上下元素的影响
例如在写用户评论哪一块，类似朋友圈。
根据数据，点赞、评论、图片等元素可能有可能没有，还要考虑各个元素间的距离

目前解决办法也和第二条一样，在写约束的时候，写在trueSB 和 falseSB 里面
```Swift
heart.trueSB = { [weak self] v in
  v.snp.remakeConstraints { make in
    make.left.equalTo(ContentEdge.left)
    make.top.equalTo(self!.content.snp.bottom).offset(10)
  }
}
heart.falseSB = { [weak self] v in
  v.snp.remakeConstraints { make in
    make.left.equalTo(ContentEdge.left)
    make.height.equalTo(0)
    make.top.equalTo(self!.content.snp.bottom).offset(0)
  }
}
```
后面两条主要是让样式代码和逻辑代码，更加高聚合低耦合

ps: 其实也可以通过写一个set方法来调用block，达到同样的效果，不必使用rxcocoa