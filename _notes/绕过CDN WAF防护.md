up::[[Red Team能力建设]]-[[初始访问 Initial Access]]-[[Bypass]]
- # 找到源主机IP
	- 记的很久之前在乌云上看到篇文章，通过使用 zmap 对全 CN 的 IP 的 80、443 端口进行扫描，通过banner对比得到真实IP。
	- 上面的方案是有一定的缺陷的：
		- 1. 无法对响应做匹配，且字符串匹配可能不准确
		- 2. 源站只允许CDN访问，无法直接访问怎么办？
- ## 问题一：使用工具Hakoriginfinder
	- Hakoriginfinder 是一个 golang 工具，它会向每个 IP 地址发出 HTTP 和 HTTPS 请求，尝试通过将响应与真实网站的响应进行比较来找到源主机，并且使用 Levenshtein 算法找到相似的响应。
	- 速度肯定不如zmap那么快。
	- https ://github.com/hakluke/hakoriginfinder
- ## 问题二：使用CDN基础设施来绕过限制
	- 以AWS环境为例
		- CloudFlare 和 CloudFront 等CDN服务，会使用共享的 IP 范围来对源站进行访问
		- 使用CloudFront时，我们可以在HTTP请求添加任意header
	- 这些条件可以绕过问题二提到的限制，方法是通过控制的一个路由，这个未经过滤的请求可以绕过源头的任何 IP 限制，因为它来自与合法请求相同的 IP 范围，如下图：
		- <img src="/assets/Pasted image 20221104160205.png">
	- 这里的核心涉及AWS CDN使用的HTTP header字段
		- **Cdn-Proxy-Origin**, 此字段是必填的，需要设置为通过 CloudFront 转发后，路由要到达目标的主机名或IP
		- **Cdn-Proxy-Host**, 此字段是可选的，内容为通过 CloudFront 转发后，Host 字段的值，如果未设置，它将默认为 Cdn-Proxy-Origin 的值
		- **X-Forwarded-For**, 此字段是可选的，内容为通过 CloudFront 转发后，HTTP header中的xff字段，如果未设置，则为一个随机IP
	- 比如，下面的请求通过 CloudFront 后转发到公共 ifconfig.me 服务，返回的 IP 将是我们从 CloudFront 网络发出的请求的源 IP
		```
		  curl -H 'Cdn-Proxy-Origin: ifconfig.me' -H 'Cdn-Proxy-Host: ifconfig.me' XXXXXXXXXXXXX.cloudfront.net
		```
	- 参考
		- https://blog.ryanjarv.sh/2022/03/16/bypassing-wafs-with-alternate-domain-routing.html