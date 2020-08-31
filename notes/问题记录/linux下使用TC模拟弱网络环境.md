# linux下使用TC模拟弱网络环境

## 原理
TC 通过建立处理数据包队列，并定义队列中数据包被发送的方式，从而实现进行流量控制。TC 模拟实现流量控制功能使用的队列分为两类：


* **01.无类队列** 规定是对进入网络设备(网卡) 的数据流不加区分统一对待的队列规定。

  * 使用无类队列规定形成的队列能够接受数据包以及重新编排、延迟或丢弃数据包。这类队列规定形成的队列可以对整个网络设备(网卡) 的流量进行整形， 但不能细分各种情况。

  * 常用的无类队列规定主要有 pfifo_fast (先进现出) 、TBF ( 令牌桶过滤器) 、SFQ(随机公平队列) 、ID (前向随机丢包)等等。这类队列规定使用的流量整形手段主要是排序、限速和丢包。


* **02.分类队列** 规定是对进入网络设备的数据包根据不同的需求以分类的方式区分对待的队列规定。

  * 数据包进入一个分类的队列后， 它就需要被送到某一个类中， 也就是说需要对数据包做分类处理。对数据包进行分类的工具是过滤器，过滤器会返回一个决定，队列规定就根据这个决定把数据包送入相应的类进行排队。

  * 每个子类都可以再次使用它们的过滤器进行进一步的分类。直到不需要进一步分类时， 数据包才进入该类包含的队列排队。

  * 除了能够包含其它队列规定之外， 绝大多数分类的队列规定还能够对流量进行整形。 这对于需要同时进行调度( 如使用 SFQ ) 和流量控制的场合非常有用。

* **03.基本原理**

  * 接收包从输入接口（Input Interface）进来后，经过流量限制（Ingress Policing）丢弃不符合规定的数据包，由输入多路分配器（Input De-Multiplexing）进行判断选择......

  * 如果接收包的目的是本主机，那么将该包送给上层处理；否则需要进行转发，将接收包交到转发块（Forwarding Block）处理。转发块同时也接收本主机上层（TCP、UDP等）产生的包。转发块通过查看路由表，决定所处理包的下一跳。然后，对包进行排列以便将它们传送到输出接口（Output Interface）。

  * 一般我们只能限制网卡发送的数据包，不能限制网卡接收的数据包，所以我们可以通过改变发送次序来控制传输速率。

  * Linux 流量控制主要是在输出接口排列时进行处理和实现的。

## TC规则
### 1. 流量控制
> 流量控制包括以下4种方式

* SHAPING(限制)\
当流量被限制，它的传输速率就被控制在某个值以下。限制值可以大大小于有效带宽，这样可以平滑突发数据流量，使网络更为稳定。shaping（限制）只适用于向外的流量。

* SCHEDULING(调度)\
通过调度数据包的传输，可以在带宽范围内，按照优先级分配带宽。SCHEDULING(调度)也只适于向外的流量。

* POLICING(策略)\
SHAPING 用于处理向外的流量，而 POLICIING(策略)用于处理接收到的数据。

* DROPPING(丢弃)\
如果流量超过某个设定的带宽，就丢弃数据包，不管是向内还是向外。

### 2. 流量控制处理对象
> 流量的处理由三种对象控制，它们是：qdisc(排队规则)、class(类别)和 filter(过滤器)。

#### 2.1 QDisc

* QDisc(排队规则)是 queueing discipline 的简写，它是理解流量控制(traffic control)的基础。

* 无论何时，内核如果需要通过某个网络接口发送数据包，它都需要按照为这个接口配置的 qdisc(排队规则)把数据包加入队列。然后，内核会尽可能多地从 qdisc 里面取出数据包，把它们交给网络适配器驱动模块。

* 最简单的 QDisc 是 pfifo 它不对进入的数据包做任何的处理，数据包采用先入先出的方式通过队列。不过，它会保存网络接口一时无法处理的数据包。

* QDISC 的分为 CLASSLESS QDisc 和 CLASSFUL QDISC 类别,即不可分类 QDisc 和可分类 QDisc

#### 2.2 CLASSLESS QDisc

* 无类别 QDISC 包括

  * **[p|b] fifo**\
  使用最简单的 qdisc，纯粹的先进先出。只有一个参数：limit，用来设置队列的长度， pfifo 是以数据包的个数为单位；bfifo 是以字节数为单位。

  * **pfifo_fast**\
  在编译内核时，如果打开了高级路由器(Advanced Router)编译选项，pfifo_fast 就是系统的标准 QDISC 。它的队列包括三个波段(band)。在每个波段里面，使用先进先出规则。而三个波段(band)的优先级也不相同，band 0 的优先级最高，band 2 的最低。如果 band 里面有数据包，系统就不会处理 band 1 里面的数据包，band 1 和 band 2 之间也是一样。数据包是按照服务类型(Type of Service,TOS)被分配多三个波段(band)里面的。

  * **red**\
  red 是 Random Early Detection(随机早期探测)的简写。如果使用这种 QDISC，当带宽的占用接近于规定的带宽时，系统会随机地丢弃一些数据包。它非常适合高带宽应用。

  * **sfq**\
  sfq 是 Stochastic Fairness Queueing 的简写。它按照会话(session--对应于每个 TCP 连接或者 UDP 流)为流量进行排序，然后循环发送每个会话的数据包。

  * **tbf**\
  tbf 是 Token Bucket Filter 的简写，适合于把流速降低到某个值。

* 无类别 QDisc 的配置

  * 如果没有可分类 QDisc，不可分类 QDisc 只能附属于设备的根,使用命令 tc qdisc add dev DEV root QDISC QDISC-PARAMETERS

  * 要删除一个不可分类 QDisc，需要使用命令 tc qdisc del dev DEV root

  * 一个网络接口上如果没有设置 QDisc，pfifo_fast 就作为缺省的 QDisc。

#### 2.3 CLASSFUL QDISC

可分类 QDISC 包括：

* CBQ

  * CBQ 是 Class Based Queueing(基于类别排队)的缩写。

  * 它实现了一个丰富的连接共享类别结构，既有限制(shaping)带宽的能力，也具有带宽优先级管理的能力。

  * 带宽限制是通过计算连接的空闲时间完成的。

  * 空闲时间的计算标准是数据包离队事件的频率和下层连接(数据链路层)的带宽。

* HTB

  * HTB 是 Hierarchy Token Bucket 的缩写。

  * 通过在实践基础上的改进，它实现了一个丰富的连接共享类别体系。使用 HTB 可以很容易地保证每个类别的带宽，虽然它也允许特定的类可以突破带宽上限，占用别的类的带宽。

  * HTB 可以通过 TBF(Token Bucket Filter)实现带宽限制，也能够划分类别的优先

* PRIO

  * PRIO QDisc 不能限制带宽，因为属于不同类别的数据包是顺序离队的。

  * 使用 PRIO QDisc 可以很容易对流量进行优先级管理，只有属于高优先级类别的数据包全部发送完毕，才会发送属于低优先级类别的数据包。

  * 为了方便管理，需要使用 iptables 或者 ipchains 处理数据包的服务类型(Type Of Service,ToS)。

### 2. 操作原理

* 类(Class)组成一个树，每个类都只有一个父类，而一个类可以有多个子类。某些 QDisc(例如：CBQ 和 HTB)允许在运行时动态添加类，而其它的 QDisc(例如：PRIO)不允许动态建立类。允许动态添加类的 QDisc 可以有零个或者多个子类，由它们为数据包排队。

* 此外，每个类都有一个叶子 QDisc ，默认情况下，这个叶子 QDisc 使用 pfifo 的方式排队，我们也可以使用其它类型的 QDisc 代替这个默认的 QDisc 。而且，这个叶子 QDisc 有可以分类，不过每个子类只能有一个叶子 QDisc 。当一个数据包进入一个分类 QDisc ，它会被归入某个子类。

* 如果过滤器附属于一个类，相关的指令就会对它们进行查询。过滤器能够匹配数据包头所有的域，也可以匹配由 ipchains 或者 iptables 做的标记。

* 树的每个节点都可以有自己的过滤器，但是高层的过滤器也可以直接用于其子类。如果数据包没有被成功归类，就会被排到这个类的叶子 QDisc 的队中。相关细节在各个 QDisc 的手册页中

### 3. 命令规则

>所有的 QDisc、类和过滤器都有 ID。ID 可以手工设置，也可以有内核自动分配。ID 由一个主序列号和一个从序列号组成，两个数字用一个冒号分开。

* QDISC，一个 QDisc 会被分配一个主序列号，叫做句柄(handle)，然后把从序列号作为类的命名空间。句柄采用像 10: 一样的表达方式。习惯上，需要为有子类的 QDisc 显式地分配一个句柄。

* 类(CLASS)，在同一个 QDisc 里面的类分享这个 QDisc 的主序列号，但是每个类都有自己的从序列号，叫做类识别符(classid)。类识别符只与父 QDisc 有关，和父类无关。类的命名习惯和 QDisc 的相同。

* 过滤器(FILTER)，过滤器的 ID 有三部分，只有在对过滤器进行散列组织才会用到。详情请参考 tc-filters 手册页。

## TC命令

> tc 可以使用以下命令对 QDisc 、类和过滤器进行操作：

* **add:** 在一个节点里加入一个 QDisc 、类或者过滤器。添加时，需要传递一个祖先作为参数，传递参数时既可以使用 ID 也可以直接传递设备的根。如果要建立一个 QDisc 或者过滤器，可以使用句柄(handle)来命名；如果要建立一个类，可以使用类识别符(classid)来命名。

* **remove:** 删除有某个句柄(handle)指定的 QDisc ，根 QDisc(root)也可以删除。被删除 QDisc 上的所有子类以及附属于各个类的过滤器都会被自动删除。

* **change:** 以替代的方式修改某些条目。除了句柄(handle)和祖先不能修改以外，change 命令的语法和 add 命令相同。

* **replace:** 对一个现有节点进行近于原子操作的删除／添加。如果节点不存在，这个命令就会建立节点。

* **link:** 只适用于 DQisc，替代一个现有的节点。

```shell
tc qdisc [ add | change | replace | link ] dev DEV [ parent qdisc-id | root ] [ handle qdisc-id ] qdisc [ qdisc specific parameters ]

tc class [ add | change | replace ] dev DEV parent qdisc-id [ classid class-id ] qdisc [ qdisc specific parameters ]

tc filter [ add | change | replace ] dev DEV [ parent qdisc-id | root ] protocol protocol prio priority filtertype [ filtertype specific parameters ] flowid flow-id

tc [-s | -d ] qdisc show [ dev DEV ]

tc [-s | -d ] class show dev DEV tc filter show dev DEV
```

## 具体操作

> Linux 流量控制主要分为建立队列、建立分类和建立过滤器三个方面。

### 1. 基本步骤

* 针对网络物理设备（如以太网卡 eth0）绑定一个队列 QDisc；

* 在该队列上建立分类 class；

* 为每一分类建立一个基于路由的过滤器 filter；

* 最后与过滤器相配合，建立特定的路由表。

### 2. 模拟实例

流量控制器上的以太网卡(eth0) 的 IP 地址为 192.168.1.66，在其上建立一个 CBQ 队列。假设包的平均大小为 1000 字节，包间隔发送单元的大小为 8 字节，可接收冲突的发送最长包数目为 20 字节。

假如有三种类型的流量需要控制:

* 是发往主机 1 的，其 IP 地址为 192.168.1.24。其流量带宽控制在 8Mbit，优先级为 2；

* 是发往主机 2 的，其 IP 地址为 192.168.1.30。其流量带宽控制在 1Mbit，优先级为 1；

* 是发往子网 1 的，其子网号为 192.168.1.0，子网掩码为 255.255.255.0。流量带宽控制在 1Mbit，优先级为 6。

#### 2.1 建立队列

一般情况下，针对一个网卡只需建立一个队列。

将一个 cbq 队列绑定到网络物理设备 eth0 上，其编号为 1:0；网络物理设备 eth0 的实际带宽为 10 Mbit，包的平均大小为 1000 字节；包间隔发送单元的大小为 8 字节，最小传输包大小为 64 字节。

```shell
tc qdisc add dev eth0 root handle 1: cbq bandwidth 10Mbit avpkt 1000 cell 8 mpu 64
```

#### 2.2 建立分类

分类建立在队列之上。

一般情况下，针对一个队列需建立一个根分类，然后再在其上建立子分类。对于分类，按其分类的编号顺序起作用，编号小的优先；一旦符合某个分类匹配规则，通过该分类发送数据包，则其后的分类不再起作用。

* 创建根分类 1:1 ；分配带宽为 10Mbit ，优先级别为 8 。

  * `tc class add dev eth0 parent 1:0 classid 1:1 cbq bandwidth 10Mbit rate 10Mbit maxburst 20 allot 1514 prio 8 avpkt 1000 cell 8 weight 1Mbit`

  * 该队列的最大可用带宽为 10Mbit ，实际分配的带宽为 10Mbit ，可接收冲突的发送最长包数目为 20 字节；最大传输单元加 MAC 头的大小为 1514 字节，优先级别为 8 ，包的平均大小为 1000 字节，包间隔发送单元的大小为 8 字节，相应于实际带宽的加权速率为 1Mbit 。

* 创建分类 1:2 ，其父分类为 1:1 ，分配带宽为 8Mbit ，优先级别为 2 。

  * `tc class add dev eth0 parent 1:0 classid 1:1 cbq bandwidth 10Mbit rate 10Mbit maxburst 20 allot 1514 prio 8 avpkt 1000 cell 8 weight 1Mbit`

  * 该队列的最大可用带宽为 10Mbit ，实际分配的带宽为 8Mbit ，可接收冲突的发送最长包数目为 20 字节；最大传输单元加 MAC 头的大小为 1514 字节，优先级别为 1 ，包的平均大小为 1000 字节，包间隔发送单元的大小为 8 字节，相应于实际带宽的加权速率为 800Kbit ，分类的分离点为 1:0 ，且不可借用未使用带宽。

* 创建分类 1:3 ，其父分类为 1:1 ，分配带宽为 1Mbit ，优先级别为 1 。

  * `tc class add dev eth0 parent 1:1 classid 1:3 cbq bandwidth 10Mbit rate 1Mbit maxburst 20 allot 1514 prio 1 avpkt 1000 cell 8 weight 100Kbit split 1:0`

  * 该队列的最大可用带宽为 10Mbit ，实际分配的带宽为 1Mbit ，可接收冲突的发送最长包数目为 20 字节；最大传输单元加 MAC 头的大小为 1514 字节，优先级别为 2 ，包的平均大小为 1000 字节，包间隔发送单元的大小为 8 字节，相应于实际带宽的加权速率为 100Kbit ，分类的分离点为 1:0 。

* 创建分类 1:4 ，其父分类为 1:1 ，分配带宽为 1Mbit ，优先级别为 6 。

  * `tc class add dev eth0 parent 1:1 classid 1:4 cbq bandwidth 10Mbit rate 1Mbit maxburst 20 allot 1514 prio 6 avpkt 1000 cell 8 weight 100Kbit split 1:0`

  * 该队列的最大可用带宽为 10Mbit ，实际分配的带宽为 1Mbit ，可接收冲突的发送最长包数目为 20 字节；最大传输单元加 MAC 头的大小为 1514 字节，优先级别为 6 ，包的平均大小为 1000 字节，包间隔发送单元的大小为 8 字节，相应于实际带宽的加权速率为 100Kbit ，分类的分离点为 1:0 。

#### 2.3. 建立过滤器

过滤器主要服务于分类。

一般只需针对根分类提供一个过滤器，然后为每个子分类提供路由映射。

* 应用路由分类器到 cbq 队列的根，父分类编号为 1:0 ；过滤协议为 ip ，优先级别为 100 ，过滤器为基于路由表。
```shell
tc filter add dev eth0 parent 1:0 protocol ip prio 100 route
```

* 建立路由映射分类 1:2, 1:3, 1:4
```shell
tc filter add dev eth0 parent 1:0 protocol ip prio 100 route to 2 flowid 1:2
tc filter add dev eth0 parent 1:0 protocol ip prio 100 route to 3 flowid 1:3
tc filter add dev eth0 parent 1:0 protocol ip prio 100 route to 4 flowid 1:4
```

#### 2.4 建立路由

该路由是与前面所建立的路由映射一一对应。

* 发往主机 192.168.1.24 的数据包通过分类 2 转发(分类 2 的速率 8Mbit)
```shell
ip route add 192.168.1.24 dev eth0 via 192.168.1.66 realm 2
```

* 发往主机 192.168.1.30 的数据包通过分类 3 转发(分类 3 的速率 1Mbit)
```shell
ip route add 192.168.1.24 dev eth0 via 192.168.1.66 realm 2
```

* 发往子网 192.168.1.0/24 的数据包通过分类 4 转发(分类 4 的速率 1Mbit)
```shell
ip route add 192.168.1.0/24 dev eth0 via 192.168.1.66 realm 4
```

**注：** 一般对于流量控制器所直接连接的网段建议使用 IP 主机地址流量控制限制，不要使用子网流量控制限制。如一定需要对直连子网使用子网流量控制限制，则在建立该子网的路由映射前，需将原先由系统建立的路由删除，才可完成相应步骤。

#### 2.5 监视

主要包括对现有队列、分类、过滤器和路由的状况进行监视。

* 显示队列的状况
```shell
# 简单显示指定设备(这里为eth0)的队列状况
tc qdisc ls dev eth0
# 详细显示指定设备(这里为eth0)的队列状况
tc -s qdisc ls dev eth0
```

* 显示分类的状况
```shell
# 简单显示指定设备(这里为eth0)的分类状况
tc class ls dev eth0
# 详细显示指定设备(这里为eth0)的分类状况
tc -s class ls dev eth0
```

* 显示过滤器的状况
```shell
tc -s filter ls dev eth0
```

* 显示现有路由的状况
```shell
ip route
```

#### 2.6 维护

主要包括对队列、分类、过滤器和路由的增添、修改和删除。

增添动作一般依照"队列->分类->过滤器->路由"的顺序进行；修改动作则没有什么要求；删除则依照"路由->过滤器->分类->队列"的顺序进行。

* 队列的维护\
一般对于一台流量控制器来说，出厂时针对每个以太网卡均已配置好一个队列了，通常情况下对队列无需进行增添、修改和删除动作了。

* 分类的维护

  * 增添，增添动作通过 tc class add 命令实现

  * 修改，修改动作通过 tc class change 命令实现
  ```shell
  tc class change dev eth0 parent 1:1 classid 1:2 cbq bandwidth 10Mbit rate 7Mbit maxburst 20 allot 1514 prio 2 avpkt 1000 cell 8 weight 700Kbit split 1:0 bounded
  ```
  对于bounded命令应慎用，一旦添加后就进行修改，只可通过删除后再添加来实现。

  * 删除，删除动作只在该分类没有工作前才可进行，一旦通过该分类发送过数据，则无法删除它了。因此，需要通过shell文件方式来修改，通过重新启动来完成删除动作。

* 过滤器的维护

  * 增添，增添动作通过 tc filter add 命令实现

  * 修改，修改动作通过 tc filter change 命令实现
  ```shell
  tc filter change dev eth0 parent 1:0 protocol ip prio 100 route to 10 flowid 1:8
  ```

  * 删除，删除动作通过 tc filter del 命令实现
  ```shell
  tc filter change dev eth0 parent 1:0 protocol ip prio 100 route to 10 flowid 1:8
  ```

* 与过滤器一一映射路由的维护

  * 增添，增添动作通过 ip route add 命令实现，如前面所示。

  * 修改，修改动作通过 ip route change 命令实现
  ```shell
  ip route change 192.168.1.30 dev eth0 via 192.168.1.66 realm 8
  ```

  * 删除，删除动作通过 ip route del 命令实现
  ```shell
  ip route del 192.168.1.30 dev eth0 via 192.168.1.66 realm 8
  ip route del 192.168.1.30 dev eth0 via 192.168.1.66 realm 8
  ```

## 模拟延迟传输简介

netem 与 tc: netem 是 Linux 2.6 及以上内核版本提供的一个网络模拟功能模块。该功能模块可以用来在性能良好的局域网中,模拟出复杂的互联网传输性能,诸如低带宽、传输延迟、丢包等等情况。使用 Linux 2.6 (或以上) 版本内核的很多发行版 Linux 都开启了该内核功能,比如 Fedora、Ubuntu、Redhat、OpenSuse、CentOS、Debian 等等。 tc 是Linux 系统中的一个工具,全名为 traffic control(流量控制)。tc 可以用来控制 netem 的工作模式,也就是说,如果想使用 netem ,需要至少两个条件,一个是内核中的 netem 功能被包含,另一个是要有 tc。

>需要注意的是:本文介绍的流控只能控制发包动作,不能控制收包动作,同时,它直接对物理接口生效,如果控制了物理的 eth0,那么逻辑网卡(比如 eth0:1)也会受到影响,反之,如果您在逻辑网卡上做控制,该控制可能是无效的。(注:虚拟机中的多个网卡可以在虚拟机中视为多个物理网卡)。

```shell
tc qdisc add dev eth0 root netem delay 100ms
# 该命令将 eth0 网卡的传输设置为延迟 100 毫秒发送。
# 更真实的情况下,延迟值不会这么精确,会有一定的波动,我们可以用下面的情况来模拟出
```

* 带有波动性的延迟值:

```shell
tc qdisc add dev eth0 root netem delay 100ms 10ms
# 该命令将 eth0 网卡的传输设置为延迟 100ms ± 10ms (90 ~ 110 ms 之间的任意值)发送。
```

还可以更进一步加强这种波动的随机性:

```shell
tc qdisc add dev eth0 root netem delay 100ms 10ms 30%
# 该命令将 eth0 网卡的传输设置为 100ms ,同时,大约有 30% 的包会延迟 ± 10ms 发送。示例:现在 ping 一下 216 机器:
# 可以看出数据明显的波动性。
```

* 模拟网络丢包:

```shell
tc qdisc add dev eth0 root netem loss 1%
# 该命令将 eth0 网卡的传输设置为随机丢掉 1% 的数据包。示例:在 216 上执行
```

```shell
tc qdisc add dev eth0 root netem loss 10%
# 显示 16 个包只有 13 个收到了。也可以设置丢包的成功率:
```

```shell
tc qdisc add dev eth0 root netem loss 1% 30%
# 该命令将 eth0 网卡的传输设置为随机丢掉 1% 的数据包,成功率为 30% 。
```

* 删除网卡上面的相关配置:将之前命令中的 add 改为 del 即可删除配置:

```shell
tc qdisc del dev eth0 root netem #自己加的配置
# 该命令将 删除 eth0 网卡的相关传输配置
```

* 模拟包重复:

```shell
tc qdisc add dev eth0 root netem duplicate 1%
# 该命令将 eth0 网卡的传输设置为随机产生 1% 的重复数据包 。6 模拟数据包损坏:
```

```shell
tc qdisc add dev eth0 root netem corrupt 0.2%
# 该命令将 eth0 网卡的传输设置为随机产生 0.2% 的损坏的数据包。 (内核版本需在 2.6.16 以上)
```

* 模拟数据包乱序:

```shell
tc qdisc change dev eth0 root netem delay 10ms reorder 25% 50%
# 该命令将 eth0 网卡的传输设置为:有 25% 的数据包(50%相关)会被立即发送
# 其他的延迟 10 秒。

# 新版本中,如下命令也会在一定程度上打乱发包的次序:
tc qdisc add dev eth0 root netem delay 100ms 10ms
```

* 查看已经配置的网络条件:

```shell
tc qdisc show dev eth0
# 该命令将 查看并显示 eth0 网卡的相关传输配置9
```

```shell
eg:
tc qdisc add dev ens32 root netem delay 2000ms 100ms 20%
tc qdisc del dev ens32 root netem delay 2000ms 100ms 20%
tc qdisc del dev ens32 root
```
