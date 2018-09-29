# SwiftTips (๑•̀ㅂ•́)و✧
记录iOS开发中的一些知识点  

[![Language: Swift 4.2](https://img.shields.io/badge/language-swift4.2-f48041.svg?style=flat)](https://developer.apple.com/swift)
![Platform: iOS 11](https://img.shields.io/badge/platform-iOS-blue.svg?style=flat)


[1.常用的高阶函数 sorted、map、compactMap、filter、reduce](#1)  
[2.高阶函数扩展](#2)

<h2 id="1">1.常用的高阶函数 sorted、map、compactMap、filter、reduce</h2>  

函数式编程在swift中有着广泛的应用,下面列出了几个常用的高阶函数.
#### 1. sorted
常用来对数组进行排序.顺便感受下函数式编程的多种姿势.

##### 1. 使用sort进行排序,不省略任何类型

```swift
let intArr = [13, 45, 27, 80, 22, 53]

let sortOneArr = intArr.sorted { (a: Int, b: Int) -> Bool in
    return a < b
}
// [13, 22, 27, 45, 53, 80]
```
##### 2. 编译器可以自动推断出返回类型,所以可以省略

```swift
let sortTwoArr = intArr.sorted { (a: Int, b: Int) in
    return a < b
}
// [13, 22, 27, 45, 53, 80]
```

##### 3. 编译器可以自动推断出参数类型,所以可以省略

```swift
let sortThreeArr = intArr.sorted { (a, b) in
    return a < b
}
// [13, 22, 27, 45, 53, 80]
```

##### 4. 编译器可以自动推断出参数个数,所以可以用$0,$1替代

```swift
let sortFourArr = intArr.sorted {
    return $0 < $1
}
// [13, 22, 27, 45, 53, 80]
```

##### 5. 如果闭包中的函数体只有一行,且需要有返回值,return可以省略

```swift
let sortFiveArr = intArr.sorted {
    $0 < $1
}
// [13, 22, 27, 45, 53, 80]
```

##### 6. 最简化: 可以直接传入函数`<`

```swift
let sortSixArr = intArr.sorted(by: <)
// [13, 22, 27, 45, 53, 80]
```

#### 2. map和compactMap
##### 1. map: 对数组中每个元素做一次处理.

```swift
let mapArr = intArr.map { $0 * $0 }
// [169, 2025, 729, 6400, 484, 2809]
```
##### 2. compactMap: 和map类似,但可以过滤掉nil,还可以对可选类型进行解包.

```swift
let optionalArr = [nil, 4, 12, 7, Optional(3), 9]
let compactMapArr = optionalArr.compactMap { $0 }
// [4, 12, 7, 3, 9]
```

#### 3. filter: 将符合条件的元素重新组合成一个数组

```swift
let evenArr = intArr.filter { $0 % 2 == 0 }
// [80, 22]
```

#### 4. reduce: 将数组中的元素合并成一个

```swift
// 组合成一个字符串
let stringArr = ["1", "2", "3", "*", "a"]
let allStr = stringArr.reduce("") { $0 + $1 }
// 123*a

// 求和
let sum = intArr.reduce(0) { $0 + $1 }
// 240
```
#### 5. 高阶函数可以进行链式调用.比如,求一个数组中偶数的平方和

```swift
let chainArr = [4, 3, 5, 8, 6, 2, 4, 7]

let resultArr = chainArr.filter {
                            $0 % 2 == 0
                        }.map {
                            $0 * $0
                        }.reduce(0) {
                            $0 + $1
                        }
// 136
```


<h2 id="2">2.高阶函数扩展</h2> 


#### 1. map函数的实现原理

```swift
extension Sequence {
    
    // 可以将一些公共功能注释为@inlinable,给编译器提供优化跨模块边界的泛型代码的选项
    @inlinable
    public func customMap<T>(
        _ transform: (Element) throws -> T
        ) rethrows -> [T] {
        let initialCapacity = underestimatedCount
        var result = ContiguousArray<T>()
        
        // 因为知道当前元素个数,所以一次性为数组申请完内存,避免重复申请
        result.reserveCapacity(initialCapacity)
        
        // 获取所有元素
        var iterator = self.makeIterator()
        
        // 将元素通过参数函数处理后添加到数组中
        for _ in 0..<initialCapacity {
            result.append(try transform(iterator.next()!))
        }
        // 如果还有剩下的元素,添加进去
        while let element = iterator.next() {
            result.append(try transform(element))
        }
        return Array(result)
    }
}

```
map的实现无非就是创建一个空数组,通过for循环遍历将每个元素通过传入的函数处理后添加到空数组中,只不过swift的实现更加高效一点.

关于其余高阶函数的实现,有兴趣的同学可以看下源码,:[Sequence.swift
](https://github.com/apple/swift/blob/swift-4.2-branch/stdlib/public/core/Sequence.swift)

#### 2. 关于数组中用到的其他的一些高阶函数

```swift
class Pet {
    let type: String
    let age: Int
    
    init(type: String, age: Int) {
        self.type = type
        self.age = age
    }
}

var pets = [
            Pet(type: "dog", age: 5),
            Pet(type: "cat", age: 3),
            Pet(type: "sheep", age: 1),
            Pet(type: "pig", age: 2),
            Pet(type: "cat", age: 3),
            ]
```
###### 1. 遍历所有元素
```swift         
pets.forEach { p in
   print(p.type)
}
```
###### 2. 是否包含满足条件的元素
```swift
let cc = pets.contains { $0.type == "cat" }
```
###### 3. 第一次出现满足条件的元素的位置
```swift
let firstIndex = pets.firstIndex { $0.age == 3 }
// 1
```
###### 4. 最后一次出现满足条件的元素的位置
```swift
let lastIndex = pets.lastIndex { $0.age == 3 }
// 4
```
###### 5. 根据年龄从大到小进行排序
```swift
let sortArr = pets.sorted { $0.age < $1.age }
```
###### 6. 获取age大于3的元素
```swift
let arr1 = pets.prefix { $0.age > 3 }
// [{type "dog", age 5}]
```
###### 7. 获取age大于3的取反的元素
```swift
let arr2 = pets.drop { $0.age > 3 }
// [{type "cat", age 3}, {type "sheep", age 1}, {type "pig", age 2}, {type "cat", age 3}]

```
###### 8. 将字符串转化为数组
```swift
let line = "BLANCHE:   I don't want realism. I want magic!"

let wordArr = line.split(whereSeparator: { $0 == " " })
// ["BLANCHE:", "I", "don\'t", "want", "realism.", "I", "want", "magic!"]
```















