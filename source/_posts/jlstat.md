---
title: JLSTAT
date: 2019-10-03 10:35:00
tags:
---

前面为了方便 给视图添加多状态操作, 是通过继承的方式添加属性来操作的.
这样的缺点是不得不为每一个空间都要写一个控件写一个子类.

后面通过参照rxswift, 来给所有的view 添加多状态操作.

主要也是先通过给view 添加一个 jl 的命名空间, 然后再通过运行时, 为 view 关联一个block.
设置状态时, 再调用关联的block.

[Demo](https://github.com/JemesL/JLSTAT)


```Swift
extension JLSpace where Base: UIView {
    
    func setStatBlock<E>(defaultValue: E, block: @escaping (Base, E) -> ()) {
        typealias STATBE = STAT<Base, E>
        let stat = STATBE(e: defaultValue, block: block)
        objc_setAssociatedObject(self.base, &AssociatedKeys.statKey, stat, .OBJC_ASSOCIATION_RETAIN)
        
        block(self.base, defaultValue)
    }
    
    func setStat<E>(e: E) {
        typealias STATBE = STAT<Base, E>
        if var stat = objc_getAssociatedObject(self.base, &AssociatedKeys.statKey) as? STATBE {
            stat.block?(self.base, e)
            stat.e = e
            objc_setAssociatedObject(self.base, &AssociatedKeys.statKey, stat, .OBJC_ASSOCIATION_RETAIN)
        }
    }
    
    func getStat<E>() -> E? {
        if let statBlock = objc_getAssociatedObject(self.base, &AssociatedKeys.statKey) as? STAT<Base, E> {
            return statBlock.e
        }
        return nil
    }
}

struct STAT<Base, E> {
    typealias STATBLOCK = (Base, E) -> ()
    
    var block: STATBLOCK?
    var e: E?
    init(e: E?, block: STATBLOCK?) {
        self.block = block
        self.e = e
    }
}
```