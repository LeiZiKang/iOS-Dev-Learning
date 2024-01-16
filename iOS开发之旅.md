# iOS开发之旅

## 循环
```swift
import Foundation

var forLoop = 0
// for
for i in 0..<3 {
    forLoop += i
}
// while
while forLoop > 1 {
    forLoop -= 1
}

// do-while
repeat {
    forLoop *= 2
} while (forLoop <= 2^10)
            
print(forLoop)
print(2^10) // 此处是2进制吗？```
```

swift中函数是一等公民，可以当变量去用

## 闭包

可以理解为没有函数名的函数
```swift
//: [Previous](@previous)

import Foundation

// 闭包就是匿名函数
var oldArray = [1,5,3,2,4]
var array = oldArray.sorted(by: { (a,b) -> Bool in
    return a < b
})
print(array)

let 偶数 = oldArray.filter {
    $0 % 2 == 0
}
print(偶数)

// 例子：算n的阶乘
// 正常的函数法
func facorial(of n: Int64) -> Int64 {
    return (1...n).reduce(1, *)
}
facorial(of: 20)

let yes : (Int) -> Int  = { num in
    return (1...num).reduce(1, *)
}
yes(20)

//: [Next](@next)
```