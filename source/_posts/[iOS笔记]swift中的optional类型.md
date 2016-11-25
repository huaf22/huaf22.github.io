# [iOS笔记]Swift中的Optional类型 (可选类型)

<!--more-->

### Optional类型表示: 有值 / 没有值

在Objective-C中并没有Optional类型, 只有nil,并且nil只能用于表示对象类型无值,

并不能用于基础类型(int, float),枚举和结构体,基础类型需要返回类似NSNotFound的特殊值来表示无值,

所以在Swift中定义了Optinal类型来表示各种类型的无值状态,并规定了nil不能用于非可选的常量和变量,只能用于Optinal类型

### 先看一下基本的用法:

```
var serverResponseCode: Int? = nil
// serverResponseCode 现在不包含值

serverResponseCode = 404
// serverResponseCode 包含一个可选的 Int 值 404 
Int? 表示serverResponseCode可以是有值的也可以是无值的

var surveyAnswer: String?
//surveyAnswer 被自动设置为 nil

```

### 在if语法中的用法

使用 if 语句和 nil 来判断一个Optional类型是否有值

```
var convertedNumber: Int? 

if convertedNumber != nil {
     print("convertedNumber contains some integer value.")
}

```

### 可选绑定( optional binding )
使用optional binding来判断optional类型是否有值,并且如果有值就把值赋给一个常量或者临时变量。

optional binding可以用在 if 和  while 语句中:

```
if let constantName = someOptional {
     print("get the value: \(constantName)")
}

//如果someOptional这个optional类型有值,创建一个叫做constantName的常量并将someOptional包含的值赋给它。
//并且因为constantName已经被optional类型包含的值初始化过,所以不需要再使用 ! 后缀来获取它的值。

```

###  强制解析/解包 ( forced unwrapping )

当确定可选类型确实包含值之后,可以在可选的名字后面加一个感叹号( ! )来获取值;

当Option == nil时,使用 ! 来获取会导致运行时错误。所以使用 ! 来强制解析值之前,一定要确定Option类型不是nil的


### 隐式解析可选类型 ( implicitly unwrapped optionals )

有时候在编写程序中,一个optional类型第一次被赋值后,可以确定它以后总会有值,

在这种情况下,使用它时每次都还需要判断和解包optional类型是非常低效的,因为可以确定它总会有值,

所以Swift中定义了一个隐式解析可选类型,它在使用时可以自动解析;

把( String? )改成感叹号( String! )就可以声明一个隐式解析可选类型。

```
let possibleString: String? = "An optional string."
let forcedString: String = possibleString! // 需要惊叹号来获取值

let assumedString: String! = "An implicitly unwrapped optional string." 
let implicitString: String = assumedString // 不需要感叹号
```

但是需要注意:

- 如果你在隐式解析可选类型没有值的时候取值,会触发运行时错误,
  
  这和你在无值的普通Optional类型后面加一个惊叹号的情景一样;


- 如果一个变量以后还可能变成 nil 的话,不要使用隐式解析可选类型;

- 并且如果需要在变量的生命周期中判断是否 是 nil 的话,请使用普通可选类型;


#### 可以理解为隐式解析可选类型用于声明初始化时可能为nil,但之后会立刻被赋值,并且不会再变成nil的Optional类型对象

