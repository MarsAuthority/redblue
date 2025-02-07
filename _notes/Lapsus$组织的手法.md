up::[[Red Team基础设施]]-[[实践参考]]
- # 微软报告
	- 3月22日，微软MSTIC、DART、MS365 Defender威胁情报团队发布了《DEV-0537 criminal actor targeting organizations for data exfiltration and destruction》报告，详细分析了攻击者的攻击手法（TTPs），并给出了安全建议
	- 攻击者收集了目标的大量信息，包括：组织架构、help desk，应急响应流程、供应链关系等，主要是用于发送钓鱼邮件，以及重置目标用户的凭证。手段包括：
		- 部署Redline Stealer恶意软件，窃取密码和session token
		- 购买凭证和session token
		- 向攻击目标企业（或供应商和合作伙伴）的员工购买证书和通过MFA认证
		- 在公共代码库中搜索公开凭据
		- 然后使用泄露的凭证或session token访问面向互联网的系统和应用，包括：VPN、RDP、VDI、Okta。
	- 对于使用MFA的目标企业，攻击者用两种技术突破MFA机制：session token replay and using stolen passwords
		- session token replay 会话令牌重放可能是实现MFA功能时不规范，代码机制在时效、范围时有错误。
		- using stolen passwords 是指窃取密码后，再钓鱼诱导受害者触发点击MFA认证通过的提示（二次认证的确认机制时get请求、csrf风险、或者提示不明显）。
		- 还有一个姿势是窃取员工的个人邮箱，通过个人账户做第二因素身份验证或密码恢复。
	- 通过招募目标企业员工成功进入攻击目标（员工提供身份凭据并通过MFA的提示，或者安装anydesk或其他远程管理软件）
	- 攻击者进入目标后，会访问JIRA、Gitlab和Confluence（用这些软件的企业是不是菊花一紧^_^），并尝试使用JIRA、Gitlab、Confluence的漏洞进行提权，以及使用DC Sync、Mimikatz、Ntdsutil
	- 攻击者拥有专门的 [[Red Team基础设施]] ，而且知道企业的安全检测规则（不同地理位置登录账户的检测），会选择地理上与目标相似的VPN出口，然后从目标下载敏感数据
	- 攻击者也会尝试攻击目标的云租户特权账户（AWS、Azure），并有创建账户和删除数据的行为
	- 攻击者还会监听组织应急响应处置和内部讨论（Slack、Teams、电话会议），深入了解被攻击者的心理状态
- # twitter网友爆料Mandiant给okta做的应急报告
	- Lapsus$居然用的都是常用、默认的工具，没有黑魔法，可见安全基础能力的重要性，很多公司地基没打好就开始造高楼了
	- <img src="/assets/Pasted image 20221104151423.png">
	- <img src="/assets/Pasted image 20221104151431.png">
- # 思考
	- 国外安全公司的文化就是会公开透明对待安全事件，这点难能可贵，我们也可以从中学到很多，此次LAPSUS$事件，相关公司也做了公开披露，参考
		- Cloudflare
			- https://blog.cloudflare.com/cloudflare-investigation-of-the-january-2022-okta-compromise/
		- Okta
			- https://www.okta.com/blog/2022/03/oktas-investigation-of-the-january-2022-compromise/
	- 其他事件的申明包括：
		- [[AWS 的 S3 故障回顾和思考]]
		- [Gitlab误删数据](https://about.gitlab.com/blog/2017/02/01/gitlab-dot-com-database-incident/)
- # 来源
	- https://mp.weixin.qq.com/s/AG-ITyHlwesxS2k-5BzgKg
	- https://www.microsoft.com/security/blog/2022/03/22/dev-0537-criminal-actor-targeting-organizations-for-data-exfiltration-and-destruction/