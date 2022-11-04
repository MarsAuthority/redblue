up::[[行业观察 Industry Research]]-[[Google]]
- # 安全合规要求
	- 满足全球不同国家地区的法律、规章制度、标准。
	- <img src="/assets/Pasted image 20221104160643.png">
- # 安全架构模型
	- 基本上就是参考了NIST CSF和CIS Control，中规中矩。
		- <img src="/assets/Pasted image 20221104160648.png">
		- <img src="/assets/Pasted image 20221104160655.png">
- # Google Cloud 在安全上做了什么？
	- 当组织开始在云中构建 IT 基础架构时，会考虑以下几个基本安全组件来建立环境。
		- <img src="/assets/Pasted image 20221104160703.png">
		- <img src="/assets/Pasted image 20221104160711.png">
		- 具体如下：
			- 1. **安全治理 Governance**：定义了安全架构路线图，以及如何管理和遵循各种标准和过程以确保环境安全稳定；最好是遵循或参考NIST等业界标准化框架；同时也包含了风险管理（定义成本、了解风险、努力减轻这些风险）和安全人员培训。
			  2. **DevSecOps**：融入DevOps流程，理解要部署的应用/Workload，在漏洞产生前就消灭他们。
			  3. **IAM**：强大的 IAM 工具和流程将是您构建经过身份验证和授权的环境的第一道防线。
			  4. **网络安全**：在将数据中心传统迁移到云端的过程中，应用程序和工具通常可能对外暴露访问，需要使用包括隔离、VPC控制、云防火墙、DDoS防护等来进行防护。
			  5. **计算和基础设施安全**：提供包括云原生监控和日志服务等多个方面能力。默认情况下，它的各种底层组件（如 Monitoring、Trace、Logging、Error Reporting 和 Debug）提供了整个 GCP 基础架构的实时可见性。
			  6. **应用程序安全**：需要紧急预防或缓解 OWASP Top 10 等威胁，使用包括扫描器、WAF等手段。
			  7. **数据安全**：防止数据泄露是整个安全基础设施和流程的基石。大多数在云中构建的数据库现在都带有本机安全控制，具有授权视图和列级安全等功能，应用程序可以配置为利用这些控制进行授权访问。
			  8. **用户终端和连接安全性**：利用其庞大的内部网络将公司网络保持在内部网络中，避免不必要的暴露。
			-
- # 数据和事件驱动架构
	- 保护运行环境的推荐架构之一是密切关注数据和事件，并努力补救或提醒适当的团队。
	- <img src="/assets/Pasted image 20221104160718.png">
- # 安全组件概览
	- <img src="/assets/Pasted image 20221104160725.png">
- # 参考
	- https://info-on-security.medium.com/gcp-security-architecture-whitepaper-da19fc9b4ba2
	- https://www.youtube.com/watch?v=LZyhEEC7jcg&t=2316s