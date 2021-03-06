## 简洁篇 - HTTP/HTTPS并发

---

#### HTTP并发

需求：1000并发\(此为演示，实际并发支持很高\)

环境：eth1已up，web服务器IP为:66.66.66.66/8\(若使用sos作为服务端，则默认带HTTP服务\)

步骤：

\#**配置测试口ip，并复用自带模板**

```
ifconfig eth1 66.67.67.67 netmask 255.0.0.0
cp /sos/client/curl-loader-0.56/conf-examples/10K.conf /sos/case/1000.conf
```

\#**修改HTTP并发配置，修改1000.conf中对应项，改为以下内容\(未列出的内容，不要删除\)，保存**

```
BATCH_NAME= 1000                     #任务名称
CLIENTS_NUM_MAX=1000                 #并发数
CLIENTS_NUM_START=20                 #初始化并发数
CLIENTS_RAMPUP_INC=20                #每秒增长数
INTERFACE =eth1                      #绑定网口
NETMASK=8                            #并发IP掩码
IP_ADDR_MIN= 66.67.67.68             #并发IP起始范围
IP_ADDR_MAX= 66.167.167.167          #并发IP结束范围
CYCLES_NUM= 1                        #任务执行次数，默认为-1(一直执行，直到CTRL+C)，设置为1则执行次
URLS_NUM= 1                          #访问的URL个数
URL=http://66.66.66.66/index.html    #访问的URL，支持多个(设置多个URL时，URLS_NUM为URL数量
```

**\#进入/sos/tmp目录\(因为在此执行，会生成测试日志\)，执行并发任务，**

```
cd /sos/tmp
curl-loader-boss -f ../case/1000.conf
```

**\#查看连接数：建议在客户端和服务端上都执行**

```
net-monitor
```

**\#查看吞吐：建议在客户端和服务端上都执行**

```
bmon -b
```

**\#注意：记得停止流量**

```
curl-loader-boss -s
```

#### HTTPS并发

**\#完全按照HTTP并发步骤，只需要将1000.conf基础上的对应项，改为以下内容，执行即可**

**\#备注：若使用sos作为服务端，则默认带HTTPS服务**

```
URL=https://66.66.66.66/index.html
TIMER_URL_COMPLETION = 0
TIMER_AFTER_URL_SLEEP =1000
```

**\#查看连接数：建议在客户端和服务端上都执行**

```
net-monitor
```

**\#查看吞吐：建议在客户端和服务端上都执行**

```
bmon -b
```

**\#注意：记得停止流量**

```
curl-loader-boss -s
```



