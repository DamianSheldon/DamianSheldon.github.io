---
layout: post
title: "iOS Development--Certificates, Provisioning Profiles"
date: 2014-10-09 16:24:09 +0800
comments: true
categories: [Archives, iOS Development]
keywords: Private Key, Certificate Signing Request, CSR, Certificates, Provisioning Profiles
discription: Introduce Certificates and Provisioning Profiles
---
iOS App开发过程的真机调试和开发完成的发布要用合法的 Signing Identity 进行签名，并且要制作相应的Provising Profile。  

<img name="LaunchApp" src="/images/LaunchApp.png" width="697" height="573">  


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



### Private Key, Certificate, Provisioning Profile 的作用  

Private Key --私钥， 在iOS App 开发过程中，Xcode用它来签署应用。    

Certificate --证书，它包含公钥，用来认证已签名的程序，通过认证来确定应用的来源是可信任的，并且代码是完整的， 未经修改的。  

<img name="Certificate" src="/images/Certificate.png" width="696" height="471"> 

Provisioning Profile --供应配置文件，它包含证书， App ID, 设备信息，它决定Xcode用哪个证书/私钥组合来签署程序, 开发设备也通过它来决定如何认证安装在设备上的程序。  

<img name="ProvisioningProfile" src="/images/ProvisioningProfile.png" width="618" height="377">  


### Private Key, Certificate, Provisioning Profile 的制作过程  

使用KeyChain申请 Certificate Signing Request (CSR)，这个过程就能生成代码签名的公、私钥对，私钥会保存在KeyChain中，公钥则包含在Certificate中。  

Provisioning Profile的制作要复杂些，它要包含App 相应的Certificate， App ID, Development Provision Profile 还会包含 Device 信息。  

### Tips

团队开发时，我们可以通过邮件等方式分发Private Key，这样只需要制作一次 Private Key, Certificate, Provisioning Profile。  

Xcode3.2.3预发布版本加入了新功能Team Provisioning Profile,它包含一个Wildcard App ID(*, 匹配所有应用程序)，Team中所有的Development Certificates和所有开发设备信息，增加新设备后，Xcode会自动更新Team Provisioning Profile, 因此， 团队成员可以通过设置Xcode的Provisioning Profile为Team Provisioning Profile，从而可以在所有的开发设备上调试应用程序。  

<img name="TeamProvisioningProfile" src="/images/TeamProvisioningProfile.png" width="712" height="406">  
