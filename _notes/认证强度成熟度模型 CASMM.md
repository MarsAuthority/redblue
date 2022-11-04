up::[[Blue Team能力建设]]-[[安全规范 Security regulations]]
- # 并非所有的 MFA 都是一样的，他们的差异在于
	- <img src="/assets/Pasted image 20221104155002.png">
	- 现在，越来越多的钓鱼攻击不仅提示输入用户名和密码，还提示输入 MFA 信息。这意味着传统的 MFA 在逐渐失效。这就提出了一个问题：是否有一种身份验证方法可以防止这种情况发生？换句话说，是否有一种多因素认证模式可以抵抗网络钓鱼？答案是肯定的。
	- FIDO 代表 Fast Identity Online，它使用通用身份验证框架 (UAF) 和通用第二因素 (U2F) 协议。该系统使用公钥加密和物理访问令牌。私钥存储在令牌上并随身携带，而公钥存储在您要对其进行身份验证的服务中。当进行身份验证时，向客户端/token证明你就是你（指纹、PIN、语音等），并且客户端会创建一个发送到服务的签名请求。使用公钥对该请求进行解密和身份验证，这证明该请求是使用私钥发出的，然后就可以通过身份验证。
- # 来源
	- https://danielmiessler.com/blog/casmm-consumer-authentication-security-maturity-model/