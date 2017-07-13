## 简洁篇 - HTTP/HTTPS并发

#### **HTTP并发**

需求：1000并发\(此为演示，实际并发支持很高\)

环境：eth1已up，web服务器IP为:66.66.66.66/8\(若使用sos作为服务端，则默认带HTTP服务\)

步骤：

**\#配置测试口ip**

ifconfig eth1 66.67.67.67 netmask 255.0.0.0

**\#复用自带模板**

cp /sos/client/curl-loader-0.56/conf-examples/10K.conf /sos/case/1000.conf

**\#修改HTTP并发配置，修改1000.conf中对应项，改为以下内容\(未列出的内容，不要删除\)，保存**

_\#任务名称_

BATCH\_NAME= 1000

_\#并发数_

CLIENTS\_NUM\_MAX=1000

_\#初始化并发数_

CLIENTS\_NUM\_START=20

_\#每秒增长数_

CLIENTS\_RAMPUP\_INC=20

_\#绑定网口_

INTERFACE   =eth1

_\#并发IP掩码_

NETMASK=8

_\#并发IP段_

IP\_ADDR\_MIN= 66.67.67.68

IP\_ADDR\_MAX= 66.167.167.167

_\#任务执行次数，默认为-1\(一直执行，直到CTRL+C\)，设置为1则执行次_

CYCLES\_NUM= 1

URL=[http://66.66.66.66/index.html](http://66.66.66.66/index.html)

**\#进入/sos/tmp目录，因为在此执行，会生成测试日志**

cd /sos/tmp

**\#执行，并发任务为一直跑**

curl-loader -f ../case/1000.conf

**\#可以使用以下命令观察是否在跑**

iftop -N -n -i eth1

或者

_\#建议在客户端和服务端上都执行_

while :; do netstat -anp\|grep EST\|grep :80\|wc -l;sleep 1;done

#### **HTTPS并发**

**完全按照HTTP并发步骤，只需要将1000.conf基础上的对应项，改为以下内容，执行即可**

备注：若使用sos作为服务端，则默认带HTTPS服务

URL=[https://66.66.66.66/index.html](http://66.66.66.66/index.html)

TIMER\_URL\_COMPLETION = 0

TIMER\_AFTER\_URL\_SLEEP =1000

**\#可以使用以下命令观察是否在跑**

iftop -N -n -i eth1

或者

_\#建议在客户端和服务端上都执行_

while :; do netstat -anp\|grep EST\|grep :4430\|wc -l;sleep 1;done
