up::[[Blue Team能力建设]]-[[检测工程方法论 Detection Engineering Methodology]]
- # 背景
	- 有效的威胁检测是确保安全营运正常运作的关键要求之一，要先能“看见”，才能做后续的响应和处置；对于安全运营同学来说，在制定检测策略和后期运营时会遇到几个问题：
		- 缺乏规范的文档和标准，整体质量难以保证，导致低质量的检测策略部署到生产系统，造成产生过多/无效的告警（False Positive）会造成安全团队的疲惫感和不信任感，而更加危险的是检测策略未覆盖的部分（False Negative），我们无法应对“未知”的风险。
		- 检测规则产生的安全告警缺少有效的上下文信息，无法帮助值班的应急人员；他们可能不熟悉漏洞/应用的相关信息、无法判断是否为误报、如何处置的真实告警等。
		- 没有版本控制和同行评审，告警的优先级可能不一致或不明确。
	- Palantir的安全团队分享了内部是如何构建威胁检测策略的，提出了自己的威胁检测框架，试图从文档模板、流程和约定规则方面来提高检测策略的质量和效率。
- # 威胁检测框架—Alerting and Detection Strategy Framework（ADS）
	- 该框架帮助我们构建检测规则的生成、测试和管理，包含以下部分：
		- 目标：给出了应该检测的行为类型的简短明文描述。
		- 分类：提供了到MITRE ATT&CK相关条目的映射。
		- 策略摘要：描述了检测策略正在寻找什么、使用了哪些技术和数据源、相关上下文以及减少误报的优化措施等。
		- 技术背景：提供了解检测策略和告警所需的详细信息和背景；目标是为运营人员提供一个独立的参考，以对任何潜在的安全告警做出判断。
		- 盲点和假设：没有检测策略是绝对完美的，识别假设和盲点可以帮助其他工程师了解检测策略可能被绕过的具体情况。
		- 误报：由于配置错误、环境特性或其他非恶意场景而导致检测策略误报的已知实例。
		- 验证：列出了生成触发此告警所需的步骤；类似单元测试，可以是用于生成告警的POC/脚本等。
		- 优先级：描述了各种安全告警级别和相关标准。
		- 响应：触发此告警时的一般响应步骤，这些步骤指导运营人员对告警进行分类和调查。
		- 其他资源：可能有助于理解的任何其他内部、外部或技术参考资料。
	- 在部署新的检测策略之前，必须完成上述的每个部分。这保证了任何安全告警都有足够的文档，并经过验证。所有文档都会受版本控制和集中管控；在检测策略后续的优化迭代过程中，需要不断修改变更对应的文档，确保文档保持最新状态。
- # 实际案例
	- ## 策略名称：Unusual-Powershell-Host-Process
		- **目标**
			- 检测“没有 PowerShell 的 PowerShell”攻击，这些活动涉及不直接调用 powershell.exe 执行的PowerShell 命令和脚本。
		- **分类**
			- [Execution / Powershell](https://attack.mitre.org/wiki/Technique/T1086)
		- **策略摘要**
			- 该策略的作用如下：
				- 通过Windows系统的终端agent监控模块加载（module loads）行为
				- 监控任何加载 powershell DLL 的进程（system.management.automation.dll 或system.management.automation.ni.dll）
				- 排除已知路径/进程名的正常powershell主机进程（host processes）
				- 告警任何异常的powershell主机进程（host processes）
			- SIGMA规则示例如下，具体受篇幅限制就省略了 :)
				```
				  title: Detection of PowerShell Execution via DLL
				  logsource:
				      category: process_creation
				      product: windows
				  detection:
				      selection1:
				          Image|endswith:
				              - '\\rundll32.exe'
				      selection2:
				          Description|contains:
				              - 'Windows-Hostprozess (Rundll32)'
				      selection3:
				          CommandLine|contains:
				              - 'Default.GetString'
				              - 'FromBase64String'
				      condition: (selection1 or selection2) and selection3
				```
		- **技术背景**
			- PowerShell 命令和脚本可以通过加载底层 System.Management.Automation 命名空间来执行，该命名空间通过 .NET 框架和 Windows 公共语言接口 (CLI) 公开，Powershell 实际上是一个名为system.management.automation.dll 的 DLL，它也可能以本机映像格式存在，如system.management.automation.ni.dll。
			- Powershell DLL 可以加载到多个进程中，这些进程称为 powershell 主机进程。包括 powershell.exe 或 powershell 集成脚本环境 (powershell_ise.exe) 、以及 Exchange 和 Azure Active Directory 的同步进程等；Powershell 主机进程一般都有微软签名的数字签名。
			- 攻击者喜欢利用 Powershell进行攻击，因为它提供了一个高级接口来与操作系统交互，而无需在 C、C# 或 .NET 中开发功能。虽然许多攻击者会直接使用Powershell进程，但更老练的攻击者可能会选择更加隐秘的方法，将 powershell 注入非原生的Powershell主机进程来绕过应用程序白名单和环境限制，详见：[unmanaged powershell](https://github.com/leechristensen/UnmanagedPowerShell)
			- 许多工具允许使用 DLL 运行 PowerShell。例如，“PowerShdll”和“NoPowerShell” ，他们依赖于 LOLBIN（无落地攻击文件），如 rundll32.exe、installutil.exe、regsvcs.exe、regasm.exe 和 regsvr32.exe 来调用 DLL。这些 LOLBIN 由微软签名并经常列入白名单。如下图是PowerShell 使用 rundll32 调用 DLL ，会出现一个带有 PowerShell 控制台的新窗口，这是进程中是不存在powershell.exe的。
			- <img src="/assets/Pasted image 20221104151615.png">
		- **盲点和假设**
			- 该策略依赖于以下假设：
				- Windows系统的终端agent正常运行
				- 正常监控所有模块加载（module loads）的行为
				- 终端收集的日志被发往SOC/SIEM等平台
				- SOC/SIEM加载检测策略进行分析
			- 如果违反任何假设，就会出现盲点。例如，以下内容会触发告警：
				- 合法的 Powershell 主机进程被滥用（例如 powershell.exe）
				- 列入白名单的 Powershell 主机进程被滥用
				- 终端Agent被攻击者控制
		- **误报**
			- 有几种情况会发生误报：
				- 合法的 powershell 触发告警，并且没有通过白名单进行加白
			- 合法的 powershell 进程通常有如下特征：
				- 调用方有合法的数字签名
				- Powershell 使用标准方法（例如 LoadLibrary）将本机 Powershell 库加载到内存中
				- 信任的二进制文件
		- **优先级**
			- 在所有条件下，优先级都设置为中等
		- **验证**
			- 通过在 MacOS 主机上执行以下命令来验证：
				```
				  Copy-Item C:\\Windows\\System32\\WindowsPowerShell\\v1.0\\powershell.exe -Destination C:\\windows\\temp\\unusual-powershell-host-process-test.exe -Force 
				  Start-Process C:\\windows\\temp\\unusual-powershell-host-process-test.exe -ArgumentList '-NoProfile','-NonInteractive','-Windowstyle Hidden','-Command {Get-Date}' 
				  Remove-Item 'C:\\windows\\temp\\unusual-powershell-host-process-test.exe' -Force -ErrorAction SilentlyContinue
				```
		- **响应**
			- 如果触发此警报，建议采用以下响应程序：
				- 将可疑的 powershell 进程与白名单上进行比较
					- 请注意是否存在由于路径或驱动器号差异导致的小问题
				- 检查数字签名
					- 使用工具或 powershell 来确定二进制文件是否经过数字签名
					- 对签名者和二进制文件进行置信度确认
				- 确定二进制文件是否对应于已安装的应用程序
					- 查看 osquery 以查找可能与二进制文件匹配的已安装软件包
				- 查看二进制文件的行为
					- 是否进行了任何异常的网络连接？
					- 是否产生了任何子进程？
					- 是否进行了任何可疑的文件修改？如果二进制文件不可信，或无法追踪到合法安装的应用程序，将其视为潜在危害并升级为安全事件。
		- **其他资源**
			- https://github.com/Mr-Un1k0d3r/PowerLessShell
			- https://github.com/leechristensen/UnmanagedPowerShell
			- https://www.blackfog.com/fileless-powershell-protection/
- # 思考
	- 如何构建好的威胁检测能力？首先，**目标应该明确**，模糊的的目标会导致无法落地执行；其次是要**做出一些假设**，因为我们不可能掌握所有信息，这个世界充满了不确定性，当面对不确定性时，必须通过假设，来不断证伪和修正。比如我们想检测CoblatStrike，是检测beacon还是它执行的一个或者多个功能呢？是否要考虑在不同环境、不同版本、不同网络架构的区别？在终端层、网络层、身份认证数据等不同检测维度下，分别该如何检测和联动？等等这些问题都需要在一个个的假设下去完成。最后是**上下文信息**，当我们看到一个警告时，从标题和内容并不知道它是用来做什么的、它的检测机制是怎样的、应该用它做什么？
	- Palantir的威胁检测框架在很大程度上在尝试解决上述的问题，当然，在这个过程中，离不开专家知识、人工处置、足够全的数据和自动化操作等。
- # 参考
	- https://blog.palantir.com/alerting-and-detection-strategy-framework-52dc33722df2
	- https://www.paloaltonetworks.com/blog/security-operations/stopping-powershell-without-powershell
	- https://cloud.withgoogle.com/cloudsecurity/podcast