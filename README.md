# my-Ethernet-learning
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

![TSN Diagram](https://iebmedia.com/wp-content/uploads/2021/05/6.png)



**Enery-Efficient-Ethernet(EEE)**：高效节能以太网，相关标准在IEEE 802.3az EEE在2010年9月制定完成。https://blog.csdn.net/wll1228/article/details/106364795



## 802.1Q 详细介绍 ##

### reference ###

1.https://blog.csdn.net/xiaohaijiejie/article/details/70208549 802.1Q VLAN 技术原理---理解PVID和VID

**交换机的核心思想：源地址学习，目的地址转发**

VLAN技术就是主要针对于交换机提出的

PVID？：当端口收到一个UNTAGED数据帧时，无法确定在哪个VLAN中进行交换，PVID定义了在这种情形下交换该帧的VLAN。从某种意义上讲，可以把PVID理解为端口的default VLAN。在支持VLAN的交换机中，每个端口都有一个PVID值，该值有一个缺省值，当然你也可以更改它。 

一个VLAN可以包含多个端口，而一个端口也可以属于多个VLAN。一个端口在一个VLAN中有不同的属性，TAG的添加和移除原则就是根据这个属性而定的。 
      TAGED：如果一个端口在一个VLAN中的属性是TAG的，那么，从该端口转发出去的数据帧就是TAGED。（当然，该数据帧是在该VLAN中交换的） 
      UNTAGED：如果一个端口在一个VLAN中的属性是UNTAG的，那么，从该端口转发出去的数据帧就是UNTAGED。（当然，该数据帧是在该VLAN中交换的）

**那它能一会tag，一会untag吗？** 每个MAC可能存在于多个VLAN域之中，但只在唯一的一个VLAN域中属于untag类型。如果不属于任何一个域，那么PVID就是它默认的VLAN域，且该MAC在该域内属于untag类型。

是否属于某一VLAN，以及在某一VLAN内是何属性，决定了网络节点如何接收和转发报文。具体见reference1.

## AVB详细介绍 ##

### reference ###

1.https://zhuanlan.zhihu.com/p/265799490 AVB概述。解释了框架，和上下游的联系

2.

AVB 在ISO中的位置：

![image-20220614152842189](C:\Users\alex.liu\AppData\Roaming\Typora\typora-user-images\image-20220614152842189.png)

AVB协议上层为音视频传输协议**1722，简称AVTP**，AVTP协议主要用于封装音视频流，而AVB系统协议为AVTP提供基础架构，确保AVTP流的确定性传输。

AVTP报文如下。它需要带VLAN tag。VLAN Tag中的Priority对流控十分重要。

![image-20220614164831982](C:\Users\alex.liu\AppData\Roaming\Typora\typora-user-images\image-20220614164831982.png)



### AVB由以下4个主要协议组成： ###

#### 1. AVB系统协议 ####

IEEE 802.1BA

#### 2. 流预留协议 ####

IEEE802.1Qat

#### 3. 时间同步协议 ####

IEEE802.1AS （1588的精简和改进）

#### 4. 交换机流整形协议 ####

IEEE802.1Qav
