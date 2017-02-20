---
layout: post
title: "iOS Development--Certificates, Provisioning Profiles"
date: 2014-10-09 16:24:09 +0800
comments: true
categories: [Archives, iOS Development]
keywords: Certificates, Provisioning, App ID, UDID, CSR 
discription: Introduce Certificates and Provisioning Profiles
---
在 iOS 开发中，Certificate 和 Provisioning Profle 可能是我们容易困惑的一部分内容。这篇文章我打算来梳理下这部分知识，只有理解了其中的内容，我们才能解决在开发过程可能遇到的代码签名问题。  

先来看张图，所谓一图胜千言：

<div style="text-align: center" markdown="1"> 
	<img name="LaunchApp" src="/images/LaunchApp.png" width="697" height="573">  
</div>

从这张图上我们可以看到，只有应用的 bundle ID 匹配 App ID, certificate 匹配到 Provisioning Profile 中的 Certificate, device ID 匹配到 Provisioning Profile 中的 device ID。嗯，这中间牵涉到的内容就有：

* App ID
* Certificate
* Device ID 
* Provisioning Profile

<!-- more -->

### App ID

App ID 是我们应用的唯一标识符。

### Certificate

Certificate 就是我们通常讨论的证书，它包含信任实体的信息，例如Common Name, Country, Public info等等。它的作用就是让别人知道这个证书的拥有者是谁，他是否可信，它之所以能实现这些是基于数学上的非对称加密。

### Device ID 

Device ID 是物理设备的唯一标识符，我们通常称为 UDID。

### Provisioning Profile

Provisiong Profile 我们称之为授权配置文件，它包含了上面所有的东西。

开发过程的真机调试需要：
Private Key -- 私钥
iPhone Development Certificate -- 开发证书
Development Provisioning profile

发布到 App Store 需要：
私钥
iPhone Distribution Certificate
App Store Distribution Provisioning profile

通过 Ad Hoc 发布需要：
私钥
iPhone Distribution Certificate
Ad Hoc Distribution Provisioning profile

### Certificate, Provisioning Profile 的制作过程  

使用KeyChain申请 Certificate Signing Request (CSR)，这个过程就能生成代码签名的公、私钥对，私钥会保存在KeyChain中，公钥则包含在certSigningRequest中。 

Certificate制作具体步骤：

* Certificate Signing Request (CSR)  
KeyChain Access > Certificate Assitant > Request a Certificate From a Certificate      Authority...

Certificate Infomation

User Email Address:xxx(you favarite address)  
Common Name:xxx(you name)  
CA Email:(Keep empty)  
Request is: save to disk  

之后会弹出保存路径选择对话框，选择你想保存的目标路径。

* 制作Certificate  
developer.apple.com > Certificates, Identifiers & Profiles > Certificates > + > 选择需要的Certificate类型 > 上传之前创建的CSR > 得到Certificate

* 安装Certificate  
下载生成的Certificate > 保存好（如改个容易识别的名字，保存到安全的地方） > 双击它，安装到Key Chain.

* 导出Private Key  
从KeyChain中导出Private Key，团队其他成员可以省略制作Certificate的繁琐操作。

Provisioning Profile的制作要复杂些，它要包含App 相应的Certificate， App ID, Development Provision Profile 还会包含 Device 信息。  

### Tips

团队开发时，我们可以通过邮件等方式分发Private Key，这样只需要制作一次 Private Key, Certificate, Provisioning Profile。  

Xcode3.2.3预发布版本加入了新功能Team Provisioning Profile,它包含一个Wildcard App ID(*, 匹配所有应用程序)，Team中所有的Development Certificates和所有开发设备信息，增加新设备后，Xcode会自动更新Team Provisioning Profile, 因此， 团队成员可以通过设置Xcode的Provisioning Profile为Team Provisioning Profile，从而可以在所有的开发设备上调试应用程序。  
<div style="text-align: center" markdown="1"> 

	<img name="TeamProvisioningProfile" src="/images/TeamProvisioningProfile.png" width="712" height="406">  
</div>

## Reference
[iOS Code Signing: Under The Hood](https://www.raywenderlich.com/2915/ios-code-signing-under-the-hood) 
