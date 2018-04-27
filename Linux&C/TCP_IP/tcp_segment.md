# The TCP/IP 传输层


## TCP segment structure

- TCP 报文分段由头和数据文本组成。

     -------------------------------------------
     |                                         |
     |  TCP HEADER (20 bytes + optional part)  |
     |                                         |
     -------------------------------------------
     |                                         |
     |  DATA (65535-20-optional-20 bytes)      |
     |                                         |
     -------------------------------------------

- TCP头部至少20字节的固定格式信息，以及可选部分（总是一个整数乘以4字节）。
- 紧跟头部之后至少有长度为6535-20-20的可选（如果有的话）数据字节。
- 因此，任何一个TCP报文分段总共至少拥有65355-20（即，2的16次方减20）字节。实际上，这个数字通常被MTU（maximum transfer unit， 最大传输单元）严格的限制为1500字节。

    0                   1                   2                   3
    0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
   |          Source Port          |       Destination Port        |
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
   |                        Sequence Number                        |
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
   |                    Acknowledgment Number                      |
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
   |  Data |           |U|A|P|R|S|F|                               |
   | Offset| Reserved  |R|C|S|S|Y|I|            Window             |
   |       |           |G|K|H|T|N|N|                               |
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
   |           Checksum            |         Urgent Pointer        |
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
   |                    Options                    |    Padding    |
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
   |                             data                              |
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+

源端口（）：16比特
目的端口（）：16比特

- 端口是一个用于进程间通信的逻辑地址。在一台主机或者一个进程内可以提供多个端口。
- 小于256的熟知端口有：21，23，25，80，110
- 小于1024的端口号为操作系统保留，仅root权限。
- IP加端口组成了TSAP（Transport Service Access Point，传输服务访问点，有时也称为socket

### 序号

- 32比特
- 每个字节的数据都被编号

### 确认号

- 32比特
- 指定了除去发送者本身（并不是正确接收最后一个字节的）的下一个字节编号。
- 通常是接收者返回给发送者。
- 如果

### 头部长度

- URG：

### 16比特窗口

### 可选部分
- 


## 解码以太网帧

```
-------------------------------------------------------------
                         Ethernet header
-------------------------------------------------------------
00 A0 92 48 72 45 | dest. MAC address = 0:a0:92:48:72:45
00 00 0C 05 C3 58 | source MAC address = 0:0:c:5:c3:58
08 00             | network protocol = 0x0800 (IP)
-------------------------------------------------------------
                         IP header
-------------------------------------------------------------
4                 | IP version = 4
5                 | header length = 5 4-bytes words
00                | type of service = 0 (normal)
00 29             | length = 0x29 octets = 41 (dec.)
DB FB             | datagram identification
40 00             | don't fragment
FE                | TTL = 254
06                | transport protocol type = 6 (TCP)
7D CB             | header checksum
81 6E 1E 1A       | source IP address = 129.110.30.26
81 6E 02 11       | destination IP address = 129.110.2.17
-------------------------------------------------------------
                        TCP header
-------------------------------------------------------------
02 8B             | source port = 0x028b (651 dec.)
02 03             | dest. port = 0x0203 (515 dec., printer)
6A 86 7B 57       | source seqno = 1787198295 (dec.)
B6 B6 B0 20       | acknowledgment no = 3065425952 (dec.)
50                | header length = 5 words
10                | indicates an ACK
24 00             | window size = 0x2400 (9216 dec.)
15 89             | TCP checksum
00 00             | urgent pointer off
-------------------------------------------------------------
                         Data
-------------------------------------------------------------
02                | Data byte
54 41 4D 49 4C    | Padding to make a 46 byte IP datagram
-------------------------------------------------------------
D7 87 6C A4       | Ethernet checksum (Ethernet trailer)
-------------------------------------------------------------
```


## TCP滑动窗口

- TCP依赖滑动窗口进行流程和拥塞控制
    - 窗口是
    - 窗口大小：
- 发送者窗口：

 Sequence-number space
                  x       y             z
  |<-------A----->|<--B-->|<-----C----->|<---------D-------->|
                          |<----------->|
                          Sender's Window

A - 已被确认的旧序列号
B - 未经确认数据的序列号
C - 允许用在新的数据传输上的序列号
D - 未被允许的未来序列号

x = 第一个未被确认字节
y = 下一个将被发送的字节
z = x加上发送者窗口的长度


- 窗口总大小 = 2^32 -1 bytes = 4,294,967,295 bytes


接收者窗口:

  Receiver's Sequence-number space
                  x               y
  |<-------A----->|<------B------>|<----C---->|
                  |<------------->|
                  Receiver's Window

  A - old sequence numbers which have been acknowledged
  B - sequence numbers allowed for new reception
  C - future sequence numbers which are not yet allowed
 
  x = Next byte to receive
  y = Last byte that can be received
  y-x = Receiver's window size
  
- 问题：主机或许会接收适合窗口的包含序列号的旧报文分段
    - 被包裹
    - 主机崩溃和重连

```
```

## 总结

- TCP = Transport Control Protocol
- A reliable end-to-end byte stream over an unreliable internetwork.
- Independent of network architecture, topology, speed.
- Robust in the face of many kinds of failures.
- Defined in RFC's 793, 1122, 1323.
A machine that supports TCP must have a single "TCP entity" (like a transportation ministry in a state) present at its operating system kernel (Linux) or as a user process. This "entity" is also in charge of interfacing to the IP layer.
TCP somtimes mean a protocol (a set of rules), and sometimes it means a transport entity (a running computer process).
Two machines establish a TCP connection by creating (or using existing) connection end-points that are called sockets or ports. Each machine can have up to 65535 (2^16-1) open ports. So it is possible to have many connections between two machines (how many in principle?). One port can be involved in many connections (with different ports on the the other host, or with the same port on different hosts).
A TCP connection is a point-to-point full-duplex and byte-stream.
TCP receives data from the Application layer. It may send it immediately or buffer it until it collects a large amount to send at once. However, it is possible to force TCP to flush its buffers.
TCP breaks the data into segments (TCP packets). Each segment is shipped seperately from the others (and even may take a different route than others). Segments may arrive to their destination not in order (and even some of them may be lost). It's up to the TCP entity at the other end to reassemble, report missing segments, etc, and deliver the data to the receiving process.
After TCP sends a segment it maintains a timer for receipt of an acknowledgment from the other end of the connection
Every received segment is acknowledged
Timeout/retransmission is adaptive
Checksum on TCP pseudo-header
A bad segment is discarded without a NAK
duplicated segments are discarded
Sender times out and retransmits
Each byte has a sequence number. Used to reassemble segments in order Each sequence number must be acknowledged Acknowledgement of each octet: Impractical Initial sequence numbers should be assigned randomly to minimize problems with duplicate numbers from different connections
Duplicate segments are discarded by the receiving TCP IP may deliver duplicate datagrams
Flow control (end-to-end) (sliding windows algorithm) Ensures that a fast sender does not overrun a slow receiver
Congestion control (host-network interaction) Prevents too much data from being injected into the network
TCP avoids sending small packets by accumulating octets until a buffer is full or until a timer expires (default 2 ms).