up::[[Red Team基础设施]]
- # 1. SMTP
	- 配置SMTP是为了执行如下两个操作：
	- **Sendmail**
		- 删掉之前的server headers，在/etc/mail/sendmail.mc结尾增加：
			- ```define(`confRECEIVED_HEADER',`by $j ($v/$Z)$?r with $r$. id $i; $b')dnl```
		- 在/etc/mail/access结尾增加：
			```
			IP-to-Team-Server *TAB* RELAY
			Phish-Domain *TAB* RELAY
			```
		- 参考：
			- [Removing Sender’s IP Address From Email’s Received From Header](https://www.devside.net/wamp-server/removing-senders-ip-address-from-emails-received-from-header)
			- [Removing Headers from Postfix setup](https://major.io/2013/04/14/remove-sensitive-information-from-email-headers-with-postfix/)
		- **catch-all address：**把任何发送给*@phishdomain.com的邮件转发到到指定的邮件地址。
			- ```echo PHISH-DOMAIN >> /etc/mail/local-host-names```
		- 在/etc/mail/sendmail.mc的//Mailer Definitions//（结尾处）之前添加以下行：
			- ```FEATURE(`virtusertable', `hash -o /etc/mail/virtusertable.db')dnl```
		- 在/etc/mail/virtusertable结尾增加：
			- ```@phishdomain.com  external-relay-address```
	- **Postfix**
		- Postfix是替换Sendmail的一个更好的选择，还提供了更好的兼容性；Postfix还提供了全面的IMAP支持，使得测试人员能够实时地与对原始消息做出响应，，而不必依靠catch-all address。
		- 配置参考：[Mail Servers Made Easy](https://blog.inspired-sec.com/archive/2017/02/14/Mail-Server-Setup.html)
- # 2. DNS
	- <img src="/assets/Pasted image 20221104150341.png">
	- **Socat**
		- socat可将53端口上的DNS数据包重定向到我们的cs team server
			- ```socat udp4-recvfrom:53,reuseaddr,fork udp4-sendto:<IPADDRESS>; echo -ne```
		- 参考：[Redirecting Cobalt Strike DNS Beacons - Steve Borosh](https://medium.com/rvrsh3ll/redirecting-cobalt-strike-dns-beacons-e3dcdb5a8b9b)
	- **iptables**
		- iptables转发DNS规则可以和我们的cs team server配合使用
			```
		  iptables -I INPUT -p udp -m udp --dport 53 -j ACCEPT
		  iptables -t nat -A PREROUTING -p udp --dport 53 -j DNAT \
		  --to-destination <IP-GOES-HERE>:53
		  iptables -t nat -A POSTROUTING -j MASQUERADE
		  iptables -I FORWARD -j ACCEPT
		  iptables -P FORWARD ACCEPT
		  sysctl net.ipv4.ip_forward=1
			```
	- **DNS重定向也可以在NAT后面完成**
		- 有时候可能需要在没有公网IP的内部网络上托管C2服务器，可以使用iptables、socat和反向SSH隧道的组合来实现。
		- <img src="/assets/Pasted image 20221104150435.png">
- # 3. HTTP(S)
	- 实践可以参考 [[CIA 如何实现C&C基础设施]]
	- **socat**
		- socat可用于将指定端口上的任何 TCP 数据包重定向到你的服务器。将localhost上的 TCP 端口 80 重定向到另一台主机上的端口 80 的基本语法是：
			- ```socat TCP4-LISTEN:80,fork TCP4:<REMOTE-HOST-IP-ADDRESS>:80```
		- 如果要指定网络接口，可以使用
			- ```socat TCP4-LISTEN:80,bind=10.0.0.2,fork TCP4:1.2.3.4:80```
	- **iptables**
		- 除了socat, iptables可以通过 NAT 执行dumb pipe重定向。 要将转向器的本地端口80转发到远程主机，配置命令如下：
			```
			  iptables -I INPUT -p tcp -m tcp --dport 80 -j ACCEPT
			  iptables -t nat -A PREROUTING -p tcp --dport 80 -j DNAT \
			  --to-destination <REMOTE-HOST-IP-ADDRESS>:80
			  iptables -t nat -A POSTROUTING -j MASQUERADE
			  iptables -I FORWARD -j ACCEPT
			  iptables -P FORWARD ACCEPT
			  sysctl net.ipv4.ip_forward=1
			```
	- **SSH**
		- 首先，必须设置GatewayPorts转发，否则将无法正常使用：
			```
			  # Allow the SSH client to specify which hosts may connect
			  GatewayPorts yes
			  
			  # Allow both local and remote port forwards
			  AllowTcpForwarding yes
			```
		- 要将redirector的本地端口 80 转发到你的内部服务器，命令如下：
			```
			  tmux new -S redir80
			  ssh <redirector> -R *:80:localhost:80
			  Ctrl+B, D
			```
		- 还可以转发多个端口，例如，同时打开 443 和 80：
		  id:: 624ed011-fe17-47ab-a287-cbab172044be
			```
			  tmux new -S redir80443
			  ssh <redirector> -R *:80:localhost:80 -R *:443:localhost:443
			  Ctrl+B, D
			```
- # 4. Payloads 与 Web
	- 当Payloads(服务)和网络资源开启时，我们希望降低被检测到的风险，同时希望无论是建立C2还是收集信息payload都能被高效的执行。
	- <img src="/assets/Pasted image 20221104150545.png">
	- 下面是 Jeff Dimmock 关于Apache的mod_rewrite的用法和示例：
		- Strengthen Your Phishing with Apache mod_rewrite https://bluescreenofjeff.com/2016-03-22-strengthen-your-phishing-with-apache-mod_rewrite-and-mobile-user-redirection/
		- Invalid URI Redirection with Apache mod_rewrite https://bluescreenofjeff.com/2016-03-29-invalid-uri-redirection-with-apache-mod_rewrite/
		- Operating System Based Redirection with Apache mod_rewrite https://bluescreenofjeff.com/2016-04-05-operating-system-based-redirection-with-apache-mod_rewrite/
		- Combatting Incident Responders with Apache mod_rewrite https://bluescreenofjeff.com/2016-04-12-combatting-incident-responders-with-apache-mod_rewrite/
		- Expire Phishing Links with Apache RewriteMap https://bluescreenofjeff.com/2016-04-19-expire-phishing-links-with-apache-rewritemap/
		- Apache mod_rewrite Grab Bag https://bluescreenofjeff.com/2016-12-23-apache_mod_rewrite_grab_bag/
		- Serving Random Payloads with Apache mod_rewrite https://bluescreenofjeff.com/2017-06-13-serving-random-payloads-with-apache-mod_rewrite/
	- 其他Apache的mod_rewrite的用法和示例：
		- mod_rewrite rule to evade vendor sandboxes from Jason Lang @curi0usjack https://gist.github.com/curi0usJack/971385e8334e189d93a6cb4671238b10
		- Serving random payloads with NGINX - Gist by jivoi https://gist.github.com/jivoi/a33ace2e25515a31aa2ffbae246d98c9
	- 要在redirector上自动设置Apache的mod_rewrite，请看[Mod_Rewrite Automatic Setup](https://blog.inspired-sec.com/archive/2017/04/17/Mod-Rewrite-Automatic-Setup.html) and [the accompanying tool](https://github.com/n0pe-sled/Apache2-Mod-Rewrite-Setup)
- # 5. C2重定向
	- 重定向 C2 流量有两个目的：1、隐藏后端真实服务器；2、躲避相关的调查者，让他们以为这是个合法网站。 通过使用Apache的mod_rewrite和自定义C2配置文件或其他代理（比如使用Flask），我们可以高效的过滤出来自调查 C2 的流量。
		- Cobalt Strike HTTP C2 Redirectors with Apache mod_rewrite - Jeff Dimmock https://bluescreenofjeff.com/2016-06-28-cobalt-strike-http-c2-redirectors-with-apache-mod_rewrite/
		- Securing your Empire C2 with Apache mod_rewrite - Gabriel Mathenge (@_theVIVI) https://thevivi.net/2017/11/03/securing-your-empire-c2-with-apache-mod_rewrite/
		- Expand Your Horizon Red Team – Modern SAAS C2 - Alex Rymdeko-Harvey (@killswitch-gui) https://cybersyndicates.com/2017/04/expand-your-horizon-red-team/
		- Hybrid Cobalt Strike Redirectors  https://zachgrace.com/2018/02/20/cobalt_strike_redirectors.html
	- 基于上述 “C2重定向”，另一种方法是可以在重定向服务器使用Apache的SSL代理引擎来接受入站SSL请求，并将这些请求代理到HTTPS listener上。 整个阶段使用SSL加密，可以根据需要在redirector上转发SSL证书。
	- 假如你已经使用了LetsEncrypt（aka CertBot），为了使mod_rewrite规则能够工作，需要在“/etc/apache2/sites-available/000-default-le-ssl.conf” 安装配置证书。 另外，要启用SSL ProxyPass引擎，相关配置如下：
		```
		  # Enable the Proxy Engine
		  SSLProxyEngine On
		  
		  # Tell the Proxy Engine where to forward your requests
		  ProxyPass / https://DESTINATION_C2_URL:443/
		  ProxyPassReverse / https://DESTINATION_C2_URL:443/
		  
		  # Disable Cert checking, useful if you're using a self-signed cert
		  SSLProxyCheckPeerCN off
		  SSLProxyCheckPeerName off
		  SSLProxyCheckPeerExpire off
		```
- # 来源
	- https://github.com/bluscreenofjeff/Red-Team-Infrastructure-Wiki