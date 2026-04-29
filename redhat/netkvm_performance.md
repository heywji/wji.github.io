# Netperf Performance Metrics: A Comprehensive Reference Guide



sshpass -p kvmautotest [root@dell-per760-33.lab.eng.pek2.redhat.com](mailto:root@dell-per760-33.lab.eng.pek2.redhat.com)

/home/kar: 

internal_cfg/test_loops/qemu/performance_net.cfg

mac_82599_host = '90:e2:ba:7d:33:3d'

mac_82599_exhost = '00:1b:21:8e:b2:b9'



1. cd /root ; wget --no-check-certificate[ https://gitlab.cee.redhat.com/wquan/kvm-perf/-/raw/master/perf/configure_network.sh](https://gitlab.cee.redhat.com/wquan/kvm-perf/-/raw/master/perf/configure_network.sh)[ https://gitlab.cee.redhat.com/wquan/kvm-perf/-/raw/master/perf//set_irq_affinity.sh](https://gitlab.cee.redhat.com/wquan/kvm-perf/-/raw/master/perf//set_irq_affinity.sh)
2. yum -y install /usr/bin/mpstat /usr/bin/numactl
   1. sshpass -p kvmautotest ssh root@dell-per760-35.lab.eng.pek2.redhat.com "yum -y install /usr/bin/numactl"
3. cd /home/kar/workspace/avocado-vt/ ; git pull origin pull/4284/head --no-rebase ; python3 setup.py install
4. cd /home/kar ; git remote add wquan[ https://gitlab.cee.redhat.com/wquan/kar/](https://gitlab.cee.redhat.com/wquan/kar/) ; git fetch wquan ; git cherry-pick 8565f7025de033b374227667983d32547361ba58
5. python3 ./ConfigTest.py --testcase=boot --driveformat=virtio_blk --guestname=Win11.x86_64 --platform=x86_64 --machines=q35 --firmware=ovmf --debug
6. python3 ConfigTest.py --category=performance_net --customsparams="only 82599.bridge_test.1q.acceptance..netperf..host_guest" --ksm=no --driveformat=virtio_blk --vcpu=4 --mem=4096 --guestname=Win11.x86_64 --machine=q35 --firmware=ovmf --debug
7. Refer:[ https://docs.google.com/document/d/1j0DiTQQpGIY-uxVcaYRLNjjQoPhuffg6oSYhaWGpnoc/edit?tab=t.kq5qljmji8ku#heading=h.x8b8auwix94w](https://docs.google.com/document/d/1j0DiTQQpGIY-uxVcaYRLNjjQoPhuffg6oSYhaWGpnoc/edit?tab=t.kq5qljmji8ku#heading=h.x8b8auwix94w) 

# I. Core Tests (Found in your results)

**1. TCP_STREAM (Receiver)**

- **Layman's terms:** Like downloading a massive 4K movie at top speed. It measures how fast data can pour *into* your machine.
- **Professional terms:** Evaluates unidirectional bulk data transfer throughput over TCP. It assesses the Network Interface Card (NIC) bandwidth limit and the operating system's TCP congestion control for long-lived receiving connections.

**2. TCP_MAERTS (Transmitter)**

- **Layman's terms:** Like uploading a huge file to a cloud drive as fast as possible. ("MAERTS" is simply "STREAM" spelled backwards.) It measures how fast data can pour *out* of your machine.
- **Professional terms:** Evaluates unidirectional bulk data transmission throughput over TCP. This is the reverse test of TCP_STREAM, primarily used to identify asymmetric link performance and receiver-side processing bottlenecks.

**3. TCP_RR (Request/Response)**

- **Layman's terms:** Like a rapid-fire Q&A game over a phone call. You ask, they answer, without ever hanging up the phone. It measures how many back-and-forth rounds you can do in one second.
- **Professional terms:** Evaluates TCP synchronous request/response transaction rate (Transactions Per Second) over a persistent connection. It tests the network stack's interrupt handling and context switch latency for small packets.

# II. Appendix (Other Netperf test modes)

- **UDP_STREAM:** Like firing a machine gun blindly. Measures raw packet-sending speed without caring if they arrive (Good for live-streaming/gaming tests).
- **UDP_RR:** Like shouting into a valley and waiting for an echo. Measures the absolute minimum latency by stripping away TCP connection overhead.
- **TCP_CRR (Connect/Request/Response):** Extreme speed dating. Creates a brand new connection for every single transaction (Critical for high-concurrency Web server testing).
- **UNIX_STREAM / UNIX_RR:** Two people in the same house passing notes. Tests internal machine communication (IPC) by bypassing the NIC hardware.



------



# 一、 核心测试 (你的结果中包含的)

**1. TCP_STREAM (接收端测试)**

- **小白版：** 你打开迅雷疯狂下载一个超大电影，看看这根网线最高能跑多少兆（只管收）。
- **专业版：** 评估基于 TCP 协议的单向大批量数据传输极限吞吐量（Throughput），侧重考察网卡硬件带宽和系统 TCP 拥塞控制算法的长连接处理能力。

**2. TCP_MAERTS (发送端测试)**

- **小白版：** 你往百度网盘里疯狂上传一个超大文件，看看这根网线最高能跑多少兆（只管发）。
- **专业版：** 评估基于 TCP 协议的单向大批量数据发送极限吞吐量，是 STREAM 的反向测试，常用于排查网络链路收发不对称及接收端处理瓶颈。

**3. TCP_RR (请求/响应测试)**

- **小白版：** 两个人通电话快问快答，你发“在吗”，他回“在”，电话不挂断，看一秒钟能聊几个来回。
- **专业版：** 评估长连接状态下的 TCP 同步请求/应答事务处理率（TPS），主要考验网络协议栈处理小包（Small Packets）的中断响应及上下文切换极低延迟性能。

# 二、 附录 (其他 Netperf 测试模式)

- **UDP_STREAM：** 闭上眼睛端起冲锋枪朝对面狂扫射。测纯粹的发包速度，不管对面收到没有（常用于直播、游戏丢包测试）。
- **UDP_RR：** 对着空旷的山谷大喊一声等回音。排除了 TCP 的干扰，测最纯粹、最底层的网络反应时间。
- **TCP_CRR (相亲狂魔测试)：** 疯狂相亲。每次打招呼前都要重新握手建连接（衡量 Web 服务器高并发处理能力的核心指标）。
- **UNIX_STREAM / UNIX_RR：** 两个人住在同一个屋檐下传纸条。连门都不出，纯测电脑内部两个程序说话的速度，不经过网卡。



------



Investigation Report on Network Isolation Anomaly & Bug Reproduction Guide

## 1. Executive Summary

During the automated performance testing of Win11 Virtio-net (Intel 82599 10G), a "Phantom IP" (10.72.143.x) was observed on the test dedicated Virtio NIC. This report confirms that the test execution period was strictly isolated and the anomaly was caused by the host's automatic network management after the test cleanup phase. It also provides a definitive guide to manually reproduce the Win11 Packet-Per-Second (PPS) drop bug while avoiding IP contamination.

![image-20260412163400293](/home/wjiwji/projects/erlinux.github.io/redhat/typora/image-20260412163400293.png)

## 2. Infrastructure & Topology Details

### *Physical Hosts*

- **Host A (System Under Test):** dell-per760-33
  - **Management NIC:** eno8303 (MAC: c4:cb:e1:da:68:08) -> Connected to bridge switch (Lab Public Network).
  - **Test NIC (82599):** ens8f1 (MAC: 90:e2:ba:7d:33:3d) -> Dedicated for P2P traffic.
  - **Private Bridge:** atbr0 (Dynamically created/deleted by Avocado-VT).
- **Host B (Traffic Generator):** dell-per760-35
  - **Test NIC (82599):** ens7f1 (MAC: 00:1b:21:8e:b2:b9) -> Physically back-to-back connected to Host A's ens8f1 via fiber.

### *Virtual Machine (Guest)*

- **VM Name:** Win11 (avocado-vt-vm1)
  - **Management vNIC (RTL8139):** MAC 9a:37:37:37:37:6e -> Connected to Host's switch bridge. Used for Control Plane (SSH/WinRM).
  - **Test vNIC (Virtio-net):** MAC 9a:e6:39:39:fc:c6 -> Corresponds to host tap0. Used for Data Plane (Performance Test).

## 3. Root Cause Analysis (The Phantom IP)

The P2P isolation was intact during the actual performance measurement. The appearance of the 10.72.x.x IP on the Virtio card was a post-test side effect:

1. **Isolation Phase:** During the test, tap0 and ens8f1 were isolated within the atbr0 bridge.
2. **Cleanup Phase:** The Avocado script executed ip link delete atbr0 after data collection.
3. **Automatic Handover:** The host's NetworkManager or libvirtd detected the orphaned tap0 and automatically added it to the default public bridge switch.
4. **DHCP Trigger:** Win11 detected the Link UP event on the new bridge and requested a DHCP IP from the lab network.

## 4. Manual Bug Reproduction & Isolation Guide

To manually reproduce the Win11 "low TX packet count (PPS drop)" bug without hitting the 10.72.x.x IP contamination issue, follow these exact phases:

### *Phase 1: Prevent IP Contamination (Solving the 10.72.x.x issue)*

On **Host A (33)**, lock the interfaces to prevent NetworkManager from moving them to the public bridge:

\# Prevent NM from managing the physical NIC and the virtual TAP interface

nmcli device set ens8f1 managed no

nmcli device set tap0 managed no

### *Phase 2: Establish the Isolated P2P Network*

**On Host B (35):**

ip addr add 192.168.100.35/24 dev ens7f1

ip link set ens7f1 up

netserver -D

**On Host A (33) & Win11 Guest:**

1. Manually create the bridge: ip link add name br_test type bridge; ip link set ens8f1 master br_test; ip link set br_test up; ip link set ens8f1 up.
2. Map the VM's Virtio NIC to br_test.
3. In Win11 Guest, **disable the RTL8139 management adapter**.
4. Configure the Virtio NIC with static IP 192.168.100.33, Mask 255.255.255.0. **Leave Default Gateway and DNS completely empty.**
5. Disable Windows Firewall: netsh advfirewall set allprofiles state off.

### *Phase 3: Trigger the Bug & Collect Evidence*

In the Win11 Guest, run the netperf tests that caused the massive regression (-21.4% and -36.1% throughput drop):

\# Trigger extreme small packet (64 bytes) PPS regression

netperf -H 192.168.100.35 -t TCP_STREAM -l 60 -- -m 64



\# Trigger medium packet (1024 bytes) regression

netperf -H 192.168.100.35 -t TCP_STREAM -l 60 -- -m 1024

Simultaneously, on **Host A (33)**, capture the PPS metric to prove the low TX rate from Win11:

sar -n DEV 1 | grep br_test

\# Record the rxpck/s (packets received by host bridge from VM) to attach to the Bugzilla.



------



性能测试环境网络隔离异常调查与 Bug 复现指南

## 1. 摘要

在执行 Win11 Virtio-net (Intel 82599 10G) 自动化性能测试期间，观测到测试专属的 Virtio 网卡获取了“幻象 IP” (10.72.143.x)。本报告确认测试执行期间链路处于严格隔离状态，该异常是由测试清理阶段后宿主机自动化的网络管理机制触发的。此外，本报告提供了完整的手动复现 Win11 发包量（PPS）骤降 Bug 的标准操作流程（SOP），同时彻底解决 IP 污染问题。



## 2. 基础设施与拓扑细节

### *物理机*

- **物理机 A (被测端):** dell-per760-33
  - **管理网卡:** eno8303 (MAC: c4:cb:e1:da:68:08) -> 连接至网桥 switch (实验室公网)。
  - **测试网卡 (82599):** ens8f1 (MAC: 90:e2:ba:7d:33:3d) -> 专用于 P2P 流量。
  - **私有网桥:** atbr0 (由 Avocado-VT 动态创建/删除)。
- **物理机 B (打流端):** dell-per760-35
  - **测试网卡 (82599):** ens7f1 (MAC: 00:1b:21:8e:b2:b9) -> 通过光纤与物理机 A 的 ens8f1 物理点对点直连。

### *虚拟机 (Guest)*

- **虚拟机名称:** Win11 (avocado-vt-vm1)
  - **管理虚拟网卡 (RTL8139):** MAC 9a:37:37:37:37:6e -> 连接至宿主机的 switch 网桥。用于控制面 (SSH/WinRM)。
  - **测试虚拟网卡 (Virtio-net):** MAC 9a:e6:39:39:fc:c6 -> 对应宿主机 tap0 设备。用于数据面 (性能压测)。

## 3. 根因分析（幻象 IP 的产生）

在实际性能指标测量期间，P2P 隔离是完整的。Virtio 网卡出现 10.72.x.x IP 是测试后的副作用：

1. **隔离阶段:** 测试期间，tap0 和 ens8f1 被严格限制在 atbr0 网桥内。
2. **清理阶段:** 数据采集完成后，Avocado 脚本执行了 ip link delete atbr0 粗暴删除了网桥。
3. **自动接管:** 宿主机的 NetworkManager 侦测到 tap0 变成孤儿接口，自动将其抢夺并添加到了默认的公用网桥 switch 中。
4. **DHCP 触发:** Win11 探测到新网桥的链路 UP 事件，发起 DHCP 请求并从实验室公共网络获取了 10.72.x.x IP。

## 4. Bug 手动复现与网络隔离避坑指南

为了手动复现 Win11“发包量极低（PPS Drop）”的 Bug，并彻底解决获取到 10.72.x.x IP 的污染问题，请严格按照以下阶段操作：

### *阶段 1：防止 IP 污染（解决 10.72.x.x 问题）*

在 **Host A (33 机)** 上，必须首先封印网卡，防止系统网络大管家 (NetworkManager) 捣乱：

\# 禁止 NM 管理物理测试网口和虚拟机的 TAP 通道

nmcli device set ens8f1 managed no

nmcli device set tap0 managed no

### *阶段 2：建立完全隔离的 P2P 打流环境*

**在 Host B (35 机，服务端) 操作：**

ip addr add 192.168.100.35/24 dev ens7f1

ip link set ens7f1 up

netserver -D

**在 Host A (33 机) 及 Win11 虚拟机内部操作：**

1. 手动建桥连通物理与虚拟：

ip link add name br_test type bridge

ip link set ens8f1 master br_test

ip link set br_test up

ip link set ens8f1 up

1. 确保虚拟机的 Virtio 网卡映射到了 br_test 上。
2. 在 Win11 内部的“网络连接”面板中，**直接禁用 RTL8139 管理网卡**，彻底切断物理后路。
3. 为 Virtio 网卡配置静态 IP 192.168.100.33，子网掩码 255.255.255.0。**【警告】默认网关和 DNS 必须留空！**
4. 关闭 Win11 防火墙：CMD 执行 netsh advfirewall set allprofiles state off。

### *阶段 3：触发发包量 (PPS) 骤降 Bug 并收集证据*

在 Win11 虚拟机中，执行导致测试报告中严重性能退化（-21.4% 和 -36.1%）的 netperf 命令：

\# 触发 64 字节极小包的 PPS 瓶颈 Bug

netperf -H 192.168.100.35 -t TCP_STREAM -l 60 -- -m 64



\# 触发 1024 字节包的吞吐量断崖 Bug

netperf -H 192.168.100.35 -t TCP_STREAM -l 60 -- -m 1024

**定罪证据收集：**

当 Win11 正在艰难发包时，在 **Host A (33 机)** 上运行以下命令监控真实的收包率（即 Win11 的发包率）：

sar -n DEV 1 | grep br_test

\# 截取此时低迷的 rxpck/s（每秒收包数）指标，将此作为核心证据提交至 Bugzilla。



# Notes - Discuss: test metrics regarding netkvm performance

## Mar 13, 2026 | [Discuss: test metrics regarding netkvm performance](https://www.google.com/calendar/event?eid=NmVvbW5zbGhramJmY2hlNGZvazJsNW9pcGQgd2ppQHJlZGhhdC5jb20)

Attendees: [Qianqian Zhu](mailto:qizhu@redhat.com) [Wecom Ji](mailto:wji@redhat.com) [Wenli Quan](mailto:wquan@redhat.com) [Xiaoling Gao](mailto:xiagao@redhat.com)

Notes

- Test results: http://virtqetools.lab.eng.pek2.redhat.com/kvm_autotest_job_log/?jobid=12425915
- Refer:[ ](https://docs.google.com/document/d/19xscOm6QC5ftmGDJkyAa5ZP_Hb2NDPYowK--pjpTmwA/edit?tab=t.0)[Netperf Performance Metrics: A Comprehensive Reference Guide](https://docs.google.com/document/d/19xscOm6QC5ftmGDJkyAa5ZP_Hb2NDPYowK--pjpTmwA/edit?tab=t.0)
  - (RHEL-10.2-20260309.2 + virtio-win-1.9.53-0.el10.iso)
    - RHEL10.1 GA version + <driver related to rhel10.1.z>
- Hi team, setting up a quick sync to align on our netkvm performance testing matrix. I've completed the initial test on RHEL 10.2 with the latest driver, but I'm getting different suggestions on the baseline comparison: should we keep the Host OS constant and change the driver version, or keep the driver constant and change the Host OS? 
- Goal 1: The driver
  - keeps the Host OS constant and changes the driver version
- Goal 2: customer
  - keeps the driver constant and changes the Host OS?
- Wenli: Test the whole customer environment first, checking the driver version, then if needed.
- Qianqian: 10.2 is the baseline, 10.2.z (host is 10.2 GA version, driver is 10.2.z) ; 10.3 (host is 10.3, driver is 10.3)

Action items

- Question 1: Should we keep the Host OS constant and change the driver version, or keep the driver constant and change the Host OS? 
- Question 2: VBS + IOMMU
  - No IOMMU, no VBS
  - IOMMU (iommu_platform=on, ats=on)
  - **No IOMMU, VBS running**
  - Both IOMMU and VBS
- Refer: https://issues.redhat.com/browse/RHEL-116446



# NETKVM Lazy memory allocation



### **By Yuri Benditovich ybendito@redhat.com**

### **NETKVM Lazy memory allocation**

#### **Preamble**

This document explains the feature in general aiming to cover feature-specific tests and conduct the feature acceptance process. Some of the recommendations below might be used in future to extend regression or sanity testing procedures.

The recommendations are based on the details of feature implementation and on the results of preliminary engineering tests, the exact parameters might be changed in the final testing procedures.

#### **Feature description and testing recommendations**

The “Lazy memory allocation” feature was developed to address a problem of long initialization time of NETKVM device driver (NDIS miniport).

 

During initialization the driver needs to allocate memory buffers for RX buffers (they are used for incoming packets) and auxiliary TX buffers (they are used to transmit outgoing packets efficiently). Typically, without any additional parameters, the QEMU creates a virtio-net device with a single queue pair having RX queue of 256 packets and TX queue of 256 packets. In such a case the driver allocates all the required buffers and makes the device ready for usage in reasonable time.

However there are several cases when the driver needs to spend much more time to initialize the device.

- The device may have multiple queue pairs, this may be useful on large-scale systems with many CPUs. 
- QEMU can be configured for a bigger size of RX and TX queues (up to 1024).
- In case the virtio-net device is a physical device the size of RX and TX queues might be even bigger, depending on hardware design, and queue size of 4096 elements is not rare. 

As an example, if we have a device with 32 queues with 1024 entries in each RX and TX queue the driver needs to make 512 times more memory allocations than in the case of a single queue pair with 256 entries in each queue. 



Left side: Single Queue Pair with 256 entries per queue

This setup requires a total of 512 allocations (256 entries × 1 RX queue + 256 entries × 1 TX queue).

Right side: 32 Queue Pairs with 1024 entries per queue

This setup requires 65,536 allocations (1024 entries × 32 RX queues + 256 entries × 32 TX queues).

Also, the system may include more than one virtio-net device, the initialization process is sequential and the next devices are not usable until previous ones finish their initialization.

The idea of the “Lazy memory allocation” feature is to split the initialization process into two parts: 

- Minimal initialization to make the device ready for basic networking functionality as fast as possible
- Lazy initialization that is completed when all the required allocations are done

The heavy part of typical initialization is the allocation of RX buffers (each of them is as large as 64K), so during “minimal initialization” stage the driver initializes all the TX queues and allocates (at the moment of writing this document) just 16 RX buffers for each RX queue. At this point the driver reports the “ready” state and the OS can start using the device. 

On the second (“”lazy”) stage of the initialization the driver allocates the rest of required RX buffers, doing that as a background job in a separated thread in parallel with network activity..

#### **Configurability**

There is a configuration parameter to disable the feature and get back to the previous behavior. 

| Parameter name | Default | Possible values | Note                                                       |
| -------------- | ------- | --------------- | ---------------------------------------------------------- |
| **FastInit**   | 1       |                 | The feature is enabled                                     |
|                |         | 0               | Disable the feature and get back to previous behavior      |
|                |         | 1               | Enable the feature and make the device availability faster |

#### 

#### **Metrics**

There are additional metrics that can help in feature validation and diagnostic, they are accessible via the netkvm-wmi.cmd batch file as “**netkvm-wmi.cmd cfg**”:

| Field               | Units        | Note                                                         |
| ------------------- | ------------ | ------------------------------------------------------------ |
| **InitTimeMs**      | milliseconds | The time that the driver spent till it reports “ready” state to OS |
| **LazyAllocTimeMs** | milliseconds | The additional time that the driver spent till it completes all the initialization. -1 if Lazy Allocation job is not done yet |
|                     |              |                                                              |

There are two existing metrics whose behavior was modified when lazy allocation is enabled (they are also under “**netkvm-wmi.cmd cfg**” data):

**RxQueueSize** - indicates actual number of entries in each RX queue. When lazy allocation job is done, this returns maximal number of entries in RX queue (i.e. smaller of device RX queue size and Init.MaxRxBuffers driver’s configuration parameter)

**MemoryKB** - indicates actual size of DMA memory allocations made by the driver in kilobytes. When the lazy allocation job is done, this returns the maximal amount of allocated DMA memory.

#### 

#### **Note for testing procedures**

**Important! When performing tests related to performance, when the result of the test is measured in time or throughput, the driver verifier must be disabled.** 

Note that defining a virtio-net with many queues and running the system with a smaller number of CPUs does not make sense, some of the queues will not be used. Exact number of queues in use is available via “netkvm-wmi.cmd cfg”, field ‘NumOfQueues’. 

If the device has many queues and the size of the RX queue is also big (1024) the ‘LazyAllocTime’ might be long (up to several minutes depending on exact numbers). 

When possible - check that the network adapter acquires its IP address via DHCP and responds to ping requests when its ‘lazy allocation job’ is still active (LazyAllocTimeMs=-1).



#### **Notes for testing matrix**

Possible testing matrix for the feature may look like this:

|           | S (1x256)    | …              |                |              |                |                |
| --------- | ------------ | -------------- | -------------- | ------------ | -------------- | -------------- |
|           | No fast Init | Fast:init time | Fast:Lazy time | No fast Init | Fast:Init time | Fast:Lazy time |
| Cold boot | [time (ms) ] | …              | …              | …            | …              | …              |
| Hot boot  | …            | [time (ms)]    | …              | …            | …              | …              |
| Enable    | …            | …              | …              | …            | …              | …              |

Columns are various cases of Rx memory size (number of queues * queue size)

Row are initialization scenarios

Value in each cell is the time spent for initialization: for case when “Fast Init” is disabled this is “Init time” as reported by “netkvm-wmi.cmd cfg” , for case when “Fast init” is enabled - “Init time” and “Lazy time” respectively.

##### Testing matrix (columns)

Testing matrix with **enabled and disabled ‘FastInit’** should include various numbers of queues and various queue sizes, for example:

- 1 queue, default sizes (S)
- 4 queues, default sizes (M)
- 4 queues, RX 1024, TX 256 (L) 
- 32 queue, RX 1024, TX 256 (XXL) - should have enough RAM (~2GB will be used for DMA allocations) 
- Maximal number of devices (S)

Probably it is better to present the test results of “multiple devices” test case (like 27 virtio-net devices) in a separate table, “**netkvm-wmi.cmd cfg**” for many devices outputs the data for all the devices that finished their minimal initialization. It is possible that the sum of all the “Init time” values will be close to the total delay until all the devices are ready as it is visible to the end user..

##### Testing matrix (rows)

Testing matrix should include 3 scenarios of initialization flows

- Cold boot
- Hot boot (restart)
- Enable from disabled state

#### **Disable flow**

When the OS disables the driver before the lazy allocation job is done the driver should gracefully stop the lazy allocation process, after that the disable operation is similar to regular one. So in feature testing we need to check that all the allocations are freed (driver verifier helps with this), the device becomes disabled and further enable operation also executed successfully.

One of the scenarios in the testing should be “enable” - “short wait” (much shorter than typical time to full initialization) - “disable”. 

#### **Enable flow**

The results in enable flow might be different from ones of hot or cold boot, this is the reason why it makes sense to check enable flow separately.

#### **Shutdown/reboot flows**

Shutdown/reboot flows should be tested just for regression. It is recommended to cover the case when shutdown reboot happens after disable-enable with XXL queues configuration (as described above) before the device is fully initialized (i.e. when its Lazy initialization is still in progress).

#### **Power management flows**

RedHat recommends to disable power management for the VM, setting both “disable_S3” and “disable_S4” global flags to “1” (they do not present any advantage to the end user and are prone to trigger complicated flows). Still it makes sense to check them once during the feature acceptance test. For that the “disable_s4” global parameter of qemu should be set to “0”, hibernation on the VM should be enabled and “Fast startup” should be turned on.

Hibernation flow in such a case will be similar to one of a bare-metal machine, i.e. driver’s state and applications state will be restored on next start of qemu. 

Shutdown flow of the OS will actually be a hibernation for drivers layer only. Further boot will take the driver out of hibernation.

We recommend testing the hibernation and shutdown 

**Notes:**

- **disable “Fast startup” after the test is done before shutting down the system**
- **never change qemu command line if the system was hibernated or shut down with “Fast startup” enabled**

 

#### **Final word**

All the results of feature testing should be reviewed/discussed. It is possible that we’ll find that “This is good but not good enough”, in this case further work will be based on respective Jira issues.

All the problems and things that look like a problem should be communicated and discussed in context of respective Jira issues.

 