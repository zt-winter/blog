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
导入corerulest时，其中会有一个crs-setup.conf文件，该文件命令了一个通用变量集，在规则集中会引用这些变量，从而实现所有规则集的变量控制。
```conf
SecAction \
  "id:900990,\
  phase:1,\
  pass,\
  t:none,\
  nolog,\
  tag:'OWASP_CRS',\
  ver:'OWASP_CRS/4.14.0-dev',\
  setvar:tx.crs_setup_version=4140,\
  setvar:tx.detection_paranoia_level=1"
```
setvar:XXX就设置了变量，后续在coreruleset识别扫描的规则集中会获取该变量，并做出不同的探测级别设置，比如REQUEST-913-SCANNER-DETECTION.conf中`SecRule TX:DETECTION_PARANOIA_LEVEL "@lt 1" "id:913011"`，会判断TX:DETECTION_PARANOIA_LEVEL是否小于1。在crs-setup.conf文件中会对通用变量的含义有详细解释。