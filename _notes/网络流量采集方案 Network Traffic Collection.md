- # 1. 基础知识
	- 两种最常见的网络流量采集方法是网络分路器(TAPs) 和交换机端口镜像(SPANs)
	- **网络分路器(TAPs) **
		- 一种直接连接到网络基础设施的物理设备。 TAP 位于两个设备之间，而不是两个交换机或路由器之间；并行方案，通过拷贝流量副本进行监控，不影响原始网络通讯，这种技术确保任何大小的数据包都将被拷贝并且**不容易超载**。
		- 如下图所示，Web服务器和交换机之间的流量数据被TAP设备捕获拷贝(TxA:A->B, TxB:B->A)，大多数TAP设备分别复制两个方向的流量数据，并将它们发送到单独的监控端口（TxA和TxB）
		- <img src="https://raw.githubusercontent.com/MarsAuthority/redblue/master/assets/Pasted%20image%2020221104134953.png">
	- **交换机端口镜像(SPANs)**
		- SPAN 端口（有时称为镜像端口）是一种内置于交换机或路由器的软件功能，可拷贝通过设备的选定流量数据包的副本，并将它们发送到指定的 SPAN 端口。 由于交换机或路由器的主要目的是转发生产数据包，因此 SPAN 数据在设备上的优先级较低，SPAN 还使用单个出口端口来聚合多个链路，因此**很容易超载**。SPAN 最适合在未安装 TAP 的位置对少量数据进行临时监控。（注：SPAN 仍然是访问某些类型数据的唯一方式，例如在同一交换机上跨端口到端口的数据）
		- <img src="https://raw.githubusercontent.com/MarsAuthority/redblue/master/assets/Pasted%20image%2020221104135009.png">
- # 2. 东西向流量采集
	- 传统TAP和SPAN的方案在东西向流量采集有两个难点：**1、无法采集同一宿主机内的两台虚拟机之间的流量。2、封装了vxlan等隧道包头的数据流量无法识别**。
	- 业界主流的方案有（TAP/SPAN的基础上）：
		- SDN：通过SDN控制器的全网控制能力，实现东西向流量采集
		- API接口：通过调用云计算系统的标准引流API，实现东西向流量采集
		- 代理：非SDN网络下，添加一个引流的代理，实现东西向流量采集
		- Agent：通过在终端部署软件来实现东西向流量采集
		- vxlan：在虚机上做镜像，把流量通过vxlan导出去，送到一台支持vxlan终结的交换机上去，这台交换机把vxlan剥离后，送给旁挂的设备进行流量采集
- # 3. Gigamon Solution
	- **无源TAP**
		- 无源TAP是在其网络端口之间没有物理隔离的设备，它从网络的两个方向接收流量，以便在监视工具上看到100％的流量。无源TAP不需要任何电源，增加了一层冗余，几乎不需要维护，并减少了总费用。
	- **有源TAP**
		- 有源TAP在网络端口之间具有物理隔离。因此，它们需要一种故障安全机制，以确保在TAP断电时网络保持可操作状态；**有源TAP通常在下面的场景使用（无源TAP不适用）**：
			1. Locations where the light levels are too low to use a splitter → regeneration provides a viable solution
			2. Copper infrastructures → where electricity is used to move electrons (instead of photons)
			3. Signal conversions → since an active TAP regenerates the signal anyway, it can also be designed to create a signal of a different type (such as 10Gb SR converted to 10Gb LR)
			4. SFP-based links that cannot otherwise be broken (such as TwinAX cabling) → regeneration works here as well
	- **虚拟TAP**
		- 用于网络功能虚拟化 (NFV) 以及私有和公共云基础架构，以访问托管环境中的东西向和南北向流量。
		- <img src="https://raw.githubusercontent.com/MarsAuthority/redblue/master/assets/Pasted%20image%2020221104135055.png">
- # 参考资料
	- https://www.gigamon.com/resources/resource-library/white-paper/to-tap-or-to-span.html
	- https://www.gigamon.com/resources/resource-library/white-paper/understanding-network-taps-first-step-to-visibility.html
	- https://www.secrss.com/articles/12651
	- http://blog.nsfocus.net/east-west-flow-sum