up::[[终端基础设施 Endpoint Infrastructure]]-[[EDR]]
<img src="/assets/Pasted image 20221104145425.png">
- # 1. 市场趋势
	- 到 2023 年底，云交付的 EPP 解决方案将超过 95% 。
	- 到 2025 年，50% 使用 EDR 的组织将使用托管检测和响应功能（MDR）。
	- 到 2025 年，60% 的 EDR 解决方案将包含来自多个安全控制源的数据，例如身份认证、CASB 和 DLP。
- # 2. 市场定义/描述
	- Gartner对EPP市场的定义如下：EPP提供了将agent或sensor部署到托管终端（包括 PC、服务器和其他设备）的工具；这些旨在防止一系列已知和未知的恶意软件和威胁，并提供免受此类威胁的保护；此外，它们还提供调查和补救任何逃避保护控制的事件的能力。
	- EPP的核心功能是：
		- 预防和保护安全威胁，包括使用基于文件和无文件漏洞的恶意软件。
		- 将控制（允许/阻止）应用于软件、脚本和进程的能力 。
		- 使用对设备活动、应用程序和用户数据的行为分析来检测和预防威胁的能力 。
		- 当漏洞利用逃避保护控制时进一步调查事件和/或获得补救指导的设施 。
	- EPP中经常出现的可选功能可能包括：
		- 终端设备的资产、配置和策略管理的收集和报告 。
		- 磁盘加密、本地防火墙设置等操作系统安全控制状态的管理和报告 。
		- 扫描系统漏洞和报告/管理安全补丁安装的设施 。
		- 能够报告互联网、网络和应用程序活动，以获得潜在恶意活动的其他迹象 。
- # 3. 市场概览
	- 勒索软件目前是所有组织面临的最大风险。
	- 远程办公显著加速了基于云的解决方案落地（存量占60%，增量占95%）。
	- 无文件攻击已经非常普及，使得基于行为的检测防护成为EDR工具的关键能力。
	- 采用EDR工具的最大障碍仍然是操作它们所需的技能和增加的总成本。
	- 为了缩小技能差距，提供监控和警报分类的MDR服务正变得越来越流行。
	- SolarWinds供应链攻击说明了EDR的价值和缺点；EDR并没有实时检测到SolarWinds攻击，但在阻止新发现的恶意行为非常有用。
	- SolarWinds 攻击还表明需要至少更好地集成来自身份和电子邮件的遥测数据，以及需要有效的篡改保护以确保代理不被禁用。
	- XDR正在成为EPP解决方案的最新关键功能。
	- 所有组织都需要优先级更高的加固指导。
	- EPP解决方案还可以添加移动端的威胁防御和与统一终端管理的集成，以减少整体管理负担。
- # 4. 准入准出原则
	- 所有供应商必须至少提供以下12项：
		- 无需更新规则即可防御。
		- 可以根据进程行为检测恶意活动。
		- 可以将IOC/IOA数据集中存储分析。
		- 能够检测防御无文件恶意文件的攻击。
		- 检测到恶意软件时自动删除恶意软件，删除/隔离文件/终止进程。
		- 能够在控制台压制/忽略误报。
		- EPP控制台必须是基于云的SaaS服务，由供应商管理维护。
		- 控制台和报告能够显示完整的进程树，以识别进程是如何产生的，并进行可操作的根本原因分析。
		- 需提供威胁狩猎能力，控制台提供跨终端的IOC/IOA（如文件哈希、源/目标 IP、注册表项）搜索。
		- 能够识别恶意软件并给出修复步骤或者实现回滚。
		- 提供选项将威胁情报集成到管理控制台中。
		- 能够防护常见应用程序漏洞和内存利用攻击。
		- 当终端设备位于公司网络之外时，必须继续收集可疑事件数据。
		- 能对文件夹、驱动器或设备（例如 USB 驱动器）执行静态、按需恶意软件检测扫描。
		- 所有功能都必须在单个agent/sensor中提供或直接集成到操作系统中。
- # 参考
	- https://www.gartner.com/doc/reprints?ct=210506&id=1-25Z4TEJM&st=sb