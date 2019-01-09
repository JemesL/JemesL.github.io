---
title: RXSwift
date: 2018-09-14 11:40:14
tags:
---

最近新的项目采用了rxswift + mvvm 来写。
网络框架用的moya 的rxswift版。

总的体验来说，mvvm架构中，确定体会到了把数据逻辑剥离出来并且方便复用的便利。
采用了响应式编程理念，用了rxswift。在某些场景下，rxswift 的体验非常好.
不过现在也造成了在问题追踪上会比较麻烦。

这几个月当中也是踩了不少坑，结合实际开发简单说说rxswift的一些实际运用。

### 实际应用当中的一些使用场景
#### 1. 简化tableview等数据绑定流程
结合 RXDataSource 简化 tableview、collectionview 等的数据绑定
#### 2. 简化复杂页面多状态间的逻辑
类似登录、购物车等这些多个状态互相之间影响的，使用 rxswift 会极大的简化逻辑流程。
#### 3. 同步状态信息 
类似很多页面出现相同的数据列表，列表还会展示状态信息，比如点赞、收藏等。
这些状态信息同步可以通过发送事件，在所有相关列表订阅这个事件，然后处理改变各自列表的状态

### rxswift学习笔记
写的并不全，主要是对一些常用的地方做个简单的记录
#### 1. **share(replay:scope:)**
在某些情况下，可能会对Observer 进行多次订阅，如下
```Swift
var ob = Observable.of(1,2,3).map { print($0) }.share(replay: 1, scope: .forever)
ob.subscribe(onNext: { res in
    print("first sub")
})

ob.subscribe(onNext: { res in
    print("second sub")
})
```
执行结果如下，多次订阅会将整个链式多次执行，但大多数情况下只希望map 只执行一次，比如map里面可能是网络请求等等。
```
1
first sub
1
second sub
```
可以在map后面添加share
```Swift
var ob = Observable.of(1,2,3).map { print($0) }.share(replay: 1, scope: .forever)
```
现在链式只执行了一次
```
1
first sub
second sub
```
#### 2. **retry**
当遇到error 时，会重新订阅该序列
retry(2)里面数字代表重试次数，默认1次
```
enum MyError: Error {
    case A
    case B
}

var count = 1
let sequenceThatErrors = Observable<String>.create { observer in
    observer.onNext("a")
    observer.onNext("b")

    //让第一个订阅时发生错误
    if count == 1 {
        observer.onError(MyError.A)
        print("Error encountered")
        count += 1
    }

    observer.onNext("c")
    observer.onNext("d")
    observer.onCompleted()

    return Disposables.create()
}

sequenceThatErrors
    .retry(2)  //重试2次（参数为空则只重试一次）
    .subscribe(onNext: { print($0) })

```
#### 3. **debug**
可以打印相关信息，方便调试
多个debug时，添加字符串予以区分：debug("调试")
```Swift
var ob = Observable.of(1).map { print($0) }.share(replay: 1, scope: .forever                    )
ob.debug("frist").subscribe(onNext: { res in
    print("first sub")
})

ob.debug("second").subscribe(onNext: { res in
    print("second sub")
})
```
打印：
```
2018-12-24 15:07:29.612: frist -> subscribed
1
2018-12-24 15:07:29.614: frist -> Event next(())
first sub
2018-12-24 15:07:29.614: frist -> Event completed
2018-12-24 15:07:29.614: frist -> isDisposed
2018-12-24 15:07:29.655: second -> subscribed
2018-12-24 15:07:29.656: second -> Event next(())
second sub
2018-12-24 15:07:29.656: second -> Event completed
2018-12-24 15:07:29.656: second -> isDisposed
```

#### 4. **Single**
Single 是 Observable 的另外一个版本。但它不像 Observable 可以发出多个元素，它要么只能发出一个元素，要么产生一个 error 事件。
* 发出一个元素，或一个 error 事件
* 不会共享状态变化

#### 5. **Subjects**
subjects 即是 Observer，也是 Observable。它可以动态接受新的值，然后通过 Event 将新值发送给所有的订阅者

subjects 一共有四种：PublishSubject、BehaviorSubject、ReplaySubject、Variable
共同点：
* 一旦发送 .complete or .error 事件，该subject 将终结，不再发送 .next 事件
* 对于在终结后订阅该sunject 的订阅者，也会收到一条.complete or .error，告诉新的订阅者已经终结了
区别：
* 最大的区别在于新的订阅者能不能收到旧的event，能的话能收到多少个。

1. **PublishSubject**
* 最普通的subject，不需要初始值就能创建
* 只能收到订阅之后的event，无法收到旧的 event

```Swift
let s = PublishSubject<String>()
s.onNext("first")
s.subscribe(onNext: { s in
    print("sub \(s)")
}).disposed(by: disposeBag)
s.onNext("second")
```
```
sub second
```

2. **BehaviorSubject**
* 需要一个初始值来创建
* 一个新的订阅者会立刻收到上一个event，之后照常

3. **ReplaySubject**
* 创建的时候需要设置一额bufferSize，表示它对发送过的event的缓存个数
* 它会缓存bufferSize个 .next 事件，新的订阅者会立刻收到缓存的 .next事件
* 如果是在subject 结束后订阅的，则除了缓存的 .next事件 还会收到 .complete or .error

4. **Variable**
* 是对 BehaviorSubject 的一层封装，必须通过一个初始值来创建
* 具有 BehaviorSubject 相同的功能，能向新的订阅者发送上一个event 还有之后新的event
* 不同的是， BehaviorSubject会吧值作为自己的属性保存下来，在销毁时会自动发送 .complete。
* 不需要也不能给它发送 .complete or .error 来结束它

5. **BehaviorRelay**
* 是对 BehaviorSubject 的一层包装
* 和 BehaviorSubject 不同的是不能用 .error or .complete 来结束

#### 6. **drive**
如果我们的序列满足如下特征，就可以使用它：
* 不会产生 error 事件
* 一定在主线程监听（MainScheduler）
* 共享状态变化（shareReplayLatestWhileConnected）

#### 7. **flatMap**
* flatMap 会对每一个元素应用一个转换方法，转换成一个 Observable，等于是变成一个Observables的序列，然后再对Observables的元素合并之后再发出来，等于又将其降维成一个Observable
* 这个用处很大，如果一个Observable 的元素同样也是一个Observable时，利用flatMap可以将其元素都发送出来。
```Swift
let subject1 = BehaviorSubject(value: "A")
let subject2 = BehaviorSubject(value: "1")
 
let variable = Variable(subject1)
 
variable.asObservable()
    .flatMap { $0 }
    .subscribe(onNext: { print($0) })
 
subject1.onNext("B")
variable.value = subject2
subject2.onNext("2")
subject1.onNext("C")
```

#### 8. **ControlProperty**
ControlProperty 是专门用来描述UI控件属性，拥有该类型的属性都是被观察者Observable
有一下特征：
* 不会产生error 事件
* 一定在 MainScheduler 订阅
* 一定在 MainScheduler 监听

#### 9. **ControlEvent**
ControlEvent 是专门用来描述UI所产生的事件，拥有该类型的属性都是被观察者Observable有一下特征：
* 不会产生error 事件
* 一定在 MainScheduler 订阅
* 一定在 MainScheduler 监听
* 共享状态变化
