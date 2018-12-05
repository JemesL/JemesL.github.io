---
title: Swift 4.0 编码规范
date: 2018-12-05 14:28:44
tags:
---

### 1.编码格式

* 每行最多160个字符，这样可以避免一行过长。 (Xcode->Preferences->Text Editing->Page guide at column: 设置成160即可)
* 确保每行都不以空白字符作为结尾 （Xcode->Preferences->Text Editing->Automatically trim trailing whitespace + Including whitespace-only lines).
* 使用二元运算符(+, -，==, 或->)的前后都需要添加空格
```Swift
let value = (1 + 2) * 3
```
* 在逗号、冒号后面加一个空格
```Swift
let arr = [1, 2, 3]
let dict = ["123": 123]
```
* 方法的左大括号不要另起，并和方法名之间留有空格，注释空格
```Swift
// 注释
func makeSomething {
  // coding
}
```
* 判断语句不用括号
```Swift
if isTrue = true {
  // coding
}
```
* 方法的左大括号不要另起，并和方法名之间留有空格，注释空格
```Swift
if isTrue = true {
  // coding
}
```
* 尽量不使用self. 除非方法参数与属性同名
```Swift
func setContent(name: String, count: Int) {
    self.name = name
    c = count
}
```
* 在访问枚举类型时，使用更简洁的点语法
```Swift
enum ShareToPlatform {
  case WX
  case TimeLine
  case QQ
  case Sina
}
let platform = .QQ
```

* 使用 // MARK: -，按功能、协议、代理等分组
```Swift
// MARK: - UITableViewDelegate

// MARK: - Action

// MARK: - Request
```
* 使用extension 隔离方法集，相关协议、类似功能放在一个extension里
```Swift
// MARK: - UICollectionViewDelegate
extension JLViewController: UICollectionViewDelegate {
	// 代理方法
}
```
* 当对外接口不兼容时，使用@available(iOS x.0, *) 标明接口适配的起始系统版本号
```Swift
@available(iOS 10.0, *)
func handleSome() {
    //
}
```

### 2.命名规范
* 常量，变量，函数，方法的命名规则使用小驼峰规则，首字母小写，类型名使用大驼峰规则，首字母大写。
```Swift
@available(iOS 10.0, *)
func handleSome() {
    //
}
```
* 当命名里出现缩写词时，缩写词要么全部大写，要么全部小写，以首字母大小写为准
```Swift
let htmlString = "https://www.baidu.com"
let urlString:  URLString
let userID:  UserID

class HTMLModel {
    //
}
```
* bool类型命名时，使用is作为前缀
```Swift
var isHave: Bool = false
```
* 使用前缀 k + 大骆驼命名法 为所有非单例的静态常量命名。
```Swift
class MyClassName {
    // 基元常量使用 k 作为前缀
    static let kSomeConstantHeight: CGFloat = 80.0
    // 非基元常量也是用 k 作为前缀
    static let kDeleteButtonColor = UIColor.redColor()
    // 对于单例不要使用k作为前缀
    static let sharedInstance = MyClassName()
    /* ... */
}
```
### 3.代码风格
* 如果需要把访问修饰符放到第一个位置。
* 尽可能的多使用let，少使用var。
* 当需要遍历一个集合并变形成另一个集合时，推荐使用函数 map, filter 和 reduce。
```Swift
// 推荐
let stringOfInts = [1, 2, 3].flatMap { String($0) }
// ["1", "2", "3"]
// 不推荐
var stringOfInts: [String] = []
for integer in [1, 2, 3] {
    stringOfInts.append(String(integer))
}
// 推荐
let evenNumbers = [4, 8, 15, 16, 23, 42].filter { $0 % 2 == 0 }
// [4, 8, 16, 42]
// 不推荐
var evenNumbers: [Int] = []
for integer in [4, 8, 15, 16, 23, 42] {
    if integer % 2 == 0 {
        evenNumbers(integer)
    }
}
```
* 如果变量类型可以依靠推断得出，不建议声明变量时指明类型。
* 不要使用 as! 或 try!。
* 我们推荐使用提前返回的策略，而不是 if 语句的嵌套。使用 guard 语句可以改善代码的可读性。
```Swift
// 推荐
func eatDoughnut(atIndex index: Int) {
    guard index >= 0 && index < doughnuts else {
        // 如果 index 超出允许范围，提前返回。
        return
    }
    let doughnut = doughnuts[index]
    eat(doughnut)
}
// 不推荐
func eatDoughnuts(atIndex index: Int) {
    if index >= 0 && index < donuts.count {
        let doughnut = doughnuts[index]
        eat(doughnut)
    }
}
```

### 4.注释规范
* MARK，用于方法或函数的注释。加上 ‘-’ 再函数列表会多一个横线分割
```Swift
// MARK: - 描述内容
// MARK: 描述内容
```
* TODO，表示这里代码有没有完成，还要处理。加上 ‘-’ 再函数列表会多一个横线分割
```Swift
// TODO: - 描述内容
// TODO: 描述内容
```
* FIXME，需要修改代码，bug等。加上 ‘-’ 再函数列表会多一个横线分割，且显示的icon 与mark和todo不通，更醒目
```Swift
// FIXME: - 描述内容
// FIXME: 描述内容
```
* 添加文档注释（xcode 快捷键⌘⌥/）
```Swift
/// 获取用户的帐密信息
///
/// - Returns: 信息元组的 Observable
func fetchLocalInfo() -> Observable<(String, String)?> {
    // coding
}
```