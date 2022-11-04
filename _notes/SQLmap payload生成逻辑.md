up::[[Red Team能力建设]]-[[初始访问 Initial Access]]-[[SQL 注入]]
- # test和boundary组合生成payload
	- 这部分主要梳理sqlmap生成payload的方式，sqlmap扫描规则文件位于xml文件夹中的boundaries.xml文件和payloads文件夹，payloads文件夹中存放了六种注入类型的xml文件。
		- boolean_blind
		- error_based
		- inline_query
		- stacked_queries
		- time_blind
		- union_query
	- 最终payload的生成方式为: prefix + payload + comment + suffix, 其中prefix和suffix由boundaries中的子节点prefix和suffix提供，payload和comment由payloads文件夹中的test的子节点payload和comment提供。
- # 解析test
	- test节点的格式如下：
		```
		  <test>
		      <title></title>
		      <stype></stype>
		      <level></level>
		      <risk></risk>
		      <clause></clause>
		      <where></where>
		      <vector></vector>
		      <request>
		          <payload></payload>
		          <comment></comment>
		          <char></char>
		          <columns></columns>
		      </request>
		      <response>
		          <comparison></comparison>
		          <grep></grep>
		          <time></time>
		          <union></union>
		      </response>
		      <details>
		          <dbms></dbms>
		          <dbms_version></dbms_version>
		          <os></os>
		      </details>
		  </test>
		 ```
	- title
		- 当前测试Payload的标题，通过标题可以了解当前的注入类型以及测试的数据库类型
	- stype
		- SQL注入的类型, 可能的值和代表的意义如下
			- Boolean-based blind SQL injection
			- Error-based queries SQL injection
			- Inline queries SQL injection
			- Stacked queries SQL injection
			- Time-based blind SQL injection
			- UNION query SQL injection
	- level
		- 测试SQL注入的深度，共有5个级别，级别越高发送的请求数越多。这和boundary子节点level的含义是一致的
			- Always (<100 requests)
			- Try a bit harder (100-200 requests)
			- Good number of requests (200-500 requests)
			- Extensive test (500-1000 requests)
			- You have plenty of time (>1000 requests)
	- risk
		- 测试payload破坏数据完整性的可能性。
			- Low risk
			- Medium risk
			- High risk
	- clause[多个值用,分隔]
		- payload在哪个字段位置生效, 这和boundary子节点clause的含义是一致的
			- Always
			- WHERE / HAVING
			- GROUP BY
			- ORDER BY
			- LIMIT
			- OFFSET
			- TOP
			- Table name
			- Column name
			- Pre-WHERE (non-query)
	- where[多个值用,分隔]
		- 添加完整payload`<prefix>`,`<payload>`,`<comment>`,`<suffix>`的地方, 这和boundary子节点where的含义是一致的
		- Append the string to the parameter original value
		- Replace the parameter original value with a negative random integer value and append our string
		- Replace the parameter original value with our string
	- vector
		- 注入向量
	- request
		- 注入测试发送的请求
			- payload
				- 测试使用的payload, 其中的[RANDNUM]，[DELIMITER_STAR]，[DELIMITER_STOP]分别代表着随机数值与字符, 扫描时会用对应的随机数替换掉。
			- comment
				- 添加在payload后面的注释
			- char
				- 在union注入测试中暴力破解列数所使用的字符
			- columns
				- 在union注入测试中测试列的范围， 如<columns>[COLSTART]-[COLSTOP]</columns>
	- response
		- 从响应中识别是否成功注入
			- comparison(布尔盲注)
				- 应用比较算法，比较两次请求响应的不同，一次请求的payload使用request子节点中的payload, 另一次请求的payload使用response子节点中的comparison
			- grep(报错注入)
				- 响应中匹配的正则表达式
			- time(时间盲注和堆叠注入)
				- 响应返回之前等待的时间(单位为秒)
			- union(union注入)
				- 调用 unionTest()函数
	- details
		- 注入成功能得到的关于操作系统和数据库相关的信息
			- dbms
		- 使用的数据库类型
			- dbms_version
		- 数据库版本
			- os
		- 运行数据库得操作系统
- # 解析boundaries
	- boundary节点的格式如下:
		```
		  <boundary>
		      <level></level>
		      <clause></clause>
		      <where></where>
		      <ptype></ptype>
		      <prefix></prefix>
		      <suffix></suffix>
		  </boundary>
		```
		- 其中`level`, `clause`, `where`表示的含义和test节点中所表示的含义是一致的。
	- ptype
		- 测试参数的类型
			- Unescaped numeric
			- Single quoted string
			- LIKE single quoted string
			- Double quoted string
			- LIKE double quoted string
	- prefix
		- ``<payload>``,``<comment>``添加的前缀
	- suffix
		- `<payload>`,`<comment>`添加的后缀
- # 源码解析payload组合
	- sqlmap使用`lib/controller/checks.py`文件中的`checkSqlInjection()`函数进行sql注入的测试, 其中利用test和boundary生成payload的主要代码如下，忽略其他逻辑判断
		```
		  # Favoring non-string specific boundaries in case of digit-like parameter values
		  if value.isdigit():
		      kb.cache.intBoundaries = kb.cache.intBoundaries or sorted(copy.deepcopy(conf.boundaries), key=lambda boundary: any(_ in (boundary.prefix or "") or _ in (boundary.suffix or "") for _ in ('"', '\'')))
		      boundaries = kb.cache.intBoundaries
		  else:
		      boundaries = conf.boundaries
		  
		  tests = getSortedInjectionTests()
		  
		  while tests:
		      test = tests.pop(0)
		  
		      for boundary in boundaries:
		          injectable = False
		  
		          # Skip boundary if it does not match against test's <clause>
		          # Parse test's <clause> and boundary's <clause>
		          clauseMatch = False
		  
		          for clauseTest in test.clause:
		              if clauseTest in boundary.clause:
		                  clauseMatch = True
		                  break
		  
		          if test.clause != [0] and boundary.clause != [0] and not clauseMatch:
		              continue
		  
		          # Skip boundary if it does not match against test's <where>
		          # Parse test's <where> and boundary's <where>
		          whereMatch = False
		  
		          for where in test.where:
		              if where in boundary.where:
		                  whereMatch = True
		                  break
		  
		          if not whereMatch:
		              continue
		  
		          # For each test's <where>
		          for where in test.where:
		              # generage payload
		              templatePayload = agent.payload(..., where=where)
		 ```
	- 利用test和boundary生成payload的流程为:
		- 循环遍历每一个test,
		- 对某个test,循环遍历boundary
		- 若boundary的where包含test的where值，并且boundary的clause包含test的clause值, 则boundary和test可以匹配
		- 循环test的where值,结合匹配的boundary生成相应的payload
- # 来源
	- http://www.beesfun.com/2017/03/30/sqlmap%E6%BA%90%E7%A0%81%E8%A7%A3%E6%9E%90%E4%B9%8Btest%E5%92%8Cboundary%E7%BB%84%E5%90%88%E7%94%9F%E6%88%90payload/