up::[[Blue Team能力建设]]-[[命令与控制 Command and Control]]
- # 阅读目录(Content)
	- DNS隧道简介
		- DNS隧道木马分类
			- IP直连型 DNS隧道木马
			- 域名型 DNS隧道木马 - DNS迭代查询中继隧道
			- IP直连型和域名解析型异同点
	- Powershell+dnscat2实现DNS隐蔽隧道反弹Shell
		- C&C域名绑定
		- 在C&C服务器上安装DNS C&C Server
		- 部署客户端dnscat2 client
		- 连接建立后，C&C控制端可以执行指令
			- 获取shell
			- 端口转发
		- wireshark抓包分析
			- UDP直连模式
			- 域名解析模型
	- 可用于DNS tunnel的检测思路 - 基于UDP DNS会话
		- DNS query type成分组成异常检测
			- DNS Tunnel
			- DNS FF Botnet
			- DNS Query types Numbers
		- 基于Zipf定律的异常检测 - Frequency Analysis
		- DNS query/answer文本特征
			- n-gram文本特征
			- 基于CNN深度神经网络，从文本角度判断单条DNS query是否存在可疑Tunnel特征
			- Query/Answer长度特征
		- 基于会话聚类维度的DNS tunnel行为特征
			- DNS会话时长
			- DNS会话中数据包总数
			- “上行大包”占请求报文总数的比例
			- “下行小包”占响应报总数的比例
			- 有效载荷的上传下载比
			- 有效载荷部分是否加密
			- 域名对应的主机名数量
			- FQDN数异常检测
			- 总的query 报文Payload载荷量
		- 响应时间相关特征
			- Response wait time特征
		- 信息熵
		- 发包频率行为
	- 可用于DNS tunnel的检测思路 - 基于DNS QUERY维度
		- Network Features - 网络访问行为方面的特征
			- 域名被访问频率角度特征
			- TTL (last seen - first seen)
			- window
			- dns name change frequency
		- Lexical Features
			- 域名本身词频特征层面特征
			- 域名的香侬熵
			- 字符分布特征 -可读性/易读性方面的体现
		- DNS session会话相关特征
			- Number of IP subnetworks
			- DNS对应的src_ip/dst_ip count相关特征
			- DNS query type组成成分相关特征
		- DNS域名相关meta信息
			- whois-based features
	- 异常数据清洗
		- 基线异常过滤思路
		- 无监督异常abnormal检测算法
- # 来源
	- https://www.cnblogs.com/LittleHann/p/8656621.html