## [Swift学习笔记]-ObjectMapper-框架
<!--more-->
[Hearst-DD](https://github.com/Hearst-DD)/**[ObjectMapper](https://github.com/Hearst-DD/ObjectMapper)**

ObjectMapper是用Swift语言实现对象和JSON相互转换的框架

ObjectMapper框架支持的数据结构类型:
* Int
* Bool
* Double
* Float
* String
* RawRepresentable(Enums)
* Array<AnyObject>
* Dictionary<String, AnyObject>
* Object<T: Mappable>
* Array<T: Mappable>
* Array<Array<T: Mappable>>
* Set<T: Mappable>
* Dictionary<String, T: Mappable>
* Dictionary<String, Array<T: Mappable>>
* Optionals of all the above //上述的可选类型
* Implicitly Unwrapped Optionals of the above  //上述的隐式解析可选类型

其中Mappable是ObjectMapper框架中定义的一个接口
```
public protocol Mappable {
 /// This function can be used to validate JSON prior to mapping.
 /// Return nil to cancel mapping at this point
 init?(_ map: Map)
 /// This function is where all variable mappings should occur. 
 ///It is executed by Mapper during the mapping (serialization and deserialization) process.
 mutating func mapping(map: Map)
}
```

#### 配置
* 在项目的podfile中添加:
```
pod 'ObjectMapper', '~> 1.3'
```

* 运行 pod install

#### 案例
```
class User: Mappable {
    var username: String?
    var age: Int?
    var weight: Double!
    var array: [AnyObject]?
    var dictionary: [String : AnyObject] = [:]
    var bestFriend: User?                       // Nested User object
    var friends: [User]?                        // Array of Users
    var birthday: NSDate?
    var imageURLs: Array<NSURL>?

    required init?(_ map: Map) {

    }

    // Mappable
    func mapping(map: Map) {
        username    <- map["username"]
        age         <- map["age"]
        weight      <- map["weight"]
        array       <- map["arr"]
        dictionary  <- map["dict"]
        bestFriend  <- map["best_friend"]
        friends     <- map["friends"]
        birthday    <- (map["birthday"], DateTransform())
        posterURL   <- (map["image"], URLTransform())
    }
}
```
自定义的Model需要实现Mappable接口,并在mapping(map: Map)方法中将Model的属性与JSON结构的Key相映射, 如果ObjectMapper支持该属性的类型的转换, 只需要写 
```
username <- map["username"]
```
如果ObjectMapper不支持转换就需要调用Mappable额外提供的类, NSDate类型可以用DateTransform()类转换:
```
birthday <- (map["birthday"], DateTransform())
```
如果ObjectMapper也没有提供类型转化方法就需要自定义了转换类了, 这里就自定义了一个URLArrayTransform类:
```
import Foundation
import ObjectMapper

class URLArrayTransform: TransformType {
    typealias Object = Array<NSURL>
    typealias JSON = Array<AnyObject>
    
    init() {}
    
    func transformFromJSON(value: AnyObject?) -> Array<NSURL>? {
        if let URLStrings = value as? [String] {
            var listOfUrls = [NSURL]()
            for item in URLStrings {
                if let url = NSURL(string: item) {
                    listOfUrls.append(url)
                }
            }
            return listOfUrls
        }
        return nil
    }
    
    func transformToJSON(value: [NSURL]?) -> JSON? {
        if let urls = value {
            var urlStrings = [String]()
            for url in urls {
                urlStrings.append(url.absoluteString)
            }
            return urlStrings
        }
        
        return nil
    }
}
```
自定义的转换类需要实现 ObjectMapper 的 TransformType 接口, 从 URLArrayTransform 的实现中可以看出自定义转换类还是比较简单的, 主要就是重写 transformFromJSON 和 transformToJSON 方法.

完整实现了 User 类后, 就可以是 User 和 JSON 字符串相互转换了

```
let user = User(JSONString: JSONString)

let JSONString = user.toJSONString(prettyPrint: true)  //prettyPrint参数用于生成JSON字符串是否格式化, 以便于打印

使用Mapper类转换
let user = Mapper<User>().map(JSONString: JSONString)

let JSONString = Mapper().toJSONString(user, prettyPrint: true)
```

## 补充: AlamofireObjectMapper

[tristanhimmelman](https://github.com/tristanhimmelman)/**[AlamofireObjectMapper](https://github.com/tristanhimmelman/AlamofireObjectMapper)**

该框架可以结合 Alamofire 和 ObjectMapper 使用, 为Alamofire的Request类扩展出了responseObject 和 responseArray 方法, 更方便的将网络通信返回的JSON数据转换成对象
#### 配置
* 在项目的podfile中添加:
```
pod 'AlamofireObjectMapper', '~> 3.0'
```
* 运行 pod install

#### 案例
```
let URL = "..."
Alamofire.request(.GET, URL).responseObject { (response: DataResponse<WeatherResponse>) in

    let weatherResponse = response.result.value

    if let threeDayForecast = weatherResponse?.threeDayForecast {
        for forecast in threeDayForecast {
            print(forecast.day)
            print(forecast.temperature)           
        }
    }
}
```
