---
title: 多重NAT网络下的BT下载
published: 2025-09-19
description: ''
image: ''
tags: [Bittorrent, VPN, Proxy]
category: '教程'
draft: false 
lang: ''
---

# 前言
在国内网络资源日益收紧, 各大运营商对个人用户PCDN严打的环境下(其实是对所有高上传的负价值用户打击), 现在的BT下载可以说是一潭死水, 在不采取任何优化措施的情况下, 打开一个种子基本都没什么速度, 就算是热门种子, 也有一堆迅雷或者伪装成其他客户端的吸血鬼, 属实难绷. 另外, 阅前提醒:  本文方法不适用PT

# 软件选择
Windows下我一般用`Bitcomet`或者`qBittorrent Enhanced Edition` (下文称为QBEE), Bitcomet的长效种子有一定用, 但不知为什么只有开始下载时能连上, 下到后面就连不上了, 可能是我网络问题? 总之还是不如更好的网络环境来的实在

Linux下只用QBEE, 没别的, 就为了跨平台webui

另外, 使用`PeerBanHelper` + BTN网络的方式来阻止吸血鬼

## QBEE调优
`连接` 下的上传窗口建议根据自己的上传带宽来调, 除非你的带宽异常小(<=5Mbps), 一般建议每Torrent设置8-10个上传窗口(30M左右的上传速度), 代表单个任务上传流量分配至8个用户左右, 这个值不能太大, 否则会占用过多的CPU资源

`每Torrent最大连接数`这个不用太大, 我设的64, 再多没意义, 毕竟正常下载过程中你的下载速度主要由一小部分用户提供, 别的用户每人一般贡献10KB/s的流量. 这个值过大可能会导致你的网络变卡, 因为运营商家宽的连接数不会太多

另外`Peer连接协议`保持默认的`TCP+uTP`即可, 除非你的环境存在严重的UDP限速

`Bittorrent`下的加密选项推荐开启, 最好是强制加密, 一定程度上规避某些运营商的限速

## QBEE代理相关
[Ref](https://cherr.cc/qb_proxy) <br>
`通过代理查找主机名`: 是否通过代理连接Tracker <br>
`对 BitTorrent 目的使用代理`: 控制 qBittorrent 上报给 tracker 的 IP, 选择Yes时, 上报的IP是代理服务的IP <br>
`使用代理服务器进行用户连接`: 是否使用代理来连接Peer, 用于隐藏自己的IP, 开启此选项*无法实现使用代理来做种*, 开启此选项后会向Tracker服务器上报本地监听端口为1,导致其他用户无法连接<br>

## BitComet
`使用代理连接Tracker`: 上报给Tracker的IP是代理的IP

:::NOTE
BitComet貌似开启了`使用代理连接Peer`后还是会上报自己的监听端口, 这有助于更好地连接Peer
:::
# Tracker选择
Tracker首选: https://trackerslist.com/#/zh <br>
另外根据种子来源添加对应的Tracker, 如`nyaa.si`上的种子就添加`http://sukebei.tracker.wf:8888/announce`这个Tracker <br>

:::TIP
不推荐从国内各种种子站来下载资源, 这些资源基本都是被加了广告的二手资源, 另外, 由于这些站点的用户绝大多是是国人, 而其中的绝大多数都只会用迅雷下载, 你从这里获得的种子更不太可能有什么速度
:::

# 网络优化
众所周知, 国内大部分地区的家庭用户是没有公共IPv4的, 并且还会套好几层NAT, 就算你有公网IP, 运营商也会对你的连接数进行限制, 或者UDP QoS限速

由于国内用到基本是光猫路由一体的机器, 这类机器没有动态超级密码的情况下你改不了什么东西, 而大部分地区又要不到公网IP, 这种情况下, 网上说的什么在路由器那配置DMZ Upnp 防火墙什么的我个人感觉没多大意义, 但我还是会把这些写出来, 多一条选择. <br>

尽管IPv6的广泛运用一定程度上缓解了这些问题, 但仍治标不治本. 国内用户的IPv6虽然是正常的, 但路由器防火墙不一定会允许外部主动连接.

## 传统方法
`DMZ`: 进路由器找DMZ, 打开该选项后, 应该会让你填一个IP或者mac, 这里填你进行BT下载的电脑即可 <br>
`UPNP`: 这个只需要打开就行, BT软件会自动配置 <br>

## 有VPS用户的专属方法
Bittorrent下载需要和Peer建立连接, 这分为两种: `你连接到Peer` & `Peer连接到你` <br>

前者可以通过BT软件内置的代理, 让我们通过代理连接到Peer, 避免因GFW等网络原因导致的无法连接 <br>
> 使用代理时, 你的出口IP是VPS的IP, 但出口端口不是监听的端口

这种方法需要在VPS上部署一个代理服务, http socks, 随你便, 但这种弱鸡代理太容易被GFW拉黑了, 如果你不想被GFW拦截或丢弃掉你的流量, 就需要使用更为高级的代理软件如Xray或者Hysteria2、Singbox <br>

如果你选择用国内的VPS(:spoiler[不推荐, 因为可能连不上国外Peer])作为代理下载BT, 就不需要设置过于复杂的代理, 只需要设置好认证就行, 或者直接在国内VPS上部署BT软件, 使用NFS硬盘的方式, 这也不错<br>
:::WARNING
VPS选择要慎重, 由于国外对版权管理比较严格, 在VPS上使用BT可能会导致你的VPS被DMCA投诉, 我自己用的腾讯云的海外服务器, 平时也不下载一些可能会被版权炮的东西
:::
例如一种: <br>
本地BT通过socks代理连接到 运行在本地或局域网的代理转发软件(如Xray), Xray将BT流量封装成更抗GFW的流量, 送到你的VPS上, VPS上的Xray还原成BT流量, 完成访问 <br>
这种方法其实异常像Openwrt主路由/旁路由魔法上网的形式, 唯一的区别是用户是一个BT软件, 而不是个人用户 <br>

至于如何实现`Peer连接到你`, 这就简单了许多, 这个其实就是内网穿透, 把本地服务暴露到外网, 如SSH反向代理等, 都是为了将VPS收到的连接转发到本地指定端口, 如下是一个**SSH反代**的配置示例: <br>
```text title="/etc/ssh/sshd_config"
# 找到这个选项并设为yes
GatewayPorts yes
```
后重启sshd服务: `sudo systemctl restart sshd` <br>
现在, 你就能在下载BT的机器上运行如下命令来暴露端口
```shell
# 将VPS上5678端口的TCP流量转发到本地5678端口, 后文都以这个端口作为BT监听端口
ssh -NR 5678:127.0.0.1:5678 root@VPS_IP
```
> 在VPS上使用`lsof -i :5678`验证一下端口是不是`*:5678 (LISTEN)`的形式, 如果没有星号*出现, 代表GatewayPorts项没正确配置 <br>

**SSH反代的缺点是只能转发TCP流量, 而UDP不行** <br>

或者, 使用FRP, 不过这个要在VPS服务器和本地都配置一下 <br>

```toml title="frps.toml"
# 注意bingPort是数字, 不需要加 "", 此项为frps和frpc通信用的端口, 并不是最终监听的端口
bindPort = 5555
auth.token = "Password"
```
```toml title="frpc.toml"
serverAddr = "VPS_IP"
serverPort = 5555
auth.token = "Password"
tls_enable = true

[[proxies]]
name = "QBEE_tcp"
type = "tcp"
localIP = "127.0.0.1"
localPort = 5678
remotePort = 5678

[[proxies]]
name = "QBEE_udp"
type = "udp"
localIP = "127.0.0.1"
localPort = 5678
remotePort = 5678
```
以上两种方法的缺点是: 在BT软件里看连接的用户时, 会有一堆IP为127.0.0.1或者其他内网IP的用户, 这是正常的, 流量在应用层转发时丢失了源IP信息, 如果不想要这种, 尝试下面的方法: <br>

## 在VPS上使用隧道VPN + iptables/nftables转发流量到本地BT
由于本地BT在nat内网下, VPS显然不能直接访问本地BT, 因而要创建一个三层TUN隧道, 在网络层转发流量到BT主机 <br>
:::TIP
只要能保证Peer连接到VPS的流量能原封不动的转发到本地, 就能实现传入连接, 至于转发到本地的哪个接口无所谓, 一般是转发到自己上网的那个接口
:::
假设隧道内, BT下载机器的IP为172.16.1.2, VPS的IP是172.16.1.1 <br>
隧道软件不唯一, 自己选, 但无论如何, 请保证如下命令能正确返回结果: <br>
```shell
# 本地为Windows端
ping -S 172.16.1.2 172.16.1.1
ping -S 172.16.1.2 8.8.8.8
curl --interface 172.16.1.2 test.ipw.cn
```

确定隧道一切正常后, 在VPS上执行如下命令: (其中172.16.1.2是BT下载机器上的隧道的IP, 5678是BT监听的端口)
```shell
# 在路由前DNAT一下流量, 这个转发不会改变SRC_IP SRC_PORT
iptables -t nat -I PREROUTING -p tcp --dport 5678 -j DNAT --to-destination 172.16.1.2:5678
iptables -t nat -I PREROUTING -p udp --dport 5678 -j DNAT --to-destination 172.16.1.2:5678
```
> iptables在不指定`--table`参数时默认使用filter表, `-I`参数即`--insert`, 后面跟链, 同`--append -A`或者`--delete -D`, 每个iptables命令都要指定到哪个表的哪个链

之后下载种子时, 如果QBEE里能看到用户标签里有`I`, 或者BC里`发起`为`远程`, 即代表成功

# Summary
[Ref](https://www.bilibili.com/opus/1104998878057332745) <br>
这个教程内的隧道用的是gost, 通过wss进行传输, 另外这个教程里socks5也是用wss传输的 <br>

如果想用VPS辅助下载, 有如下几种方式
  - 本地BT连接VPS代理 + 远程端口转发到本地
  - VPS上部署隧道, BT绑定隧道, 远程端口转发到隧道 (所有流量走隧道, 注意流量消耗)
  - VPS上部署BT和隧道, 本地加入隧道后把硬盘挂载到VPS上 (这种方法对网络有较高的要求, 如果流量不够, 不建议使用)

无论哪种, 核心都是BT软件上报给Tracker的IP要为VPS的IP, 想办法让VPS上的对应端口转发到本地 <br>

如果BT绑定到全部接口, 在eth0和tun0上都能访问互联网, 此时如果尝试连接一个IP, 会走哪个接口? (猜测是走路由表来决定)<br>

# 后记
文章写完时我想到一个方法: <br>
本地BT连接VPS代理 + 隧道(但不绑定) + 远程转发到隧道 <br>
毕竟`Peer连接到你`, 只需要VPS上能转发到本地即可, 至于转发到本地的哪个接口并不重要(前提是目标接口必须被监听)<br>
而`本地BT连接VPS代理`这一步是很容易实现分流的, 让Tracker和国外Peer走VPS, 国内Peer直接直连, 听着像方法一的Plus版本

# 其他
## 在BT软件里绑定上网接口
适用于有多个网口, 且这些网口属于不同的网络的用户
  - QBEE在设置里能改
  - BitComet要进高级设置, 找`network.prefered_network_adapter_XXXX`
