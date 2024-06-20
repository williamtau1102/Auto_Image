#eMMC #UFS

# 参考资料

1. Universal Flash Storage (UFS) Version 4.0 (JESD220F)
2. Embedded Multi-Media Card (e•MMC) Electrical Standard (5.1A) (JESD84-B51A)

# 一 非易失性存储器

Non-Volatile Memory,NVM 主要由三部分构成：

1. NAND：存储器的载体、直接决定了存储器的容量大小，其素质也在一定程度上影响存储器的稳定性，如读写次数等。

2. 控制器：对存储器的稳定性有重要影响，也关系到数据的传输速度。如果一个程序反复擦除写入同一闪存单元，会产生坏道故障，而控制器则尽可能有序地将工作压力均匀于每一个存储器单元上。存储器设备都包括某种程度的垃圾收集，一旦每块存储器单元都已被写入一次，存储器控制器将会标记碎片文件，将其换成正等待被删除，新的数据可以被写入，不同的控制器处理速度不同，直接影响存储器性能。

3. 接口：是传输数据的通路，对存储器的数据传输速率起到决定性作用。

![[Pasted image 20240617102149.png]]

# 二 e.MMC

## 2.1 e.MMC 简介

e.MMC 全称为"embedded Multi Media Card,嵌入式多媒体存储卡"，eMMC 的明显优势是在封装中集成了一个控制器，它提供标准接口并管理闪存。

eMMC 闪存基于并行数据传输技术，其内部存储单元与主控之间拥有 8 个数据通道，传输数据时 8 个通道同步工作，工作模式为半双工，即每个通道都可以进行读写传输，但同一时刻只能执行读或写的操作。

![image.png](https://cdn.nlark.com/yuque/0/2023/png/22935284/1696992746241-152c90c2-541c-4150-af07-5f7c7229c235.png#averageHue=%23e4e4e1&clientId=u0ff26b48-0764-4&from=paste&height=263&id=ue17908db&originHeight=339&originWidth=512&originalType=url&ratio=1&rotation=0&showTitle=false&size=80742&status=done&style=none&taskId=u23a5482d-52d0-47d4-8197-3b35fe668e3&title=&width=397)

eMMC 的技术前身是 MMC，后来为了突出这个设备是嵌入在电路板上，在前面加入了 embedded，所以前面的字母是小写；

![image.png](https://cdn.nlark.com/yuque/0/2023/png/22935284/1696992746257-2911ce88-ee6d-4acd-b063-59d020c4a39b.png#averageHue=%23a4a08f&clientId=u0ff26b48-0764-4&from=paste&height=232&id=ub230b2ae&originHeight=280&originWidth=611&originalType=url&ratio=1&rotation=0&showTitle=false&size=132125&status=done&style=none&taskId=uc8e706c5-72bc-4f70-9d37-3f532fffbfa&title=&width=506)

JEDEC(固态技术协会)制定的标准规格。这种标准与 eMMC 的区别主要在于接口，eMMC 采用的是 8bit 并行接口，随着技术升级，标准理论值最高可以达到**400MB/s**，但这一速率无法满足逐渐的传输需求，所以 UFS 出现；

![image.png](https://cdn.nlark.com/yuque/0/2023/png/22935284/1696992746301-0b01c149-cb78-4820-b522-6c9e6b117513.png#averageHue=%23d3dbc5&clientId=u0ff26b48-0764-4&from=paste&id=u1d75711e&originHeight=433&originWidth=1024&originalType=url&ratio=1&rotation=0&showTitle=false&size=227671&status=done&style=none&taskId=u27dfac3b-14a6-4c9f-b20c-8f704d0b16e&title=)

eMMC 采用的是**半双工并行通道**，**每位数据都必须保持同一步调并排前进，效率会受到影响**；eMMC 只支持单通道，**不能同时进行读写数据**，也不支持多线程；想提升性能，只能不断改进 eMMC 的总线接口数量；

![[Pasted image 20240617113401.png|725]]

## 2.2 e.MMC 引脚介绍

![[Pasted image 20240617113529.png|725]]

- DAT : Data lines are bidirectional signals.Host adn iNAND operate in push-pull mode.
	支持1bit/4bit/8bit三种模式。
- CMD : A bidirectional channel used for device initialization and command transfers.

- CLK : Each cycle directs a 1-bit transfer on the command and DAT lines.
	可变时钟频率范围：0-20MHz、0-26MHz和0-52MHz。
- RST_n : Hardware Reset.

- VCC : Positive supply voltage for Core.
	3.3V
- VCCQ : Positive supply voltage for I/O.
	3.3V or 1.8V

![[Pasted image 20240617113229.png|800]]

- VDDi : Internal power mode. Connect 0.1uF capacitor from VDDi to ground.

## 2.3 工作原理介绍

![[Pasted image 20240617114852.png|775]]

# 三 UFS

UFS 全称为"Universal Flash Storage,通用闪存存储"，同样一种内嵌式存储器标准规格，同样是整合有主控芯片的闪存。但是它使用的是 PC 平台上常见的 SCSI 结构模型并支持对应的 SCSI 指令集，速度远超 eMMC5.1。

UFS 闪存则是基于串行数据传输技术，其内部存储单元与主控之间虽然只有两个数据通道，但由于采用串行数据传输，其实际数据传输时速远超基于并行技术的 eMMC 闪存；而且其支持全双工模式，所有数据均可以同时执行读写操作，在数据读写响应速度上也比 eMMC 要快的多。

UFS 采用的是**全双工差分信号**，**串行通道；可以同时进行读写数据**，UFS3.0 的接口带宽最高速率可达 23.2Gbps=2.9GB/s(UFS2.1 的最大接口速率是 11.6Gbps)。由 Flash 颗粒和 Flash controller 组成。

![image.png](https://cdn.nlark.com/yuque/0/2023/jpeg/22935284/1696992746219-ca613eea-c88e-4ceb-96ce-8a59fa22b267.jpeg#averageHue=%23e9e7b7&clientId=u0ff26b48-0764-4&from=paste&id=u4448f28d&originHeight=360&originWidth=500&originalType=url&ratio=1&rotation=0&showTitle=false&size=29585&status=done&style=none&taskId=u865e14d1-78ce-4554-89ff-e7ffcd5eac9&title=)

![[Pasted image 20240604154043.png|500]]

5G 网的最高理论速率可达 20Gbps，也就是 2.5GB/s，只有 UFS3.0 的带宽才能与 5G 密切结合。

![[Pasted image 20240604154105.png]]

![image.png|750](https://cdn.nlark.com/yuque/0/2023/png/22935284/1696992746269-a246f66e-ac39-4f57-8790-590b3f41a3dc.png#averageHue=%23f9f8f6&clientId=u0ff26b48-0764-4&from=paste&id=u8be3c633&originHeight=695&originWidth=1557&originalType=url&ratio=1&rotation=0&showTitle=false&size=103654&status=done&style=none&taskId=ua13fda19-7455-41c7-b243-f59c7d79c05&title=)

![image.png](https://cdn.nlark.com/yuque/0/2023/png/22935284/1696992748893-4c6feb54-d8c7-44d0-bf4f-b6b9ce4b8458.png#averageHue=%23d1d0c9&clientId=u0ff26b48-0764-4&from=paste&id=u7b9ec092&originHeight=865&originWidth=1807&originalType=url&ratio=1&rotation=0&showTitle=false&size=1055572&status=done&style=none&taskId=uf3d90211-664c-4056-b6ad-2c50ffff07e&title=)<br />![image.png](https://cdn.nlark.com/yuque/0/2023/png/22935284/1696992748987-4c984459-93b2-425f-9dd2-6bba20efd345.png#averageHue=%23d0d0c8&clientId=u0ff26b48-0764-4&from=paste&id=u7f4036f9&originHeight=869&originWidth=1796&originalType=url&ratio=1&rotation=0&showTitle=false&size=1169120&status=done&style=none&taskId=ua245f81d-30c8-47d7-a889-b7661a401ec&title=)

## 3.1 UFS 架构

UFS 芯片主要分为**UFS controller**和**NAND Flash**两部分，另外还有一个很重要的接口**M-PHY**。

![image.png](https://cdn.nlark.com/yuque/0/2023/png/22935284/1696992748190-0bc06f71-1506-408c-a927-0a81eadeb40d.png#averageHue=%23f4f3f3&clientId=u0ff26b48-0764-4&from=paste&id=ua218682b&originHeight=463&originWidth=916&originalType=url&ratio=1&rotation=0&showTitle=false&size=44180&status=done&style=none&taskId=u191a686e-bf89-4894-8001-4be3f943acb&title=)

![image.png](https://cdn.nlark.com/yuque/0/2023/png/22935284/1696992748308-aeabf00c-e034-49c7-bc09-83a3a5ce043d.png#averageHue=%23c1cdb6&clientId=u0ff26b48-0764-4&from=paste&id=u960d1221&originHeight=482&originWidth=1024&originalType=url&ratio=1&rotation=0&showTitle=false&size=352723&status=done&style=none&taskId=u2ff32220-e6b2-4afc-8446-6bd0cce3514&title=)

## 3.2 引脚功能介绍

![image.png](https://cdn.nlark.com/yuque/0/2023/png/22935284/1696992748258-16a275a3-13d2-4197-a078-8c0bbcbe428d.png#averageHue=%23f8f8f7&clientId=u0ff26b48-0764-4&from=paste&id=uc089e206&originHeight=746&originWidth=1024&originalType=url&ratio=1&rotation=0&showTitle=false&size=257113&status=done&style=none&taskId=uc0651205-1b62-4a59-bcd3-61027c794fa&title=)

### 3.2.1 供电引脚

UFS 共有两路供电，其中 VCC 是为 Nand Flash 颗粒供电，VCCQ 是为 controller 和 PHY interface 供电。两者供电都由 LDO 输出。

![image.png](https://cdn.nlark.com/yuque/0/2023/png/22935284/1696992750636-5d5e1cc5-597f-4ac9-867c-559c87f34b70.png#averageHue=%23ebdecb&clientId=u0ff26b48-0764-4&from=paste&id=ufe5bdd53&originHeight=105&originWidth=1029&originalType=url&ratio=1&rotation=0&showTitle=false&size=12596&status=done&style=none&taskId=u94d6eece-0cfb-44e7-9cdc-28c49f48fb1&title=)

![image.png](https://cdn.nlark.com/yuque/0/2023/png/22935284/1696992750426-c29315ed-55fa-4051-a773-7d501f9ea004.png#averageHue=%23f1f0ef&clientId=u0ff26b48-0764-4&from=paste&id=u8fab8cdb&originHeight=249&originWidth=1024&originalType=url&ratio=1&rotation=0&showTitle=false&size=66059&status=done&style=none&taskId=ucde83485-2812-4985-974a-0d7e935a069&title=)

### 3.2.2 信号引脚

UFS 的信号线共十根，其中复位信号 1 根，时钟信号 1 根，全双工差分数据线 8 根（RXD 2lane 差分；TXD 2lane 差分）。

![image.png](https://cdn.nlark.com/yuque/0/2023/png/22935284/1696992750953-62f54bf9-27e6-4ee3-888f-022ec7d8a75e.png#averageHue=%23f2eee8&clientId=u0ff26b48-0764-4&from=paste&id=u96535cfe&originHeight=537&originWidth=1024&originalType=url&ratio=1&rotation=0&showTitle=false&size=161411&status=done&style=none&taskId=uc9f5758f-554e-4250-a12e-235722b1c2f&title=)

## 3.3 时钟特性

![image.png](https://cdn.nlark.com/yuque/0/2023/png/22935284/1696992751469-56ecb7e0-fb72-4777-b5fa-3f53480e576b.png#averageHue=%23f3f3f2&clientId=u0ff26b48-0764-4&from=paste&id=u04316bfc&originHeight=446&originWidth=1024&originalType=url&ratio=1&rotation=0&showTitle=false&size=89782&status=done&style=none&taskId=u45d6f0c7-16c6-45d1-b02d-56c8ed70cdb&title=)

## 3.4 性能/模式介绍

### 3.4.1 性能介绍

特别要注意的是顺序读写速度的单位是 MB/s；随机读写速度的单位是 IOPS(Input/Output Operation Per Second)，指的是单位时间内系统能处理的 I/O 请求数量。

随机读写频繁的应用，如小文件存储(图片)、OLTP 数据库、邮件服务器，关注随机读写性能，IOPS 是关键衡量指标。

顺序读写频繁的应用，传输大量连续数据，如电视台的视频编辑，视频点播 VOD(Video On Demand)，关注连续读写性能。数据吞吐量是关键衡量指标。

IOPS 和数据吞吐量适用于不同的场合：

读取 10000 个 1KB 文件，用时 10 秒 Throught(吞吐量)=1MB/s ，IOPS=1000 追求 IOPS

读取 1 个 10MB 文件，用时 0.2 秒 Throught(吞吐量)=50MB/s, IOPS=5 追求吞吐量

简而言之：

**磁盘的 IOPS，也就是在一秒内，磁盘进行多少次 I/O 读写。**

**磁盘的吞吐量，也就是每秒磁盘 I/O 的流量，即磁盘写入加上读出的数据的大小**。

![image.png](https://cdn.nlark.com/yuque/0/2023/png/22935284/1696992752502-b20a9ee5-a49f-48eb-997d-a034b4d0678b.png#averageHue=%23c7af94&clientId=u0ff26b48-0764-4&from=paste&id=u33d54366&originHeight=298&originWidth=1024&originalType=url&ratio=1&rotation=0&showTitle=false&size=111047&status=done&style=none&taskId=u43319565-c998-4e68-ba89-9337e469185&title=)

### 3.4.2 模式介绍

1、Support for High Speed Gears : up to HS-GEAR4 2Lane <br />2、Low-speed mode (PWM) : LS mode (Gear 1) <br />3、High-speed burst mode : HS mode (Gear 1,2,3,4)

Gear1-4：代表的是传输速率，数字每大 1 带宽增加一倍。

## 3.5 UFS4.0 新 feature 总览

![image.png](https://cdn.nlark.com/yuque/0/2023/png/22935284/1696992752549-3607e2f4-09dc-4abd-85b0-d08e647c6844.png#averageHue=%23c1cec6&clientId=u0ff26b48-0764-4&from=paste&id=ud20b4c3f&originHeight=532&originWidth=1024&originalType=url&ratio=1&rotation=0&showTitle=false&size=244107&status=done&style=none&taskId=u01f0638c-ebef-4e80-ae8d-3f766c777d8&title=)

adb shell cat proc/devinfo/ufs\* # 获取 UFS 的 chipID

### 3.5.1 写入增强器(Write Booster)

也叫 Write Turbo，利用 TLC 来模拟 SLC 缓存技术。SCL 的每个存储单元可以存储 1bit 数据，这种工作模式可靠、读写速度快、性能好。 TLC 对于控制电压的控制要求更高，需要 8 个电位。

![image.png](https://cdn.nlark.com/yuque/0/2023/png/22935284/1696992840353-b95d575a-0ce6-4087-8d80-7f0095e7f83a.png#averageHue=%237dc0d0&clientId=u348a221a-974b-4&from=paste&id=u207a2f4c&originHeight=342&originWidth=899&originalType=url&ratio=1&rotation=0&showTitle=false&size=302374&status=done&style=none&taskId=u9f84b3f0-3887-4a5c-b2e9-3036850e456&title=)

### 3.5.2 Deep Sleep

### 3.5.3 性能调整通知(Performance Throttling Notification)

闪存在温度过高时会影响其性能，该功能就是当温度过高时，UFS 控制器告诉系统该情况。

### 3.5.4 Host Performance Booster

Host Control Mode。提升器件的随机读的性能，降低由于存储碎片化引起的卡顿。让 UFS 控制器把映射表缓存在手机内存中。

![image.png](https://cdn.nlark.com/yuque/0/2023/png/22935284/1696992840330-69d5a78b-4d76-4a5e-a2b6-16348795c768.png#averageHue=%23cfccc4&clientId=u348a221a-974b-4&from=paste&id=u6fdc6554&originHeight=221&originWidth=1907&originalType=url&ratio=1&rotation=0&showTitle=false&size=248744&status=done&style=none&taskId=u13c62d20-2948-44e5-9ebc-ca3eb0f1168&title=)
