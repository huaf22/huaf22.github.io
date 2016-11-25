## 配置-fastlane-框架自动化打包发布-iOS-应用
<!--more-->
[fastlane](https://github.com/fastlane)/**[fastlane](https://github.com/fastlane/fastlane)**
![](https://github.com/fastlane/fastlane/raw/master/fastlane/assets/fastlane_text.png)


#### 安装RVM
```
curl -sSL https://get.rvm.io | bash -s stable --ruby
```
安装 RVM 是需要管理多版本的 Ruby 环境

#### 安装fastlane
```
gem install fastlane --verbose
```
#### fastlane初始化
在项目目录下运行
```
fastlane init
```
命令运行时需要输入Apple ID, 运行成功后会生成 fastlane/Fastlane 文件, Fastlane 是一个 ruby 脚本

#### 使用 bundler 管理依赖
[官网介绍](https://docs.fastlane.tools/getting-started/ios/setup/#use-a-gemfile)
bundler 用来管理 fastlane 自身版本和 fastlane 运行时的相关依赖版本, 相当于 iOS 开发中的 CocoaPods 框架, 使用方法也和 CocoaPods 如出一辙
* 安装 bundler
```
sudo gem install bundler
``` 
* 在项目根目录下新建 Gemfile 文件并写入

  ```
  source "https://gems.ruby-china.org"

  gem "fastlane"
  gem "cocoapods"
  ```
* 安装依赖库, 生成  Gemfile.lock 文件, 这个文件和我们平常接触的 Podfile.lock 文件功能一致, 配置 CI 时也需要在每次构建前调用该命令
  ```
  [sudo] bundle install
  ```
* 更新 Gemfile.lock 文件
```
[sudo] bundle update
```
* 运行 fastlane 框架
```
bundle exec fastlane [lane]
```


####安装match
[github官网介绍](https://github.com/fastlane/fastlane/tree/master/match)
match 是 fastlane 的一个功能组件,  采取了集中化方式来管理证书和 profile,  新建一个私有远程 git 库用来保存证书和 profile, 一个 team 的开发者共用同一套证书, 方便了管理和配置, 同时 match 在证书过期时还会自动从苹果官网下载新的证书并 push 到私有的 git 库中, 保证证书同步, 不得不为这个想法点赞!
```
gem install match
xcode-select --install
```
#### match初始化
```
match init

```

命令运行时需要输入新建的用来保存证书的git地址, 运行成功后会生成 Matchfile 配置文件
```
.
└── fastlane
    ├── Appfile
    ├── Fastfile
    ├── Matchfile
    └── README.md
```
接着运行下面的两条命令来生成证书和 profile
```
match appstore
match development
```
命令运行时需要输入Apple ID 和 密码, 该密码保存在系统的 keychain 中
> 如果不希望用 match 重新生成证书和 profile, 可以参考[Michał Laskowski](http://macoscope.com/blog/simplify-your-life-with-fastlane-match/#main)的博客

如果当前项目的证书和 profile 比较混乱, 可以用一下两个命令来清空 Apple 官网上当前全部证书和 profile 文件
```
match nuke development
match nuke distribution
```
#### match 扩展
如果 Team 来了新成员, 在 clone 了项目工程后运行下面的命令后, match 就从私有证书库中下载develop 证书和 profile, 并安装到新机上, 新成员就可以调试代码了
```
match development --readonly
```

#### Jenkins 配置

![jekins.png](http://upload-images.jianshu.io/upload_images/2591396-1393ffa8757aac0b.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


#### 遇到的问题
* 运行 match appstore 时遇到 openssl 库版本问题, 更新 openssl 库

```
AfluyFileSystem:Remote afluy$ match appstore
[19:53:01]: Successfully loaded '/Users/afluy/WorkCodeIOS/Remote/fastlane/Matchfile' 📄
....
[19:53:01]: Cloning remote git repo...
/Library/Ruby/Gems/2.0.0/gems/fastlane_core-0.53.0/lib/fastlane_core/command_executor.rb:46: warning: Insecure world writable dir /Users/afluy/Applications/dex2jar-2.0 in PATH, mode 040777
[19:53:02]: 🔓  Successfully decrypted certificates repo
[19:53:02]: Verifying that the certificate and profile are still valid on the Dev Portal...
[19:53:20]: 🔒  Successfully encrypted certificates repo
[19:53:20]: -----------------------------------------------------------------------
[19:53:20]: Connection reset by peer - SSL_connect
[19:53:20]:
[19:53:20]: SSL errors can be caused by various components on your local machine.
[19:53:20]: Apple has recently changed their servers to require TLS 1.2, which may
[19:53:20]: not be available to your system installed Ruby (2.0.0)
[19:53:20]:
[19:53:20]: The best solution is to install a new version of Ruby
[19:53:20]:
[19:53:20]: - Make sure OpenSSL is installed with Homebrew: `brew update && brew upgrade openssl`
[19:53:20]: - If you use system Ruby:
[19:53:20]:   - Run `brew update && brew install ruby`
[19:53:20]: - If you use rbenv with ruby-build:
[19:53:20]:   - Run `brew update && brew upgrade ruby-build && rbenv install ruby-2.3.1`
[19:53:20]:   - Run `rbenv global ruby-2.3.1` to make it the new global default Ruby version
[19:53:20]: - If you use rvm:
[19:53:20]:   - First run `rvm osx-ssl-certs update all`
[19:53:20]:   - Then run `rvm reinstall ruby-2.3.1 --with-openssl-dir=/usr/local
[19:53:20]:
[19:53:20]: If that doesn't fix your issue, please google for the following error message:
[19:53:20]:   'Connection reset by peer - SSL_connect'
[19:53:20]: -----------------------------------------------------------------------

[!] Connection reset by peer - SSL_connect
```


* 运行 match development 时遇到 Apple 账号权限问题, 请使用可以上传 profile 的账号

```
AfluyFileSystem:Remote afluy$ match development
[20:35:02]: Successfully loaded '/Users/afluy/WorkCodeIOS/Remote/fastlane/Matchfile' 📄
...
[20:35:02]: Cloning remote git repo...
[20:35:03]: 🔓  Successfully decrypted certificates repo
[20:35:03]: Verifying that the certificate and profile are still valid on the Dev Portal...
[20:35:17]: Couldn't find a valid code signing identity in the git repo for development... creating one for you now
...
[20:35:17]: Starting login with user 'zhao.xianhua@whaley.cn'
[20:35:22]: Successfully logged in
[20:35:25]: $ security import /var/folders/_n/kyvgh8495bv5zmmdg7f85_1r0000gn/T/d20161027-57678-otquli/certs/development/K2935G9MQS.p12 -k '/Users/afluy/Library/Keychains/login.keychain' -T /usr/bin/codesign -T /usr/bin/security
[20:35:25]: ▸ 1 key imported.
[20:35:25]: $ security import /var/folders/_n/kyvgh8495bv5zmmdg7f85_1r0000gn/T/d20161027-57678-otquli/certs/development/K2935G9MQS.cer -k '/Users/afluy/Library/Keychains/login.keychain' -T /usr/bin/codesign -T /usr/bin/security
[20:35:25]: ▸ 1 certificate imported.
[20:35:25]: Successfully generated K2935G9MQS which was imported to the local machine.
security: SecKeychainSearchCopyNext: The specified item could not be found in the keychain.
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100  1062  100  1062    0     0    825      0  0:00:01  0:00:01 --:--:--   825
[20:35:28]: Verifying the certificate is properly installed locally...
[20:35:28]: Successfully installed certificate K2935G9MQS

...

[20:35:29]: Starting login with user '...'
[20:35:31]: Successfully logged in
[20:35:31]: Fetching profiles...
[20:35:31]: No existing profiles found, that match the certificates you have installed locally! Creating a new provisioning profile for you
[20:35:35]: Creating new provisioning profile for '...' with name 'match Development ...'
[20:35:38]: An error occured while verifying your certificates and profiles with the Apple Developer Portal.
[20:35:38]: If you already have your certificates stored in git, you can run `match` in readonly mode
[20:35:38]: to just install the certificates and profiles without accessing the Dev Portal.
[20:35:38]: To do so, just pass `readonly: true` to your match call.
[20:35:38]: 🔒  Successfully encrypted certificates repo

Looking for related GitHub issues on fastlane/fastlane...

Found no similar issues. To create a new issue, please visit:
https://github.com/fastlane/fastlane/issues/new

[!] Apple provided the following error info:
	You are not permitted to create provisioning profiles for team.  Please contact one of your team admins, who can create a profile on your behalf.
```

* Apple 官网和本地机器上已经有证书, 需要删除官网和本地的证书, 也可以运行上面说的 match nuke 命令来清空已有的证书

```
AfluyFileSystem:Remote afluy$ match development
[10:53:48]: Successfully loaded '/Users/afluy/WorkCodeIOS/Remote/fastlane/Matchfile' 📄
...
[10:53:48]: Cloning remote git repo...
[10:53:54]: 🔓  Successfully decrypted certificates repo
[10:53:54]: Verifying that the certificate and profile are still valid on the Dev Portal...
[10:54:00]: Couldn't find a valid code signing identity in the git repo for development... creating one for you now
...
[10:54:00]: Starting login with user 'zhao.xianhua@whaley.cn'
[10:54:02]: Successfully logged in
[10:54:05]: 🔒  Successfully encrypted certificates repo

Looking for related GitHub issues on fastlane/fastlane...

➡️  match: Could not create another certificate, reached the maximum number of available certificates.
   https://github.com/fastlane/fastlane/issues/5765 [closed] 4 💬
   17 Aug 2016

➡️  Could not create another certificate, reached the maximum number of available certificates.
   https://github.com/fastlane/fastlane/issues/3472 [closed] 9 💬
   5 weeks ago

➡️  Could not create another certificate, reached the maximum number of available certificates
   https://github.com/fastlane/fastlane/issues/3123 [closed] 10 💬
   5 weeks ago

and 10 more at: https://github.com/fastlane/fastlane/search?q=Could%20not%20create%20another%20Development%20certificate,%20reached%20the%20maximum%20number%20of%20available%20Development%20certificates.&type=Issues&utf8=✓

[!] Could not create another Development certificate, reached the maximum number of available Development certificates.
```
* Team 中其他开发者需要在 Xcode 中选择 Automatically manage signing
* 在 Setting up CocoaPods master repo 卡住, 这个是 CocoaPods 需要全部 clone 下来, 我最终下载了大概 700M 的文件~~
*  如果不能将改动提交到私有的存储证书的远程 git 库, 可以检查该 git 库的master分支是否被保护

#### 参考资料
https://codesigning.guide/
https://github.com/fastlane/fastlane
https://icyleaf.com/2016/07/intro-fastlane-automation-for-ios-and-android/
https://rvm.io/rvm/install
https://ruby-china.org/wiki/rvm-guide
https://ruby-china.org/wiki/install_ruby_guide
http://macoscope.com/blog/simplify-your-life-with-fastlane-match/#main
http://stackoverflow.com/questions/36963467/how-do-i-manually-add-existing-provisioning-profiles-and-certificates-to-fastlan
http://macoscope.com/blog/simplify-your-life-with-fastlane-match/#migration
