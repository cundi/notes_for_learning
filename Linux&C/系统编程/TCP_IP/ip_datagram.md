# IP

IP报文由头和文本数据组成。

```txt
     -------------------------------------------
     |                                         |
     |               IP HEADER                 |
     |                                         |
     -------------------------------------------
     |                                         |
     |               DATA                      |
     |                                         |
     -------------------------------------------
```

IPv4头部大小：

1. 20字节固定部分
2. 以及可选的变长部分

```txt
0                     1                   2                   3
    0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
   |Version|  IHL  |Type of Service|          Total Length         |
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
   |         Identification        |Flags|      Fragment Offset    |
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
   |  Time to Live |    Protocol   |         Header Checksum       |
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
   |                       Source Address                          |
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
   |                    Destination Address                        |
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
   |                    Options                    |    Padding    |
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
```

版本(Version, 4bit):为4代表ipv4, 为6代表ipv6

报头长度(Header Length, 4bit):一般为5, 代表IP首部一共占用20个字节. (4*5)

服务类别(Type Of Service, 8bit):

报文长度(Length, 16bit):  IP首部+数据部分的总长度, 一般再加上以太头长度即为数据包的长度.

标识(Identification, 16bit): 用于在IP层对数据包进行分段的时候,标识数据包.

标志(IP Flags, 3bit):

Bit 0: reserved, must be zero
Bit 1: (DF) 0 = May Fragment, 1 = Don’t Fragment.
Bit 2: (MF) 0 = Last Fragment, 1 = More Fragments.

0x80---保留字段, 0x40---不分段, 0x20---还有更多的分段

分段偏移(Fragment Offset, 13bit): 本数据包的数据在分段中的偏移(需要再乘以8).

生存时间(Time To Live, 8bit): 本数据包的TTL.

用户协议(Protocol, 8bit): 1---icmp, 2---igmp, 6---tcp, 17---udp, 89---ospf.

报头校验和(Header Checksum, 16bit): IP首部的校验和.

源IP地址(Source IP, 32bit): 源IP地址.

目的IP地址(Destination IP, 32bit): 目的IP地址.



## 其他注意事项

IP头的中的

## 分片

问题点:

1. IP层的分片和重组分别发生在什么时候?
IP分片发生在IP层，不仅源端主机会进行分片，中间的路由器也有可能分片，因为不同的网络的MTU是不一样的，如果传输路径上的某个网络的MTU比源端网络的MTU要小，路由器就可能对IP数据报再次进行分片。而分片数据的重组只会发生在目的端的IP层。

2. IP分片和TCP分段有什么区别?
IP分片的原因是MTU, TCP分段的原因是MSS.


3. IP协议对分片进行重组的时候, 是不是也要像TCP协议一样, 启动一个定时器, 等到3片都到齐?
启用定时器.但只是一个针对分片重组的定时器。如果定时器超时还没有收到完整的数据包,则直接将数据包丢弃.

注: 关于这个问题, 今天专门去分析了协议栈的源代码. lwIP中并没有在ip层启动定时器, 只是简单的回收最旧的ip分片组. linux内核(2.6.18)的协议栈中确实有在ip层对分片做了定时器处理. 注册了一个timer, 当过期的时候, 回调函数会调用从而收受内存. 在linux内核协议栈中, ip分片的默认超时时间为600秒. (ipfragment.c中有源代码ip4_frags.secret_interval = 10 * 60 * HZ;)

最近刚好在关注TI5, secret站队很强大. 不知道内核源代码开发者为什么要用secret_interval 这样的名字~~


4. IP层会不会对错误的数据包进行重传?
IP层不会对丢弃或者错误的数据包进行重传,但是上层协议可能会提供重传机制(tcp等).


5. 为什么要避免IP分片
IP层是没有超时重传机制,一旦分片中有一个数据包丢失就只能由上层协议重传整个数据包,效率低下. 

使用UDP开发的时候，我们需要在应用层去限制每个包的大小，一般不要超过1472字节(即以太网MTU—UDP首部—IP首部),来避免IP层的分片。TCP程序由于有MSS, TCP协议本身会避免IP分片,则不用考虑这些.
 

### 分片攻击

　　这种攻击的原理是：在IP的分片包中，所有的分片包用一个分片偏移字段标志分片包的顺序，但是，只有第一个分片包含有TCP端口号的信息。当IP分片包通过分组过滤防火墙时，防火墙只根据第一个分片包的Tcp信息判断是否允许通过，而其他后续的分片不作防火墙检测，直接让它们通过。

这样，攻击者就可以通过先发送第一个合法的IP分片，骗过防火墙的检测，接着封装了恶意数据的后续分片包就可以直接穿透防火墙，直接到达内部网络主机，从而威胁网络和主机的安全。

------------------------------

### 邪恶比特：

将IP头中本应该设置为比特 0 的IP flag字段，出于恶意目的而改为了 1.

比如，执行恶意的端口扫描，就需要设置邪恶比特。

Scapy：强大的包制作和操控工具。

使用Scapy制作包含邪恶比特的TCP Syn 包：

```
# myevilpacket.py
#!/usr/bin/python
from scapy.all import *

ip=IP(src="192.168.1.121", dst="192.168.1.2", flags=4, frag=0)
tcpsyn=TCP(sport=1500, dport=80, flags="S", seq=4096)
send(ip/tcpsyn)
```

to briefly mention the usage of flags=4. As you could see in the IPv4 header image the IP Flags field uses 3 bits.  These 3 bits are the highest bits in the 6th byte of the IP Header.  To set the Evil bit we need to set the value to 100 in binary or 4 in hex/integer.

## 相关趣闻

Apart of the Evil bit one that is really hilarious is the RFC 5841 which proposes a TCP option to denote packet mood. For example happy packets which are happy because they received their ACK return packet within less than 10ms. Or the Sad Packets which are sad because they faced retransmission rates greater than 20% of all packets sent in a session.

https://en.wikipedia.org/wiki/April_Fools'_Day_Request_for_Comments


