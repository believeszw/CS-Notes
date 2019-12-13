# linux下使用TC模拟弱网络环境

## 模拟延迟传输简介

netem 与 tc: netem 是 Linux 2.6 及以上内核版本提供的一个网络模拟功能模块。该功能模块可以用来在性能良好的局域网中,模拟出复杂的互联网传输性能,诸如低带宽、传输延迟、丢包等等情 况。使用 Linux 2.6 (或以上) 版本内核的很多发行版 Linux 都开启了该内核功能,比如 Fedora、Ubuntu、Redhat、OpenSuse、CentOS、Debian 等等。 tc 是Linux 系统中的一个工具,全名为 traffic control(流量控制)。tc 可以用来控制 netem 的工作模式,也就是说,如果想使用 netem ,需要至少两个条件,一个是内核中的 netem 功能被包含,另一个是要有 tc。

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
