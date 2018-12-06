## 服务器虚拟化 ##

在规划阶段，考虑以下几点：服务器与存储数量，服务器网卡与虚拟机网络，冗余电源，CPU，存储方面优先配置共享的存储，其次是添加本地磁盘。

注意：服务器的存储和应用是分离的。分为存储服务器和应用服务器。

### 服务器性能与容量规划 ###

在实施的前期，需要有虚拟机容量规划，即一台物理机上最多能放多少虚拟机。这是个综合性问题，要考虑CPU，内存，磁盘，也要考虑运行时需要的资源，实际使用时，系统需要至少30%的冗余，不能让一个主机上的资源利用率超过80%，一旦达到这个数值，系统响应会变慢。

估算虚拟化容量时，只考虑CPU的情况下，可以将物理CPU与虚拟CPU按照1:4~1:10甚至更高的比例规划，例如一台物理助理具有4个8核心CPU，在内存和存储足够的情况，按照1:5的比例，则能虚拟出4*8*5=160个vcpu，假设每个虚机需要2个vcpu，则可以创建80个虚机；实际实施虚拟化项目中，大多数虚机对cpu要求并不是非常高，所以及时为虚机分配4个或更多的cpu，但实际上该虚机的cpu使用率只有10%以下，消耗物理机cpu资源不足0.5个。

一般在虚拟化的项目中，对内存占用是最大、要求最高的，实际使用中物理机内存使用率接近80-90%，在同一物理机上规划的虚机数量较多，分配的内存总是超过实际物理机的内存；在给物理机配置内存时，要考虑在物理机上运行多少虚机、这些虚机以供需要多少内存，一般每个虚机内存需求在1~4GB甚至更多，还要为物理机用的虚拟化软件(VMware ESX等)预留部分内存。通常配置了4\*8核心cpu的主机，一般要配置96GB+的内存；配置2\*6个核心cpu的主机，通常要配置32~64GB内存。

### 统计现有服务器容量

统计内容包括：cpu型号，数量，现有利用率，内存容量及利用率，硬盘数量、大小、RAID使用情况；

计算方式：

	实际cpu资源 = 该服务器cpu频率 * cpu数量 * cpu使用率

	实际内存资源 = 该服务器内存 * 内存使用率

	实际硬盘空间 = 硬盘 - 剩余

整体项目中cpu负载率60~75%。

硬盘需求：如果虚机保存在服务器的本地存储上，而不是网络存储，则为服务器配置6个硬盘做RAID5，或者8个硬盘做RAID50。


### Hypervisor

虚拟机监控器，是运行在物理机和虚拟机之间的软件层，hypervisor可以将虚机和主机分离开，并且可以根据需要为每个虚机动态分配计算资源；

分为`Native`和`Hosted`；

原生型：hypervisor直接运行在硬件上来控制硬件资源并管理虚拟机，常见的有VMware ESXi和MS Hyper-V；

宿主型：hypervisor运行在传统OS上，可以模拟出一套虚拟硬件平台，常见的有VMware workstation和oracle virtualbox；

从性能角度来看两种方式都有性能的损耗，但`Native`比`Hosted`损耗小，使用企业生产环境使用的一般是原生型，宿主型的用在实验或测试环境；

### ESXi

ESXi是vSphere