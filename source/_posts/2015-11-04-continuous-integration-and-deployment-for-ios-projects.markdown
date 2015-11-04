---
layout: post
title: "iOS 项目持续集成和部署笔记"
date: 2015-11-04 09:38:13 +0800
comments: true
categories: [Archives]
keywords: iOS, Continuous Integration, Deployment
discription: Continuous Integration & Deployment for iOS Projects
---
本文是我做 iOS 项目持续集成和部署的笔记。

### 安装工具

1. [jenkins](http://jenkins-ci.org/);

```
$ brew update && brew install jenkins
```

2. [fastlane](https://github.com/fastlane/fastlane);

```
$ sudo gem install fastlane --verbose
```

3. [fir-cli](https://github.com/FIRHQ/fir-cli/blob/master/README.md);

```
$ sudo gem install fir-cli
``` 

### fastlane 自动构建

* cd [your_project_folder]
* fastlane init
* Follow the setup assistant, which will set up fastlane for you
* Edit Fastfile

```
fastlane_version "1.25.0"

default_platform :ios


platform :ios do
  before_all do
    ENV["SLACK_URL"] = "https://hooks.slack.com/services/..."
    #cocoapods

    increment_build_number

    gym(scheme: "srsApp") # Build your app - more options available

    # xctool # run the tests of your app
  end

  desc "Runs all the tests"
  lane :test do
    snapshot
  end

  desc "Submit a new Beta Build to fir"
  desc "This will also make sure the profile is up to date"
  lane :fir do
    #snapshot
    sh "sigh --team_id your_team_id"
    #deliver(beta: true)

    #"ipa distribute:pgyer -u USER_KEY -a APP_KEY" # 上传到蒲公英

    #"ipa distribute:fir -u USER_TOKEN -a APP_ID" # 上传到fir.im

    # sh"ipa distribute:fir -u USER_KEY -a APP_KEY"
    # You can also use other beta testing services here
    sh "fir p ../srsApp.ipa -T USER_TOKEN"
  end

  desc "Submit a new Beta Build to pgyer "
  desc "This will also make sure the profile is up to date"
  lane :pgyer do
    #snapshot
    sh "sigh --team_id your_team_id"
    #deliver(beta: true)

    # You can also use other beta testing services here
    sh "curl -F \"file=@/Users/dongmeiliang/Documents/HJKApp/srsApp/srsApp.ipa\" -F \"uKey=USER_TOKEN\" -F \"_api_key=APP_KEY\" http://www.pgyer.com/apiv1/app/upload"
  end

  # You can define as many lanes as you want

  after_all do |lane|
    # This block is called, only if the executed lane was successful

     slack(
       message: "Successfully deployed new App Update."
     )
  end

  error do |lane, exception|
     slack(
       message: exception.message,
       success: false
     )
  end
end
```
使用 `fastlane fir`测试能否正常构建。

### 自动测试
可以利用 xcodebuild 命令来实现自动测试。
```
$ xcodebuild -project UI\ Testing\ In\ Xcode.xcodeproj -scheme UI\ Testing\ In\ Xcode -destination 'platform=iOS Simulator,name=iPhone 6s,OS=9.1' test
```

### jenkins 持续集成

1. 创建自动构建 job

* Open http://localhost:8080 with safari
* Install plugins
	* HTML Publisher Plugin
	* AnsiColor Plugin
	* Rebuild Plugin
	* GIT Plugin
	
* Create job > input item name > Freestyle project > OK
* Configuration
	Source Code Management
	Git
	Repository URL /Users/dongmeiliang/Documents/HJKApp
	Branches to build */test
	Add build step > Execute shell
	
	```
	#!/bin/bash 
	source ~/.bash_profile
	cd srsApp/
	fastlane  fir
	```
* Save > build

2. 创建自动测试 job


##Reference  
Testing with Xcode
[fastlane](https://github.com/fastlane/fastlane)  
[Jenkins Integration](https://github.com/fastlane/fastlane/blob/master/docs/Jenkins.md)  
[iOS可持续化集成: Jenkins + bundler + cocoapods + shenzhen + fastlane + pgyer](http://blog.csdn.net/colorapp/article/details/47007329)  
	
	

