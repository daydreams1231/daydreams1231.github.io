---
title: Linux下的SSD Trim
published: 2025-09-28
description: ''
image: ''
tags: [Linux, SSD, Trim]
category: ''
draft: false 
lang: ''
---

# TRIM
[Ref](https://www.ichenfu.com/2022/10/05/enable-trim-on-usb-attached-scsi-ssds) <br>
[ArchWiki](https://wiki.archlinux.org/title/Solid_state_drive#External_SSD_with_TRIM_support) <br>

使用`lsblk -D`命令来查看trim启用情况 <br>
如果`DISC-GRAN`非0, 代表这个设备支持TRIM <br>
如果`DISC-MAX`非0, 代表这个设备启用了TRIM <br>

DISC-GRAN   DISC-MAX <br>
        0          0    不支持TRIM <br>
        1          0    支持TRIM, 但没启用 <br>
        1          1    支持且启用TRIM <br>

对于外置移动硬盘里的SSD, 需要通过如下命令查看Trim情况: <br>
```shell
sudo apt install sg3-utils
# 查看输出中的: Logical block provisioning: lbpme=0, lbprz=0, 关键是lbpme=0, 代表内核默认不会discard
sudo sg_readcap -l /dev/sda

# 查看输出Logical block provisioning VPD page (SBC)部分, 主要看第一行的LBPU=XXX, 如果该值为1, 代表芯片支持unmap指令, 即支持Trim
sudo sg_vpd -a /dev/sda

# 查看目前内核识别的设备的provisioning_mode, 第一个XXXXXX是设备名称, 如sda, 第二个XXXXXX是硬件编号, 直接TAB自动补全即可
cat /sys/block/XXXXXX/device/scsi_disk/XXXXXX/provisioning_mode
# 如果输出为full, 代表内核当前是没有检测到设备支持Trim特性, 解决方法也比较简单，直接echo unmap到这个文件
echo unmap > /sys/block/XXXXXX/device/scsi_disk/XXXXXX/provisioning_mode

# 设置设备在插入时自动更新其状态
echo 'ACTION=="add|change", ATTRS{idVendor}=="0bda", ATTRS{idProduct}=="9220", SUBSYSTEM=="scsi_disk", ATTR{provisioning_mode}="unmap"' >>/etc/udev/rules.d/10-uas-discard.rules
# 记得重启系统, 让规则生效
```
对于一个SSD, 如果格式化为ext4, 则在挂载时最好指定`discard`参数, 来启用TRIM功能 <br>
如果是f2fs, FS会自动定期在空闲时TRIM(前提是没手动禁用trim) <br>

:::TIP
mount命令显示设备的挂载参数, 也能用来看是否启用了trim
:::

:::INFO
Debian下有一个自动TRIM服务: /usr/lib/systemd/system/fstrim.service <br>
默认情况下, 它会一周进行一次TRIM
:::

