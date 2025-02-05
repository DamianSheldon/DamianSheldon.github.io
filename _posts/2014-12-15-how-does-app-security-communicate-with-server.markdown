---
layout: post
title: "在 iOS App 中使用自签名证书"
date: 2014-12-15 11:53:11 +0800
comments: true
categories: [Archives, iOS Development]
keywords: Self-signed, Certificate, iOS, TLS/SSL, HTTPS 
description: App如何与Server安全交互
---

大多数App都需和Server通信来提供服务，这中间就牵涉到网络通信安全。网络通信安全是一个很大的话题，本文不打算全面覆盖，而是来理理HTTPS。

移动设备可能会处于不安全的网络环境中，比如连接了某个公共热点，攻击者不需要访问设备，只需访问设备所在的网络，就能获取到用户信息，所以，当应用中用户的信息需要保护时，开发者需要保证通信的安全性。

最简单直接的解决办法是采用HTTPS,在web服务器上安装一个自签名证书，启用HTTPS,然后对NSURLSession进行配置以接受该自签名证书。

HTTPS是如何做到通信安全的呢？答案是TLS/SSL协议。TLS(Transport Layer Security)/SSL(Secure Socket  Layer)协议是专门为解决网络通信安全设计的。它的基石是非对称加密。

TLS/SSL链路中的数据是加密的，客户端给服务器发送的数据是用服务器的公钥加密的，由于非对称加密的数学特性，只有拥有私钥的服务器才能正确解密数据。服务器给客户端发送的数据则是用自己的私钥加密的，客户端用公钥解密。

那么我们如何判断服务器发给我们的公钥是值得信任的呢？通常商业网站的数字证书都是由中级证书或根证书来签名，而根证书是一开始就内置在设备中，不是通过网络交换的，这样当某个服务器声明说我是某某，我们可以通过证书链来判断真伪。


根证书其实是一个自签名证书，我们的应用也可以用自签名证书来确保网络通信安全，还可以省掉很大一笔证书费用，只要私钥足够安全，它甚至比商业证书更安全。

##创建自签名证书

为了方便创建自签名证书来测试 TLS, Apple 为我们提供一个工具 Certificate Assitant，它内置在 OS X 中，我们可以通过 KeyChain 打开它；我们也可以使用 openssl。新手的话还是建议使用 Certificate Assitant. 详细步骤参考[Creating Certificates for TLS Testing](https://developer.apple.com/library/ios/technotes/tn2326/_index.html#//apple_ref/doc/uid/DTS40014136-CH1-SECISSUE_C).

## 为服务端配置证书

我使用的是 Apache，配置如下:

```
#/etc/apache2/httpd.conf
<VirtualHost *:443>
#ServerName www.example.com
SSLEngine on
SSLCertificateFile "/etc/apache2/server.crt"
SSLCertificateKeyFile "/etc/apache2/server.key"
</VirtualHost>
```

由于 Apache 是使用 PEM 格式的证书和私钥，所以我们需要格式转换下：

```
#Extracting a digital identity for use with Apache

$ # First extract the server certificate.
$
$ openssl pkcs12 -in "Deep Thought.p12" -nokeys -out server.crt
Enter Import Password: ****
MAC verified OK
$
$ # Next extract the server private key.
$
$ openssl pkcs12 -in "Deep Thought.p12" -nocerts -nodes -out server.key
Enter Import Password: ****
MAC verified OK
```

重启 Apache, 我们可以使用 openssl 的 s_client 子命令来测试下。

```
// Failed
$ openssl s_client -connect myserver.com:443

// Success
$ openssl s_client -connect myserver.com:443 -CAfile ./MyCACertificate.pem
```

##接受自签名证书

### URLSession

我们需要介入到 TLS 的授权过程，基本做法是判断是与我们指定的服务器通信需要授权，然后把自签名证书加入锚中。代码如下：

```
// Authentication Challenges and TLS Chain Validation
func urlSession(_ session: URLSession, task: URLSessionTask, didReceive challenge: URLAuthenticationChallenge, completionHandler: @escaping (URLSession.AuthChallengeDisposition, URLCredential?) -> Void) {

    print("authentication method \(challenge.protectionSpace.authenticationMethod)\n host: \(challenge.protectionSpace.host)")

        if challenge.protectionSpace.authenticationMethod == NSURLAuthenticationMethodServerTrust && challenge.protectionSpace.host == "dongmeiliangsmacbook-pro.local" {

            // Custom evaluating a trust object
            let serverTrust = challenge.protectionSpace.serverTrust!
                let policy = SecPolicyCreateSSL(true, "dongmeiliangsmacbook-pro.local" as CFString)

                SecTrustSetPolicies(serverTrust, [policy] as CFArray)

                let path = Bundle.init(for: ViewController.self).path(forResource: "ServerCertificates", ofType: "cer")

                do {
                    let certData = try NSData(contentsOfFile: path!, options: NSData.ReadingOptions(rawValue: 0))
                        if let certificate = SecCertificateCreateWithData(nil, certData as CFData) {
                            SecTrustSetAnchorCertificates(serverTrust, [certificate] as CFArray)

                                var allowConnection = false

                                var trustResult: SecTrustResultType = .invalid

                                let err = SecTrustEvaluate(serverTrust, &trustResult)

                                if err == noErr {
                                    allowConnection = (trustResult == .unspecified) || (trustResult == .proceed)
                                }

                            print("err:\(err)\nallowConnection:\(allowConnection)")

                                    if
                                        allowConnection
                                        {
                                            completionHandler(.useCredential, URLCredential(trust:serverTrust))
                                        }
                                    else
                                    {
                                        completionHandler(.cancelAuthenticationChallenge, nil)
                                    }

                        }
                        else
                        {
                            print("certificate create with data failed")
                            completionHandler(.cancelAuthenticationChallenge, nil)
                        }

                }
            catch
            {
                print("read certificate data failed:\(error)")
                completionHandler(.cancelAuthenticationChallenge, nil)
            }

        }
        else
        {
            completionHandler(.performDefaultHandling, nil)
        }
}
```

### AFNetworking

AFNetworking 只需要我们配置下 securityPolicy，代码如下：

```
lazy var apiClient: AFHTTPSessionManager = {
    let client = AFHTTPSessionManager(baseURL: URL(string: "https://dongmeiliangsmacbook-pro.local/"))
        let selfSignedCertificates = AFSecurityPolicy.certificates(in: Bundle.init(for: ViewController.self))

        client.securityPolicy = AFSecurityPolicy(pinningMode: .certificate, withPinnedCertificates: selfSignedCertificates)
        client.securityPolicy.allowInvalidCertificates = true

        return client
}()
```

###注意点

客户端是把服务端的证书加入锚中。

### 完整示例

[SelfSignedCertificate](https://github.com/DamianSheldon/SelfSignedCertificate)


### Reference
[AFNetworking SSL Pinning With Self-Signed Certificates](http://initwithfunk.com/blog/2014/03/12/afnetworking-ssl-pinning-with-self-signed-certificates/)  
[Creating Certificates for TLS Testing](https://developer.apple.com/library/ios/technotes/tn2326/_index.html#//apple_ref/doc/uid/DTS40014136-CH1-SECISSUE_C)  
[HTTPS Server Trust Evaluation](https://developer.apple.com/library/content/technotes/tn2232/_index.html#//apple_ref/doc/uid/DTS40012884-CH1-SECSTRICTER)  
URL Session Programming Guide
