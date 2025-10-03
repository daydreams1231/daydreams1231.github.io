---
title: Docker自用仓库及常用教程
published: 2025-09-28
description: ''
image: ''
tags: [Docker, Linux]
category: '教程'
draft: false 
lang: ''
---

# 安装
```shell
wget https://get.docker.com -O get-docker.sh && sudo bash get-docker.sh
```

# 卸载
```shell
sudo apt-get purge docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin docker-ce-rootless-extras
sudo rm -rf /var/lib/docker
sudo rm -rf /var/lib/containerd
sudo rm /etc/apt/sources.list.d/docker.list
sudo rm /etc/apt/keyrings/docker.asc
```

# 调试
如果Docker无法正常启动, 可使用如下命令来调试:
```shell
sudo dockerd --debugg
```

# 换用nftables (可能有兼容性问题, 慎用)
```json title="/etc/docker/daemon.json"
{
    "iptables": false,
    "ip6tables": false
}
```
ubuntu貌似为了兼容考虑, 把`iptables`命名成了`iptables-legacy`, 实际`iptables`是`nftables`, 如需切换回来:
```shell
sudo update-alternatives --set iptables /usr/sbin/iptables-legacy
```

# 换源
```json title="/etc/docker/daemon.json"
{
    "registry-mirrors": ["https://docker.1panel.live"]
}
```

# 换容器存储位置
```json title="/etc/docker/daemon.json"
{
    "data-root": "/mnt/docker"
}
```

# 换容器使用的DNS
默认情况下, 容器使用的是在其被启动时系统的DNS文件, 如果需要更改: <br>
```json title="/etc/docker/daemon.json"
{
    "dns": ["223.5.5.5", "8.8.8.8"]
}
```
该配置会让所有`新创建`的容器的DNS为配置指定的内容 <br>

或者, 在创建容器时指定 `--dns 223.5.5.5` 参数 <br>

# 容器使用代理
在创建容器时指定:
```
-e http_proxy=<URL> -e https_proxy=<URL>
```
代理URL形式: <br>
`http或者socks5h` + **://** + `认证用户`:`认证密码`@`代理地址`:`代理端口` <br>

# 个人常用镜像
## OpenList
```shell
docker run -d --name openlist --user 1000:1000 --restart unless-stopped -v /root/config/openlist:/opt/openlist/data -p 5244:5244 -e UMASK=022 openlistteam/openlist:latest
```

## Watchtower (容器自动更新)
```shell
docker run -d --name watchtower --restart unless-stopped --volume /var/run/docker.sock:/var/run/docker.sock containrrr/watchtower --cleanup --interval 43200
```

## LibreSpeed内网测速
```shell
docker run -d --name speedtest -p 5200:8080 -e MODE=standalone -v /root/config/librespeed:/database ghcr.io/librespeed/speedtest
```

## qBittorrent Enhanced Edition
```yaml title="docker-compose.yml"
services:
  QBEE:
    image: 'superng6/qbittorrentee:latest'
    container_name: QBEE
    user: '1000:1000'
    network_mode: "host"
    volumes:
      - '/root/config/qbee:/config'
      - '/mnt/external/qbee:/downloads'
    environment:
      - WEBUIPORT=8080
      - PUID=0
      - PGID=0
      - TZ=Asia/Shanghai
      - ENABLE_DOWNLOADS_PERM_FIX=false
    restart: unless-stopped
```

## Aria2
```shell
sudo docker run -d \
    --name aria2-pro \
    --restart unless-stopped \
    --log-opt max-size=1m \
    --network host \
    -e PUID=1000 \
    -e PGID=1000 \
    -e RPC_SECRET=<PASSWORD> \
    -e RPC_PORT=6800 \
    -e LISTEN_PORT=23456 \
    -e IPV6_MODE=true \
    -v /root/config/aria2:/config \
    -v /mnt/disk:/downloads \
    --dns 223.5.5.5 \
    p3terx/aria2-pro
```

## Bitwarden / vault-warden 自建密码管理器
如果你使用反向代理, 则要填入DOMAIN参数
```shell
sudo docker run -d \
  --name vaultwarden \
  -v /root/config/bitwarden:/data \
  -e DOMAIN="" \
  --restart unless-stopped \
  -p 127.0.0.1:80:80 \
  vaultwarden/server:latest-alpine
```

## natfrp
```shell
docker run -d \
   --name natfrp \
   --restart unless-stopped \
   -v /root/config/natfrp:/run \
  -p 7102:7102 \
   natfrp.com/launcher:latest
```

## Adguard Home
conf目录下只有一个yaml配置文件, work目录下存放查询日志, 过滤器等东西(work目录最好映射到外置存储上)
```shell
docker run -d \
    --name AdguardHome \
    --restart unless-stopped \
    --network host \
    -v /mnt/disk/adg_work:/opt/adguardhome/work \
    -v /root/config/adg/conf:/opt/adguardhome/conf \
    adguard/adguardhome:latest
```

## PeerBanHelper
```shell
docker run -d \
    --name PeerBanHelper \
    --restart unless-stopped \
    --stop-timeout 30 \
    -p 9898:9898 \
    -v /root/config/pbh:/app/data \
    -e PUID=1000 \
    -e PGID=1000 \
    -e TZ=UTC \
    ghostchu/peerbanhelper:latest
```

## FRP
```shell
# Client
docker run -d --restart=unless-stopped --network host -v /root/config/frp/frpc.toml:/etc/frp/frpc.toml --name frpc snowdreamtech/frpc
# Server
docker run -d --restart=unless-stopped --network host -v /root/config/frp/frps.toml:/etc/frp/frps.toml --name frps snowdreamtech/frps
```

## 百度网盘:(5800网页，5900VNC)
```shell
docker run -d \
    --name=baidunetdisk \
    -p 5800:5800 \
    -p 5900:5900 \
    -v /root/config/baidu:/config \
    -v /disk:/config/baidunetdiskdownload \
    --restart on-failure \
    johngong/baidunetdisk:latest
```

## 迅雷
```shell
docker run -d \
  --name xunlei \
   -v /root/config/xunlei:/xunlei/data \
   -v /disk:/xunlei/downloads \
   -p 2345:2345 \
   --privileged \
   cnk3x/xunlei
```

## Zerotier
```shell
docker run --name Zerotier --rm --cap-add NET_ADMIN --device /dev/net/tun zerotier/zerotier:latest <NETWORK ID>
```

## Firefox浏览器 (5800网页 5900VNC)
```shell
docker run -d --name firefox -e TZ=Asia/Shanghai  --network host -e DISPLAY_WIDTH=1280 -e DISPLAY_HEIGHT=720 -e KEEP_APP_RUNNING=1 -e ENABLE_CJK_FONT=1  -e VNC_PASSWORD=<VNC_PASSWORD> -v /root/config/firefox:/config:rw --shm-size 3g jlesage/firefox
```