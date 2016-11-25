## [iOS笔记]Swift中的闭包(Closures)

<!--more-->
闭包是自包含的函数代码块,Swift 中的闭包与 C 和 Objective-C 中的代码块(blocks)以及其他一些编程语言中的匿名函数比较相似。

闭包表达式语法的如下一般形式:
```
 { (parameters) -> returnType in
     statements
}
```
举个例子:
```
func backwards(s1: String, s2: String) -> Bool {
    return s1 > s2
}
var reversed = names.sort(backwards)
```

上面例子对应的闭包表达式版本的代码:

```
 reversed = names.sort({ (s1: String, s2: String) -> Bool in
    return s1 > s2
})
```
## 1. 闭包特性
#### 1.1 根据上下文推断类型(Inferring Type From Context)

任何情况下,通过内联闭包表达式构造的闭包作为参数传递给函数或方法时,都可以推断出闭包的参数和返回值类型。
```
 reversed = names.sort( { s1, s2 in return s1 > s2 } )

```

#### 1.2 单表达式闭包隐式返回(Implicit Return From Single-Expression Clossures)
单行表达式闭包可以通过省略 return 关键字来隐式返回单行表达式的结果。

```
reversed = names.sort( { s1, s2 in s1 > s2 } )
```
#### 1.3 参数名称缩写(Shorthand Argument Names)
Swift 自动为内联闭包提供了参数名称缩写功能,您可以直接通过 $0 , $1 , $2 来顺序调用闭包的参数,以此类推。
```
reversed = names.sort( { $0 > $1 } )
```
> ##### P.S. 我们可以进一步精简
###### 运算符函数(Operator Functions):
Swift 的 String 类型定义了关于大于号( > )的字符串实现,其作为一个函数接受两个 String 类型的参数并返回 Bool 类型的值。
```
reversed = names.sort(>)
```

## 2. 尾随闭包(Trailing Closures)
如果需要将一个很长的闭包表达式作为最后一个参数传递给函数,可以使用尾随闭包来增强函数的可读性。
```
//函数声明
func someFunctionThatTakesAClosure(closure: () -> Void) { 
	// 函数体部分
}

// 以下是不使用尾随闭包进行函数调用
someFunctionThatTakesAClosure({
	// 闭包主体部分 
})

// 以下是使用尾随闭包进行函数调用 
someFunctionThatTakesAClosure() {
	// 闭包主体部分 
}
```

```
//之前的例子可以写成:
reversed = names.sort() { $0 > $1 }

//如果函数只需要闭包表达式一个参数,当使用尾随闭包时,甚至可以把 () 省略掉:
reversed = names.sort { $0 > $1 }
```

## 3. 捕获(capturing)
闭包可以捕获和存储其所在上下文中任意常量和变量的引用。
Swift 会管理在捕获过程中涉及到的所有内存操作,包括释放不再需要的变量。

即使定义这些常量和变量的原作用域已经不存在,闭包仍然可以在闭包函数体内引用和修改这些值。

Swift 中,可以捕获值的闭包的最简单形式是嵌套函数,也就是定义在其他函数的函数体内的函数。
嵌套函数可以捕获其外部函数所有的参数以及定义的常量和变量。

```
func makeIncrementor(forIncrement amount: Int) -> () -> Int {
     var runningTotal = 0
     //声明内嵌函数
     func incrementor() -> Int { 
         runningTotal += amount //内嵌函数引用了外部函数的amount和runningTotal变量
         return runningTotal
     }
     return incrementor
 }

let incrementByTen = makeIncrementor(forIncrement: 10)

incrementByTen() // 返回的值为10
incrementByTen() // 返回的值为20
incrementByTen() // 返回的值为30

```
> P.S 上面的方法调用可以引申理解为makeIncrementor函数体是一个class,
而incrementor是class中的函数,incrementor函数引用了class中的属性amount和runningTotal

如果创建了另一个 incrementor ,它会有属于它自己的一个全新、独立的 runningTotal 变量的引用:

```
let incrementBySeven = makeIncrementor(forIncrement: 7)
incrementBySeven() // 返回的值为7
```

## 4. 闭包是引用类型(Closures Are Reference Types)

上面的例子中, incrementBySeven 和 incrementByTen 是常量,但是这些常量指向的闭包仍然可以增加其捕获的变量的值。这是因为函数和闭包都是引用类型。
无论将函数或闭包赋值给一个常量还是变量,实际上都是将常量或变量的值设置为对应函数或闭包的引 用。上面的例子中,指向闭包的引用 incrementByTen 是一个常量,而并非闭包内容本身。

> ##### 这也意味着如果将闭包赋值给了两个不同的常量或变量,两个值都会指向同一个闭包!

## 5. 非逃逸闭包(Nonescaping Closures)
当一个闭包作为参数传到一个函数中,但是这个闭包在函数返回之后才被执行,我们称该闭包从函数中逃逸。
你可以在参数名之前标注 @noescape ,用来指明这个闭包是不允许“逃逸”出这个函数的。
将闭包标注 @noescape 能使编译器知道这个闭包的生命周期(闭包只能在函数体中被执行,不能脱离函数体执行,所以编译器明确知道运行时的上下文),从而可以进行一些比较激进的优化。
```
func someFunctionWithNoescapeClosure(@noescape closure: () -> Void) {
    closure()
}

//函数接受的闭包被添加到一个函数外定义的数组中
var completionHandlers: [() -> Void] = []
func someFunctionWithEscapingClosure(completionHandler: () -> Void) {
    completionHandlers.append(completionHandler)
}

class SomeClass {
    var x = 10
    func doSomething() {
        //将闭包标注为 @noescape 使能在闭包中隐式地引用 self
        someFunctionWithEscapingClosure { self.x = 100 } 
        someFunctionWithNoescapeClosure { x = 200 }
    }
}

let instance = SomeClass()

instance.doSomething()
print(instance.x) // prints "200"

completionHandlers.first?()
print(instance.x) // prints "100"
```

## 6. 自动闭包(Autoclosures)
这种闭包不接受任何参数,当它被调用的时候,会返回被包装在其中的表达式的值。
```
var customersInLine = ["Chris", "Alex", "Ewa", "Barry", "Daniella"]

// customerProvider 的类型不是 String ,而是 () -> String 
let customerProvider = { customersInLine.removeAtIndex(0) }

print("Now serving \(customerProvider())!") // prints "Now serving Chris!"
print(customersInLine.count)   // prints "4"
```

```
// customersInLine is ["Alex", "Ewa", "Barry", "Daniella"]
func serveCustomer(customerProvider: () -> String) {
    print("Now serving \(customerProvider())!")
}
serveCustomer( { customersInLine.removeAtIndex(0) } )
// prints "Now serving Alex!"


使用@autoclosure
func serveCustomer(@autoclosure customerProvider: () -> String) {
     print("Now serving \(customerProvider())!")
}

serveCustomer(customersInLine.removeAtIndex(0))
 // prints "Now serving Ewa!"
```

@autoclosure特性暗含了@noescape 特性,如果你想让这个闭包可以“逃逸”,则应该使用@autoclosure(escaping) 特性.

参考资料:The Swift Programming Language
