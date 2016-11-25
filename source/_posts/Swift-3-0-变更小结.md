## Swift-3-0-变更小结
<!--more-->

[Apple 官网 Swift 3.0 版本变更文档](https://developer.apple.com/library/content/documentation/Swift/Conceptual/Swift_Programming_Language/RevisionHistory.html)
### 新增访问符关键字: open, fileprivate
[Apple 官网解释](https://developer.apple.com/library/content/documentation/Swift/Conceptual/Swift_Programming_Language/AccessControl.html#//apple_ref/doc/uid/TP40014097-CH41-ID3)
**open**:  公开访问接口, 类和成员变量是可以被模块内外  **override**
**public**:  公开访问接口, 但是只能在模块内被 **override**
**internal**: 只在模块中访问
**fileprivate**: 只在当前文件中访问
**private**: 只在当前文件中访问, 并且其 extension 里也不能访问

所以使用 **open** 访问符时需要考虑可能被模块外的代码复写

### 方法签名和调用的变更
举例:
```
     enum PullToRefreshState {
-        case Pulling
-        case Triggered
-        case Refreshing
-        case Stop
+        case pulling
+        case triggered
+        case refreshing
+        case stop
     }

```
声明枚举中的case需要小写开头
```
-    func application(application: UIApplication, didFinishLaunchingWithOptions launchOptions: [NSObject: AnyObject]?) -> Bool {
+    func application(_ application: UIApplication, didFinishLaunchingWithOptions launchOptions: [UIApplicationLaunchOptionsKey: Any]?) -> Bool {

```
```
-        self.posterImageView.contentMode = .ScaleAspectFill
-        self.posterImageView.frame = CGRectMake(0, 0, self.view.wly_width, BarViewHeight);
+        self.posterImageView.contentMode = .scaleAspectFill
+        self.posterImageView.frame = CGRect(x: 0, y: 0, width: self.view.wly_width, height: BarViewHeight);
```
```
-        self.customBar.frame = CGRectMake(0, 0, self.view.wly_width, BarViewHeight);
+        self.customBar.frame = CGRect(x: 0, y: 0, width: self.view.wly_width, height: BarViewHeight);

```
```
-        self.tableView.registerClass(WLYArticleTableViewCell.self , forCellReuseIdentifier: WLYArticleTableViewCell.identifier)
+        self.tableView.register(WLYArticleTableViewCell.self , forCellReuseIdentifier: WLYArticleTableViewCell.identifier)
```
```
-    func tableView(tableView: UITableView, cellForRowAtIndexPath indexPath: NSIndexPath) -> UITableViewCell {
-        let cell = tableView.dequeueReusableCellWithIdentifier(WLYArticleTableViewCell.identifier) as! WLYArticleTableViewCell
+    func tableView(_ tableView: UITableView, cellForRowAt indexPath: IndexPath) -> UITableViewCell {
+        let cell = tableView.dequeueReusableCell(withIdentifier: WLYArticleTableViewCell.identifier) as! WLYArticleTableViewCell

```
```
-        tableView.deselectRowAtIndexPath(indexPath, animated: true)
+        tableView.deselectRow(at: indexPath, animated: true)
```
```
-        self.collectionView.scrollEnabled = false
-        self.collectionView.backgroundColor = UIColor.whiteColor()
-        self.collectionView.registerClass(WLYArticleDetailCell.self, forCellWithReuseIdentifier: WLYArticleDetailCell.identifier)
+        self.collectionView.isScrollEnabled = false
+        self.collectionView.backgroundColor = UIColor.white
+        self.collectionView.register(WLYArticleDetailCell.self, forCellWithReuseIdentifier: WLYArticleDetailCell.identifier)
```
```
-        self.arrow.center = CGPointMake(self.frame.size.width / 2, self.frame.size.height / 2)
-        self.arrow.frame = CGRectOffset(arrow.frame, 0, 0)
+        self.arrow.center = CGPoint(x: self.frame.size.width / 2, y: self.frame.size.height / 2)
+        self.arrow.frame = arrow.frame.offsetBy(dx: 0, dy: 0)

```
```
-        self.titleLabel.font = UIFont.systemFontOfSize(15)
+        self.titleLabel.font = UIFont.systemFont(ofSize: 15)

```
```
-                    self.themeArray?.insert(self.homePageTheme(), atIndex: 0)
+                    self.themeArray?.insert(self.homePageTheme(), at: 0)
```
```
-        self.stateButton.setTitle("夜间", forState: .Normal)
-        self.stateButton.setTitleColor(UIColor.wly_darkTextColor, forState: .Normal)
-        self.stateButton.setImage(UIImage(named: "Menu_Dark"), forState: .Normal)
-        self.stateButton.titleLabel?.font = UIFont.systemFontOfSize(13)
+        self.stateButton.setTitle("夜间", for: UIControlState())
+        self.stateButton.setTitleColor(UIColor.wly_darkTextColor, for: UIControlState())
+        self.stateButton.setImage(UIImage(named: "Menu_Dark"), for: UIControlState())
+        self.stateButton.titleLabel?.font = UIFont.systemFont(ofSize: 13)

```
```
-            guard let hexString: String = rgba.substringFromIndex(rgba.startIndex.advancedBy(1)),

+            guard let hexString: String = rgba.substring(from: rgba.characters.index(rgba.startIndex, offsetBy: 1)),


```
#### GCD
GCD 使用了面向对象的方式调用
```
-            let time = dispatch_time(DISPATCH_TIME_NOW, (Int64)(3 * NSEC_PER_SEC))
-            dispatch_after(time, dispatch_get_main_queue()) {
+            let time = DispatchTime.now() + Double((Int64)(3 * NSEC_PER_SEC)) / Double(NSEC_PER_SEC)
+            DispatchQueue.main.asyncAfter(deadline: time) {
```

#### Block: @escaping && @autoclosure
[Apple 官网介绍](https://developer.apple.com/library/content/documentation/Swift/Conceptual/Swift_Programming_Language/Closures.html#//apple_ref/doc/uid/TP40014097-CH11-ID546)

* 当闭包作为参数传入一个方法, 并且该闭包是在方法返回后才被调用, 我们称为闭包的"逃逸",需要添加 @escaping 标识, 否则编译器会报错
* 当使用了@escaping标识了一个闭包时, 必须在闭包中明确地引用self

```
-    static func requestLatestArticles(completion: (WLYDailyArticle?, NSError?) -> Void) {
+    static func requestLatestArticles(_ completion: @escaping (WLYDailyArticle?, NSError?) -> Void) {
```
