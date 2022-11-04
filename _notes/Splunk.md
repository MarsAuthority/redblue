up::[[Blue Team能力建设]]-[[日志分析 Log Analysis]]-[[检测规则 Detection Rules]]
- # 介绍
	- Splunk提供映射到ATT&CK/KILL CHAIN/CIS Controls TTPs相关的USE CASE和检测规则， 它们包括检测规则、搜索语法、机器学习算法和 Splunk Phantom 剧本——所有这些都旨在协同工作以检测、调查和响应威胁。
- # 覆盖范围
	- 要查看使用ATT&CK标记的所有内容的最新检测覆盖图，请访问：https://mitremap.splunkresearch.com 包含目前检测覆盖范围的技术的快照。 蓝色的阴影越深，对这种特定技术的检测就越多；该地图在每次发布时自动更新，并从 generate-coverage-map.py 生成。
- # 检测规则
	- 包含了终端、网络和云相关的200多种检测规则，示例如下:
		```
		  name: Eventvwr UAC Bypass
		  id: 9cf8fe08-7ad8-11eb-9819-acde48001122
		  version: 1
		  date: '2021-03-01'
		  author: Michael Haag, Splunk
		  type: TTP
		  datamodel:
		  - Endpoint
		  description: The following search identifies Eventvwr bypass by identifying the registry
		    modification into a specific path that eventvwr.msc looks to (but is not valid)
		    upon execution. A successful attack will include a suspicious command to be executed
		    upon eventvwr.msc loading. Upon triage, review the parallel processes that have
		    executed. Identify any additional registry modifications on the endpoint that may
		    look suspicious. Remediate as necessary.
		  search: '| tstats `security_content_summariesonly` count values(Registry.registry_key_name)
		    as registry_key_name values(Registry.registry_path) as registry_path min(_time)
		    as firstTime max(_time) as lastTime FROM datamodel=Endpoint.Registry where  Registry.registry_path="*mscfile\\shell\\open\\command\\*"  by
		    Registry.user, Registry.dest , Registry.registry_value_name| `security_content_ctime(lastTime)`
		    | `security_content_ctime(firstTime)` | `drop_dm_object_name(Registry)` | `eventvwr_uac_bypass_filter`'
		  how_to_implement: To successfully implement this search you need to be ingesting information
		    on process that include the name of the process responsible for the changes from
		    your endpoints into the `Endpoint` datamodel in the `Registry` node.
		  known_false_positives: Some false positives may be present and will need to be filtered.
		  references:
		  - https://blog.malwarebytes.com/malwarebytes-news/2021/02/lazyscripter-from-empire-to-double-rat/
		  - https://github.com/redcanaryco/atomic-red-team/blob/master/atomics/T1548.002/T1548.002.md
		  - https://attack.mitre.org/techniques/T1548/002
		  - https://enigma0x3.net/2016/08/15/fileless-uac-bypass-using-eventvwr-exe-and-registry-hijacking/
		  tags:
		    analytic_story:
		    - Windows Defense Evasion Tactics
		    - IcedID
		    automated_detection_testing: passed
		    confidence: 100
		    context:
		    - Source:Endpoint
		    - Stage:Defense Evasion
		    dataset:
		    - https://media.githubusercontent.com/media/splunk/attack_data/master/datasets/attack_techniques/T1548.002/atomic_red_team/windows-sysmon.log
		    impact: 80
		    kill_chain_phases:
		    - Exploitation
		    - Privilege Escalation
		    message: Registry values were modified to bypass UAC using Event Viewer on $dest$
		      by $user$.
		    mitre_attack_id:
		    - T1548.002
		    - T1548
		    observable:
		    - name: user
		      type: User
		      role:
		      - Victim
		    - name: dest
		      type: Hostname
		      role:
		      - Victim
		    product:
		    - Splunk Enterprise
		    - Splunk Enterprise Security
		    - Splunk Cloud
		    required_fields:
		    - _time
		    - Registry.registry_key_name
		    - Registry.registry_path
		    - Registry.user
		    - Registry.dest
		    - Registry.registry_value_name
		    risk_score: 80
		    security_domain: endpoint
		```
	- 完整内容见：
		- https://github.com/splunk/security_content/tree/develop/detections
- # USE CASEs
	- ProxyShell的示例如下:
		```
		  name: ProxyShell
		  id: 413bb68e-04e2-11ec-a835-acde48001122
		  version: 1
		  date: '2021-08-24'
		  author: Michael Haag, Teoderick Contreras, Mauricio Velazco, Splunk
		  type: batch
		  description: ProxyShell is a chain of exploits targeting on-premise Microsoft Exchange Server - CVE-2021-34473, CVE-2021-34523, and CVE-2021-31207.
		  narrative: 'During Pwn2Own April 2021, a security researcher demonstrated an attack chain targeting on-premise Microsoft Exchange Server. August 5th, the same researcher publicly released further details and demonstrated the attack chain. \
		    1. CVE-2021-34473 - Pre-auth path confusion leads to ACL Bypass (Patched in April by KB5001779) \
		    1. CVE-2021-34523 - Elevation of privilege on Exchange PowerShell backend (Patched in April by KB5001779) \
		    1. CVE-2021-31207 - Post-auth Arbitrary-File-Write leads to RCE (Patched in May by KB5003435) \
		    Upon successful exploitation, the remote attacker will have `SYSTEM` privileges on the Exchange Server. In addition to remote access/execution, the adversary may be able to run Exchange PowerShell Cmdlets to perform further actions.'
		  references:
		    - https://y4y.space/2021/08/12/my-steps-of-reproducing-proxyshell/
		    - https://www.zerodayinitiative.com/blog/2021/8/17/from-pwn2own-2021-a-new-attack-surface-on-microsoft-exchange-proxyshell
		    - https://www.youtube.com/watch?v=FC6iHw258RI
		    - https://www.huntress.com/blog/rapid-response-microsoft-exchange-servers-still-vulnerable-to-proxyshell-exploit#what-should-you-do
		    - https://i.blackhat.com/USA21/Wednesday-Handouts/us-21-ProxyLogon-Is-Just-The-Tip-Of-The-Iceberg-A-New-Attack-Surface-On-Microsoft-Exchange-Server.pdf
		  tags:
		    analytic_story:
		    - ProxyShell
		    category:
		    - Adversary Tactics
		    - Ransomware
		    product:
		    - Splunk Enterprise
		    - Splunk Enterprise Security
		    - Splunk Cloud
		    usecase: Advanced Threat Detection
		```
- # 攻击样本数据 Attack Data Repository
	- 无需准备攻击环境/工具， 快速验证检测规则的覆盖和有效性。
	- 数据使用YML，格式如下：
|字段|描述|
|--|--|
|id|唯一标识UUID|
|name|作者名称|
|date|最后修改日期|
|dataset|dataset关联URLs|
|description|简介|
|environment|运行环境|
|technique|对应的ATT&CK手法|
|references|参考信息|
|sourcetypes|来源类型|
	
- 示例如下:
	```
  id: 405d5889-16c7-42e3-8865-1485d7a5b2b6
  author: Patrick Bareiss
  date: '2020-10-08'
  description: 'Atomic Test Results: Successful Execution of test T1003.001-1 Windows
	Credential Editor Successful Execution of test T1003.001-2 Dump LSASS.exe Memory
	using ProcDump Return value unclear for test T1003.001-3 Dump LSASS.exe Memory using
	comsvcs.dll Successful Execution of test T1003.001-4 Dump LSASS.exe Memory using
	direct system calls and API unhooking Return value unclear for test T1003.001-6
	Offline Credential Theft With Mimikatz Return value unclear for test T1003.001-7
	LSASS read with pypykatz '
  environment: attack_range
  technique:
  - T1003.001
  dataset:
  - https://media.githubusercontent.com/media/splunk/attack_data/master/datasets/attack_techniques/T1003.001/atomic_red_team/windows-powershell.log
  - https://media.githubusercontent.com/media/splunk/attack_data/master/datasets/attack_techniques/T1003.001/atomic_red_team/windows-security.log
  - https://media.githubusercontent.com/media/splunk/attack_data/master/datasets/attack_techniques/T1003.001/atomic_red_team/windows-sysmon.log
  - https://media.githubusercontent.com/media/splunk/attack_data/master/datasets/attack_techniques/T1003.001/atomic_red_team/windows-system.log
  references:
  - https://attack.mitre.org/techniques/T1003/001/
  - https://github.com/redcanaryco/atomic-red-team/blob/master/atomics/T1003.001/T1003.001.md
  - https://github.com/splunk/security-content/blob/develop/tests/T1003_001.yml
  sourcetypes:
  - XmlWinEventLog:Microsoft-Windows-Sysmon/Operational
  - WinEventLog:Microsoft-Windows-PowerShell/Operational
  - WinEventLog:System
  - WinEventLog:Security
	```
- # 来源
	- https://github.com/splunk/attack_data