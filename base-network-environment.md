## 常用篇 - 基础网络环境

---

##### **\#以下各环境，先进入SOS网络环境构建模式：**

```
root@SOS:~# sos netenv

vyos@SOS:~$ configure
```

##### **\#所有**配置，在配置好命令后，使用以下两个进行保存，重启后依然生效

```
vyos@SOS# commit save

vyos@SOS# save
```

#### 默认服务

| **默认已启动** | **端口** | **根目录** | **公共目录** |
| :--- | :--- | :--- | :--- |
| HTTP/HTTPS | 80/443 | /usr/share/nginx/html/ | /sangfor/ |
| FTP | 21 | /sos/server/feitp-server/REV/ | /sangfor/ |
| CIFS | 445 | /home/samba/test/ | /sangfor/ |

#### 静态路由

**\#方法1：使用root登录或者vyos用户执行sudo**

`route add -net 192.168.20.0/24 gw 192.168.10.195`

**\#方法2：使用configure模式的set protocols命令**

`set protocols static route 192.168.20.0/24 next-hop 192.168.10.195 distance '1'`

**\#方法3：修改/etc/network/interfaces，保存，重启网络**

`up route add -net 192.168.20.0/24 gw 192.168.10.195`

#### 策略路由

**\#使用vyos用户configure模式的set命令**

```
 set policy route FILTER-WEB rule 1000 destination port 80 
 #设置策略路由FILTER-WEB匹配数据包的目的端口是80
 set policy route FILTER-WEB rule 1000 protocol tcp
 #设置策略路由FILTER-WEB匹配的协议是TCP
 set policy route FILTER-WEB rule 1000 set table 100
 #将策略路由FILTER-WEB加到100表
 set protocols static table 100 route 0.0.0.0/0 next-hop 10.255.0.2
 #设置策略路由FILTER-WEB的下一跳为10.255.0.2
 set interfaces ethernet eth1 policy route FILTER-WEB
 #将策略路由FILTER-WEB应用到eth1口
```

#### VLAN

**\#方法1：使用configure模式set interfaces命令**

`set interfaces ethernet eth0 vif 10 address 192.168.10.196/24`

此种方法配置的vlan，当该接口收到不带vlan\_tag的报文后会照样查路由表转发

**\#方法2：使用root登录下的sos-vlan命令**

```
说明：此方式配置vlan，通过修改子接口mac地址方式，解决路由器收到不带vlan_tag报文仍然查路由转发问题
      此方式通过配置物理子接口来实现vlan功能；若在eth1口配置vlan10，会生成一个eth1.10的子接口    
例如：添加vlan：sos-vlan -c 1 -i eth1 -v 10 -a 1.1.1.1/24 -m 00:90:0b:49:22:4b
      删除vlan：sos-vlan -c 0 -i eth1 -v 10 -a 1.1.1.1/24
  -c   configure   值为1添加vlan，值为0删除vlan
  -i   interface   待配置vlan的物理网口
  -v   vlan_id     vlan_id，<0-4094>
  -a   address     vlan接口的IP地址，Example：1.1.1.1/24
  -m   mac         vlan接口的mac地址，可自定义，不能物理口相同
```

#### PPPOE

**\#使用root用户执行sos-pppoe命令**

```
用法 : sos-pppoe [-c] -i interface -s start_ip -l last_ip -g gateway_ip-u user -p password
范例1：设置pppoe服务器：sos-pppoe -i eth1 -s 22.22.22.22 -l 22.22.22.30 -g 22.22.22.25 -u test -p test
范例2：设置pppoe且延时为100ms：sos-pppoe -i eth1 -s 22.22.22.22 -l 22.22.22.30 -g 22.22.22.25 -u test -p test -t 100
范例3：删除pppoe用户：sos-pppoe -c -u test
    -c   cancel      默认为设置pppoe，加-c为删除pppoe配置
    -i   interface   指定侦听接口
    -s   start_ip    资源池起始ip
    -l   last_ip     资源池终止ip
    -g   gateway     给拨号网段分配的网关
    -u   user        本地用户名
    -p   password    本地用户密码
    -d   dns-server  dns服务器，默认值为8.8.8.8
    -t   delay-time  侦听接口时延(ms)，默认为0
    -h   help        查看帮助信息
```

#### TC

**\#使用root用户执行sos-tc命令**

```
用法: sos-tc -i interface [cdDhlr]
设置10ms延时       : sos-tc -i eth0 -d 10
设置100ms+-10ms延时: sos-tc -i eth0 -d 100-10
设置1%丢包         : sos-tc -i eth0 -l 1
设置1%-3%丢包      : sos-tc -i eth0 -l 1-3
清空eth0配置       ：sos-tc -i eth0 -r
选项：
    -i interface     网口,eg: eth0
    -r recover       清除tc设置，恢复环境
    -d delay         时延，单位ms
    -l loss          数据包丢包率，单位%
    -D duplicate     数据包重传率，单位%
    -c corrupt       数据包失真率，单位%
    -h help          帮助
```

#### DNS

**\#使用root用户执行sos-dns命令**

```
说明: sos-dns一款便捷的dns服务器!

注意: 启动服务前，请使用sos-dns -c先配置好dns记录(使用sos-dns -e查看配置事例子)
      若启动失败，请检查是否内存不足，若是，则可以执行sos swapon开启交换分区来解决！
用法: sos-dns 加以下参数
选项：以下每行|分隔的参数等价
    -s|start                    启动dns服务
    -k|stop                     停止dns服务
    -l|status                   查看dns服务状态
    -c|config                   配置和查看dns记录
    -e|example                  查看dns记录配置事例
    -h|help                     查看帮助说明
```

#### MAIL

**\#使用root用户执行sos-mail命令**

```
说明: sos-mail一款便捷的邮件服务器(暂时支持smtp)!

注意: 默认邮件域名为test1.sos.com，所以请在sos-dns中配置好相关记录，若要定制域名，请使用sos-mail -c定制，并同步dns服务器!
用法: sos-mail 加以下参数
选项：以下每行|分隔的参数等价
    -s|start                    启动邮件服务
    -k|stop                     启动邮件服务
    -l|status                   查看邮件服务状态
    -c|config                   定制邮件配置
    -e|example                  查看邮件配置事例
    -h|help                     查看帮助说明
```

#### SYSLOG

**\#使用root用户执行sos-syslog命令**

```
说明： sos-syslog是采用rsyslog的syslog服务器

选项：以下每行|分隔的参数等价
    -s|--start                  启动sos-syslog服务器
    -h|--help                   查看帮助说明

例子：sos-syslog -s  启动sos-syslog服务器
```

#### DHCP

**\#先给dhcp服务对应的网口\(例如eth1\)，配置好ip和双工模式**

```
set interfaces ethernet eth1 address 192.168.10.127/24

set interfaces ethernet eth1 duplex auto
```

**\#再配置DHCP服务相关参数**

```
set service dhcp-server shared-network-name 'LAN' authoritative enable

set service dhcp-server shared-network-name 'LAN' subnet '192.168.10.0/24' start '192.168.10.128' stop '192.168.10.254'

set service dhcp-server shared-network-name 'LAN' subnet '192.168.10.0/24' default-router '192.168.10.1'

set service dhcp-server shared-network-name 'LAN' subnet '192.168.10.0/24' dns-server '192.168.10.1'

set service dhcp-server shared-network-name 'LAN' subnet '192.168.10.0/24' domain-name 'internal-net'

set service dhcp-server shared-network-name 'LAN' subnet '192.168.10.0/24' lease 86400
```

#### 流控

**\#首先新增两个流控策略**

```
set traffic-policy shaper WAN-OUT bandwidth '50Mbit'

set traffic-policy shaper WAN-OUT default bandwidth '50%'

set traffic-policy shaper WAN-OUT default ceiling '100%'

set traffic-policy shaper WAN-OUT default queue-type 'fair-queue'

set traffic-policy shaper LAN-OUT bandwidth '200Mbit'

set traffic-policy shaper LAN-OUT default bandwidth '50%'

set traffic-policy shaper LAN-OUT default ceiling '100%'

set traffic-policy shaper LAN-OUT default queue-type 'fair-queue'
```

**\#将策略应用到网口**

```
set interfaces ethernet eth2 traffic-policy out 'WAN-OUT'

set interfaces ethernet eth0 traffic-policy out 'LAN-OUT'
```

**\#设置延时和丢包（延时100ms、丢包1%）**

```
tc qdisc add dev eth0 root netem delay 100ms loss 1%

tc qdisc add dev eth2 root netem delay 100ms loss 1%
```

#### 广域网环境

**\#在eth0口模拟100ms的时延**

```
tc qdisc add dev eth0 root netem delay 100ms
```

**\#删除在eth0口设置的时延**

```
tc qdisc del dev eth0 root netem delay XXXms
```

**\#在eth0口模拟100ms＋－10ms（随机分布）的时延**

```
tc qdisc change dev eth0 root netem delay 100ms 10ms 25%
tc qdisc change dev eth0 root netem delay 100ms 10ms
```

**\#在eth0口模拟时延，在100ms＋－10ms之间的正态分布**

```
tc qdisc change dev eth0 root netem delay 100ms 10ms distribution normal
```

**\#在eth0口模拟丢包（丢包率为0.1％）**

```
tc qdisc change dev eth0 root netem loss 0.1%
```

**\#在eth0口模拟丢包（丢包率为0.1％－33.33％之间随机分布）**

```
tc qdisc change dev eth0 root netem loss 0.3% 33.33%
```

**\#在eth0口模拟数据包重传（重传率为1％）**

```
tc qdisc change dev eth0 root netem duplicate 1%
```

**\#在eth0口模拟数据失真\(数据能传过去，但数据错误）**

```
tc qdisc change dev eth0 root netem corrupt 0.1%
```

**\#在eth0口模拟重新请求数据包**

```
tc qdisc change dev eth0 root netem gap 5 delay 10ms 
备注：第五、十、十五个（每隔5个）包会立即发送，而其余的包会延时10ms再发送
```

```
tc qdisc change dev eth0 root netem delay 10ms reorder 25% 50% 
备注：25％-50％的数据包会被立即发送，其余的延时10ms再发送，这种可以模拟乱序的情况
```

**\#模拟传输速率**

```
# tc qdisc add dev eth0 root handle 1:0 netem delay 100ms
# tc qdisc add dev eth0 parent 1:1 handle 10: tbf rate 256kbit buffer 1600 limit 3000
```

**\#原理**

```
对互联网而言，一切都是数据包，操控网络实际上是在操控数据包，操控它如何产生，路由，传输，分片等等。
TC在数据包离开系统的时候进行控制，在IP层与网卡之间做手脚，实际上，负责将数据包传递到物理层的正是TC模块，
这意味着在系统内核中，TC作为数据包的调度者是一直运作的，甚至在你不想用他的时候，一般情况下，TC维持一个
先进先出的数据队列。

数据包入队的时候首先调用根队列规定的过滤器，根据过滤器定义的规则将数据包交给某个类，如果该类不是叶子类，
将会调用该类定义的过滤器进一步分类，若该类没有定义过滤器，就会交给包含他的队列规定的默认类来处理，若接
收到数据包的类是叶子类，数据包将进入到叶子类的队列规定里面排队，需要注意的是：过滤器只能将数据包交给某个
类，类再将数据包放入自己的队列规定进行排队，而不能直接交给某个队列规定。

接受包从数据接口进来后，经过流量限制，丢弃不合规定的数据包，然后输入多路分配器判断：若接受包的目的地是
本主机，那么将该包送给上册处理，否则需转发，将接受包交到转发块处理。转发块同时也接收本主机上层产生的包。
转发块通过查看路由表，决定所处理包的下一跳，然后对包排序以便将他们传送到输出接口。

Linux的TC主要是在输出接口排列时进行处理和实现的。
```



