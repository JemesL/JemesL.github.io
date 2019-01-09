---
title: 一个比较少见的循环引用
date: 2019-01-09 18:11:42
tags:
---

循环引用应该大部分人都知道怎么回事.

####较常见的就有
* A实例 强引用 B实例, B 强引用 A.
* A实例强引用 Block B, B 里面 调用了self

在开发的时候发现另一个类型, 不过有点类似上述的第二个

```Swift
import Foundation
class Person {

    var info: Info?

    struct Info {
        var name: String
        var family: String
        var age: Int
    }

    func printInfo() {
        if let info = info {
            print(info)
        }
    }

    func transform() -> Info {
        self.info = Info(
            name: "吴彦祖",
            family: "吴氏家族",
            age: 20
        )
        return self.info!
    }

    deinit {
        print("deinit")
    }
}

let block = {
    print("begin")
    let p = Person()
    p.transform()
    print(CFGetRetainCount(p as CFTypeRef))
    print("end")
}
block()
```
上述代码 run 一下 结果如下, 
```Swift
begin
2
end
deinit
```
打印 deinit 了,说明 p 正常释放了, 没有循环引用


然后现在改一下 Struct Info 和 transform 方法, 在里面添加 prinfFunc 方法
```Swift
struct Info {
    var name: String
    var family: String
    var age: Int
    var printFunc: () -> ()
}
func transform() -> Info {
    return Info(
        name: "吴彦祖",
        family: "吴氏家族",
        age: 20,
        printFunc: printInfo
    )
}
```

再次 run 一下, 打印结果如下
```Swift
begin
3
end
deinit
```
依旧可以打印出 deinit, 但是 p 的引用计数 +1.
说明在 struct 里面持有 某实例的方法会让该实例引用计数 +1.

现在如果把 transform 调整如下, 返回 info 之前让 赋值给 self.info
```Swift
func transform() -> Info {
    self.info = Info(
        name: "吴彦祖",
        family: "吴氏家族",
        age: 20,
        printFunc: printInfo
    )
    return self.info!
}
```

打印如下:
```Swift
begin
3
end
```
这一次没有 deinit, p 没有被释放, 形成了循环引用.
造成这个循环主要是没想到 struct 里面持有方法,也会造成计数 +1.
