up::[[Blue Team能力建设]]-[[应急响应 Incident Response]]
- # 目录
	```
	  antivirus.yaml 
	  applications.yaml 
	  cloud_services.yaml 
	  config_files.yaml 
	  containerd.yaml 
	  docker.yaml 
	  hadoop.yaml 
	  installed_modules.yaml 
	  instant_messaging.yaml 
	  java.yaml 
	  kaspersky_careto.yaml 
	  kubernetes.yaml 
	  legacy.yaml 
	  linux.yaml 
	  linux_proc.yaml 
	  macos.yaml 
	  ntfs.yaml 
	  tomcat.yaml 
	  unix_common.yaml 
	  webbrowser.yaml 
	  webservers.yaml 
	  windows.yaml 
	  windows_dll_hijacking.yaml 
	  wmi.yaml
	```
- # 案例- antivirus.yaml内容如下：
	```
	  # Anti-Virus artifacts.
	  
	  name: EsetAVQuarantine
	  doc: Eset Anti-Virus Quarantine (Infected) files.
	  sources:
	  - type: FILE
	    attributes: {paths: ['/Library/Application Support/ESET/esets/cache/quarantine/*']}
	  supported_os: [Darwin]
	  labels: [Antivirus]
	  ---
	  name: MicrosoftAVQuarantine
	  doc: Microsoft Anti-Virus Quarantine (Infected) files.
	  sources:
	  - type: FILE
	    attributes:
	      paths:
	      - '%%environ_allusersappdata%%\Microsoft\Microsoft Antimalware\Quarantine\**'
	      - '%%environ_allusersappdata%%\Microsoft\Windows Defender\Quarantine\**'
	      separator: '\'
	  supported_os: [Windows]
	  labels: [Antivirus]
	  ---
	  name: MicrosoftAVLogs
	  doc: Microsoft Anti-Virus log files.
	  sources:
	  - type: FILE
	    attributes:
	      paths:
	      - '%%environ_allusersappdata%%\Microsoft\Windows Defender\Support\MPLog-*.log'
	      - '%%environ_allusersappdata%%\Microsoft\Windows Defender\Support\MPDetection-*.log'
	      separator: '\'
	  supported_os: [Windows]
	  labels: [Antivirus, Logs]
	  ---
	  name: WindowsDefenderExclusions
	  doc: |
	    Directories, processes and extensions configured not to be scanned by Windows Defender.
	    The can be set locally or through group policy objects (GPO).
	  
	    Certain malware families (for example, Tofsee) are known to add directories to the
	    Paths list in order to avoid being detected by Windows Defender. Other malware
	    (for example, REvil) use the existing exclusions to be ignored by Anti-Virus products.
	  sources:
	  - type: REGISTRY_KEY
	    attributes:
	      keys:
	      - 'HKEY_LOCAL_MACHINE\Software\Microsoft\Windows Defender\Exclusions\Paths\*'
	      - 'HKEY_LOCAL_MACHINE\Software\Microsoft\Windows Defender\Exclusions\Processes\*'
	      - 'HKEY_LOCAL_MACHINE\Software\Microsoft\Windows Defender\Exclusions\Extensions\*'
	      - 'HKEY_LOCAL_MACHINE\Software\Microsoft\Windows Defender\Exclusions\TemporaryPaths\*'
	      - 'HKEY_LOCAL_MACHINE\Software\Policies\Microsoft\Windows Defender\Exclusions\Paths\*'
	      - 'HKEY_LOCAL_MACHINE\Software\Policies\Microsoft\Windows Defender\Exclusions\Processes\*'
	      - 'HKEY_LOCAL_MACHINE\Software\Policies\Microsoft\Windows Defender\Exclusions\Extensions\*'
	      - 'HKEY_LOCAL_MACHINE\Software\Policies\Microsoft\Windows Defender\Exclusions\TemporaryPaths\*'
	  supported_os: [Windows]
	  urls:
	  - 'https://blog.malwarebytes.com/detections/pum-optional-msexclusion/'
	  - 'https://answers.microsoft.com/en-us/protect/forum/all/windows-defender-how-to-remove-exclusions/2a0cc465-97b2-46ea-ae77-b87075ed124e'
	  - 'https://blog.talosintelligence.com/2019/05/threat-roundup-0503-0510.html'
	  - 'https://news.sophos.com/en-us/2021/07/04/independence-day-revil-uses-supply-chain-exploit-to-attack-hundreds-of-businesses/'
	  ---
	  name: SophosAVLogs
	  doc: Sophos Anti-Virus log files.
	  sources:
	  - type: FILE
	    attributes: {paths: ['/Library/Logs/Sophos*.log']}
	    supported_os: [Darwin]
	  - type: FILE
	    attributes:
	      paths: ['%%environ_allusersappdata%%\Sophos\Sophos Anti-Virus\Logs\*']
	      separator: '\'
	    supported_os: [Windows]
	  supported_os: [Darwin, Windows]
	  labels: [Antivirus, Logs]
	  ---
	  name: SophosAVQuarantine
	  doc: Sophos Anti-Virus Quarantine (Infected) files.
	  sources:
	  - type: FILE
	    attributes: {paths: ['/Users/Shared/Infected/*']}
	    supported_os: [Darwin]
	  - type: FILE
	    attributes:
	      paths: ['%%environ_allusersappdata%%\Sophos\Sophos Anti-Virus\INFECTED\*']
	      separator: '\'
	    supported_os: [Windows]
	  supported_os: [Darwin, Windows]
	  labels: [Antivirus]
	  ---
	  name: SymantecAVLogs
	  doc: Symantec Anti-Virus Log Files.
	  sources:
	  - type: FILE
	    attributes:
	      paths:
	      - '%%environ_allusersappdata%%\Symantec\Symantec Endpoint Protection\*\Data\Logs\*.log'
	      - '%%environ_allusersappdata%%\Symantec\Symantec Endpoint Protection\*\Data\Logs\AV\*.log'
	      - '%%users.localappdata%%\Symantec\Symantec Endpoint Protection\Logs\*.log'
	      separator: '\'
	    supported_os: [Windows]
	  supported_os: [Windows]
	  labels: [Antivirus, Logs]
	  ---
	  name: SymantecAVQuarantine
	  doc: Symantec Anti-Virus Quarantine (Infected) files.
	  sources:
	  - type: FILE
	    attributes:
	      paths: ['%%environ_allusersappdata%%\Symantec\Symantec Endpoint Protection\**5\*.vbn']
	      separator: '\'
	    supported_os: [Windows]
	  supported_os: [Windows]
	  labels: [Antivirus, Logs]
	 ```
- # 使用
	```
	  $WindowsArtifacts=$(curl https://raw.githubusercontent.com/ForensicArtifacts/artifacts/master/data/windows.yaml)
	  $obj = ConvertFrom-Yaml $WindowsArtifacts.Content -AllDocuments
	  
	  $count=0;
	  foreach ($Artifact in $obj){
	  $Artifacts = [pscustomobject][ordered]@{
	  	Name = $obj.name[$count]
	  	Description = $obj.doc[$count]
	  	References = $obj.urls[$count]
	  	Attributes = $obj.sources.attributes[$count]
	  }
	  $count++;
	  $Artifacts | FL;
	  }
	```
- # 来源
	- https://www.jaiminton.com/cheatsheet/DFIR/#disclaimer
	- https://github.com/ForensicArtifacts/artifacts/tree/main/data