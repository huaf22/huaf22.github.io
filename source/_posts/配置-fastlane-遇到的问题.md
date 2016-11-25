## 配置-fastlane-遇到的问题
<!--more-->
#### 下面的问题是因为 bundler 库本身的一个尚未解决的bug引起的:
```
You'll need to update your bundle to a different version of json (1.8.3) that hasn't been removed in order to install. (Bundler::GemNotFound)
```
面的问题是因为 bundler 库本身的一个尚未解决的bug引起的,
```
[11:48:47]: Exit status of command 'fir publish ../build/Remote_debug.ipa' was 1 instead of 0.
/System/Library/Frameworks/Ruby.framework/Versions/2.0/usr/lib/ruby/2.0.0/universal-darwin16/rbconfig.rb:213: warning: Insecure world writable dir /usr/local/lib in PATH, mode 040777
/usr/local/lib/ruby/gems/2.3.0/gems/bundler-1.13.6/lib/bundler/rubygems_integration.rb:336:in `block (2 levels) in replace_gem': fir-cli is not part of the bundle. Add it to Gemfile. (Gem::LoadError)
	from /usr/local/bin/fir:22:in `<main>'
```

https://rubygems.org/gems/fir-cli
gem 'fir-cli', '~> 1.5'
