title: modsecurity rule grammar
tags:
    - modsecurity
    - WAF
date: 2025/04/23
categories: security
---

# modsecurity规则语法
modsecurity core rule规则可以参考[官方wiki文档](https://github.com/owasp-modsecurity/ModSecurity/wiki/Reference-Manual-(v3.x))
SecRule的基本语法
SecRule VARIABLES OPERATOR [ACTIONS]
* VARIABLES:一般是指定请求和响应的对象，如REQUEST_BODY、REQUEST_HEADERS、REQUEST_METHOD、REQUEST_URI(包含完整的请求URL,有请求的字符串数据，比如/index.php?p=x)
* OPERATOR:可以理解为一个操作函数，一个入参是VARIABLES，一个是操作函数后面的入参，@用于标识操作函数，比如`SecRule TX:DETECTION_PARANOIA_LEVEL "@lt 1" "id:913011"`，lt函数比较VARIABLES与1的值，如果VARIABLES小于1，,则执行action。pmFromFIle函数匹配variables与入参，不区分大小写，入参可以有多个，可以送入文件。`SecRule REQUEST_HEADERS:User-Agent "@pmFromFile scanners-user-agents.data scanners-user-agents-1.data" "id:913011"`。
* ACTIONS:满足OPERATOR的要求后执行的动作，有多个action时，用逗号隔开，常用的一些aciton如下：
  * "id:xxxx":对规则进行分类编号
  * "allow":停止规则匹配，允许通信继续执行
  * "auditlog":标记本次通信，并记录audit log
  * “block":阻止通信，但没有语法支持如何阻止通信，具体使用细节参考[block说明](https://github.com/owasp-modsecurity/ModSecurity/wiki/Reference-Manual-(v3.x)#user-content-block)

# coreruleset实例
[coreruleset](https://github.com/coreruleset/coreruleset)是modsecurity官方给出的一些规则集，其中有一些针对特定攻击或探测的规则，可供参考。
