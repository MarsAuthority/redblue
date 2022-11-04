up::[[Blue Team能力建设]]-[[日志分析 Log Analysis]]-[[检测规则 Detection Rules]]

- 内置100+检测规则可直接使用，如下图：
![[Pasted image 20221104172950.png]]
- 案例
	- 检测计划任务（scheduled task）是否被创建
		- **Rule type**：EQL
		- **Rule indices**:
			- winlogbeat-*
			- logs-system.*
		- **Severity**：Low
		- **Risk score**：21
		- **Runs every**：5 min
		- **Searches indices from**：now-9m
		- **Maximum alerts per execution**：100
		- **References**：https://docs.microsoft.com/en-us/windows/security/threat-protection/auditing/event-4698
		- **Potential false positives**：安装新软件时，也可能触发创建合法的计划任务
		- 规则：
		```
		iam where event.action == "scheduled-task-created" and /* excluding 
		tasks created by the computer account */ not user.name : "*$" and
		/* TaskContent is not parsed, exclude by full taskname noisy ones */
		not winlog.event_data.TaskName : ("\\OneDrive Standalone
		Update Task-S-1-5-21*", "\\Hewlett-Packard\\HP Web
		Products Detection", "\\Hewlett-Packard\\HPDeviceCheck")
		```
		- **Framework**: MITRE ATT&CKTM
			- Tactic:
				- Name: Persistence
				- ID: TA0003
				- Reference URL: [https://attack.mitre.org/tactics/TA0003/](https://attack.mitre.org/tactics/TA0003/)
			- Technique:
				- Name: Scheduled Task/Job
				- ID: T1053
				- Reference URL: [https://attack.mitre.org/techniques/T1053/](https://attack.mitre.org/techniques/T1053/)
- 参考
	- https://www.elastic.co/guide/en/security/current/prebuilt-rules.html