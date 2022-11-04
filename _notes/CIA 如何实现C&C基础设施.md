up::[[Red Team基础设施]]-[[实践参考]]
- # 1. 背景
	- Wikileaks 泄漏的资料 “Vault 7: CIA Hacking Tools Revealed” 有提到CIA是怎么做的。
- # 2. 整体架构
	- <img src="/assets/Pasted image 20221104151004.png">
	- 分为五个部分：
		- Implanted Host: 被控制的机器
		- VPS Server:  随时会销毁替换，隐藏真实C2
		- Proxy / VPN Server: 代理服务器，转发C2通信流量到C2服务器
		- Backend Server: 后台服务器
		- OSN: 存储日志的安全区
	- 其中：
		- Implanted Host到VPS Server 使用VPN连接（通过TLS加密）
		- VPS Server、Proxy / VPN Server、Backend Server和OSN之间的通信全部使用mTLS加密
		- 所有的VPS Server都是对公网提供服务的服务器（通常托管在空壳公司ISP）
		- Proxy / VPN Server 代理服务器负责过滤掉“无效”流量（如果mTLS认证失败则返回虚假网站 Cover Server）
	- VPN可能使用的是OpenVPN，来自：
		- <img src="/assets/Pasted image 20221104151017.png">
	- 架构中的Proxy / VPN Server 代理服务器为Nginx，在系统中被称为“Switchblade”
- # 3. VPS Server
	- VPS Server是红队基础设施的第一道防线，即用即抛和安全可靠是最重要的功能，它的iptables配置文件如下，也可参考 [[重定向器 Redirectors]]
		```
		  iptables -P INPUT DROP
		  iptables -P FORWARD DROP
		  iptables -p OUTPUT DROP
		  iptables -A INPUT -p tcp --dport 22 -j ACCEPT
		  iptables -A OUTPUT -p tcp --sport 22 -j ACCEPT
		  DNAT
		  iptables -t nat -A PREROUTING -i eth0 -p tcp --sport 1024:65535 -d 10.3.2.174 --dport 53 -j DNAT --to-destination 172.16.63.101:443
		  iptables -t nat -A PREROUTING -i eth0 -p tcp --sport 1024:65535 -d 10.3.2.174 --dport 80 -j DNAT --to-destination 172.16.63.101:443
		  iptables -t nat -A PREROUTING -i eth0 -p tcp --sport 1024:65535 -d 10.3.2.174 --dport 443 -j DNAT --to-destination 172.16.63.101:443
		  FORWARDING
		  iptables -A FORWARD -i eth0 -o p3p2 -m state --state ESTABLISHED,RELATED -j ACCEPT
		  iptables -A FORWARD -i p3p2 -o eth0 -m state --state ESTABLISHED,RELATED -j ACCEPT
		  iptables -A FORWARD -i eth0 -o p3p2 -p tcp --sport 1024:65535 -d 172.16.63.101 --dport 53 -m state --state NEW -j ACCEPT
		  iptables -A FORWARD -i eth0 -o p3p2 -p tcp --sport 1024:65535 -d 172.16.63.101 --dport 80 -m state --state NEW -j ACCEPT
		  iptables -A FORWARD -i eth0 -o p3p2 -p tcp --sport 1024:65535 -d 172.16.63.101 --dport 443 -m state --state NEW -j ACCEPT
		  SNAT
		  iptables -t nat -A POSTROUTING -o p3p2 -j MASQUERADE
		```
- # 4. Switchblade
	- Switchbalde 是与 Hive 和 Madison 等其他代理服务一起使用的身份验证代理服务器。Switchblade 使用自签名公钥证书与 Nginx 和 Linux Iptables相结合，将经过身份验证的数据传递给C2，未经身份验证的数据到虚假网站 Cover Server。
	- 架构图如下：
		- <img src="/assets/Pasted image 20221104151033.png">
		- 根据Wikileaks的另一份泄漏信息 [AmazonAtlas](https://wikileaks.org/amazon-atlas/) “Amazon是美国情报界领先的云提供商。2013 年，Amazon与CIA签订了一份价值 6 亿美元的合同，以构建一个云计算，供情报机构使用，以处理被列为绝密的信息”，其中右下角的“Protected Operational Network 受保护的运营网络”可能是AWS提供的专用网络区域。
		- Switchbalde的Nginx配置文件如下：
			```
			  # HTTPS server
			  server {
			          listen          172.16.63.113:443 ssl; 
			          server_name     nginx.edb.devlan.net;
			          ssl_certificate                 /etc/nginx/certs/domainA/server.crt;
			          ssl_certificate_key             /etc/nginx/certs/domainA/server.key;
			          ssl_client_certificate          /etc/nginx/certs/domainA/ca.crt;
			          ssl_verify_client               optional;
			          ssl_verify_depth                2;
			          ssl_protocols                   TLSv1 TLSv1.1 TLSv1.2;
			          ssl_prefer_server_ciphers       on;
			                                                                                   
			          proxy_set_header        Host    $host:$proxy_port;                       
			          location / {                                                              
			                  if ($ssl_client_verify = SUCCESS) {                              
			                          proxy_pass   http://172.16.64.100:4098;                  
			                  }
			                  proxy_pass      http://172.16.64.12:44302;
			          }
			  }
			```
- # 5. 流量伪装（Resignaturing）
	- CIA找了AFD对hive 2.5做了一次评测，以验证其通信流量是否会被外部检测出来，包括ICMP、TCP和UDP等协议，CIA也对这些协议做了一些修改（DNS和TFTP明文信息很多，所以暂未修改）
	- **ICMP**
		- <img src="/assets/Pasted image 20221104151049.png">
	- AFD发现前6字节是一个明显的检测特征；通过第1个字节的key对后面的IP进行XOR混淆，CIA对其进行了修改：XOR key为随机值（后续会变成更长的key）
	- **TCP 和 UDP**
		- AFD并未发现明显的特征，但发现
			- 存在固定的9个数据包序列，长度为：74-74-66-70-66-466-66-66-54，其中466为通信payload，其余为正常协议的长度。
			- AFD还指出，这些数据包不符合各自的协议规范，可能会引起注意。
		- CIA对数据包进行了如下修改：
			- 数据包长度是随机的
			- TCP和UDP使用相同的编码策略，见下图：
				- <img src="/assets/Pasted image 20221104151056.png">
- # 6. 自毁机制 self delete
	- 用于确保任何处于休眠状态的 implanted 机器（未成功外连C2），在预定的时间内能有效地自毁，唯一残余的东西是是“配置文件” (.config) 和 /var 目录中留下的日志文件 (.log)。
	- 在正常操作期间，.log 文件是空的，其最后修改时间表示最后一次联系的时间，当 self-delete 执行时，Hive 二进制文件将从主机中删除，并使用格式 yymmddHHMMSS 的时间戳创建日志文件。 （插入文件的时间戳应该与文件的最后修改时间一致）
	- 自毁的算法伪代码如下：
		```
		  last_time = 0 [This is the initial condition.] Set last_time to current time.
		  If current time > (last_time + CHECK_INTERVAL) AND (current time - last_time) ≤ MAX_TIME_DIFF (1 minute), then the system time is OK. If the time anomaly counter > 0, decrement it by 1, otherwise, check the file configuration time Set last_time to current time.
		  If current time > (last_time + CHECK_INTERVAL) AND (current time - last_time) > MAX_TIME_DIFF, then the system time changed. Increment time anomaly counter. Set last_time to current time.
		  If the current time < last_time, then the system time changed. Increment time anomaly counter. Set last_time to current time.
		  If the time anomaly counter > TIME_ANOMALY_LIMIT, then self delete.
		```
- # 参考
	- https://byt3bl33d3r.substack.com/p/taking-the-pain-out-of-c2-infrastructure-3c4?s=r
	- https://wikileaks.org/vault7/document/hive-Operating_Environment/hive-Operating_Environment.pdf
	- https://wikileaks.org/vault7/document/hive-Infrastructure-Switchblade/hive-Infrastructure-Switchblade.pdf
	- https://wikileaks.org/vault7/document/hive-DevelopersGuide/hive-DevelopersGuide.pdf