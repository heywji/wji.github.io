

# RHEL Week - VirtIO-Win-NetKVM - UDP Segmentation Offload
>  本文档将使用中文完整记录测试流程和结果，方便大家查阅。

[TOC]


## 一、客户端配置

### 1. 系统环境

![image-20250513165414881](/home/wji/Pictures/typora-user-images/image-20250513165414881.png)

### 2. 网卡驱动版本

![image-20250513165716490](/home/wji/Pictures/typora-user-images/image-20250513165716490.png)

**Tips: ** 目前客户可以拿到的最新的版本是： **virtio-win-1.9.45-0.el10_0.rpm** 需要将它解压后得到 **virtio-win-1.9.45-0.el10_0.iso** 并挂载到虚拟机内。

```bash
Name         : virtio-win
Version      : 1.9.45
Release      : 0.el10_0
Architecture : noarch
Size         : 258 M
Source       : virtio-win-1.9.45-0.el10_0.src.rpm
Repository   : beaker-AppStream
Summary      : VirtIO para-virtualized drivers for Windows(R)
URL          : http://www.redhat.com/
License      : Apache-2.0 AND BSD-3-Clause AND GPL-2.0-only AND GPL-2.0-or-later
Description  : VirtIO para-virtualized Windows(R) drivers for 32-bit and 64-bit
             : Windows(R) guests.

```

![image-20250514112256573](/home/wji/Pictures/typora-user-images/image-20250514112256573.png)

![image-20250514112209137](/home/wji/Pictures/typora-user-images/image-20250514112209137.png)

### 3. iperf3 测试工具

![image-20250513153747856](/home/wji/Pictures/typora-user-images/image-20250513153747856.png)

### 4. Wireshark & Winpcap 抓包工具

![image-20250513165523077](/home/wji/Pictures/typora-user-images/image-20250513165523077.png)


## 二、服务端配置

### 1. 系统环境

```
[root@dell-per750-51 ~]# cat /etc/redhat-release 
Red Hat Enterprise Linux release 10.1 Beta (Coughlan)
[root@dell-per750-51 ~]# uname -r
6.12.0-74.el10.x86_64
[root@dell-per750-51 ~]# rpm -q qemu-kvm
qemu-kvm-10.0.0-1.el10.x86_64
```

### 2. 开启 Libvirt 服务

```
[root@dell-per750-51 ~]# systemctl enable libvirtd # 设置开机自启
[root@dell-per750-51 ~]# systemctl start libvirtd  # 立即启动服务
[root@dell-per750-51 ~]# virsh net-list            # 检查 libvirt Network 是否存在
 Name      State    Autostart   Persistent
--------------------------------------------
 default   active   yes         yes

[root@dell-per750-51 ~]# ip addr show dev virbr0   # Linux 软件桥接（bridge）设备
16: virbr0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 9000 qdisc htb state UP group default qlen 1000
    link/ether 52:54:00:a7:62:4c brd ff:ff:ff:ff:ff:ff
    inet 192.168.122.1/24 brd 192.168.122.255 scope global noprefixroute virbr0
       valid_lft forever preferred_lft forever
[root@dell-per750-51 ~]# 
```

### 3. 开启 iperf3 server 端

```
[root@dell-per750-51 ~]# iperf3 -s -p 12345
-----------------------------------------------------------
Server listening on 12345 (test #1)
-----------------------------------------------------------


```

### 4. 调整 MTU 数值


> “MTU” 是 “Maximum Transmission Unit” 的缩写，中文通常称为 “最大传输单元”。链路层 MTU 决定了“单帧”在一个网络跳（交换机→路由器→网卡）上传输时的最大长度（例如 1500 B 或 9000 B）。它指的是在一条链路层（例如以太网、Wi-Fi）上，单个数据帧（或包）的 最大 大小（以字节为单位），超过这个大小就必须拆分或分段才能发送。

- 默认 MTU：1500 字节

这是绝大多数传统以太网接口（包括大部分交换机和路由器）出厂时的标准设置，适用于 IPv4/IPv6 的一般流量。

- Jumbo Frame（巨型帧） MTU：常见 9000 字节

许多现代网卡和交换机支持将 MTU 调到更大，一般在 9000 B 左右，以降低分片开销、提升大流量吞吐（例如存储网络、虚拟化环境中常用）。

> 只要网络全链路都支持 jumbo frame，把 MTU 调大就能显著提速。


```[root@dell-per750-51 ~]# systemctl enable --now libvirtd
[root@dell-per750-51 ~]# ip link set virbr0 mtu 1500
[root@dell-per750-51 ~]# ip addr show dev virbr0
16: virbr0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc htb state UP group default qlen 1000
    link/ether 52:54:00:a7:62:4c brd ff:ff:ff:ff:ff:ff
    inet 192.168.122.1/24 brd 192.168.122.255 scope global noprefixroute virbr0
       valid_lft forever preferred_lft forever
[root@dell-per750-51 ~]# 
[root@dell-per750-51 ~]# ip link set tap0 mtu 1500
[root@dell-per750-51 ~]# ip addr show dev tap0 
31: tap0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc mq master virbr0 state UNKNOWN group default qlen 1000
    link/ether 7e:59:e2:17:88:2e brd ff:ff:ff:ff:ff:ff
    inet6 fe80::7c59:e2ff:fe17:882e/64 scope link proto kernel_ll 
       valid_lft forever preferred_lft forever
```

**Tips: **QEMU/KVM 给虚拟机分配网卡时，在 Host 创建一个 tap 二层设备，让 VM 的虚拟网卡被桥接到宿主机网络。没有开启虚拟机之前将不会存在。



## 三、效果测试

### 1/3: 测试**关闭** UDP Segmentation Offload 的 UDP 带宽
![img](/home/wji/Pictures/typora-user-images/AGV_vUe1ptCyNYPinHraVVJSkPriFFm8ND21zZ4mP1Z3S3h6CXB9pMnBgPC_iz8pR-LqeF6OJnKGI8_pH6opi-_jpWzaFOfdYwFBm6jLsl828R4Z5fuLN8cjrVQOm0iNvMIfDTwJciNE-g=s2048)


**Attention: **  UDP 报文（包括 IP 头＋UDP 头＋UDP 载荷）总长度 ≤ 本跳链路的 MTU，就不会触发 IP 分片。

> **IPv4 + UDP**
>
> - IP 头 20 B，UDP 头 8 B
> - 最大 UDP 有效载荷 ≈ 1500 − 20 − 8 = **1472 B**

![img](/home/wji/Pictures/typora-user-images/AGV_vUf0IX5fh6btuQRvFeGscPLJpAz0av3C4M68O6YlELc04k1vTzc2mEbMfAj4nzm-tJUOyBUrpoXGfuQ9ZbCA8bdvlL5w5tSLi9VsZax9VWp0WS8UM8vmUJSk1wd1aQoqI-4RHwc-=s2048)



### 2/3: 测试**打开** UDP Segmentation Offload 的 UDP 带宽 with 8972B data (Standard Mode)

![image-20250514114112659](/home/wji/Pictures/typora-user-images/image-20250514114112659.png)

**Attention: **  UDP 报文（包括 IP 头＋UDP 头＋UDP 载荷）总长度 ≤ 本跳链路的 MTU，就不会触发 IP 分片。

> **IPv4 + UDP**
>
> - IP 头 20 B，UDP 头 8 B
> - 最大 UDP 有效载荷 ≈ 9000 − 20 − 8  = **8972 B**

![image-20250514114328051](/home/wji/Pictures/typora-user-images/image-20250514114328051.png)



### 3/3: 测试**打开** UDP Segmentation Offload 的 UDP 带宽 with 65507B data (Stress-test mode)


![img](/home/wji/Pictures/typora-user-images/AGV_vUep64k6LqDRhVSu4nZpylWpM7aD3iPqEzCWffVanG8Cby_NJdjx6Zea773EbEaa4GyOvtumh9f0Imf3VMzpr_LgRB66xX7Cpbw8LZfzoxWlRWYfY-AkKqRWXdy4CyMolsx0SSyMFw=s2048)

**Attention: **  UDP 报文（包括 IP 头＋UDP 头＋UDP 载荷）总长度 > 本跳链路的 MTU，**会**触发 IP 分片。

> **IPv4 + UDP**
>
> - IP 头 20 B，UDP 头 8 B
> - 最大 UDP 有效载荷 ≈ 65 535 − 20 − 8 = **65 507 B**

![img](/home/wji/Pictures/typora-user-images/AGV_vUeqUrx8FPXJ5d-BlijuINYQAZw1OqVfgnBF8Qla61hj6bQwTlDcxlt4qGM8GzDWAEXzyS0TUxswwy5Uc74oIHc51E_JofB5bvV4XyQPX6fDe0GtdFXPnF7gzzirbNBbpRX1P7J_eQ=s2048)



## 四、可能的提问。

### Q0: CPU 利用率的问题


#### A0: 关闭 USO 后 CPU 利用率  8%

![Screenshot from 2025-05-14 12-34-16](/home/wji/Pictures/typora-user-images/Screenshot from 2025-05-14 12-34-16.png)

#### A0: 打开 USO 后 CPU 利用率 7%


![image-20250514123656238](/home/wji/Pictures/typora-user-images/image-20250514123656238.png)



### Q1: RHEL 的 UDP offload 支持吗？

### A1：我不测试 RHEL 不是很清楚。昨天和 RHEL network 的 feature owner 确定还不支持。可以通过 ethtool 列举网卡的选项。



### Q2. Windows 的 TCP offload 支持吗？

### A2: 支持的

![image-20250514120254055](/home/wji/.var/app/io.typora.Typora/config/Typora/typora-user-images/image-20250514120254055.png)

![image-20250514120907671](/home/wji/Pictures/typora-user-images/image-20250514120907671.png)