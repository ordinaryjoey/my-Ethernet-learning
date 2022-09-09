**NTP**: **Network Time Protocol** is synchronizing time to a server on the internet. It allows you to  synchronize devices in different rooms, buildings, cities, etc. because  they're all connecting to the same publicly accessible server. The  downside is its accuracy (millisecond range). 

**PTP: Precision Timing Protocol** is the general term for **IEEE-1588** which is only supported by some of  our hardware because only some of our hardware supports hardware-based  synchronization which is what you need to achieve the benefits  (nanosecond precision) of PTP. Furthermore, the synchronization is  limited to the local network. 

**TSN: Time Sensitive Networking** uses **PTP** (IEEE-802.1AS) to synchronize devices across a local network.  TSN also has features that allow for deterministic communication over  Ethernet, while still allowing other traffic over the network. 



CSR：Not sure so far

**Presentation Time in 1722:**

AVBTP defines a presentation time to achieve timing synchronization between talker and listener(s). The presentation time represents in nanoseconds the IEEE 802.1AS wall clock time when the data contained in the packet is to be presented to the AVBTP client at the listener(s).

The AVBTP presentation time is contained in the **avbtp_timestamp** field of AVBTP stream data frames.

The AVBTP presentation time may not be valid in every AVBTP packet. If an AVBTP packet contains a valid timestamp then the tv (Timestamp Valid) bit shall be set to one(1)

**CPT( current presentation time) in NXP:**

等于PTPSN的nanosecond部分。用于生成1722 frame里面所需的时间戳。

**one-step timestamp and two-step timestamp:** one-step 没有follow-up frame，这样对硬件的精确性有更高的要求。https://tekron.com/news/release/1-step-vs-2-step-ptp/



**Shapper**：整形器。整形的目的是为了让数据包发送的更规律，不至于短时间发送很多，然后很长时间不发送。

AVB使用**CBS：credit-based shapper**



TSN使用**TAS：time aware shapper**





**Enhancements to scheduled traffic (EST)**：  IEEE 802.1Qbv-2015



**TSN Technology: Ethernet Frame Preemption**：https://iebmedia.com/technology/tsn/tsn-technology-ethernet-frame-preemption/

This (for Ethernet) new technique solves the problem of a too long  network message which doesn’t fit in a cycle. Simply explained, it does  the following:

- Stop (preempt) the transmission of too long messages *before* the end of the guard band.
- Continue the transmission of the remainder of the message in the next cycle, following the express messages.

In Ethernet terminology, the preempted message is sent in multiple (>= 2) **fragments**. At the receiver, the fragments are assembled again to re-create (a copy of) the original network message.

![TSN Diagram](C:\Users\alex.liu\Desktop\Typora\pictures\guard band.png)



**Enery-Efficient-Ethernet(EEE)**：高效节能以太网，相关标准在IEEE 802.3az EEE在2010年9月制定完成。https://blog.csdn.net/wll1228/article/details/106364795



## 802.1Q 详细介绍 ##

### reference ###

1.https://blog.csdn.net/xiaohaijiejie/article/details/70208549 802.1Q VLAN 技术原理---理解PVID和VID

2.https://www.practicalnetworking.net/stand-alone/vlans/ 这个网站讲了很多网络相关的内容。

3.https://www.practicalnetworking.net/stand-alone/routing-between-vlans/

**交换机的核心思想：源地址学习，目的地址转发**

VLAN技术就是主要针对于交换机提出的，VLAN有两个主要的功能：

1.**A VLAN allows you to take one physical switch, and break it up into smaller *mini-switches*.**

2.**VLANs allow you to extend the smaller Virtual switches across multiple Physical switches**.

PVID？：当端口收到一个UNTAGED数据帧时，无法确定在哪个VLAN中进行交换，PVID定义了在这种情形下交换该帧的VLAN。从某种意义上讲，可以把PVID理解为端口的default VLAN。在支持VLAN的交换机中，每个端口都有一个PVID值，该值有一个缺省值，当然你也可以更改它。 

一个VLAN可以包含多个端口，而一个端口也可以属于多个VLAN。一个端口在一个VLAN中有不同的属性，TAG的添加和移除原则就是根据这个属性而定的。 
      TAGED：如果一个端口在一个VLAN中的属性是TAG的，那么，从该端口转发出去的数据帧就是TAGED。（当然，该数据帧是在该VLAN中交换的） 
      UNTAGED：如果一个端口在一个VLAN中的属性是UNTAG的，那么，从该端口转发出去的数据帧就是UNTAGED。（当然，该数据帧是在该VLAN中交换的）

**那它能一会tag，一会untag吗？** 每个MAC可能存在于多个VLAN域之中，但只在唯一的一个VLAN域中属于untag类型。如果不属于任何一个域，那么PVID就是它默认的VLAN域，且该MAC在该域内属于untag类型。

是否属于某一VLAN，以及在某一VLAN内是何属性，决定了网络节点如何接收和转发报文。具体见reference1.

某一个VLAN里的packet，一定无法传到其它VLAN里面吗？ 不是的。在switch里，不同的VLAN是相互独立的。但是packet经过router后，可以转到不同的VLAN switch中。**router有改变tag的功能。**

switch的功能是：The purpose of a Switch is to facilitate communication ***within*** networks.(IP)

router的功能是：to facilitate communication ***between*** networks.（reference 3）

如果一个不支持VLAN的设备要接入VLAN网络，那它需要接入一个被设置为**access**的switch port。 access类型的接口只拥有一个VLAN，且被设置为untaged。access port一般连接终端设备。

如果需要连接switch和switch，它们之间可能有多个VLAN网络，这时候可以将port设置为trunk。trunk port属于多个VLAN网络，可以使多个不同的VLAN各自建立起独立的逻辑连接关系，减少switch上用于连接其它VLAN switch的端口数量。

总的说，一个swtich上有两种端口，access port 连接终端，trunk port 连接其它switch或router。access port相当于单行道，trunk port相当于并行高速路。**离开Trunk port的packet，除了属于native VLAN的packet，都必须带有tag。离开access port 的packet，则一定没有tag.** native VLAN 是Trunk才有的。

下图为两个属于不同VLAN的节点之间数据包的传递。

![两个不同tag的设备如何传包](C:\Users\alex.liu\Desktop\两个不同tag的设备如何传包.gif)





## PTP详细介绍 ##

### reference ###

1. Understanding and Applying Precision Time Protocol



### precondition（Chapter 6.2 in 1588-2008） ###

1. PTP 网络没有cyclic forwarding
2. 可以接受有missed message, duplicated message, out-of-order message, 但是不要太常见
3. PTP通信需要多播。PTP通过多播来发现网络中的节点。如果无法多播，则需要有额外的方式来发现网络中的节点（17.5），并在通信时将单播复制给其他端口。
4. 由于网络的不对称性，时钟的精确度可能会降低。系统应该根据所需的精度，适当去配置。
5. 如果使用two-step clock， 那么general message 和 event message 在通过transparent clock传播时应保证路径相同。
6. 一个grandmaster clock 引申出的boundary clock 个数要小于255个？

### basic concepts ###

**two kinds of messages:**

1. Event message: consists of: Sync, Delay_Req, Pdelay_Req, Pdelay_Resp. Event massage 在发送和接收时都需要精确的时间戳
2. General message: consists of: Announce, Follow_Up, Delay_Resp, Pdelay_Resp_Follow_Up, Management, Signaling. General message 不需要精确的时间戳。

five device types:

1. ordinary clock: 提供同步时钟的节点（grandmaster clock） 或使用同步时钟并以此通信的节点（slave clock）。
2. boundary clock：一个作为boundary clock的device有一个slave port，用于接收同步时钟。同时还有若干个master port，用于向其它节点提供同步时钟。 boundary clock 可以用于扩展PTP网络的规模。
3. transparent clock：通常是个switch，将packet从ingress port route 到 egress port的同时会计算packet在其内部停留的时长，并在发出packet的时候将该packet对应的时间修正值填入packet。

PTP现在有三个版本的standard：1588-2002， 1588-2008， 1588-2019
还有相对应的802.1AS-2011 802.1AS-2021
PTP有V1和V2两个互不兼容的版本，分别定义于1588-2002， 1588-2008。一个网络中如果同时出现了只支持V1或V2的设备，那么它们之间的通信需要在boudary clock处进行，且该boundary clock所属的节点需要同时支持V1和V2，并拥有将两者相互转化的能力。见1588-2008 Chapter 18.
TLV：type，length，value

## AVB详细介绍 ##



### reference ###

1.https://zhuanlan.zhihu.com/p/265799490 AVB概述。解释了框架，和上下游的联系

2.

AVB 在ISO中的位置：

![image-20220614152842189](C:\Users\alex.liu\Desktop\Typora\pictures\AVB in ISO.png)

AVB协议上层为音视频传输协议**1722，简称AVTP**，AVTP协议主要用于封装音视频流，而AVB系统协议为AVTP提供基础架构，确保AVTP流的确定性传输。

AVTP报文如下。它需要带VLAN tag。VLAN Tag中的Priority对流控十分重要。

![image-20220614164831982](C:\Users\alex.liu\Desktop\Typora\pictures\AVB Packet.png)



### AVB由以下4个主要协议组成： ###

#### 1. AVB系统协议 ####

IEEE 802.1BA

#### 2. 流预留协议 ####

IEEE802.1Qat

#### 3. 时间同步协议 ####

IEEE802.1AS （1588的精简和改进）

#### 4. 交换机流整形协议 ####

IEEE802.1Qav
