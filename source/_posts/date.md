---
title: Swift 时间格式处理
date: 2019-03-20 20:46:31
tags:
---

#### 1. String -> Date
有一个需要注意的是如果字符串里面包含了时区信息, 那么设置 TimeZone 无效
```Swift
extension String {
    /// String -> Date
    ///
    /// - Parameters:
    ///   - format: 原时间字符串的格式
    ///   - timeZone: 该字符串的时区(若字符串自带时区信息, 则该设置无效), default: 当前时区
    /// - Returns: Date
    func toDate(format: String = "yyyy-MM-dd'T'HH:mm:ss.SSSZ", timeZone: TimeZone? = .current) -> Date? {
        let formatter = DateFormatter()
        formatter.timeZone = timeZone
        formatter.dateFormat = format
        return formatter.date(from: self)
    }
}
```
如下 无论时区设置成什么, 结果都一样
```Swift
var str1 = "2019-03-20T09:48:55.801Z"
// 东八区
let date11 = str1.toDate(timeZone: TimeZone(secondsFromGMT: 28800)) //"Mar 20, 2019 at 5:48 PM"
// 零时区
let date12 = str1.toDate(timeZone: TimeZone(secondsFromGMT: 0))     //"Mar 20, 2019 at 5:48 PM"
```


不包含时区信息时, 这时候时区不同是有影响的
```Swift
var str2 = "2019-03-20 20:48:55"
// 东八区
let date21 = str2.toDate(format: "yyyy-MM-dd HH:mm:ss", timeZone: TimeZone(secondsFromGMT: 28800)) //"Mar 20, 2019 at 8:48 PM"
// 零时区
let date22 = str2.toDate(format: "yyyy-MM-dd HH:mm:ss", timeZone: TimeZone(secondsFromGMT: 0))     //"Mar 21, 2019 at 4:48 AM"
```


#### 2. Date -> String
此时, 设置 timeZone 将会把时间转成时区当地时间并按格式展示
```Swift
extension Date {
    /// Date -> String
    ///
    /// - Parameter
    ///   - format: 需要转为的时间格式
    ///   - timeZone: 展示的时间时区, default: 当前时区
    /// - Returns: String
    func toString(format: String = "yyyy-MM-dd HH:mm:ss", timeZone: TimeZone? = .current) -> String {
        NSTimeZone.resetSystemTimeZone()
        let formatter = DateFormatter()
        formatter.timeZone = timeZone
        formatter.dateFormat = format

        formatter.locale = Locale(identifier: "zh_CN")
        return formatter.string(from: self)
    }
}
```

如下:
```Swift
// 零时区
date3?.toString(format: "yyyy-MM-dd HH:mm:ss", timeZone: TimeZone(secondsFromGMT: 0))     //"2019-03-20 12:48:55"
// 东八区
date3?.toString(format: "yyyy-MM-dd HH:mm:ss", timeZone: TimeZone(secondsFromGMT: 28800)) //"2019-03-20 20:48:55"
```

#### 3. Date -> String(类似朋友圈的时间格式)
```Swift
class TimeTool {
    /// Date -> String(朋友圈时间显示格式)
    ///
    /// - Parameter with: Date
    /// - Returns: String
    static func toMsgTime(with date: Date) -> String? {
        let sinceDate = date
        let curDate = Date()
        let i: Int = Int(curDate.timeIntervalSince(sinceDate))

        let minute: Int = 60
        let hour: Int = minute * 60
        let day: Int = hour * 24
        let month: Int = day * 30 // 平均值
        let year: Int = month * 12

        var str = ""
        switch i {
        case 0..<minute:
            str = "刚刚"
        case minute..<hour:
            str = "\(i/minute) 分钟前"
        case hour..<day:
            str = "\(i/hour) 小时前"
        case day..<month:
            str = "\(i/day) 天前"
        case month..<year:
            str = "\(i/month) 月前"
        case year..<LONG_MAX:
            str = "\(i/year) 年前"
        default:
            return nil
        }
        return str
    }
}
```