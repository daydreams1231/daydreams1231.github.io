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

`Bittorrent`下的加密选项推荐开启, 最好是强制加密, 一定程度上规避某些运营商的限速, `匿名`开不开没多大用, 推荐不开

## QBEE代理相关
[Ref](https://cherr.cc/qb_proxy) <br>
[qBittorrent webui中关于代理部分的源码](https://github.com/qbittorrent/qBittorrent/blob/master/src/webui/api/appcontroller.cpp#L247) <br>

`通过代理查找主机名`: 控制DNS解析是在本地还是代理上完成(看源码貌似是socks5h和socks5的区别, 所以为了防止DNS污染, 建议打开) (DNS解析操作: RSS, Tracker服务器等)<br>
`对 BitTorrent 目的使用代理`: qBittorrent 上报给 tracker 的 IP 是否是代理服务的IP <br>
`使用代理服务器进行用户连接`: 是否使用代理来连接Peer, 用于隐藏自己的IP, 开启此选项后会向Tracker服务器上报本地监听端口为1(这个我在代码里暂时没找到, 但是通过fiddler抓包发现确实是这样), 导致其他用户无法连接, 也就无法做种<br>
`高级` -> `IP 地址已报告给 Trackers`: [Wiki](https://www.libtorrent.org/reference-Settings.html#announce_ip) 控制报告给Tracker的IP, 启用后将在向tracker的报文中添加ip参数 <br>
`高级` -> `报告给 trackers的端口`: [Wiki](https://www.libtorrent.org/reference-Settings.html#announce_port) 控制报告给Tracker的端口. 如果你使用反向代理将监听端口映射到公网上不同端口, 此项才需要设置. (这里设置上报端口的优先级低于**使用代理服务器进行用户连接**, 也就是说, 只要选择用代理连接用户, 上报给tracker的端口就始终为1, 无法更改)<br>

> libtorrent wiki中说启用代理时, 代理将在绑定的网络接口上发出, 如果给定多个接口, 也只有一个生效 <br>

> libtorrent wiki中, 尽管有**listen interface**与**outgoing interface**, qb会让这两项始终保持一致性, 所以你不能在qb中监听一个端口, 用于接受传入连接, 同时在另一个接口上向peer发起连接, 而这本身听起来就怪怪的, qb这样做也不无道理.

> `连接` -> `代理服务器` 与 `高级` -> `网络接口/绑定到的可选 IP 地址(两者效果一样)`: 注意代理服务器地址必须能让绑定的目标接口访问, 例如代理服务器IP为127.0.0.1, 而eth0接口地址为192.168.1.2, 使用`ping -I eth0 127.0.0.1`发现eth0无法访问127.0.0.1, 所以代理服务器IP不能这么填, 应该填eth0所属子网内的地址. 另外, 由于改变了监听地址, 从反向代理回来的流量也得相应地改目标地址才能实现传入连接

:::INFO
翻QB源码时发现貌似只有开启`对Bittorrent目的使用代理`, 上面的`通过代理查找主机名`才会生效. 个人猜测这里的生效应该是指连接Tracker, 至于RSS与这个无关
:::
:::Tips
Tracker服务器获取你的IP有两种, 一种是在与tracker服务器通信时主动上报ip参数, 另一种是看你连接tracker时使用的IP地址, 实际上大部分服务器都不看上报的IP参数, 另外, qbee默认不会主动上报ip参数, 除非你手动配置`IP 地址已报告给 Trackers`.<br>
Tracker获取你的Port只有主动上报一种方法 <br>
:::
## BitComet
`使用代理连接Tracker`: 意如其名, 但对比QB, 开启这项并不会让上报的端口变成1 <br>
`使用代理连接Peer`: 同上, 也不会改变上报端口 <br>

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

## 未来主流方法
至少在未来几年甚至十几年, 在IPv6成为主要流量之前, 国内用户由于没公网, 只能采取中转的方式玩BT. <br>

Bittorrent下载需要和Peer建立连接, 这分为两种: `你连接到Peer` & `Peer连接到你` <br>

前者可以通过BT软件内置的代理, 让我们通过代理连接到Peer, 避免因GFW等网络原因导致的无法连接 <br>
> 使用代理时, 你的出口IP是VPS的IP, 但出口端口不是监听的端口

这种方法需要在VPS上部署一个代理服务, 如果你的VPS在国内, 就用简单的socks5代理协议就行, 但如果VPS在国外, 就需要使用更为高级的代理软件如Xray或者Hysteria2、Singbox, 原因无他, 抗GFW. <br>

> 如果使用国内VPS, 更推荐使用wireguard VPN, 让BT软件绑定这个接口

:::WARNING
VPS选择要慎重, 由于国外对版权管理比较严格, 在VPS上使用BT可能会导致你的VPS被DMCA投诉
:::
举个例子: <br>
本地BT通过socks代理连接到 运行在本地或局域网的代理转发软件(如Xray), Xray将BT流量封装成更抗GFW的流量, 送到你的VPS上, VPS上的Xray还原成BT流量, 完成访问 <br>
这种方法其实异常像Openwrt主路由/旁路由魔法上网的形式, 唯一的区别是用户是一个BT软件, 而不是个人用户 <br>

至于如何实现`Peer连接到你`, 这就简单了许多, 这个其实就是`内网穿透/反向代理`, 把本地服务暴露到外网. <br>
如下是一个**SSH反代**的配置示例: <br>
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

以下是更为进阶的方法, 使用FRP, 这种方法需要在VPS和本地主机上都要配置一遍<br>
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
由于源IP信息丢失, 这两种方法无法对抗吸血客户端 <br>

### 在VPS上使用隧道VPN + iptables/nftables转发流量到本地BT
由于本地BT在nat内网下, VPS显然不能直接访问本地BT, 因而要创建一个三层TUN隧道, 在网络层转发流量到BT主机 <br>
:::TIP
只要能保证Peer连接到VPS的流量能原封不动的转发到自己监听的那个接口, 就能实现传入连接.
:::

> 隧道软件: Wireguard OpenVPN Zerotier Easytier, 其中最方便的是Wireguard, 可惜其抗GFW能力太弱了, 只适合国内用, 用在国外很大概率封IP, OpenVPN稍微好点, 但流量特征还是非常明显, 容易被识别出来.
假设隧道内, BT下载机器的IP为172.16.1.2 (tun0), VPS的IP是172.16.1.1 <br>
在配置完隧道后, 使用如下命令来检查隧道能否上网. <br>
```shell
# 在Windows上
ping -S 172.16.1.2 172.16.1.1
ping -S 172.16.1.2 8.8.8.8
curl --interface 172.16.1.2 test.ipw.cn
# 在Linux上
ping -I tun0 172.16.1.1
ping -I tun0 8.8.8.8
curl --interface tun0 test.ipw.cn
```

确定隧道一切正常后, 在VPS上执行如下命令:
```shell
# 在路由前DNAT一下流量, 这个转发不会改变SRC_IP SRC_PORT
iptables -t nat -I PREROUTING -p tcp --dport 5678 -j DNAT --to-destination 172.16.1.2:5678
iptables -t nat -I PREROUTING -p udp --dport 5678 -j DNAT --to-destination 172.16.1.2:5678
```
> iptables在不指定`--table`参数时默认使用filter表, `-I`参数即`--insert`, 后面跟链, 同`--append -A`或者`--delete -D`, 每个iptables命令都要指定到哪个表的哪个链 <br>

之后下载种子时, 如果QBEE里能看到用户标签里有`I`, 或者BC里`发起`为`远程`, 即代表成功 <br>

# Summary
[Ref](https://www.bilibili.com/opus/1104998878057332745) <br>
这个教程内的隧道用的是gost, 通过wss进行传输, 另外这个教程里socks5也是用wss传输的 <br>

如果想用VPS辅助下载, 有如下几种方式
  - 本地BT连接VPS代理 + 远程端口转发到本地
  - VPS上部署隧道, BT绑定隧道, 远程端口转发到隧道 (所有流量走隧道, 注意流量消耗)
  - VPS上部署BT和隧道, 本地加入隧道后把硬盘挂载到VPS上 (这种方法对网络有较高的延迟与带宽要求, 如果流量不够, 不建议使用)

无论哪种, 核心都是BT软件通过VPS连接Tracker, 再想办法让VPS上的对应端口转发到本地 <br>

如果BT绑定到全部接口, 在eth0和tun0上都能访问互联网, 此时如果尝试连接一个IP, 走的接口不一定, 有可能是eth0也可能是tun0, 与默认路由优先级无关<br>

# 后记
文章写完时我想到一个方法: <br>
本地BT连接VPS代理 + 隧道(但不绑定) + 远程转发到隧道 <br>

毕竟`Peer连接到你`, 只需要VPS上能转发到本地即可, 至于转发到本地的哪个接口并不重要(前提是目标接口必须被监听)<br>
而`本地BT连接VPS代理`这一步是很容易实现分流的, 让Tracker和国外Peer走VPS, 国内Peer直接直连, 听着像方法一的Plus版本 <br>
但往后面一想, 实际意义不大, 一是你从单个Peer那不会下载太多数据(前提是你的种子不是冷门种), 其二是国内Peer就算直连, 你也不一定连的上NAT后面的Peer, 还是让自己位于公网更方便. <br>

# 自用示范 (可能会频繁更新)
## 使用QBEE + Xray
[xray Wiki](https://xtls.github.io/config/reverse.html) <br>

xray本身是支持反向代理的, 另外由于它是一个通用的代理软件, 所以常规的http/socks入站那肯定是支持的 <br>
相较于frp, xray能自由配置如何连接到VPS, 也具有更强的抗GFW性能 <br>
唯一的缺点是它也在应用层转发流量, 一样会丢失源IP信息 <br>

以下分别是xray的客户端与服务端配置, 假设本地BT监听端口5678, BT软件设置socks代理为localhost:5555, 用户名和密码均为socks123 <br>
xray客户端和服务端之间的通信端口是443 <br>
xray客户端: (socks5入站 + vless_reality_tcp出站)<br>
```json title="client.json"
{
	"inbounds": [{
		"port": 5555,
		"protocol": "socks",
		"tag": "socks-inbound",
		"settings": {
			"auth": "password",
			"accounts": [{
				"user": "socks123", // user和password自定义
				"pass": "socks123"
			}]
		}
	}],
	"outbounds": [{
		"protocol": "direct",
		"settings": {
    		"redirect": "127.0.0.1:80" // 流量目标地址
		}
	}, {
		"protocol": "vless",
		"tag": "vless-out",
		"settings": {
			"address": "<YOUR_VPS_IP>", // 填入VPS的公网IP
			"port": 443,
			"encryption": "none",
			"id": "", // xray uuid, 自己生成一个, 要和下面的保持一致
			"flow": "xtls-rprx-vision",
			"reverse": {
				"tag": "r-inbound"
			}
		},
		"streamSettings": {
			"network": "raw",
			"security": "reality",
			"realitySettings": {
				"fingerprint": "random",
				"serverName": "awneoi.com", //自定义, 随便瞎按键盘都行, 仅用作向服务器声明SNI, 这样服务器就知道你是要代理的流量, 而不是GFW主动探测的流量
				"publicKey": "", // 使用xray x25519生成, 其中Password行就是这里的内容
				"spiderX": "",
				"shortId": "4ecedee66509" // 16进制, 长度为2的整数倍
			}
		}
	}],
	"routing": {
		"domainStrategy": "AsIs",
		"rules": [{
			"inboundTag": [
				"socks-inbound"
			],
			"outboundTag": "vless-out"
		}]
	}
}
```
上面的是简化版的客户端配置, 如果未来格式变动, 也可参考文章末尾的完整客户端配置 <br>
xray服务器端: (vless_reality_tcp入站)<br>
```json title="server.json"
{
	"inbounds": [{
		"tag": "t-inbound",
		"port": 5678,
		"protocol": "tunnel"
	}, {
		"port": 443,
		"protocol": "vless",
		"settings": {
			"clients": [{
				"id": "81ed3b80-96bd-4ffc-9699-ffd9c174e95f", // 同客户端UUID
				"flow": "xtls-rprx-vision",
				"reverse": {
					"tag": "r-outbound"
				}
			}],
			"decryption": "none"
		},
		"streamSettings": {
			"network": "raw",
			"security": "reality",
			"realitySettings": {
				"show": false,
				"target": "www.apple.com:443", // 根据VPS位置自己选
				"serverNames": [
					"awneoi.com" //同上
				],
				"maxTimediff": 0,
				"privateKey": "4J9iG9Wi17mXPOf5TQD2rtkP3F_GfP6dvmRepXDaJ2U", // xray x25519 生成, 注意这里要的是私钥, 要与客户端的公钥对应
				"shortIds": [
					"4ecedee6d66509" // 同上
				]
			},
			"tcpSettings": {
				"acceptProxyProtocol": false,
				"header": {
					"type": "none"
				}
			}
		}
	}],
	"outbounds": [{
		"protocol": "direct" // essential
	}],
	"routing": {
		"rules": [{
			"inboundTag": [
				"t-inbound"
			],
			"outboundTag": "r-outbound"
		}]
	}
}
```
完成后使用 `curl --proxy socks5://socks123:socks123@127.0.0.1:5555 test.ipw.cn`来测试一下是否成功
### 完整客户端配置
```json
{
    "log": {
        "loglevel": "warning"
    },
    "routing": {
        "rules": [{
            "type": "field",
            "inboundTag": ["bridge"],
            "domain": ["full:reverse.xui"],
            "outboundTag": "interconn"
        },{
            "type": "field",
            "inboundTag": ["bridge"],
            "outboundTag": "direct"
        }]
    },
    "reverse": {
        "bridges": [{
            "tag": "bridge",
            "domain": "reverse.xui"
        }]
    },
    "inbounds": [{
        "port": 5555,
        "protocol": "socks",
        "settings": {
            "auth": "password",
            "accounts": [{
                "user": "", // user和password自定义
                "pass": ""
            }]
        }
    }
    ],
	"outbounds": [{
            "tag": "interconn",
            "protocol": "vless",
            "settings": {
                "vnext": [{
                    "address": "", // 换成你的域名或服务器 IP
                    "port": 443,
                    "users": [{
                        "id": "", // 填写你的 UUID
                        "flow": "xtls-rprx-vision",
                        "encryption": "none",
                        "level": 0
                    }]
                }]
            },
            "streamSettings": {
                "network": "tcp",
                "security": "reality",
                "realitySettings": {
                    "fingerprint": "random",
                    "serverName": "", //自定义, 随便瞎按键盘都行, 仅用作向服务器声明SNI, 这样服务器就知道你是要代理的流量, 而不是GFW主动探测的流量
                    "publicKey": "", // 使用xray x25519生成, 其中Password行就是这里的内容
                    "spiderX": "",
                    "shortId": "" // 16进制, 长度为2的整数倍
                }
            }
        },
        {
            "protocol": "freedom",
            "tag": "direct"
        }
    ]
}

```

## 使用QBEE + gost
这种方法直接让QB绑定到VPN隧道, 所以没必要用什么socks代理 <br>
示范:
  - `server`: ./gost -L=tun://:5421?net=172.16.1.1/24 -L relay+wss://:5443?bind=true
  - `client`: ./gost -L=tun://:0/:5421?net=172.16.1.2/24 -F relay+wss://服务端IP:5443
以上命令使用tun隧道, 用websocket传输数据 <br>

```shell
iptables -t nat -I PREROUTING -p tcp --dport 5678 -j DNAT --to-destination 172.16.1.2:5678
iptables -t nat -I PREROUTING -p udp --dport 5678 -j DNAT --to-destination 172.16.1.2:5678

iptables -A FORWARD -i tun0 -j ACCEPT
iptables -A FORWARD -o tun0 -j ACCEPT
iptables -t nat -A POSTROUTING -o ens5 -j MASQUERADE
```

# 其他
## 在BT软件里绑定上网接口
适用于有多个网口, 且这些网口属于不同的网络的用户
  - QBEE在设置里能改
  - BitComet要进高级设置, 找`network.prefered_network_adapter_XXXX`
