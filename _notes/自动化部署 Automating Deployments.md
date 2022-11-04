up::[[Red Team基础设施]]
- # 1. 背景
	- 前面提到的架构和各种各样的Redirector，在实际部署使用的过程中会非常繁琐；从2014年开始已经有人提出了基础设施即代码（Infrastructure as Code），作为最佳实践，基础设施及代码授权所有准备计算资源所需要做的工作都可以通过代码来完成。 这类的平台比较多，比如Terraform，Chef，Puppet Ansible，CloudFormation，Salt等等。使用 Terraform 有很多吸引力，如：
		- 设计/测试基础设施一次，即可多次重复使用
		- 配置文件可以是代码控制的
		- 可以使用单个命令创建和销毁基础设施
		- 监控状态改变
- # 2. 基础设施概览
	- 下面是一个架构图，显示了构建的基础设施——它可以而且通常更加build-out，但原则保持不变——Redirector放置在每台服务器的前面，使基础设施更加隐蔽，能够用新的服务器快速更换消耗的服务器：
		- <img src="/assets/Pasted image 20221104150641.png">
	- 总共有6台服务器，3个长期服务（钓鱼、payload 和 c2），3 个redirector（smtp、payload 和 c2）。 我们假设这些服务器在实战中会Blue team检测到并需要销毁，这就是 Terraform 自动化部分的用武之地——因为我们的环境状态是在 Terraform 配置文件中定义的，我们几乎可以立即重建那些销毁的服务器，并且操作可以继续进行而不会出现更大的中断。
- # 3. 配置基础设施
	- **基础设施是通过利用以下服务构建的：**
		- DigitalOcean Droplets(CVM)
		- DigitalOcean DNS management，需要能够为smtp redirector设置 PTR DNS 记录，以减少被目标邮件网关归类为垃圾邮件的概率
		- CloudFlare DNS management，用于控制指向长期服务器的 DNS 记录
	- 可以使用 Amazon AWS 或其他流行的 VPS 来构建服务器，只要它支持 Terraform 即可，这同样适用于 DNS 管理部分。
	- 文件结构：
		- 由以下方式组织的 terraform 配置文件定义：
			- <img src="/assets/Pasted image 20221104150656.png">
		- Configs：包括payload redirector的配置（apache: .htaccess，apache2.conf），smtp redirector（postfix: header_checks - 用于去除原始 smtp 服务器的邮件标头，master.cf - TLS 和 opendkim 的通用后缀配置，opendkim.conf - 使用 postfix 配置 DKIM 集成）
		- providers：诸如 DigitalOcean 和 CloudFlare 之类的基础设施
		- variables：terraform的API key和其他相关数据
		- sshkeys：ssh登录凭证等信息
		- dns：DNS相关配置
		- firewalls：访问控制
		- outputs：基础设施的关键 IP 和域名信息
- # 4. 变量
	- Variables.tf 存储诸如 API tokens、redirectors和 c2 域名、防火墙规则之类的内容：
		```
		  # tokens, api keys
		  variable "do-token" {
		      default = "yourdigitaloceantoken" }
		  
		  variable "cf-token" {
		      default = "yourcloudflaretoken" }
		  
		  variable "cf-email" {
		      default = "xxx@xxx.com" }
		  
		  # operator IPs
		  variable "operator-ip" {
		      default = "aaa.bbb.ccc.ddd" }
		  
		  # ssh keys
		  variable "ssh-public-key" {
		      default = "/root/.ssh/id_rsa.pub" }
		  
		  # domains & subdomains
		  variable "domain-rdir" {
		      default = "example.com" }
		  
		  variable "domain-c2" {
		      default = "example.net" }
		  
		  variable "sub1" {
		      default = "static" }
		  
		  variable "sub2" {
		      default = "ads" }
		  
		  variable "sub3" {
		      default = "js" }
		  
		  variable "sub4" {
		      default = "css" }
		  
		  variable "sub5" {
		      default = "apple" }
		  
		  variable "sub6" {
		      default = "login" }
		      
		  variable "cspw" {
		      default = "somecrazystrongpasswordgoesherelel" }
		```
- # 5. C2
	- 下面是 C2 服务器的 remote-exec Terraform 配置器，它下载 CS zip，使用给定的 CS 密码解压缩它并创建一个 cron 以确保 C2 服务器在服务器启动后启动：
		```
		  provisioner "remote-exec" {
		      inline = [
		          "apt update",
		          "apt-get -y install zip default-jre",
		          "cd /opt; wget ${var.csDownloadUrl} -O cobaltstrike.zip",
		          "echo \"@reboot root cd /opt/cobaltstrike/; ./teamserver ${digitalocean_droplet.c2-http.ipv4_address} ${var.cspw}\" >> /etc/cron.d/mdadm",
		          "unzip -P ${var.cspw} cobaltstrike.zip && shutdown -r"
		      ]
		    }
		```
- # 6. C2 Redirector
	- 使用 socat 将端口 80 和 443 上的所有传入流量重定向到运行 Cobalt Strike 团队服务器的主要 HTTP C2 服务器：
		```
		  provisioner "remote-exec" {
		      inline = [
		          "apt update",
		          "apt-get -y install socat",
		          "echo \"@reboot root socat TCP4-LISTEN:80,fork TCP4:${digitalocean_droplet.c2-http.ipv4_address}:80\" >> /etc/cron.d/mdadm",
		          "echo \"@reboot root socat TCP4-LISTEN:443,fork TCP4:${digitalocean_droplet.c2-http.ipv4_address}:443\" >> /etc/cron.d/mdadm",
		          "shutdown -r"
		      ]
		    }
		```
	- 测试 C2 及其Redirector
		- <img src="/assets/Pasted image 20221104150727.png">
- # 7. 钓鱼
	- 使用GoPhish 框架，开放端口 3333，但只允许使用 DigitalOcean 防火墙的操作员访问：
		```
		  resource "digitalocean_firewall" "phishing" {
		      name = "phishing"
		      droplet_ids = ["${digitalocean_droplet.phishing.id}"]
		  
		      inbound_rule = [
		          {
		              protocol = "tcp"
		              port_range = "3333"
		              source_addresses = ["${var.operator-ip}"]
		          }
		      ]
		      outbound_rule = [
		          {
		              protocol = "tcp"
		              port_range = "25"
		              destination_addresses = ["${digitalocean_droplet.phishing-rdr.ipv4_address}"]
		          }
		      ]
		  }
		```
- # 8. 钓鱼Redirector
	- 这是最耗时的部分。 众所周知，设置 SMTP 服务器通常是一个非常痛苦的过程。 自动化部署的价值在于一旦它被发现并销毁，您将永远不需要从头开始重建 SMTP 服务器；设置包括：
		- SPF 记录
		- DKIM
		- 加密
		- postfix 配置为中继
		- 清理电子邮件标头以混淆原始电子邮件服务器
	- 测试钓鱼及其Redirector
		- 一旦基础设施建立起来，钓鱼相关的DNS 区域应该有 spf、dkim 和 dmarc 记录，类似于这里看到的那些：
		- <img src="/assets/Pasted image 20221104150744.png">
		- 完成 DNS 设置后，可以通过中继服务器从实际的钓鱼服务器向 gmail 发送快速测试电子邮件，看看 spf、dkim 和 dmarc 是否检查 PASS，我们可以在下面看到他们在我们的案例中所做的：
		```
		  telnet redteam.me 25
		  helo redteam.me
		  mail from: olasenor@redteam.me
		  rcpt to: mantvydo@gmail.com
		  data
		  to: Mantvydas Baranauskas <mantvydo@gmail.com>
		  from: Ola Senor <olasenor@redteam.me>
		  subject: daily report
		  
		  Hey Mantvydas,
		  As you were requesting last week - attaching as promised the documents needed to keep the project going forward.
		  .
		```
		- <img src="/assets/Pasted image 20221104150805.png">
- # 9. Payload Redirector
	- Payload Redirector服务器建立在 apache2 mod_rewrite 和proxy模块上。 mod_rewrite 模块允许我们编写细粒度的 URL 重写规则，并将受害者的 HTTP 请求代理到合适位置。
	- 下面是一个 .htaccess 文件，说明何时、何地以及如何（即代理或重定向）重写传入的 HTTP 请求：
		```
		  RewriteEngine On
		  RewriteCond %{HTTP_USER_AGENT} "android|blackberry|googlebot-mobile|iemobile|ipad|iphone|ipod|opera mobile|palmos|webos" [NC]
		  RewriteRule ^.*$ http://payloadURLForMobiles/login [P]
		  RewriteRule ^.*$ http://payloadURLForOtherClients/%{REQUEST_URI} [P]
		```
	- 下面的截图应该说明了上述概念：
	- **绿色部分** - 我们使用了 curl（及其默认 UA），根据 .htaccess 文件，它应该将我们重定向到 payloadURLForOtherClients - 但因为它是一个测试主机且不可解析，所以失败了。
	- **粉红色部分** - 这次使用伪造的 UA，将 http 请求伪装成来自 iphone 的请求 - 我们可以看到 apache 正确地尝试将请求代理到 payloadURLForMobiles 主机。
	- <img src="/assets/Pasted image 20221104150826.png">
- # 10. Outputs
	- Outputs.tf 包含了DNS 及其 IP 地址，也包括smtp相关的DKIM DNS TXT等信息：
		- <img src="/assets/Pasted image 20221104150839.png">
		- <img src="/assets/Pasted image 20221104150901.png">
- # 来源
	- https://www.ired.team/offensive-security/red-team-infrastructure/automating-red-team-infrastructure-with-terraform#configuring-infrastructure