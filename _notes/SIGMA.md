up::[[Blue Team能力建设]]-[[日志分析 Log Analysis]]-[[检测规则 Detection Rules]]
- # 介绍
	- 一种通用且开放的基于签名的规则格式，可以直接描述相关的日志事件。规则格式非常灵活，易于编写，适用于任何类型的日志文件。主要目的是提供一种结构化的形式，研究人员或分析人员可以在其中描述他们曾经开发的检测方法，并使其与他人共享。
	- Sigma支持主流的SIEM工具。它有以下优点：
		- 它使分析能够在组织之间重复使用和共享。
		- 高级通用分析语言
		- 解决记录签名问题等最可靠的方法
		- 纯文本YAML文件
- # 相关Presentation
	- https://github.com/Neo23x0/Talks/blob/master/Sigma_Hall_of_Fame_20211022.pdf
- # 举例：CVE-2009-3898的Sigma规则
	```
	  title: Suspicious PsExec Execution
	  id: c462f537-a1e3-41a6-b5fc-b2c2cef9bf82
	  description: detects execution of psexec or paexec with renamed service name, this rule helps to filter out the noise if psexec is used for legit purposes or if attacker
	      uses a different psexec client other than sysinternal one
	  author: Samir Bousseaden
	  date: 2019/04/03
	  references:
	      - https://blog.menasec.net/2019/02/threat-hunting-3-detecting-psexec.html
	  tags:
	      - attack.lateral_movement
	      - attack.t1077
	  logsource:
	      product: windows
	      service: security
	      description: 'The advanced audit policy setting "Object Access > Audit Detailed File Share" must be configured for Success/Failure'
	  detection:
	      selection1:
	          EventID: 5145
	          ShareName: \\*\IPC$
	          RelativeTargetName:
	           - '*-stdin'
	           - '*-stdout'
	           - '*-stderr'
	      selection2:
	          EventID: 5145
	          ShareName: \\*\IPC$
	          RelativeTargetName: 'PSEXESVC*'
	      condition: selection1 and not selection2
	  falsepositives:
	      - nothing observed so far
	  level: high
	```
- # 项目地址
	- https://github.com/SigmaHQ/sigma