#DDR 
# 一 参考资料

1.  [https://xilinx.eetrend.com/blog/2021/100555905.html](https://xilinx.eetrend.com/blog/2021/100555905.html) 
2.  [http://www.wowotech.net/basic_tech/307.html](http://www.wowotech.net/basic_tech/307.html) 
3.  [https://adtxl.com/index.php/archives/413.html](https://adtxl.com/index.php/archives/413.html) 
4. JEDEC 相关资料
5. [DDR 基础和一些个人理解](https://zhuanlan.zhihu.com/p/622689457?utm_id=0&wd=&eqid=fa6e89bc001359150000000364620289)

---

# 二 DDR Subsystem 组成介绍

DDR Subsystem 由一条链路构成，框架大致如下图所示：
![](https://cdn.nlark.com/yuque/0/2023/png/22935284/1696861389566-ae9330f5-220a-4d6b-ad2a-28e86fe4104b.png#averageHue=%23eeeeee&id=Uuzbj&originHeight=183&originWidth=797&originalType=binary&ratio=1&rotation=0&showTitle=false&status=done&style=none&title=)![|850](https://cdn.nlark.com/yuque/0/2023/png/22935284/1696861389845-047c9e96-f21d-41c2-8ead-bcf4f8bc148e.png#averageHue=%23eaf4f2&height=435&id=OJ105&originHeight=708&originWidth=1288&originalType=binary&ratio=1&rotation=0&showTitle=false&status=done&style=none&title=&width=792)  

## 2.1 用户逻辑/系统层

如 CPU、GPU、NPU 等需要使用 DDR 存储的算力模块。

## 2.2 BUS/NOC

参考：[[13-BUS_NOC]]

用户逻辑和控制器之间的接口是用户定义的，不是标准定义。当用户逻辑向控制器发出读或写请求时，它会发出一个逻辑地址，然后控制器将此逻辑地址转换为物理地址并向 PHY 发出命令。 

## 2.3 DMC

Dynamic Memory Controller：DDR 控制器，负责初始化 DRAM，并重排读写命令以获得最大 DRAM 带宽。

用户逻辑与控制器之间的接口没有标准化，取决于设计。用户逻辑向 MC 发出读写命令时，其中的地址使用的是逻辑地址，**MC 再将逻辑地址转换为物理地址**，将用户逻辑的命令转换后向 PHY 发出。

![b975a2df069cb151fc7f94a4be7777d.jpg|950](https://cdn.nlark.com/yuque/0/2024/jpeg/22935284/1715060085669-347f8a78-6ea5-470d-b250-ef6fc174eb98.jpeg#averageHue=%23a0adae&clientId=ueabe37e0-bc7a-4&from=paste&height=813&id=u8633d59e&originHeight=3072&originWidth=4096&originalType=binary&ratio=1&rotation=0&showTitle=false&size=14145870&status=done&style=none&taskId=u649ac709-543d-4241-abfa-a371444f5e9&title=&width=1084)

### 2.3.1 逻辑地址与物理地址

当从存储介质中读取数据时，需要提供读数据的地址；在写入数据时，除地址外还需提供写数据。**由用户提供的地址，一般称为“逻辑地址”（logical address）**。逻辑地址在传输给 DRAM 前，会转换为物理地址，**转换工作通常由 MMU 完成**。

## 2.4 DFI

DDR PHY Interface，**DMC 与 PHY 之间的标准化协议接口**，符合 DFI version 2.0, 2.1, 3.0, 3.1, 4.0 and 5.0 Specifications。由于 DDR PHY 和 Controller 这两部分 IP 往往由不同设计者完成，为了实现两者之间的标准互联，需要一种 MC 和 PHY 之间的**标准通信接口，即为 DFI**，以提高独立模块利用率，降低成本，缩短项目周期。

![|650](https://cdn.nlark.com/yuque/0/2023/png/22935284/1696861390608-ba3d9040-008a-41d8-8688-0311d9d60702.png#averageHue=%23edebea&height=364&id=DXYlq&originHeight=792&originWidth=1406&originalType=binary&ratio=1&rotation=0&showTitle=false&status=done&style=none&title=&width=646)

常用的 DDR234、LPDDR234 的 ckr 的比例关系，即 dfi_clk1x : ck = 1:2；

![](https://cdn.nlark.com/yuque/0/2023/png/22935284/1696861391043-4f03ba5e-2b57-4573-9bbc-83257372c9c6.png#averageHue=%23efefef&height=717&id=k74C1&originHeight=830&originWidth=655&originalType=binary&ratio=1&rotation=0&showTitle=false&status=done&style=none&title=&width=566)

## 2.5 PHY

**DDR PHY 将 MC 命令转换为具体的底层信号，驱动 DRAM 的物理 IO 接口**，**PHY 与 DRAM 之间的接口由 JEDEC 标准化**。所以 MC 相当于读写 DDR 的大脑，而 PHY 是做出反应的肌肉。

PHY 是连接数字逻辑和物理电路的必要环节，通常这部分设计由模拟电路完成，对厂家和工艺的局限性较高。DDR PHY 主要由 3 部分构成：**延时链、控制信号逻辑和数据信号的串并转换**。

延时链，这个部分是和普通数字电路区别最大的地方：普通的数字设计希望延时越小越好，最好没有延时。但是这里要用到的恰恰就是门电路的基本延时特性。虽然写出来就是简单的 buffer 和 mux，但是在挑选器件时却要小心，要选用那些上升沿和下降沿时间特性一致的器件。而且在后端实现时，还要考虑到“线”延时对远端延时带来的额外延时。延时电路的好坏，直接决定 PHY 是否能工作。

## 2.6 SDRAM/颗粒

DDR SDRAM，Double Data Rate Synchronous Dynamic Random Access Memory，同步动态随机存取存储器。同步是指需要同步时钟，动态是指需要不停刷新和补充电量来保持数据，随机是指可以随机存读任意位置的数据。

PHY 与颗粒之间是 SDRAM 接口。

---

# 三 DDR SDRAM 介绍

## 3.1 DDR SDRAM 分类  

![](https://cdn.nlark.com/yuque/0/2023/png/22935284/1696861392274-b2dae3cc-e926-4239-ab6f-a21687615b9f.png#averageHue=%23fee598&id=lwz7t&originHeight=559&originWidth=670&originalType=binary&ratio=1&rotation=0&showTitle=false&status=done&style=none&title=)![](https://cdn.nlark.com/yuque/0/2023/png/22935284/1696861392514-d3ae64e0-4997-435e-839d-aa4e075c25ae.png#averageHue=%23fee598&id=TKKOO&originHeight=414&originWidth=674&originalType=binary&ratio=1&rotation=0&showTitle=false&status=done&style=none&title=)

![](https://cdn.nlark.com/yuque/0/2023/webp/22935284/1698650168327-5c77af81-73c2-4734-9573-1efdd1efe52d.webp#averageHue=%23b4c0a6&clientId=u24f34a98-ffbd-4&from=paste&id=ALLL5&originHeight=260&originWidth=520&originalType=url&ratio=1&rotation=0&showTitle=false&status=done&style=none&taskId=u1fbf1596-a7e2-4267-9fde-0ecf2c3225c&title=)

![](https://cdn.nlark.com/yuque/0/2023/png/22935284/1697447611644-ced74066-088e-4e10-be3c-82a062f75205.png#averageHue=%23dedddc&clientId=u68238361-3e74-4&from=paste&id=A1guK&originHeight=450&originWidth=679&originalType=url&ratio=1&rotation=0&showTitle=false&status=done&style=none&taskId=u2cdc3d30-119e-421f-ba7a-eae3d3ebd18&title=)

![image.png](https://cdn.nlark.com/yuque/0/2023/png/22935284/1697162525883-538822aa-3db1-4c13-b49a-c97a6208e1ba.png#averageHue=%23f8f6f4&clientId=u0afd0ec9-4b1b-4&from=paste&height=545&id=C7jXh&originHeight=545&originWidth=432&originalType=binary&ratio=1&rotation=0&showTitle=false&size=33903&status=done&style=none&taskId=u241a270d-72e8-4c66-908d-b1f8ef26946&title=&width=432)

![](https://cdn.nlark.com/yuque/0/2023/jpeg/22935284/1697178485772-c03ad595-0355-4d6d-9c39-cd83a4d2252f.jpeg#averageHue=%23e3e3e3&clientId=u0afd0ec9-4b1b-4&from=paste&id=V8Uhp&originHeight=395&originWidth=640&originalType=url&ratio=1&rotation=0&showTitle=false&status=done&style=none&taskId=u93896eba-4f05-4eaf-b0e4-b5a776f410a&title=)

### 3.1.1 DDR3

### 3.1.2 DDR4

DDR4 相对 DDR3 增加了 20 多项新特性。

![image.png](https://cdn.nlark.com/yuque/0/2023/png/22935284/1696989852635-b00e5c1b-bc25-436c-801b-6221510ce4e5.png#averageHue=%23f9f9f9&clientId=u33443cfc-91a7-4&from=paste&id=Ng2nv&originHeight=726&originWidth=872&originalType=url&ratio=1&rotation=0&showTitle=false&size=238299&status=done&style=none&taskId=ufa29fd22-09a0-4a52-8fe5-bc5c66adf50&title=)

![image.png](https://cdn.nlark.com/yuque/0/2023/png/22935284/1696989852567-c6bd5b97-6e6d-40f4-bfa2-e0894fa42845.png#averageHue=%23dbdce2&clientId=u33443cfc-91a7-4&from=paste&id=hvfGQ&originHeight=115&originWidth=870&originalType=url&ratio=1&rotation=0&showTitle=false&size=23643&status=done&style=none&taskId=u9c82e282-e5e5-4481-baac-409c069dc68&title=)

![image.png](https://cdn.nlark.com/yuque/0/2023/png/22935284/1696989852584-e8a4830a-d862-4013-8337-a4a7d4b6ae80.png#averageHue=%23fbfbfb&clientId=u33443cfc-91a7-4&from=paste&id=Wm1ln&originHeight=301&originWidth=863&originalType=url&ratio=1&rotation=0&showTitle=false&size=61599&status=done&style=none&taskId=u0449437e-dc22-43a0-b036-778cbea7d6b&title=)

X4/X8 : 16 个 banks(4 个 bank groups，每个 BG 有 4 个 banks) 

X16 : 8 个 banks(2 个 bank groups，每个 BG 有 4 个 banks) 

8n prefetch architecture , transfer two data words per clock cycle at the I/O pins

![image.png](https://cdn.nlark.com/yuque/0/2023/png/22935284/1696989852621-50b6e913-abaf-4cff-9a5f-8a92002252e2.png#averageHue=%23fafafa&clientId=u33443cfc-91a7-4&from=paste&id=HvWau&originHeight=660&originWidth=1370&originalType=url&ratio=1&rotation=0&showTitle=false&size=195931&status=done&style=none&taskId=u8c642aaa-eac7-49be-939c-0e8ed7e9b4c&title=)

![image.png](https://cdn.nlark.com/yuque/0/2023/png/22935284/1696989852693-4296c15c-9702-414a-8313-1abd2b2511f0.png#averageHue=%23fafafa&clientId=u33443cfc-91a7-4&from=paste&id=qVxj6&originHeight=672&originWidth=1377&originalType=url&ratio=1&rotation=0&showTitle=false&size=212559&status=done&style=none&taskId=u95f798a5-f1e6-48d0-949a-690134cf9df&title=)

![image.png](https://cdn.nlark.com/yuque/0/2023/png/22935284/1696989853556-d5817f45-b511-43b2-80d4-e3ea016e0d8c.png#averageHue=%23fbfbfb&clientId=u33443cfc-91a7-4&from=paste&id=lt4pS&originHeight=659&originWidth=1389&originalType=url&ratio=1&rotation=0&showTitle=false&size=188787&status=done&style=none&taskId=ufd2474d3-bae8-4616-96bf-11086dd53e9&title=)

### 3.1.3 DDR5

DDR5 高性能新增特征包括：突发长度增加到 16 beats，出色的刷新/预充电方案可实现更高的性能，提高通道利用率的双通道 DIMM 架构、DDR5 DIMM 上集成了稳压器、为实现更高性能而增加的存储区分组，以及命令/地址片内端接电阻 (ODT）等。

![image.png](https://cdn.nlark.com/yuque/0/2023/png/22935284/1697444950320-cdb8cf45-ef79-4055-80e9-e2551bbeb8e7.png#averageHue=%23d0c7dc&clientId=u49f79e6a-7743-4&from=paste&height=208&id=u8a5dba90&originHeight=225&originWidth=843&originalType=binary&ratio=1&rotation=0&showTitle=false&size=116586&status=done&style=none&taskId=ue456b69f-ee08-4431-b0f8-e7cd147289c&title=&width=780)

除性能更高之外，DDR5 还引入多种 RAS 功能，以保持通道提速后的稳定性。这些实现了较高 DDR5 通道稳定性的功能包括：占空比调节器 (DCA)、片上 ECC、DRAM 接收 I/O 均衡、RD 和 WR 数据的循环冗余校验 (CRC) 以及内部 DQS 延迟监控。以下内容对这些功能进行了逐个说明。

**用于补偿占空比失真的占空比调节器 (DCA)。**

占空比调节器支持主机通过调节 DRAM 内部的占空比来补偿所有 DQS（数据选通）/DQ（数据）引脚上的占空比失真。因此，DCA 功能巩固了读取数据的稳定性。

**强化 RAS 的片上 ECC**<br />DDR5 DRAM 为每 128 位数据设置 8 位的 ECC 存储空间。从而使得片上 ECC 成为一种强大的 RAS 功能，可以保护存储器阵列免受单个数位错误的影响。

**DRAM 获得 DQ 均衡以增加裕量**

DDR5 DRAM 和 LPDDR5 DRAM 一样，也支持 WR 数据均衡。该功能在 DRAM 端为 WR DQ 打开了局面，可以保护通道免受符号间干扰 (ISI) 的影响，增加了裕量，并实现了更高的数据速率。

**RD/WR 数据的循环冗余校验 (CRC)**<br />DDR4 仅支持写数据使用 CRC，而 DDR5 将 CRC 的适用范围扩展到读数据，从而提供额外保护，避免通道出错。

**内部 DQS 延迟监控。**

内部 DQS 延迟监控机制支持主机调整 DRAM 延迟，以补偿电压和温度变化。以 DDR5 速度运行的主机可以使用此功能定期重新训练通道，补偿 DRAM 中延迟引起的 VT 变化。

![image.png](https://cdn.nlark.com/yuque/0/2023/png/22935284/1697077926450-7987e4b2-10d1-448b-8785-c289a2ea968f.png#averageHue=%23f3f3f3&clientId=ub8d0bc13-bfcf-4&from=paste&height=838&id=u9b043cc2&originHeight=838&originWidth=1583&originalType=binary&ratio=1&rotation=0&showTitle=false&size=241456&status=done&style=none&taskId=u233e8dff-f0ba-4e32-8e7d-eb3f2f1c3f9&title=&width=1583)

![image.png](https://cdn.nlark.com/yuque/0/2023/png/22935284/1697077944229-3d6d49dc-16b0-463a-adf4-17005c2f8ce0.png#averageHue=%23f3f3f3&clientId=ub8d0bc13-bfcf-4&from=paste&height=854&id=u7b55f60d&originHeight=854&originWidth=1578&originalType=binary&ratio=1&rotation=0&showTitle=false&size=237678&status=done&style=none&taskId=u8819c903-1f34-4e7b-bff7-0aad2e10111&title=&width=1578)

## 3.2 最小内存单元 CELL

DRAM Storage Cell 每存储一个 bit 数据只需要一个电容，**但是电容不可避免会漏电，如果电荷不足会导致数据出错，因此电容需要被周期性刷新(预充电)，通过电容的充放电来保持数据**。由以下四个部分组成：

- Storage Capacitor，即存储电容，它通过存储在其中的电荷的多和少，或者电容两端电压差的高和低，来表示逻辑上的 1 和 0。
- Access Transistor，即访问晶体管，它的导通和截止，决定了允许或禁止对 Storage Capacitor 所存储的信息的读取和改写。
- Wordline，即字线，连接在晶体管的基极。它决定了 Access Transistor 的导通或者截止。

**此处有一个问题：当 Storage Capacitor 存储的信息为 1 时，靠近 S 极的电容引脚电压为 Vcc（即 VDD），此时如果想打开晶体管（Vgs>0），必须提供一个高于 VDD 的电压，由此引出了 VPP。**

**VPP 即为字线电压，需要外供。DDR4 中 VDD=1.2V，VPP=2.5V。之所以需要外供的原因是为了不用内建电荷泵 Charge pump，以达到省功耗的目的。而 DDR3 本身是不需要的，在其内部有电压泵，不需要外供。**

**在 DDR 内存中，VPP 必须始终大于或等于 VDD/VDDQ，这是因为 VPP 用于编程和擦除 DDR 存储芯片内存（非易失性存储器，如 EEPROM 和闪存等）中的数据。VPP 通常用于在编程或擦除期间提供高电压，以改变存储器芯片的状态。如果 VPP 电压低于 VDD/VDDQ，则无法正确编程或擦除存储器，可能会导致存储器错误或损坏。**

**另外，VPP 电压不能晚于 VDD/VDDQ 上电，这是因为在上电时，VDD/VDDQ 电压会先上升到正常工作电压，而 VPP 电压上升的速度可能会比 VDD/VDDQ 慢，如果 VPP 电压晚于 VDD/VDDQ 上电，就会导致内存无法正常工作。因此，为了保证 DDR 内存的正常工作，VPP 必须始终大于或等于 VDD/VDDQ，并且不能晚于 VDD/VDDQ 上电。**

Bitline，即位线，它是外界访问 Storage Capacitor 的唯一通道，当 Access Transistor 导通后，外界可以通过 Bitline 对 Storage Capacitor 进行读或写操作。

Storage Capacitor 的 Common 端接在 Vcc/2（**相当于外部供电的 VREFCA，即 VDD/2**）。当 Storage Capacitor 存储的信息为 1 时，另一端电压为 Vcc，此时其所存储的电荷 Q = +Vcc/2 / C；当 Storage Capacitor 存储的信息为 0 时，另一端电压为 0，此时其所存储的电荷 Q = -Vcc/2 / C。 

![|500](https://cdn.nlark.com/yuque/0/2023/png/22935284/1696861395657-4807b60e-5bef-41e9-960b-e115a7156306.png#averageHue=%23fcfcfa&height=351&id=HNUp1&originHeight=461&originWidth=836&originalType=binary&ratio=1&rotation=0&showTitle=false&status=done&style=none&title=&width=636)

![](https://cdn.nlark.com/yuque/0/2023/png/22935284/1696861395389-75a7aa63-9c66-4a37-abe0-ef96d1b27f6e.png#averageHue=%23fafafa&height=156&id=ZOM47&originHeight=135&originWidth=500&originalType=binary&ratio=1&rotation=0&showTitle=false&status=done&style=none&title=&width=576)

**写操作**：先把要写入的电平状态设定到 Bitline 上，然后打开 Access Transistor，通过 Bitline 改变 Storage Capacitor 内部的状态。写 1，通过 Bitline 信号将电容充电至 Vcc；写 0，通过 Bitline 信号将电容放电至 0。

**读操作**：Wordline 设为逻辑高电平，打开 Access Transistor，然后读取 Bitline 上的状态。
(1) Bitline 预充电到 Vref(Vcc /2) ，再打开 Wordline 选通；
(2) 如果 Cell 电容数据为 1(Vcc)，则 Bitline 电压>Vcc/2；如果 Cell 电容数据为 0(0V), 则 Bitline 电压<Vcc/2；
(3) Sense 电路对 Bitline 进行放大和采样对比输出数据。

以上操作存在以下问题，导致实际设计中引入了 **Differential Sense Amplifier**：
1、外界的逻辑电平与 Storage Capacitor 的电平不匹配。
由于 Bitline 的电容值比 Storage Capacitor 要大的多（通常为 10 倍以上），当 Access Transistor 导通后，如果 Storage Capacitor 存储的信息为 1 时，Bitline 电压变化非常小。外界电路无法直接通过 Bitline 来读取 Storage Capacitor 所存储的信息。

2、进行一次读取操作后，Storage Capacitor 存储的电荷会变化<br /> 在进行一次读取操作的过程中，Access Transistor 导通后，由于 Bitline 和 Storage Capacitor 端的电压不一致，会导致 Storage Capacitor 中存储的电荷量被改变。最终可能会导致在下一次读取操作过程中，无法正确的判断 Storage Capacitor 内存储的信息。

3、由于 Capacitor 的物理特性，即使不进行读写操作，其所存储的电荷都会慢慢变少<br /> 这个特性要求 DRAM 在没有读写操作时，也要主动对 Storage Capacitor 进行电荷恢复的操作。

Differential Sense Amplifier 包含 **Sensing Circuit** 和 **Voltage Equalization Circuit** 两个主要部分。它**主要的功能就是将 Storage Capacitor 存储的信息转换为逻辑 1 或者 0 所对应的电压，并且呈现到 Bitline 上。同时，在完成一次读取操作后，通过 Bitline 将 Storage Capacitor 中的电荷恢复到读取之前的状态**。

![](https://cdn.nlark.com/yuque/0/2023/png/22935284/1696861396045-c3ae8ce7-c64d-4cc4-aae5-7312c0dab625.png#averageHue=%23252525&height=308&id=ykDqa&originHeight=346&originWidth=967&originalType=binary&ratio=1&rotation=0&showTitle=false&status=done&style=none&title=&width=862)

一个完整的 Read Operation 包含了 **Precharge**、**Access**、**Sense**、**Restore** 四个阶段，下图中 Storage Capacitor 预设为存储高电平 Vcc。

### 3.2.1 Precharge

在这个阶段，首先会通过控制 EQ 信号，让 Te1、Te2、Te3 晶体管处于导通状态，将 Bitline 和 /Bitline 线上的电压稳定在 Vref 上, Vref = Vcc/2。然后进入到下一个阶段。

![](https://cdn.nlark.com/yuque/0/2023/png/22935284/1696861396360-d546907c-1ded-4a8b-9b5e-a1d509f1e645.png#averageHue=%23282626&height=315&id=Kqq6l&originHeight=346&originWidth=967&originalType=binary&ratio=1&rotation=0&showTitle=false&status=done&style=none&title=&width=879)

### 3.2.2 Access

经过 Precharge 阶段， Bitline 和 /Bitline 线上的电压已经被稳定在 Vref 了，此时，通过控制 Wordline 信号，将 Ta 晶体管导通。Storage Capacitor 中存储正电荷会流向 Bitline，继而将 Bitline 的电压拉升到 Vref+，而/Bitline 依然是 Vref。然后进入到下一个阶段。

![](https://cdn.nlark.com/yuque/0/2023/png/22935284/1696861396639-7f4362b3-71ac-4383-b60a-955b28444b6e.png#averageHue=%23282626&id=FDHec&originHeight=346&originWidth=967&originalType=binary&ratio=1&rotation=0&showTitle=false&status=done&style=none&title=)

### 3.2.3 Sense

由于在 Access 阶段，Bitline 的电压被拉升到 Vref+，Tn2 会比 Tn1 更具导通性，Tp1 则会比 Tp2 更具导通性。

SAN (Sense-Amplifier N-Fet Control) 会被设定为逻辑 0 的电压，SAP (Sense-Amplifier P-Fet Control) 则会被设定为逻辑 1 的电压，即 Vcc。由于 Tn2 会比 Tn1 更具导通性，/Bitline 上的电压会更快被 SAN 拉到逻辑 0 电压，同理，Bitline 上的电压也会更快被 SAP 拉到逻辑 1 电压。接着 Tp1 和 Tn2 进入导通状态，Tp2 和 Tn1 进入截止状态。

最后，Bitline 和 /Bitline 的电压都进入稳定状态，正确的呈现了 Storage Capacitor 所存储的信息 Bit。

![](https://cdn.nlark.com/yuque/0/2023/png/22935284/1696861396884-e3fa509b-3ab9-4631-9f2a-884448afd12c.png#averageHue=%23292727&id=t7RoV&originHeight=345&originWidth=966&originalType=binary&ratio=1&rotation=0&showTitle=false&status=done&style=none&title=)

### 3.2.4 Restore

在完成 Sense 阶段的操作后，Bitline 线处于稳定的逻辑 1 电压 Vcc，此时 Bitline 会对 Storage Capacitor 进行充电。经过特定的时间后，Storage Capacitor 的电荷就可以恢复到读取操作前的状态。

![](https://cdn.nlark.com/yuque/0/2023/png/22935284/1696861397146-08fc5daa-79d6-4b60-afda-68094fe7a56c.png#averageHue=%232a2727&id=Z9Vv9&originHeight=345&originWidth=966&originalType=binary&ratio=1&rotation=0&showTitle=false&status=done&style=none&title=)

最后，通过 CSL 信号，让 Tc1 和 Tc2 进入导通状态，外界就可以从 Bitline 上读取到具体的信息。

### 3.2.5 Timing

整个 Read Operation 的时序如下图所示，其中的 Vcc 即为逻辑 1 所对应的电压，Gnd 为逻辑 0。

![|675](https://cdn.nlark.com/yuque/0/2023/png/22935284/1696861397386-55d31c84-ee84-4d01-8e52-e21fe929a0f5.png#averageHue=%23f6f5f5&height=320&id=vFve7&originHeight=478&originWidth=1059&originalType=binary&ratio=1&rotation=0&showTitle=false&status=done&style=none&title=&width=708)

### 3.2.6 Write Recovery

Write Operation 的前期流程和 Read Operation 是一样的，执行 Precharge、Access、Sense 和 Restore 操作。**差异在于在 Restore 阶段后，还会进行 Write Recovery 操作**。

通过控制 WE 信号让 Tw1 和 Tw2 进入导通状态。此时，Bitline 被 input 拉到逻辑 0 电平，/Bitline 则被 /input 拉到逻辑 1 电平。经过特定的时间后，当 Storage Capacitor 的电荷被 Discharge 到 0 状态时，可以通过控制 Wordline，将 Storage Capacitor 的 Access Transistor 截止，写入 0 的操作就完成了。

![](https://cdn.nlark.com/yuque/0/2023/png/22935284/1696861397146-08fc5daa-79d6-4b60-afda-68094fe7a56c.png#averageHue=%232a2727&id=Q34Ki&originHeight=345&originWidth=966&originalType=binary&ratio=1&rotation=0&showTitle=false&status=done&style=none&title=)

## 3.3 存储阵列 Memory Array

DRAM 在设计上，将所有的 Cells 以特定的方式组成一个 Memory Array。

首先，我们在不考虑形式的情况下，最简单的组织方式，就是在一个 Bitline 上，挂接更多的 Cells，如下图所示：

![](https://cdn.nlark.com/yuque/0/2023/png/22935284/1696861397608-051e0998-614d-405a-b67b-2170a4f75cf9.png#averageHue=%231c1c1c&height=254&id=beUo5&originHeight=284&originWidth=983&originalType=binary&ratio=1&rotation=0&showTitle=false&status=done&style=none&title=&width=878)

然而，在实际制造过程中，我们并不会无限制的在 Bitline 上挂接 Cells。因为 Bitline 挂接越多的 Cells，Bitline 的长度就会越长，也就意味着 Bitline 的电容值会更大，这会导致 Bitline 的信号边沿速率下降（电平从高变低或者从低变高的速率），最终导致性能的下降。为此，我们需要限制一条 Bitline 上挂接的 Cells 的总数，将更多的 Cells 挂接到其他的 Bitline 上去。

从 Cell 的结构图中，我们可以发现，在一个 Cell 的结构中，有两条 Bitline，它们在功能上是完全等价的，因此，我们可以把 Cells 分摊到不同的 Bitline 上，以减小 Bitline 的长度。然后，Cells 的组织方式就变成了如下的形式：

![](https://cdn.nlark.com/yuque/0/2023/png/22935284/1696861397804-df45c8a5-c387-4a4d-a546-169b818c824b.png#averageHue=%23191919&id=iB9UW&originHeight=324&originWidth=983&originalType=binary&ratio=1&rotation=0&showTitle=false&status=done&style=none&title=)

当两条 Bitline 都挂接了足够多的 Cells 后，如果还需要继续拓展，那么就只能增加 Bitline 了，增加后的结构图如下： 

![](https://cdn.nlark.com/yuque/0/2023/png/22935284/1696861398059-182b86d3-297a-46e0-bef0-e4c18832006f.png#averageHue=%23191919&height=546&id=VtMg0&originHeight=644&originWidth=984&originalType=binary&ratio=1&rotation=0&showTitle=false&status=done&style=none&title=&width=834)

从图中我们可以看到，增加 Bitline 后，Sense Amplifier、Read Latch 和 Write Driver 的数量也相应的增加了，这意味着成本、功耗、芯片体积都会随着增加。由于这个原因，在实际设计中，会**优先考虑增加 Bitline 上挂接的 Cells 的数量，避免增加 Bitline 的数量**，这也意味着，**一般情况下 Wordline 的数量会比 Bitline 多很多**。

上图中，呈现了一个由 16 个 Cells 组成的 Memory Array。其中的控制信号有 8 个 Wordline、2 个 CSL、2 个 WE，一次进行 1 个 Bit 的读写操作，也就是可以理解为一个 8 x 2 x 1 的 Memory Array。

如果把 2 个 CSL 和 2 个 WE 合并成 1 个 CSL 和 1 个 WE，如下图所示。此时，这个 Memory Array 就有 8 Wordline、1 个 CSL、1 个 WE，一次可以进行 2 个 Bit 的读写操作，也就是成为了 8 x 1 x 2 的 Memory Array。

![](https://cdn.nlark.com/yuque/0/2023/png/22935284/1696861398352-9cb65bfc-b828-49dc-adcb-d9440dd500d0.png#averageHue=%23191919&height=570&id=O8eiP&originHeight=644&originWidth=984&originalType=binary&ratio=1&rotation=0&showTitle=false&status=done&style=none&title=&width=871)

按照上述的过程，不断的增加 Cells 的数量，最终可以得到一个 m x n x w 的 Memory Array，如下图所示：

![](https://cdn.nlark.com/yuque/0/2023/webp/22935284/1697440233974-b95f95bb-9889-4fa2-bb8f-c91b70ac0b3d.webp#averageHue=%23d1d0d0&clientId=u68238361-3e74-4&from=paste&height=557&id=SoZCQ&originHeight=716&originWidth=720&originalType=url&ratio=1&rotation=0&showTitle=false&status=done&style=none&taskId=u2b2f9b4f-5c65-4c4f-b365-d2a6b728301&title=&width=560)

其中，**m 为 Wordline 的数量、n 为 CSL 和 WE 控制信号的数量、w 则为一次可以进行读写操作的 Bits**。

在实际的应用中，我们通常以 **Rows x Columns x Data Width** 来描述一个 Memory Array。

1、Data Width 是指对该 Array 进行一次读写操作所访问的 Bit 位数。这个位数与 CSL 和 WE 控制线的组织方式有关。

2、Row 与 Wordline 是一一对应的，**一个 Row 本质上就是所有接在同一根 Wordline 上的 Cells**。DRAM 在进行数据读写时，选中某一 Row，实质上就是行激活 ACT 后，**该 row 上的 word line 高电平**，控制该 Row 所对应的 Wordline，打开 Cells，并**将 Cells 上的数据缓存到 Sense Amplifiers 上**。

![](https://cdn.nlark.com/yuque/0/2023/png/22935284/1696861398846-2b020acf-56b4-4eae-b616-f18b88f618fb.png#averageHue=%23ded7d7&id=k2T8C&originHeight=403&originWidth=323&originalType=binary&ratio=1&rotation=0&showTitle=false&status=done&style=none&title=)

3、**Column 是 Memory Array 中可寻址的最小单元**。一个 Row 有 n 个 Column，其中 n = Row Size / **Data Width**。下图是 Row Size = 32，Data Width = 8 时 Column 的示例。

一个 Column 的 Size 即为该 Column 上所包含的 Cells 的数量，与 Data Width 相同。Column Size 和 Data Width 在本质上是一样的，与 CSL 和 WE 控制线的组织方式有关。

![](https://cdn.nlark.com/yuque/0/2023/png/22935284/1696861399065-54e9c446-99df-480d-8bb3-80205646906b.png#averageHue=%23454545&id=TvbEx&originHeight=63&originWidth=643&originalType=binary&ratio=1&rotation=0&showTitle=false&status=done&style=none&title=)

### 3.3.1 Page Size 

Page Size ，也称 Row Size，即一个 Row 所存储的 bit 数，它是当一行被激活时加载到感应放大器的位数。对于列地址是 10bit 位宽的情况，每行有 1K 个 column。因此，对于一个 X4 设备，比特数是 1K x 4 = 4Kbit（或 512B）。同样，对于 x8 设备，每页是 1KB，对于 x16 设备，每页是 2KB。

## 3.4 Bank/Bank Group/Rank/Channel

![](https://cdn.nlark.com/yuque/0/2023/png/22935284/1696861399303-66f6ef4f-21b6-49fc-bf65-638f20e5da5e.png#averageHue=%23d99763&height=422&id=a6PQE&originHeight=804&originWidth=1988&originalType=binary&ratio=1&rotation=0&showTitle=false&status=done&style=none&title=&width=1044)

### 3.4.1 Bank

随着 Bitline 数量的不断增加，Wordline 上面挂接的 Cells 也会越来越多，Wordline 会越来越长，继而也会导致电容变大，边沿速率变慢，性能变差。因此，一个 Memory Array 也不能无限制的扩大。为了在不减损性能的基础上进一步增加容量，**DRAM 在设计上将多个 Memory Array 堆叠到一起**，如下图所示： 

![](https://cdn.nlark.com/yuque/0/2023/png/22935284/1696861399624-1256e443-fbae-4cd6-8f27-17bbe3fbf6cc.png#averageHue=%23cccccc&id=jbraj&originHeight=466&originWidth=386&originalType=binary&ratio=1&rotation=0&showTitle=false&status=done&style=none&title=)

把若干个 DRAM 阵列模块叠加在一起构成的一个单元，称作一个 Bank。如下图所示，一**个 Bank 相当于 8 个 16384 行，1024 列的 memory array 叠加在一起**。每个 array 同一时间只输出 1bit 数据，8 个 array 可同时输出 8bit 数据。从另一个角度来看，相当于每格可保存 8 位数据（对应 x8 位宽），**每行可存储 1024\*8bit=1KB。**

![](https://cdn.nlark.com/yuque/0/2023/png/22935284/1696861399937-a06721c2-ed03-4c4f-bdd1-8bb2eb86b0b2.png#averageHue=%23f0f1ec&id=DNWOf&originHeight=985&originWidth=479&originalType=binary&ratio=1&rotation=0&showTitle=false&status=done&style=none&title=)

一个 Bank 中包含若干个 Array，Array 选中“行地址”和“列地址”后，其中一个单元格就被选中，这个单元格就是一个 bit。Bank 中的所有 Array 的行和列地址都是连在一起的，选中“行地址”和“列地址”后，将一起选中所有 Array 的 bit，有多少个 array，就有多少个 bit 被选中。以 DDR3 为例，Data 线宽度是 32，prefetch 是 8，那么 Array 就有 32x8=256 个，内部一次操作会选中 256bit 的数据。**

![image.png](https://cdn.nlark.com/yuque/0/2023/png/22935284/1697509355181-15ccb11f-0754-4e3a-9369-09f8e7733c3a.png#averageHue=%23eaeaea&clientId=u68238361-3e74-4&from=paste&id=kgbDk&originHeight=588&originWidth=473&originalType=url&ratio=1&rotation=0&showTitle=false&size=30323&status=done&style=none&taskId=u28f8d66f-b345-4a85-b62c-54f7d27518c&title=)

常规意义上的 Bank 也称为 **Logic Bank(L-Bank)**，它是颗粒内部的区域概念。每一个 Bank 的 Rows、Columns、Data Width 都是一样的。在访问 DRAM 数据时，只有一个 Bank 被激活，进行数据的读写操作，我们想存取某个格子，只需要告知是哪一行哪一列就行了，这也是为什么内存可以随机存取而硬盘等则是按块存取的原因。 

**多 bank 引入是为了解决 bank 内的读写间隙问题。如果一直操作同一个 bank，比如读完第一行再读第二行，那么行与行之间要进行 precharge，这个引入了读写间隙，降低 io 的利用率。如果在第一个 bank precharge 的时候操作第二个 bank，就可以实现 pipeline 的读写。**

### 3.4.2 Bank Group

随着颗粒容量提升，bank 数越来越多，到**DDR4 时出现 Bank Group**，我们可以理解为，将多个 Bank 编成一个组，这个组就是 Bank Group。

**每个 BG 分组都有独立的激活、读取、写入和刷新操作，类似于多路操作。在一个工作时钟周期内可以同时处理多组数据，从而改进内存的整体效率和带宽。

**在 DDR 发展过程中，一直都以增加 Prefetch 为主要的性能提升手段。但到了 DDR4 时代，数据预取的增加变得更为困难，所以推出了 Bank Group 的设计。这项技术在一个 Bank Group 中执行 8 个预取，在另一个独立的 Bank Group 中可以执行另一个 8 个预取。**

每个 Bank Group 可以独立读写数据，这样一来内部的数据吞吐量大幅度提升，可以同时读取大量的数据，内存的等效频率在这种设置下也得到巨大的提升。DDR4 架构上采用了 8n 预取的 Bank Group 分组，包括使用两个或者四个可选择的 Bank Group 分组，这将使得 DDR4 内存的每个 Bank Group 分组都有独立的激活、读取、写入和刷新操作，从而改进内存的整体效率和带宽。

如果内存内部设计了两个独立的 Bank Group，相当于每次操作 16bit 的数据，变相地将内存预取值提高到了 16n，如果是四个独立的 Bank Group，则变相的预取值提高到了 32n。

DDR4 引入了 BankGroup。**跨 BG 的操作中间等待的时间，比同 BG 跨 bank 操作中间的等待时间短。本来两个 bank 之间的时间虽然比较短，但还不至于让两笔 burst 连起来，现在由于 BG 之间控制电路并行，两个 BG 之间的操作可以合并在一起**。例如两个 BL8 可以拼成一个 BL16。并且 BG 的引入不会增加预取数。

![](https://cdn.nlark.com/yuque/0/2023/png/22935284/1697019257788-ddd196c6-3e8a-4fde-86ca-066511e43575.png#averageHue=%23faf9f4&clientId=ubee3ae6b-a866-4&from=paste&height=370&id=OfjYC&originHeight=347&originWidth=594&originalType=url&ratio=1&rotation=0&showTitle=false&status=done&style=none&taskId=u83a39b11-a5fd-42ec-a549-072b8eebf16&title=&width=634)

![](https://cdn.nlark.com/yuque/0/2023/webp/22935284/1697447927136-f2aa8903-f5ce-4c9e-baff-a0ff7b57c1c1.webp#averageHue=%232c576e&clientId=u68238361-3e74-4&from=paste&id=NByBd&originHeight=464&originWidth=698&originalType=url&ratio=1&rotation=0&showTitle=false&status=done&style=none&taskId=ubb321db0-d5f6-4ed6-acfa-e1b011f3a1d&title=)

DDR4 在不同 BankGroup 之间切换需要四个时钟周期的延迟。四个时钟周期与八个突发长度相匹配。由于 4 个时钟周期等于 8 个时钟沿（包括上升沿和下降沿），因此 8 个突发长度可以在 4 个时钟周期内的每个时钟沿非常高效地输出数据或接收数据。

### 3.4.3 Rank

CPU 与内存之间的数据接口位宽是 64bit，也就意味着 CPU 在一个时钟周期内会向内存发送或从内存读取 64bit 的数据。可是，单个内存颗粒的位宽仅有 4bit、8bit 或 16bit，个别也有 32bit 的。因此，必须把多个颗粒并联起来，组成一个位宽为 64bit 的数据集合，才可以和 CPU 互连。内存条上一般都是由很多内存颗粒共同组成一根内存条。把一个由多个颗粒组成的 64bit 位宽集合，称为一个 RANK。这里 64bit 位宽指的是内存通道**（Memory Channel）的宽度。多个颗粒组成当前内存通道的位宽，这个位宽集合，称之为一个 RANK。**Rank 也称 Physical Bank(P-Bank)**。

多 Rank 的意义：假设内存通道位宽为 64bit，即一个 RANK 为 64bit，每一个 64bit 位宽都是由若干个颗粒组成的。比如，使用 16 颗 8bit 位宽内存芯片，分别组成 2 个 RANK。把原本两根物理 DIMM 的内存颗粒全部安装在一块内存印刷电路板上，使得一根内存条具备两倍的内存容量。 这就相当于，物理上虽然只有一根内存条，但是通过划分不同的 RANK，在逻辑上可以看成是 2 根内存条。

**同一个 RANK 内部的所有内存颗粒 chips，连接到同一个 CS（Chip Select，片选）信号线上**，内存控制器能够对同一个 RANK 的所有 chips 同时进行读写操作，而在同一个 RANK 的 chip 也分享同样的控制信号。在 RANK 选择好后，RANK 内部的所有内存颗粒一起被选中，共提供 64bit 的数据。**通过 CS 信号数量可以判断 RANK 数量**。

你可能会遇到另一种形式的 Dual Rank DDR——Dual-Die Package，即 DDP。DDP 将两个 DDR 颗粒封装在一起，此时你看见的就只有单个芯片了，但其实这两个颗粒还是共享总线，是一种 Dual Rank 器件。

![|525](https://cdn.nlark.com/yuque/0/2023/png/22935284/1697018076707-8f50aa0b-4527-4f1b-bd4f-dad9d468a8b2.png#averageHue=%23f6eae1&clientId=ubee3ae6b-a866-4&from=paste&height=306&id=daxBY&originHeight=428&originWidth=830&originalType=url&ratio=1&rotation=0&showTitle=false&status=done&style=none&taskId=u43f56a33-1796-46c8-909b-8dd1750dd8f&title=&width=594)

![|500](https://cdn.nlark.com/yuque/0/2023/png/22935284/1697426804225-bfc7a903-5c4c-4f11-8732-50609a559fd4.png#averageHue=%23fdfefa&clientId=u68238361-3e74-4&from=paste&id=zFjry&originHeight=391&originWidth=717&originalType=url&ratio=1&rotation=0&showTitle=false&status=done&style=none&taskId=u7867617b-ed00-4979-a5e8-104a6fb5505&title=)

RANK0 和 RANK1 共享同一组 addr/command 信号线，利用 cs 片选线选择欲读取或是写入的那一组内存颗粒，之后，就可以对这一组内存颗粒进行读写。

![](https://cdn.nlark.com/yuque/0/2023/png/22935284/1696861400352-2b483a1b-a3af-4c0b-befb-321eb58ae6b8.png#averageHue=%23eeebd2&id=bq01a&originHeight=227&originWidth=652&originalType=binary&ratio=1&rotation=0&showTitle=false&status=done&style=none&title=)

![](https://cdn.nlark.com/yuque/0/2023/png/22935284/1697685446929-dc035cc6-85f8-4f93-b0ad-80166c49941a.png#averageHue=%23d5c7be&clientId=u5f5606b7-8f3d-4&from=paste&height=449&id=ud8cf4924&originHeight=330&originWidth=438&originalType=url&ratio=1&rotation=0&showTitle=false&status=done&style=none&taskId=uc84912a5-76e5-42bf-947a-e2977f44f55&title=&width=596)

###  3.4.4 Channel

内存通道实际是对应**一组时钟、命令、地址、数据线**，这组总线也就是所谓的内存通道。一般一个内存通道，可以连接若干个 DIMM。

DDR3/DDR4 内存通道关键点，**一个通道上可以插多根 DIMM，每个 DIMM 只会占用一个通道**。

![](https://cdn.nlark.com/yuque/0/2023/png/22935284/1696861400611-5c9cf26a-919a-4315-84e4-7b0468b6f84d.png#averageHue=%23edf2e7&id=rx2hZ&originHeight=211&originWidth=773&originalType=binary&ratio=1&rotation=0&showTitle=false&status=done&style=none&title=)

**一个 DDR 控制器对外连接的通道称为一个 Channel，一颗 CPU 可内置多颗 DDR 控制器，即称为多通道技术。**常见的 PC 用 CPU 通常为 2 通道，HPC 领域的 CPU 通常会有 4 通道，甚至更高。

![](https://cdn.nlark.com/yuque/0/2023/png/22935284/1696861401012-04bc64b5-75b6-4c73-acd0-c8c01b6b7205.png#averageHue=%2330342f&id=Q8kOb&originHeight=606&originWidth=343&originalType=binary&ratio=1&rotation=0&showTitle=false&status=done&style=none&title=)

DDR5 不会为每个 DIMM 提供一个 64 位数据通道，而是为每个 DIMM 提供两个独立的 32 位数据通道（考虑 ECC 时为 40 位）。也就是说一个 DIMM 会占用 2 个通道。与传统的 DDR4 双通道相比，一个 DDR5 内存控制器对应 2 个通道，2 个 DDR5 内存控制器就对应 4 个通道。**DDR4 需要至少 2 根 DIMM 才能组成双通道；DDR5 一根 DIMM 就可以组成双通道，2 根 DIMM 就可以组成 4 通道**。

![](https://cdn.nlark.com/yuque/0/2023/png/22935284/1696861401254-c577816d-9dd0-4ae9-b32d-4b3d501fa040.png#averageHue=%23b6b1a8&id=MMxtF&originHeight=270&originWidth=620&originalType=binary&ratio=1&rotation=0&showTitle=false&status=done&style=none&title=)

![](https://cdn.nlark.com/yuque/0/2023/png/22935284/1696861401553-389bb4c5-c075-4e45-b4fe-c2d0fa3db215.png#averageHue=%23dae9d7&id=C8d8k&originHeight=282&originWidth=773&originalType=binary&ratio=1&rotation=0&showTitle=false&status=done&style=none&title=)

一个主板上可能有多个插槽，用来插多根内存。这些槽位分成两组或多组，组内共享物理信号线。这样的一组数据信号线、对应几个槽位（内存条）称为一个 channel（通道）。简单理解就是 DDRC(DDR 控制器)，**一个通道对应一个 DDRC**。CPU 外核或北桥有两个内存控制器，每个控制器控制一个内存通道。内存带宽增加一倍。

DDR5 的 DIMM 对应 2 个通道，每 2 个通道上，可以插若干个 DIMM。而每个通道上有至少一个以上的 RANK，**每个通道上 RANK 最多为 2 个**。每个 RANK 由若干个内存芯片组成，这些芯片可能是 4/8/16bit 的，他们组合的原则就是将位宽凑齐至通道位宽（32bit）。

内存控制器到内存颗粒内部逻辑，笼统上讲从大到小为：**channel ＞ DIMM ＞ rank ＞ chip ＞ bank ＞ row/column**。

![](https://cdn.nlark.com/yuque/0/2023/png/22935284/1696861401950-b4e747a6-fbb7-40e0-a652-d303fda54786.png#averageHue=%23a5f54f&id=Yix2N&originHeight=327&originWidth=565&originalType=binary&ratio=1&rotation=0&showTitle=false&status=done&style=none&title=)

![](https://cdn.nlark.com/yuque/0/2023/png/22935284/1696861402247-1d578435-ddea-45c7-977a-bfd721e150b4.png#averageHue=%23e0dabf&id=QlZbS&originHeight=353&originWidth=640&originalType=binary&ratio=1&rotation=0&showTitle=false&status=done&style=none&title=)

在这个例子中，一个 i7 CPU 支持两个 Channel（双通道），每个 Channel 上可以插俩个 DIMM,而每个 DIMM 由两个 rank 构成，8 个 chip 组成一个 rank。由于现在多数内存颗粒的位宽是 8bit,而 CPU 带宽是 64bit，所以经常是 8 个颗粒可以组成一个 rank。所以小张的内存条 2R X 8 的意思是由 2 个 rank 组成，每个 rank 八个内存颗粒(为啥我们以后讲)。由于整个内存是 4GB，我们可以算出单个内存颗粒是 256MB。

![](https://cdn.nlark.com/yuque/0/2023/png/22935284/1697102226573-426ee3d1-ed6c-4d91-aafc-970c5b2d3b77.png#averageHue=%23b6d5bb&clientId=u0d784ff7-bafa-4&from=paste&id=Nz6bZ&originHeight=516&originWidth=1173&originalType=url&ratio=1&rotation=0&showTitle=false&status=done&style=none&taskId=ucff06dbe-63b9-40b3-ad1a-116226a3aea&title=)

## 3.5 DDR 容量计算   

DDR 容量计算方法（以下图中间信息为例）：8 * 64K*（1K/8） * 8 * 8 = 4Gb（512 Megx8）。

Bank Addressing  _ Row Addressing _ (Column Addressing / 8) _ x8 Configuration _ 8-bit Prefetch 

注：Column Addressing / 8 是因为列地址有三位用来做 Burst order。Meg 代表有多少个 8bit 位宽的数据。

![](https://cdn.nlark.com/yuque/0/2023/png/22935284/1696861394859-49b9f6d4-de27-4bc9-8e3d-9513f31a0fd1.png#averageHue=%23ccaa85&height=211&id=zUupw&originHeight=407&originWidth=1648&originalType=binary&ratio=1&rotation=0&showTitle=false&status=done&style=none&title=&width=853)

## 3.6 DDR 框图介绍

DDR4 SDRAM 框图如下所示：

![image.png](https://cdn.nlark.com/yuque/0/2023/png/22935284/1697012723121-046fdb45-6e0b-43b3-8f42-d326b9073124.png#averageHue=%23fbfbfb&clientId=ubee3ae6b-a866-4&from=paste&height=832&id=uc95682be&originHeight=832&originWidth=1714&originalType=binary&ratio=1&rotation=0&showTitle=false&size=258603&status=done&style=none&taskId=u3cb49b39-0afd-4b21-af4e-a7a1b00867c&title=&width=1714)

DDR5 SDRAM 框图如下：

![image.png](https://cdn.nlark.com/yuque/0/2023/png/22935284/1697012594826-21574068-8d5b-4c19-8d68-e247a07cf021.png#averageHue=%23f4f4f4&clientId=ubee3ae6b-a866-4&from=paste&height=906&id=u9f183f92&originHeight=906&originWidth=1723&originalType=binary&ratio=1&rotation=0&showTitle=false&size=264705&status=done&style=none&taskId=u87afa501-2c84-40aa-95d2-eb8879d72aa&title=&width=1723)

### 3.6.1 Memory Arrays

Memory Array 是存储信息的主要模块。 

### 3.6.2 Sense Amplifiers 感应放大器

感应放大器有三个主要功能：
1. 感应，当存取晶体管导通并且存储电容向 bitline 放电，或者 bitline 向存储电容充电时，位线上发生的电压的微小变化。感应放大器将该位线上的电压与另一根单独的位线上提供的参考电压进行比较，并将电压差放大到极限，使得存储值可以被解析为数字 1 或 0，这是读出感应放大器在 DRAM 中的主要作用。
2. 恢复（或刷新），它在位线上的电压被感测和放大之后，恢复存储单元的值。导通存取晶体管的动作允许存储电容器与位线共享其存储的电荷。在电荷共享过程发生之后，存储单元内的电压大致等于位线上的电压，并且该电压电平不能用于另一读取操作。因此，在感测和放大操作之后，感测放大器还必须将放大的电压值恢复到存储单元。
3. 读出放大器还充当临时数据存储元件。也就是说，在存储单元中包含的数据值被感测和放大之后，感测放大器将继续驱动感测的数据值，直到 DRAM 阵列被预充电并准备好进行另一次存取。以这种方式，可以从感测放大器访问同一行单元中的数据，而无需对单元本身进行重复的行激活。在这个角色中，感测放大器阵列有效地充当缓存整行数据的行缓冲器。因此，感测放大器的阵列也被称为行缓冲器，并且设计管理策略来控制感测放大器。不同的行缓冲器管理策略规定了读出放大器阵列是将数据保留一段不确定的时间（直到下一次刷新），还是在数据被恢复到存储单元后立即将其放电。

一个基本感应放大器的电路图：

![](https://cdn.nlark.com/yuque/0/2023/png/22935284/1697615307858-ed5af887-9471-4a31-8931-e48b209d585d.png#averageHue=%23eeeeee&clientId=u887b7529-dcb8-4&from=paste&height=281&id=u9c16aeeb&originHeight=300&originWidth=865&originalType=url&ratio=1&rotation=0&showTitle=false&status=done&style=none&taskId=ue2554019-658a-4be6-96a0-d379e494773&title=&width=811)

现代 DRAM 器件中的感测放大器更复杂，除了包含上图所示的基本元件外，还有用于阵列隔离、感测放大器结构的精细平衡（类似效果就是使放大器的输入失调电压尽量小）和加速的感测能力的附加电路元件。

在上面的基本感测放大器电路图中，右边方框是均衡（EQ）信号线控制的电压均衡电路。该电路的功能是确保位线对上的电压尽可能紧密地匹配（就是使预充电后两根位线电压都是 VDD/2）。由于差分感测放大器被设计为放大位线对之间的电压差，所以在行激活之前存在于位线对上的任何电压不平衡都会降低感测放大器的有效性，所以需要有一个位线电压均衡电路。

左边方框是感应放大器的核心，一组四个交叉连接的晶体管，左边 NMOS，右边 PMOS。感测电路本质上是双稳态电路，他根据 SAN 和 SAP 感测信号被激活时（所谓激活，就是 SAN=0，SAP=VDD）位线上的相应电压来将位线对驱动到互补电压极值。

最右边的是输出结构，列选择线（CSL）控制输出晶体管的通断，使位线电平可以输出，或者外部输入电平能够作用到相应位线上。

**写回的数据从 sense amp，bitline 再次进入 cell 里，sense amp 模块带有 driver 功能双向传数据。每次读写操作，都是整行写回，读写都需要读数据到 amp 上。**

### 3.6.3 IO Gating

Colume decoder 和 Sense amp 之间还有 IO gating。IO 电路主要是用于处理数据的缓存、输入和输出。其中 Data Latch 和 Data Register 用于缓存数据，DQM Mask Logic 和 IO Gating 等则用于输入输出的控制。

### 3.6.4 Row & Column Decoder

Row Decoder 的主要功能是将 Active Command 所带的 Row Address 映射到具体的 wordline，最终打开指定的 Row。同样 Column Decoder 则是把 Column Address 映射到具体的 csl，最终选中特定的 Column。

### 3.6.5 Control Logic

Control Logic 的主要功能是解析 SDRAM Controller 发出的 Command，然后根据具体的 Command 做具体内部模块的控制，例如：选中指定的 Bank、触发 refresh 等的操作。Control Logic 包含了 1 个或者多个 Mode Register。该 Register 包含了时序、数据模式等的配置。

### 3.6.6 Refresh Counter

Refresh Counter 用于记录下次需要进行 refresh 操作的 Row。在接收到 AR 或者在 Self-Refresh 模式下，完成 一次 refresh 后，Refresh Counter 会进行更新。

### 3.6.7 DLL

延迟锁定回路 DLL，DDR SDRAM 对时钟的精确性有着很高的要求，而 DDR SDRAM 有两个时钟，一个是外部的总线时钟，一个是内部的工作时钟，在理论上 DDR SDRAM 这两个时钟应该是同步的，但由于种种原因，如温度、电压波动而产生延迟使两者很难同步。DDR SDRAM 的 tAC 就是因为内部时钟与外部时钟有偏差而引起的，它很可能造成因数据不同步而产生错误。实际上，不同步就是一种正/负延迟，如果延迟不可避免，那么若是设定一个延迟值，如一个时钟周期，那么内外时钟的上升与下降沿还是同步的。**所以需要根据外部时钟动态修正内部时钟的延迟来实现与外部时钟的同步，这就是 DLL 的任务。**

DLL 不同于主板上的 PLL，它不涉及频率与电压转换，而是生成一个延迟量给内部时钟。目前 DLL 有两种实现方法，一个是时钟频率测量法（CFM，Clock Frequency Measurement），一个是时钟比较法（CC，Clock Comparator）。

CFM 是测量外部时钟的频率周期，然后以此周期为延迟值控制内部时钟，这样内外时钟正好就相差了一个时钟周期，从而实现同步。DLL 就这样反复测量反复控制延迟值，使内部时钟与外部时钟保持同步。

![](https://cdn.nlark.com/yuque/0/2023/gif/22935284/1697705466445-0aaa6cfd-a83c-4e8f-bdf9-c56a9e50e78a.gif#averageHue=%23eaeaea&clientId=u34526b3b-c1f6-4&from=paste&id=u4ecee3e5&originHeight=270&originWidth=504&originalType=url&ratio=1&rotation=0&showTitle=false&status=done&style=none&taskId=u99e64b76-8890-4390-9937-129302e272c&title=)

CC 的方法则是比较内外部时钟的长短，如果内部时钟周期短了，就将所少的延迟加到下一个内部时钟周期里，然后再与外部时钟做比较，若是内部时钟周期长了，就将多出的延迟从下一个内部时钟中刨除，如此往复，最终使内外时钟同步。

![](https://cdn.nlark.com/yuque/0/2023/gif/22935284/1697705483750-a7a42f1d-3e21-41d3-8b64-44e786317511.gif#averageHue=%23eaeaea&clientId=u34526b3b-c1f6-4&from=paste&id=ua5d5363a&originHeight=275&originWidth=504&originalType=url&ratio=1&rotation=0&showTitle=false&status=done&style=none&taskId=ub6510043-b3bf-42c6-b796-c0c8f1b791d&title=)

CFM 与 CC 各有优缺点，CFM 的校正速度快，仅用两个时钟周期，但容易受到噪音干扰，并且如果测量失误，则内部的延迟就永远错下去。CC 更稳定可靠，如果比较失败，延迟受影响的只是一个数据（而且不会太严重），不会涉及到后面的延迟修正，但它的修正时间要比 CFM 长。DLL 功能在 DDR SDRAM 中可以被禁止，但仅限于除错与评估操作，正常工作状态是自动有效的。

## 3.7 DDR 时钟/控制/地址/数据信号

![](https://cdn.nlark.com/yuque/0/2023/png/22935284/1696861405503-0e7c927b-c67c-46b5-b4eb-8a3ff0a5863a.png#averageHue=%23f5f4f2&height=586&id=XeGby&originHeight=644&originWidth=679&originalType=binary&ratio=1&rotation=0&showTitle=false&status=done&style=none&title=&width=618)

### 3.7.1 DDR4

![image.png](https://cdn.nlark.com/yuque/0/2023/png/22935284/1696989896689-499ec8c5-2317-4678-a9ba-3613cfa9a665.png#averageHue=%23e7e4e1&clientId=u33443cfc-91a7-4&from=paste&id=x89XJ&originHeight=431&originWidth=789&originalType=url&ratio=1&rotation=0&showTitle=false&size=79693&status=done&style=none&taskId=ud2a4cdeb-c041-4907-a4c1-335a50ff463&title=)

![image.png](https://cdn.nlark.com/yuque/0/2023/png/22935284/1696989896806-6d9e13a8-2857-49ea-bda4-8deb5844a4b2.png#averageHue=%23f0f0f0&clientId=u33443cfc-91a7-4&from=paste&id=uwqcM&originHeight=686&originWidth=720&originalType=url&ratio=1&rotation=0&showTitle=false&size=325483&status=done&style=none&taskId=u3b7a49a7-961c-4247-a302-01d68d251eb&title=)

#### 3.7.1.1 A[17:0]

Address inputs: **Provide the row address for ACTIVATE commands and the column address for READ/WRITE commands to select one location out of the memory array in the respective bank. **

**A10/AP, A12/BC_n, WE_n/A14, CAS_n/A15, RAS_n/A16 have additional functions, see individual entries in this table. **The address inputs also provide the op-code during the MODE REGISTER SET command. A16 is used on some 8Gb and 16Gb parts. A17 connection is part-number specific.

第一次用于行地址，第二次用于列地址。

**A10/AP** : **Auto precharge input**:** A10 is sampled during READ and WRITE commands to determine whether auto precharge should be performed to the accessed bank after a READ or WRITE operation**. (HIGH = auto precharge; LOW = no auto precharge.) A10 is sampled during a PRECHARGE command to determine whether the PRECHARGE applies to one bank (A10 LOW) or all banks (A10 HIGH). If only one bank is to be precharged, the bank is selected by the bank group and bank addresses.

**A12/BC_n** : **Burst chop input: A12/BC_n is sampled during READ and WRITE commands to determine if burst chop (on-the-fly) will be performed**. (HIGH = no burst chop; LOW = burst chopped). See the Command Truth Table.

地址和控制信号速度没有 DQ 的速度快，以时钟的上升沿为依据采样，所以需要与时钟走线保持等长。但如果使用多片 DDR 时，地址和控制信号为一驱多的关系，需要注意匹配方式是否适合。

#### 3.7.1.2 ACT_n

**Command input: ACT_n indicates an ACTIVATE command. When ACT_n (along with CS_n) is LOW, the input pins RAS_n/A16, CAS_n/A15, and WE_n/A14 are treated as row address inputs for the ACTIVATE command.** When ACT_n is HIGH (along with CS_n LOW), the input pins RAS_n/ A16, CAS_n/A15, and WE_n/A14 are treated as normal commands that use the RAS_n, CAS_n, and WE_n signals. See the Command Truth Table.

命令激活信号，低电平时，可以通过 A[14:16]地址信号线选择激活命令的行地址。高电平时，Address 信号线正常使用。

#### 3.7.1.3 BA[1:0]

**Bank address inputs**: Define the bank (within a bank group) to which an ACTIVATE, READ, WRITE, or PRECHARGE command is being applied. Also determines which mode register is to be accessed during a MODE REGISTER SET command.

#### 3.7.1.4 BG[1:0]

**Bank group address inputs**: Define the bank group to which an ACTIVATE, READ, WRITE, or PRECHARGE command is being applied. Also determines which mode register is to be accessed during a MODE REGISTER SET command. **BG[1:0] are used in the x4 and x8 configurations. BG1 is not used in the x16 configuration.**

#### 3.7.1.5 C0/CKE1,C1/CS1_n,C2/ODT

**Stack address inputs: These inputs are used only when devices are stacked; that is, they are used in 2H, 4H, and 8H stacks for x4 and x8 configurations (these pins are not used in the x16 configuration, and are NC on the x4/x8 SDP).** DDR4 will support a traditional DDP package, which uses these three signals for control of the second die (CS1_n, CKE1, ODT1). DDR4 is not expected to support a traditional QDP package. For all other stack configurations, such as a 4H or 8H, it is assumed to be a single-load (master/slave) type of configuration where C0, C1, and C2 are used as chip ID selects in conjunction with a single CS_n, CKE, and ODT signal.

#### 3.7.1.6 CK_t,CK_c

**Differential clock inputs. All address, command, and control input signals are sampled on the crossing of the positive edge of CK_t and the negative edge of CK_c**.

差分时钟 DDR 的一个必要设计。因为温度、电阻性能的改变等原因，CK 上下沿间距可能发生变化，此时与其反相的 CK#通常被理解为第二个触发时钟，它起到了触发时钟校准的作用。**CK 上升沿快下降沿慢，CK#则是上升沿慢下降沿快**。

一般使用终端并联 100Ω 的匹配方式，差分走线差分对控制阻抗为 100Ω ，单端线 100Ω 。需要注意的是，差分线也可以使用串联匹配，使用串联匹配的好处是可以控制差分信号的上升沿缓度，对 EMI 可能会有一定的作用。

#### 3.7.1.7 CKE

**Clock enable input: CKE HIGH activates and CKE LOW deactivates the internal clock signals, device input buffers, and output drivers**. Taking CKE LOW provides PRECHARGE POWER-DOWN and SELF REFRESH operations (all banks idle), or active power-down (row active in any bank). CKE is asynchronous for self refresh exit, however, timing parameters such as XS are still calculated from the first rising clock edge where CKE HIGH satisfies tIS. After VREFCA has become stable during the power-on and initialization sequence, it must be maintained during all operations (including SELF REFRESH). CKE must be maintained HIGH throughout read and write accesses. Input buffers (excluding CK_t, CK_c, ODT, RESET_n, and CKE) are disabled during power-down. Input buffers (excluding CKE and RESET_n) are disabled during self refresh.

时钟信号使能。通过此电平，可以控制芯片是否进入低功耗模式。

#### 3.7.1.8 CS_n

**Chip select input: All commands are masked when CS_n is registered HIGH.** CS_n provides for external rank selection on systems with multiple ranks. CS_n is considered part of the command code.

DDR 芯片使能，用于多个 RANK 时的 RANK 组选择。

#### 3.7.1.9 DM_n,UDM_n,LDM_n

**Input data mask: DM_n is an input mask signal for write data. Input data is masked when DM is sampled LOW coincident with that input data during a write access. DM is sampled on both edges of DQS.** DM is not supported on x4 configurations. The UDM_n and LDM_n pins are used in the x16 configuration: UDM_n is associated with DQ[15:8]; LDM_n is associated with DQ[7:0]. The DM, DBI, and TDQS functions are enabled by mode register settings. See the Data Mask section.

掩码就是屏蔽掉我们传输不需要的数据。通过 DM，内存可以控制 IO 端口取消部分输入或输出的数据。但是在读取时，被屏蔽的数据仍然会从存储体中传出。**在写入数据时，如果 DM 信号为高，则写入数据被屏蔽；如果 DM 信号为低，则写入数据正常锁存**。

掩码每一位 bit 对应 ddr 数据的 1 个 byte。对于 4bit 位宽芯片，丙个芯片共用一个 DM 信号；对于 8bit 位宽芯片，一个芯片占用一个 DM 信号；而对于 16bit 位宽芯片，则需要两个 DM 信号。

#### 3.7.1.10 PAR

Parity for command and address input: This function can be enabled or disabled via the mode register. When enabled, the parity signal covers all command and address inputs, including ACT_n, RAS_n/A16, CAS_n/A15, WE_n/A14, A[17:0], A10/AP, A12/BC_n, BA[1:0], and BG[1:0] with C0, C1, and C2 on 3DS only devices**. Control pins NOT covered by the parity signal are CS_n, CKE, and ODT. Unused address pins that are density- and configuration-specific should be treated internally as 0s by the DRAM parity logic. Command and address inputs will have parity check performed when commands are latched via the rising edge of CK_t and when CS_n is LOW.

命令/地址信号的奇偶校验使能，可通过寄存器禁用或者使能。

#### 3.7.1.11 RAS_n/A16,CAS_n/A15,WE_n/A14

**Command inputs: RAS_n/A16, CAS_n/A15, and WE_n/A14 (along with CS_n and ACT_n) define the command and/or address being entered**. See the ACT_n description in this table.

行地址选通、列地址选通、写使能。

#### 3.7.1.12 RESET_n

**Active LOW asynchronous reset input: Reset is active when RESET_n is LOW, and inactive when RESET_n is HIGH**. RESET_n must be HIGH during normal operation. RESET_n is a CMOS rail-to-rail signal with DC HIGH and LOW at 80% and 20% of VDD (960 mV for DC HIGH and 240 mV for DC LOW).

DDR 复位信号，低电平有效。正常操作过程中，保持高电平。

#### 3.7.1.13 TEN

**Connectivity test mode input: TEN is active when HIGH and inactive when LOW. TEN must be LOW during normal operation**. TEN is a CMOS rail-to-rail signal with DC HIGH and LOW at 80% and 20% of VDD (960mV for DC HIGH and 240mV for DC LOW). On Micron 3DS devices, connectivity test mode is not supported and the TEN pin should be considered NF maintained LOW at all times.
测试模式使能信号，高电平使能测试模式。正常操作过程中，必须拉低。

![](https://cdn.nlark.com/yuque/0/2023/png/22935284/1696989875026-38b88463-e2e3-4a7d-beb1-37d760c0ad84.png?x-oss-process=image%2Fformat%2Cwebp#averageHue=%23ebe9e6&from=url&id=RZBGK&originHeight=505&originWidth=791&originalType=binary&ratio=1&rotation=0&showTitle=false&status=done&style=none&title=)

#### 3.7.1.14 DQ

**Data input/output: Bidirectional data bus. DQ represents DQ[3:0], DQ[7:0], and DQ[15:0] for the x4, x8, and x16 configurations, respectively**. If write CRC is enabled via mode register, the write CRC code is added at the end of data burst. Any one or all of DQ0, DQ1, DQ2, and DQ3 may be used to monitor the internal VREF level during test via mode register setting MR[4] A[4] = HIGH, training times change when enabled. During this mode, the RTT value should be set to High-Z. This measurement is for verification purposes and is NOT an external voltage supply pin.

**DQ 在一定规则下可以进行交换**。详细见各类型 DDR 描述。

DQ 的交换准则：

- 按照 8bit 划分为一个 slice，一个通道 64bit 可划分为 8 个 slice。

- X8/X16：同个 slice 内部的 8 位 DQ 可以自由互换；8 个 slice 之间可以任意交换；

- X4：同个 slice 可以再细拆分为低 4 位和高 4 位两组，分别为 slice[n]\_L 与 slice[n]\_H，同个 slice 的 slice[n]\_L 与 slice[n]\_H 间可以整组交换。slice 的 slice[n]\_L 内的 4 位 DQ 之间可以互相交换，slice 的 slice[n]\_H 内的 4 位 DQ 之间可以互相交换。但 slice[n]\_L 内的 DQ 与 slice[n]\_H 内的 DQ 不能进行交换。8 个 slice 之间可以任意交换；

- X8/X16 兼容内存交换准则相对宽松和灵活，但不一定能兼容 X4。

#### 3.7.1.15 DBI_n,UDBI_n,LDBI_n

**DBI input/output: Data bus inversion. DBI_n is an input/output signal used for data bus inversion in the x8 configuration. UDBI_n and LDBI_n are used in the x16 configuration; UDBI_n is associated with DQ[15:8], and LDBI_n is associated with DQ[7:0].** The DBI feature is not supported on the x4 configuration. DBI is not supported for 3DS devices and should be disabled in MR5. DBI can be configured for both READ (output) and WRITE (input) operations depending on the mode register settings. The DM, DBI, and TDQS functions are enabled by mode register settings. See the Data Bus Inversion section.

#### 3.7.1.16 DQS_t,DQS_c,UDQS_t,UDQS_c,LDQS_t,LDQS_c

**Data strobe input/output: Output with READ data, input with WRITE data. Edge-aligned with READ data, centered-aligned with WRITE data. For the x16, LDQS corresponds to the data on DQ[7:0]; UDQS corresponds to the data on DQ[15:8]. **For the x4 and x8 configurations, DQS corresponds to the data on DQ[3:0] and DQ[7:0], respectively. DDR4 SDRAM supports a differential data strobe only and does not support a single-ended data strobe.

**数据选取脉冲 DQS 是 DDR 中的重要功能，它主要用来在一个时钟周期内准确区分出每个传输周期，以便于接收方准确接收数据。它是双向信号，在读取内存时，由内存触发，DQS 的沿和 DQ 的沿对齐；写入时由 CPU/MC 触发，DQS 的中间对应 DQ 的沿。每个 byte 都有一个 DQS 信号线，可以认为 DQS 是 DQ 的同步信号。**

在读取时，**DQS 与数据信号同时生成（也是在 CK 与 CK#的交叉点）**。而 DDR 内存中的 CL 也就是从 CAS 发出到 DQS 生成的间隔，DQS 生成时，芯片内部的预取已经完毕了，由于预取的原因，实际的数据传出可能会提前于 DQS 发生（数据提前于 DQS 传出）。由于是并行传输，DDR 内存对 tAC 也有一定的要求，对于 DDR266，tAC 的允许范围是 ±0.75ns，对于 DDR333，则是 ±0.7ns，其中 CL 里包含了一段 DQS 的导入期。

**DQS 在读取时与数据同步传输，那么写入时也是以 DQS 的上下沿为准吗？不，如果以 DQS 的上下沿区分数据周期的危险很大。由于芯片有预取的操作，所以输出时的同步很难控制，只能限制在一定的时间范围内，数据在各 I/O 端口的出现时间可能有快有慢，会与 DQS 有一定的间隔，这也就是为什么要有一个 tAC 规定的原因。而在接收方，一切必须保证同步接收，不能有 tAC 之类的偏差。在写入时，芯片不再自己生成 DQS，而以发送方传来的 DQS 为基准，并相应延后一定的时间，在 DQS 的中部为数据周期的选取分割点（在读取时分割点就是上下沿），从这里分隔开两个传输周期。这样做的好处是，由于各数据信号都会有一个逻辑电平保持周期，即使发送时不同步，在 DQS 上下沿时都处于保持周期中，此时数据接收触发的准确性无疑是最高的。在写入时，以 DQS 的高/低电平期中部为数据周期分割点，而不是上/下沿，但数据的接收触发仍为 DQS 的上/下沿。

1. DDR 之前的 SDR 是使用 clock 来同步的，理论上 DQ 的读写时序完全可以由 clock 来同步。但是，速度提高之后可用的时序余量越来越小；
2. 引入 DQS 是为了降低系统设计的难度和可靠性，可以不考虑 DQ 和 clock 之间的直接关系，只用分组考虑 DQ 和 DQS 之间的关系，很容易同组同层处理；
3. DQ 和 DQS 只是组成了源同步时序的传输关系，可以保证数据在接收端被正确地锁存；
4. IC 工作时，内部真正的同步时钟是 clock 而不是 DQS，数据要在 IC 内部传输同样需要和 clock（内部时钟比外部时钟慢）去同步，就要求所有的 DQ 信号还是同步的，而且和 clock 保持一定的关系，所以就要控制 DQS 和 clock 之间的延时。
5. DQS 信号在走线时需要与同组的 DQS 信号保持等长，控制单端 50ohm 的阻抗。

![](https://cdn.nlark.com/yuque/0/2023/png/22935284/1697080628518-c5b3b860-ff83-44c8-abef-ef04cd62f300.png#averageHue=%23e8e8e8&clientId=u0d784ff7-bafa-4&from=paste&id=u9b030294&originHeight=247&originWidth=500&originalType=url&ratio=1&rotation=0&showTitle=false&status=done&style=none&taskId=u26a8b533-dc96-4429-94b9-d15cf4b3a95&title=)

#### 3.7.1.17 ALERT_n

**Alert output: This signal allows the DRAM to indicate to the system's memory controller that a specific alert or event has occurred.** Alerts will include the command/address parity error and the CRC data error when either of these functions is enabled in the mode register

报警信号，若命令/地址出现奇偶校验错误或者 CRC 错误，该 PIN 脚拉低，告知 DDR Controller。

#### 3.7.1.18 TDQS_t,TDQS_c

**Termination data strobe output: TDQS_t and TDQS_c are used by x8 DRAMs only.** When enabled via the mode register, the DRAM will enable the same RTT termination resistance on TDQS_t and TDQS_c that is applied to DQS_t and DQS_c. When the TDQS function is disabled via the mode register, the DM/TDQS_t pin will provide the DATA MASK (DM) function, and the TDQS_c pin is not used. The TDQS function must be disabled in the mode register for both the x4 and x16 configurations. The DM function is supported only in x8 and x16 configurations.Dummy loads for mixed populations of x4 based and x8 based RDIMMs.

终端数据选通，当 TDQS 使能时，DM 禁止。

该功能仅在数据位宽 x8 的内存颗粒上使用，主要用于简化 x4 位宽与 x8 位宽内存混合使用的存储器控制系统的设计，如下图所示，x8 的内存条中每 8bits 需要一对 DQS，而 x4 的内存条中每 4bits 需要一对 DQS，即 8bits 的 x4 内存条需要两对 DQS，而 x8 内存条只需要一对 DQS，这就会造成 DQS 信号的负载不均衡，从而引起信号完整性（SI）问题。因此，x4 内存条的其中一对 DQS 信号连接到 x8 内存条的 TDQS 上进行端接，并开启 x8 内存条的 TDQS 功能，从而保证负载均衡，不产生 SI 问题。

![](https://cdn.nlark.com/yuque/0/2023/png/22935284/1697687334477-fb23aedf-a97a-42d6-b688-cfb0183497f0.png#averageHue=%23fdfdfd&clientId=u5f5606b7-8f3d-4&from=paste&height=377&id=u73b023f0&originHeight=323&originWidth=529&originalType=url&ratio=1&rotation=0&showTitle=false&status=done&style=none&taskId=u7556638c-8114-4384-87e9-b57de52deac&title=&width=617)

![image.png](https://cdn.nlark.com/yuque/0/2023/png/22935284/1696989896689-499ec8c5-2317-4678-a9ba-3613cfa9a665.png#averageHue=%23e7e4e1&clientId=u33443cfc-91a7-4&from=paste&id=u7452851b&originHeight=431&originWidth=789&originalType=url&ratio=1&rotation=0&showTitle=false&size=79693&status=done&style=none&taskId=ud2a4cdeb-c041-4907-a4c1-335a50ff463&title=)

![image.png](https://cdn.nlark.com/yuque/0/2023/png/22935284/1696989896806-6d9e13a8-2857-49ea-bda4-8deb5844a4b2.png#averageHue=%23f0f0f0&clientId=u33443cfc-91a7-4&from=paste&id=u0049ede9&originHeight=686&originWidth=720&originalType=url&ratio=1&rotation=0&showTitle=false&size=325483&status=done&style=none&taskId=u3b7a49a7-961c-4247-a302-01d68d251eb&title=)

#### 3.7.1.19 ZQ

Reference ball for ZQ calibration: This ball is tied to an external 240R resistor (RZQ), which is tied to VSSQ。详细流程参考 4.2 章节 ZQ Calibration。

电阻尽可能靠近引脚，避免外部干扰。

#### 3.7.1.20 ODT

**On-die termination input: 片内端接 ODT (registered HIGH) enables termination resistance internal to the DDR4 SDRAM. When enabled, ODT (RTT) is applied only to each DQ, DQS_t, DQS_c, DM_n/DBI_n/TDQS_t, and TDQS_c signal for the x4 and x8 configurations **(when the TDQS function is enabled via mode register). For the x16 configuration, RTT is applied to each DQ, UDQS_t, UDQS_c, LDQS_t, LDQS_c, UDM_n, and LDM_n signal. The ODT pin will be ignored if the mode registers are programmed to disable RTT.

从 DDR2 开始片内端接电阻，用于连接端接电源和高速信号，起到阻抗匹配的作用，减少信号反射。

ODT 是内建核心的终结电阻，它的功能是让一些信号在终结电阻处消耗完，防止这些信号在电路上形成反射。换句话说就是在片内设置合适的上下拉电阻，以获得更好的信号完整性。

被 ODT 校准的信号包括：
- DQ， DQS， DQS# and DM for x4 configuration
- DQ， DQS， DQS#， DM， TDQS and TDQS# for X8 configuration
- DQU， DQL， DQSU， DQSU#， DQSL， DQSL#， DMU and DML for X16 configuration

![image.png](https://cdn.nlark.com/yuque/0/2023/png/22935284/1697536059910-6d51f85b-a7cd-45a2-834d-39904bb6269e.png#averageHue=%23f4f4f4&clientId=u887b7529-dcb8-4&from=paste&height=904&id=u6a7096c6&originHeight=904&originWidth=551&originalType=binary&ratio=1&rotation=0&showTitle=false&size=78398&status=done&style=none&taskId=ube7dcea3-bf9b-4c55-83dd-0ec44502113&title=&width=551)

当一个 CPU 挂了很多 DDR 芯片的时候，他们是共用控制线，地址线的，走线肯定要分叉，如果没有终端匹配电阻，肯定会产生信号完整性问题。那么如果只有一个 DDR 芯片的时候，需不需要呢？正常情况下，走线很短，有符合规则，是不需要的。

DDR3 的 PIN 定义上有一个引脚是 ODT，如果 ODT=0，DRAM Termination State 功能关闭；ODT=1，DRAM Termination State 的功能参考寄存器设置。如下是一个真值表。因为 DRAM Termination State 非常耗电，所以不用的时候最好不要打开。

![image.png](https://cdn.nlark.com/yuque/0/2023/png/22935284/1697509748560-2f49d925-9d68-4813-9391-102addbb2c2a.png#averageHue=%23eeede7&clientId=u68238361-3e74-4&from=paste&id=bv776&originHeight=69&originWidth=554&originalType=url&ratio=1&rotation=0&showTitle=false&size=19314&status=done&style=none&taskId=uad1528a1-1777-4da9-b269-abc9bd9762b&title=)

### 3.7.2 DDR5

![image.png](https://cdn.nlark.com/yuque/0/2023/png/22935284/1697530687570-61dc1767-c3d6-4d53-93cc-c55b2be2adad.png#averageHue=%23e4e1de&clientId=u773ac8d8-ebcd-4&from=paste&height=913&id=u8ab72288&originHeight=913&originWidth=778&originalType=binary&ratio=1&rotation=0&showTitle=false&size=255654&status=done&style=none&taskId=u16dd6249-0b19-41ce-abd2-fa5707adb0a&title=&width=778)

![image.png](https://cdn.nlark.com/yuque/0/2023/png/22935284/1697530713240-6b75f687-de39-4d79-95f1-9b4d5763cb28.png#averageHue=%23e5e5e5&clientId=u773ac8d8-ebcd-4&from=paste&height=134&id=ue534392d&originHeight=134&originWidth=779&originalType=binary&ratio=1&rotation=0&showTitle=false&size=34564&status=done&style=none&taskId=u9766a2bc-1df0-4629-840d-ae9a4dab68b&title=&width=779)

![image.png](https://cdn.nlark.com/yuque/0/2023/png/22935284/1697530728755-4c6d605d-a1c2-4ce2-afeb-15f7d4293294.png#averageHue=%23e6e6e6&clientId=u773ac8d8-ebcd-4&from=paste&height=345&id=ua2160d45&originHeight=345&originWidth=778&originalType=binary&ratio=1&rotation=0&showTitle=false&size=77129&status=done&style=none&taskId=u09472f5e-5494-42d0-9ba3-7d2a2fb3b0e&title=&width=778)

![image.png](https://cdn.nlark.com/yuque/0/2023/png/22935284/1697532474831-93fe9c33-7444-4120-aaef-473fec90c07f.png#averageHue=%23e2e2e2&clientId=u773ac8d8-ebcd-4&from=paste&height=104&id=ub7b79571&originHeight=104&originWidth=509&originalType=binary&ratio=1&rotation=0&showTitle=false&size=24486&status=done&style=none&taskId=udd55a2ee-8f12-4b20-b0c6-ab63bb8832b&title=&width=509)

![image.png](https://cdn.nlark.com/yuque/0/2023/png/22935284/1697532518211-281f8e44-a0b5-4a38-9dde-b4b0270caef9.png#averageHue=%23e2ddd6&clientId=u773ac8d8-ebcd-4&from=paste&height=234&id=ud57423ce&originHeight=234&originWidth=788&originalType=binary&ratio=1&rotation=0&showTitle=false&size=53422&status=done&style=none&taskId=u8f6c58cf-049c-4028-81e5-bda6ccf1e0b&title=&width=788)

![image.png](https://cdn.nlark.com/yuque/0/2023/png/22935284/1697532533547-c7802a9f-0c40-4c3c-a272-ea26bf6a7cb1.png#averageHue=%23e7e3de&clientId=u773ac8d8-ebcd-4&from=paste&height=337&id=u44fcf238&originHeight=337&originWidth=788&originalType=binary&ratio=1&rotation=0&showTitle=false&size=77229&status=done&style=none&taskId=u92f2488b-5b33-4f7f-bcd3-36867d4a215&title=&width=788)

![image.png](https://cdn.nlark.com/yuque/0/2023/png/22935284/1697532574995-a40c41bd-c39b-414a-bbfa-f1f59b85f21a.png#averageHue=%23e8e7e5&clientId=u773ac8d8-ebcd-4&from=paste&height=845&id=ucf6fc9e3&originHeight=845&originWidth=785&originalType=binary&ratio=1&rotation=0&showTitle=false&size=178139&status=done&style=none&taskId=u0d84e637-6168-4fec-b7ae-fd9313a51d8&title=&width=785)

## 3.8 DDR 供电/上电时序

### 3.8.1 VDD

VDD 是给主板主存储器供电的电源。

### 3.8.2 VDDQ

VDDQ 是给存储芯片输出缓冲器供电的电源。一般的使用中都是把 VDDQ 和 VDD 合成一个电源使用。

### 3.8.3 VPP

VPP 即为字线电压，Charge Pump Power，需要外供。DDR4 中 VDD=1.2V，VPP=2.5V。之所以需要外供的原因是为了不用内建电荷泵 Charge pump，以达到省功耗的目的。而 DDR3 本身是不需要的，在其内部有电压泵，不需要外供。

在 DDR 内存中，VPP 必须始终大于或等于 VDD/VDDQ，这是因为 VPP 用于编程和擦除 DDR 存储芯片内存（非易失性存储器，如 EEPROM 和闪存等）中的数据。VPP 通常用于在编程或擦除期间提供高电压，以改变存储器芯片的状态。如果 VPP 电压低于 VDD/VDDQ，则无法正确编程或擦除存储器，可能会导致存储器错误或损坏。

另外，VPP 电压不能晚于 VDD/VDDQ 上电，这是因为在上电时，VDD/VDDQ 电压会先上升到正常工作电压，而 VPP 电压上升的速度可能会比 VDD/VDDQ 慢，如果 VPP 电压晚于 VDD/VDDQ 上电，就会导致内存无法正常工作。因此，为了保证 DDR 内存的正常工作，VPP 必须始终大于或等于 VDD/VDDQ，并且不能晚于 VDD/VDDQ 上电。

### 3.8.4 VTT

VTT 电源需要具有吸电流和灌电流的能力，电流可能要求比较大，一般可以用专门为 DDR 设计的产生 VTT 的电源芯片来满足要求。

### 3.8.5 VREFCA

VREFCA : Reference voltage for control, command, and address pins. 可以使用电源芯片供电，也可以采用电阻分压的方式得到。

Vref = VDDQ/2，要求跟随 VDDQ。

### 3.8.6 DDR4 供电电压&上下电时序

VDD = VDDQ = 1.2V+-60mV

VPP = 2.5V,-125mV,+250mV DRAM Activating Power Supply 

DDR4 中 VPP 电压的作用：字位线电压，需要外供，DDR4 SDRAM 不用内建电荷泵 Charge pump，为了省功耗；DDR3 本身也需要的，在DDR3 SDRAM 内部有电压泵，不需要外供。

VREFCA : Reference voltage for control, command, and address pins.

ZQ : Reference ball for ZQ calibration: **This ball is tied to an external 240R resistor (RZQ), which is tied to VSSQ**。

### 3.8.7 DDR5 供电电压&上下电时序

![image.png](https://cdn.nlark.com/yuque/0/2023/png/22935284/1697077816038-68ce0215-bb40-4e74-a086-f30e9f43ae2f.png#averageHue=%23e5d9c7&clientId=ub8d0bc13-bfcf-4&from=paste&height=313&id=ue82057c1&originHeight=313&originWidth=788&originalType=binary&ratio=1&rotation=0&showTitle=false&size=69984&status=done&style=none&taskId=u0c13bb2d-1199-487d-8771-b18397f27e1&title=&width=788)

## 3.9 Prefetch / Burst Length

### 3.9.1 Prefetch

**预先存取数据**就是在 I/O 控制器发出请求之前，存储单元已经事先准备好了 2/4/8bit 数据。简单说就是把**并行传输的数据转换为串行数据流**，从而得到 1/4/8 倍速率的串行数据。由于 DRAM 的内核比接口慢得多，因此通过并行访问信息，然后将其串行化输出接口，从而弥补了这一差异。如下图所示，**从存储单元阵列到 I/O 缓冲区之间有并串的转换动作**。

![](https://cdn.nlark.com/yuque/0/2023/png/22935284/1696861406684-1dcedd3a-a976-4d15-be2c-bdd552a0e716.png#averageHue=%23e8eedd&height=359&id=LCVp1&originHeight=501&originWidth=960&originalType=binary&ratio=1&rotation=0&showTitle=false&status=done&style=none&title=&width=688)

Prefetch 相当于 DRAM core 同时修了多条高速公路连到外面的 IO 口，来**解决 IO 速率比内部核心速率快的问题**，**IO 数据速率跟核心频率的倍数关系就是 prefetch**。**这种存储阵列内部的实际位宽较大，但是数据输出位宽却比较小的设计，就是所谓的数据预取技术**，它可以让内存的数据传输频率倍增。

在实际操作中，一个时钟周期内同时将相邻列地址的数据一起取出来，这个是并排取出来的，相当于拓宽车道。并行取出 DRAM 数据，再由列地址**选择**输出。

![](https://cdn.nlark.com/yuque/0/2023/jpeg/22935284/1697428051331-9eb995cf-05bc-4856-9022-2e15482551c6.jpeg#averageHue=%23eaeae9&clientId=u68238361-3e74-4&from=paste&id=LXHix&originHeight=665&originWidth=1526&originalType=url&ratio=1&rotation=0&showTitle=false&status=done&style=none&taskId=ufb95ee8f-ee51-4af8-8b80-3a2eab9e23e&title=)

以 DDR3 为例，它的 IO gating buffer 与 FIFO 的接口宽度是 FIFO 与外部 IO 的接口宽度的 8 倍。对于 8bits 位宽的 DDR3 MEMORY chip，为了满足 8n prefetch，IO gating buffer 的宽度要达到 64 bits 的位宽。如下图所示，fifo 两端的位宽就是 8 倍的差距，所以可以实现 8bits 的 prefetch。

![](https://cdn.nlark.com/yuque/0/2023/png/22935284/1697176290391-7da4e491-b283-4bbf-bbba-8823a2360e7f.png#averageHue=%23f3f2f2&clientId=u0afd0ec9-4b1b-4&from=paste&height=445&id=zlkGZ&originHeight=617&originWidth=1066&originalType=url&ratio=1&rotation=0&showTitle=false&status=done&style=none&taskId=u7d3a4ef4-0a19-470f-aa98-4d9da5ea7db&title=&width=768)

从 DDR 才开始有的技术，之前的 SDR 没有。SDRAM 假设最开始是 1bytes 传输到 I/O 缓冲区，接着 8bits 一个周期传输到 FIFO 上，FIFO 再传到外部。**所以 SDRAM 内存芯片一次传输率的数据量就是芯片位宽，那么这个存储单元的容量就是芯片的位宽。**
1. DDR 是两位预取（2-bit Prefetch），也称为 2-n Prefetch（n 代表芯片位宽）。内部一个周期传输 2bytes 到 I/O 缓存区，由于上下沿都读取，所以连接外部的 FIFO 可以实现一个周期 2bytes 数据的传输，2-n prefetch 指的是内部通过增加线的方式，一下子传输了 2byte 数据出来。**DDR SDRAM 内部存储单元容量是芯片位宽（芯片 I/O 口位宽）的两倍**；
2. DDR2 是四位预取（4-bit Prefetch）。内部一个周期传输 4bytes 传输到 I/O 缓冲区，时钟频率变为两倍，再加上上下沿，FIFO 接到外部是就变成了了内部的 4 倍。**DDR2 SDRAM 内部存储单元容量是芯片位宽的四倍**。
3. DDR3 和 DDR4 都是八位预取（8n-bit Prefetch）。**DDR3 和 DDR4 SDRAM 内部存储单元容量是芯片位宽的八倍**。相当于 DDR3 的每一个 IO 都有一个宽度为 8 的 buffer，从 IO 进来 8 个数据后，才把这 8 个数据一次性的写入 DDR 内部的存储单元。

![image.png](https://cdn.nlark.com/yuque/0/2023/png/22935284/1697508802265-161aca6a-7c28-4df5-aa73-14cee54ca1d1.png#averageHue=%23f4f3f3&clientId=u68238361-3e74-4&from=paste&height=539&id=Svv14&originHeight=590&originWidth=759&originalType=url&ratio=1&rotation=0&showTitle=false&size=69418&status=done&style=none&taskId=u0a82327a-504c-470a-a18a-b1e84acc68b&title=&width=693)

8n-bit Prefetch 可以使得内核时钟是 DDR 时钟的四分之一。**DDR4 并没有采用 16n-bit 预取，因为数据预取数增加难度越来越大，直接上 16n-bit 比较难，因此在 DDR4 上采用 Bank Group 设计，相当于预取数提升至两个 8bit，经过两级转换，也就是相当于 16bit**。

芯片位宽的另一种说法是配置模式（Configuration），在 DDR3 时代，一般有 x4，x8，x16。**当 DDR3 为 x8 Configuration 时，一个 Cell 的容量为 8x8bits，即 8 个字节。换一句话说，在指定 bank、row 地址和 col 地址之后，可以往该地址内写入（或读取）8 Bytes。Prefetch 从某一 bank 的某一 row 某一 col 中，一次读取一个 cell（die 位宽*预取）的数据。比如 ddr4 的 x4 位宽*8n 的数据，以 x4\*BL8 的形式发送出去。两个 4n prefetch 拼接也可以以 BL8 的形式发送出去。**

**为什么 ddr4 不用 16n** **prefetch 而是用 BG？

首先 prefetch 决定了 burst 的最小长度，比如 8n prefetch 最小 burst length（BL）就是 8，（注，DDR3 中 BL4 的情况是屏蔽了后 4 个数据，其实还是读了 8 个数据）。BL 越大，数据的 efficiency 越低，万一程序用不到那么大的数据，岂不是后面读的都浪费掉了。但是使用 BG 同样可以拼接产生 BL16/BL32（这取决于 BG2 还是 BG4），却不要求两个 BL8 之间的数据是连续的，每个 BG 输出一个连续的 BL8 即可。

16n prefetch 最终会输出 64 * 16bits=128Byte 的数据，与 64byte 的 cacheline 不对齐。8n * 64=64byte 就正好对齐。（注：ddr5 靠把输出位宽拆成两个 32bit 解决了这个问题，即 32 * 16n=64Byte）。在典型的计算环境中，64 位或 72 位接口使用的是 64 bytes Cacheline，因此 8 字节的预取和 8 字节的突发长度更匹配。缓存行大小和突发长度的任何这种不匹配都会对嵌入式系统的性能产生负面影响。

另外，如果 DDR4 采用 16n prefetch，这一变化将使 DRAM 的体积大大增加，因为所有的线缆都必须包括在内。这将导致 DRAM 成本过高，因此设计人员没有采用 16 字预取，从而节省了成本。

![|399](https://cdn.nlark.com/yuque/0/2023/png/22935284/1697681792409-6415985f-856e-4fb4-b341-f6fe6a8309db.png#averageHue=%23502850&clientId=u5f5606b7-8f3d-4&from=paste&height=285&id=u6c9bc157&originHeight=226&originWidth=274&originalType=url&ratio=1&rotation=0&showTitle=false&status=done&style=none&taskId=ue483595d-16d4-4f8d-8000-8b5cb420e95&title=&width=346)

DDR5 是 16n-bit prefetch。

![](https://cdn.nlark.com/yuque/0/2023/png/22935284/1696861407118-19b8b6d0-5650-4af9-b5b5-1a162745fa71.png#averageHue=%23293744&id=mu4Tv&originHeight=646&originWidth=426&originalType=binary&ratio=1&rotation=0&showTitle=false&status=done&style=none&title=)

为什么要说内部**存储单元容量 ≠ 芯片位宽**，这就是因为预读取。比如 DDR3 预读取 8bits,所以存储单元容量是位宽的 8 倍，这样，一次就能读取 8 * 8=64bits 数据，由于内部时钟是等效时钟的 1/8，所以吞吐量刚好相等，这也就可以在保持内部存储颗粒核心频率不变的情况下（核心频率的提高花费很大，而且很难），将等效的频率提高 8 倍。

预取的缺点是，它实际上决定了 SDRAM 的最小突发长度。例如，DDR3 的预取长度为 8 个字，很难实现 4 个字的高效突发长度。BankGroup 允许设计人员保持较小的预取，同时提高性能，就像预取较大一样。

### 3.9.2 Burst Length / Type / Order

**目前内存的读写基本都是连续的，因为与 CPU 交换的数据量以一个 Cache Line（即 CPU 内 Cache 的存储单位）的容量为准，一般为 64Byte。而现有的 Rank 位宽为 8Byte，那么就要一次连续传输 8 次，这就涉及到我们经常遇到的突发传输的概念。**

突发（Burst）是指**在同一行中相邻的存储单元连续进行数据传输的方式**，连续传输的周期数就是**突发长度**（Burst Lengths，简称 BL）。

**burst length 的长度跟 CPU 的 cache line 大小有关**，**Burst length 的长度有可能大于或者等于 prefetch**。**但是如果 prefetch 的长度大于 burst length 的长度，就有可能造成数据浪费，因为 CPU 一次用不了那么多**。所以从 DDR3 到 DDR4，如果在保持 DDR3 内存 data lane 还是 64（位宽=64bit）的前提下，继续采用增加 prefetch 的方式来提高 IO 速率的话，一次 prefetch 取到的数据就会大于一个 cache line 的大小 （512bits），对于目前的 CPU 系统，反而会带来性能问题。（64bits\*16-n prefetch=1024）

**还有一个问题，列地址信号有十位，为什么寻址能力只有 128？**

**因为列地址的 A2，A1，A0 被用于 Burst Order 功能，A3 被用于 Burst Type 功能**，一般情况采用的都是**顺序读写模式**（即{A2,A1,A0}={0,0,0}），所以此时的 A3 的取值并无直接影响。表中分别对应有 Burst Length 的长度。**Burst Type 和 Burst Order 实际上就是关于 Burst Length 的读写顺序的配置**。

DDR3 内部配置采用了 8n prefetch(预取)来实现高速读写，这也导致了 DDR3 的 Burst Length 一般都是 8。当然也有 Bursth length 为 4 的设置(BC4)，是指另外 4 笔数据是不被传输的或者被认为无效而已。DDR2 内部配置采用的是 4n prefetch，Burst length 有 4 和 8 两种，对于 BL=8 的读写操作，会出现两次 4n Prefetch 的动作。

CA[2:0]的值决定了一次 Burst sequence 的读写地址顺序。比如一次 Burst Read 的时候如果 CA[2:0]=3’b001 表示低三位从地址 1 开始读取，CA3=0 的时候按顺序读取 1，2，3，0，5，6，7，4，CA3=1 的时候交错读取 1，0，3，2，5，4，7，6。

对于 Prefetch 而言，正好是 8N Prefetch，对于 Burst 而言对应 BL8。BC4 其实也是一次 BL8 的操作，只是丢弃了后一半的数据。

![](https://cdn.nlark.com/yuque/0/2023/png/22935284/1697098329474-76d26f4f-f472-4aab-abf0-91a8eb61dd90.png#averageHue=%23e4dad5&clientId=u0d784ff7-bafa-4&from=paste&height=419&id=UgDtA&originHeight=368&originWidth=560&originalType=url&ratio=1&rotation=0&showTitle=false&status=done&style=none&taskId=u3fe60159-b545-492b-bd97-1554454787e&title=&width=637)

对于一个 Bank 里面的 Memory Array，每个 Memory Cell 可以看作是一个 Byte 的集合体。CA[9:3]选中一行中的一个特定 Byte，再由 CA[2:0]选择从这个 Byte 的哪个位置开始操作。CA3 既参与了列地址译码，也决定 Burst 是连续读取还是交错读取。Prefetch 也决定了 I/O Frequency 和 Core Frequency 之间的关系。

**在进行突发传输时，只要指定起始列地址与突发长度，内存就会依次地自动对后面相应数量的存储单元进行读/写操作而不再需要控制器连续提供列地址，只要控制好两段突发读取命令的间隔周期（与 BL 相同）即可做到连续的突发传输。这样，除了第一笔数据的传输需要若干个周期（主要是之前的延迟，一般是 tRCD+CL）外，其后每个数据只需一个周期的即可获得。

在 DDR SDRAM 中，突发长度只有 **2、4、8 **三种选择，没有随机存取的操作（突发长度为 1）和全页式突发。这是为什么呢？

因为 L-Bank 一次就存取两倍于芯片位宽的数据，所以芯片至少也要进行两次传输才可以，否则内部多出来的数据怎么处理？但是，突发长度的定义也与 SDRAM 的不一样了，**它不再指所连续寻址的存储单元数量，而是指连续的传输周期数，每次是一个芯片位宽的数据**。

突发传输为了提高传输效率，涉及到预取的概念。DDR3 突发长度 BL=8，在 t1 时刻我们发起读命令，给出地址 addr1，那么因为是突发传输，所以实际我们将读出 addr1 以及它之后地址，总共 8 个地址，也就读出了 8 个数据；那么 t2 时刻，我们继续读数，给出的地址 addr=addr1+8；结合下图更容易理解：

![](https://cdn.nlark.com/yuque/0/2023/png/22935284/1697080438773-d378ddc7-d0d9-4391-9aed-89a9a27f0a0d.png#averageHue=%23e1dfde&clientId=u0d784ff7-bafa-4&from=paste&id=Kawkr&originHeight=199&originWidth=825&originalType=url&ratio=1&rotation=0&showTitle=false&status=done&style=none&taskId=uc73f834a-deea-45c9-afec-c29745b80fd&title=)

## 3.10 核心频率/时钟频率/工作频率

**核心频率**：内存 Cell 阵列(Memory Cell Array，即内部电容)的刷新频率，它是内存的真实运行频率。由于 SDRAM 采用电容作为存储介质，由于工艺和物理特性的限制，电容充放电的时间难以进一步的缩短，所以内部 Memory Array 的读写频率也受到了限制，一直保持在**133-200MHz**范围内。n-prefetch 需要内部存储单元在核心频率下多读 n 倍的数据。

**时钟频率**：也叫 I/O Buffer 的传输频率，必须和 CPU 的外频相匹配才能工作。**根据 n-bites 的 prefetch，时钟频率是核心频率的 n/2 倍**。DDR2 的时钟频率在核心频率的基础上倍频；DDR3 的时钟频率是核心频率的 4 倍。

**工作频率**：也叫等效频率、外部接口频率、数据传输频率，根据双边沿采样的原则，它的值是时钟频率的 2 倍。DDR4 时设计出了 Bank Group 设计，允许各个 Bank Group 具备独立启动操作读、写等动作特性，所以等效频率提升到了核心频率的 16 倍。

**理解核心频率与工作频率的关系需要先理解 Prefetch 的原理**。

![](https://cdn.nlark.com/yuque/0/2023/png/22935284/1696861402608-9df8251f-57c7-4a5a-a038-f74fac35be7a.png#averageHue=%23f0c144&height=266&id=W7XcK&originHeight=245&originWidth=500&originalType=binary&ratio=1&rotation=0&showTitle=false&status=done&style=none&title=&width=542)

**关于三者频率关系之间的解释**：
1、对 SDR 来说，存储单元频率、I/O 频率及数据传输率都是相同的，比如经典的 PC133，三种频率都是 133MHz。SDR 在一个时钟周期内只能读/写一次，只在时钟上升期读/写数据，当同时需要读取和写入时，就得等待其中一个动作完成之后才能继续进行下一个动作；
2、对 DDR 来说，在一个时钟周期内传输两次数据，在时钟的上升期和下降期各传输一次数据(通过差分时钟技术实现)，在存储阵列频率不变的情况下，数据传输率达到了 SDR 的两倍，此时就需要 I/O 从存储阵列中预取 2bit 数据，因此 I/O 的工作频率是存储阵列频率的两倍。
注：因为在出口处的流量增大了，所以入口的流量也要相应的增大。所以有了 2bit 预取技术。

![](https://cdn.nlark.com/yuque/0/2023/png/22935284/1696861402870-0f8d8e6a-8e40-4c51-87b6-b337b130f059.png#averageHue=%23c8e4cb&id=K4LGq&originHeight=469&originWidth=659&originalType=binary&ratio=1&rotation=0&showTitle=false&status=done&style=none&title=)

DDR 从发明之初直到 DDR4 并没有太大的变化，都是通过调整 prefetch bits 不断的增加传输速度，核心频率一直保持在 100-266MHZ 之间，prefetch 最大 8n。但是到了 DDR4 以后增加 prefetch 的方法玩不下去了，因为 cache line 最大只有 64Bytes，8bit prefetch 时一条 cahe line 需要 BL8 就可以满足了，如果是 16 bit prefetch 一次取 128bits BL8 就会是 128 Bytes，其中的 64Bytes 被浪费了，所以 DDR4 开始提升核心频率到 200-400MHZ 了。
![](https://cdn.nlark.com/yuque/0/2023/png/22935284/1697019186846-d42a828a-d9b6-4614-b851-410eb3e305b9.png#averageHue=%23f9fbfa&clientId=ubee3ae6b-a866-4&from=paste&id=VMb9t&originHeight=348&originWidth=572&originalType=url&ratio=1&rotation=0&showTitle=false&status=done&style=none&taskId=u93b8d384-499d-4ba8-8fc7-575b30d1240&title=) 

## 3.11 带宽 Bandwidth

**带宽=等效工作频率 * 通道数 * 通道位宽**。

带宽分为**理论峰值带宽、实测峰值带宽、场景实时带宽**。**如 LPDDR5 的带宽为 44GB/s，工作频率为 5500Mbps，即 44=5.5 * 4 Channel * 16 位 DQ/8**。

![|800](https://cdn.nlark.com/yuque/0/2023/png/22935284/1696861403174-b75b3ea0-fa1d-4c05-b340-694ac434d874.png#averageHue=%23eceeee&height=150&id=remas&originHeight=217&originWidth=1188&originalType=binary&ratio=1&rotation=0&showTitle=false&status=done&style=none&title=&width=822)

功耗和带宽：

![](https://cdn.nlark.com/yuque/0/2023/png/22935284/1696861403601-b856ebcf-956e-46f2-b346-5ffdfe917f1e.png#averageHue=%23e4eeec&id=BjXEQ&originHeight=534&originWidth=2528&originalType=binary&ratio=1&rotation=0&showTitle=false&status=done&style=none&title=)

有功耗拐点的原因：为提升 IO 信号质量，1.7GHz 以后开启 ODT，ODT 范围 34-240Ω。

![](https://cdn.nlark.com/yuque/0/2023/png/22935284/1696861404051-a5709ec4-53e9-4161-9c29-1ea2a83a6d30.png#averageHue=%23fcfcfc&id=yHXfj&originHeight=432&originWidth=847&originalType=binary&ratio=1&rotation=0&showTitle=false&status=done&style=none&title=)

频点和带宽：带宽和频率基本为线性关系，随着带宽增加，实际带宽/有效带宽效率逐步降低。

![](https://cdn.nlark.com/yuque/0/2023/png/22935284/1696861404319-648905cf-44d2-4034-be53-9ce1c0cec2db.png#averageHue=%23fdfcfc&height=460&id=WRSkr&originHeight=698&originWidth=680&originalType=binary&ratio=1&rotation=0&showTitle=false&status=done&style=none&title=&width=448)

## 3.12 DDR Latency/时序参数

Delay 与 Latency 虽然都有“延迟”的意思，但本质上是不同的。
Delay ：事情要在这个时间之后开始；
Latency：事情已经发生，但是还不够稳定需要一个稳定时间。
**DDR latency 指等待对系统内存中存储数据的访问完成时引起的延期**。也表示系统进入数据存取操作就绪状态前等待内存响应的时间，它通常用 4 个连着的数字来表示，如 3-4-4-8，分别指 **CL-tRCD-tRP-tRAS** 这四个数值，单位都是时钟周期。

### 3.12.1 CL

**CAS Latency，列地址脉冲选通潜伏期，最为重要，表示读取命令发出与第一个有效数据到 DDR 端口之间的延迟。相关的列地址被选中之后，将会触发数据传输，但从存储单元中输出到真正出现在内存芯片的 I/O 接口之间还需要一定的时间（数据触发本身就有延迟，而且还需要进行信号放大）。**CL 越高表示所需延迟的时间越长。下图分别表示 CL = 3 和 CL = 4 的两种情况，如果读命令在第 n 个时钟周期发出，CL = m，则读取的数据在第 n+m 个时钟时有效。

注：**CL 只是针对读取操作**。

![](https://cdn.nlark.com/yuque/0/2023/png/22935284/1697442400651-4a5a6869-525e-4ef6-bca7-849fa7121d03.png#averageHue=%23f7f6f6&clientId=u68238361-3e74-4&from=paste&height=595&id=qWvbU&originHeight=736&originWidth=1197&originalType=url&ratio=1&rotation=0&showTitle=false&status=done&style=none&taskId=uf604e389-6f20-44aa-b727-a2f8f75485d&title=&width=968)

CL 的长度在寄存器中设置：

![image.png](https://cdn.nlark.com/yuque/0/2023/png/22935284/1697443605956-75d3d054-4834-4f14-a3b6-96ecf52cd47a.png#averageHue=%23e8e6e2&clientId=u68238361-3e74-4&from=paste&height=608&id=b6kW7&originHeight=608&originWidth=405&originalType=binary&ratio=1&rotation=0&showTitle=false&size=52440&status=done&style=none&taskId=u01fb736e-4230-45fd-ac1d-6f233595a16&title=&width=405)

### 3.12.2 tRCD

表示内存行地址到列地址的延迟时间（RAS to CAS Delay），打开一行并访问其中的列所需的最小时钟周期数，即 tRCD（Row Address to Column Address Delay）。也表示从行有效到读/写命令发出之间的间隔。

### 3.12.3 tRP

表示从内存行地址控制器预充电时间（RAS Precharge），即 tRP(Row Precharge Command Period),发出预充电命令与打开下一行之间所需的最小时钟周期数。指内存从结束一个行访问到重新开始的间隔时间。

在数据读取完之后，为了腾出 sense amplifier 以供同一 Bank 其它行的寻址并传输数据，内存芯片将进行预充电操作来关闭当前工作行。如果当前寻址的存储单元是 B1，R2，C6，接下来寻址单元是 B1，R2，C4，则不用预充电，因为 sense amplifier 正在为这一行服务。但如果寻址单元是 B1，R4，C4，由于是同一 Bank 的不同行，那么就必须先把 R2 关闭，才能对 R4 寻址。从关闭现在的工作行，到可以打开新的工作行之间的间隔就是 tRP。

举例如下：Datasheet 中通常会标注 DDR 的速度等级(Speed Grade)，如-068 对应的 Cycle Time=0.682ns ，对应的 tRCD=22\*Cycle Time=13.5ns。

![image.png](https://cdn.nlark.com/yuque/0/2023/png/22935284/1698648996823-b38660d9-0d43-4e39-8dab-9c0cd36df6ce.png#averageHue=%233d3e3c&clientId=u24f34a98-ffbd-4&from=paste&height=502&id=u46e6400d&originHeight=502&originWidth=779&originalType=binary&ratio=1&rotation=0&showTitle=false&size=86138&status=done&style=none&taskId=u5439919f-ecbc-4c23-bd34-df7ca700e90&title=&width=779)

![image.png](https://cdn.nlark.com/yuque/0/2023/png/22935284/1698649070639-bb347a2a-ae17-4747-b617-266412241177.png#averageHue=%23e1e1e6&clientId=u24f34a98-ffbd-4&from=paste&height=145&id=u2fbc9f25&originHeight=145&originWidth=885&originalType=binary&ratio=1&rotation=0&showTitle=false&size=27134&status=done&style=none&taskId=u3e43c0ab-1ca8-4639-ae86-f8422c78f50&title=&width=885)

### 3.12.4 tRAS

表示内存行地址控制器激活时间 Act-to-Precharge Delay（tRAS,Row Active Time）,行活动命令与发出预充电命令之间所需的最小时钟周期数。也就是对下一次预充电时间进行限制。

## 3.12 DDR 性能功耗

### 3.12.1 功耗影响因子

存储是被动器件，单体功耗只能通过制程工艺迭代优化。功耗行为由 CPU 调度决定。

![image.png](https://cdn.nlark.com/yuque/0/2023/png/22935284/1696989618795-3af47558-9b22-4aad-be0b-c8d80afc10ee.png#averageHue=%23f5f5f5&clientId=u080925fc-7a7b-4&from=paste&id=ZFFdv&originHeight=585&originWidth=842&originalType=url&ratio=1&rotation=0&showTitle=false&size=81319&status=done&style=none&taskId=uc3083fac-ec5a-4309-9564-f05e9fb7df0&title=)

![image.png](https://cdn.nlark.com/yuque/0/2023/png/22935284/1696989606679-bbb6698d-0f3a-4271-8896-7a4a0cad3579.png#averageHue=%23e6eae8&clientId=u080925fc-7a7b-4&from=paste&id=lrgOk&originHeight=936&originWidth=1864&originalType=url&ratio=1&rotation=0&showTitle=false&size=534665&status=done&style=none&taskId=ua9c3ded0-384a-4b31-8585-8205cb94680&title=)

![image.png](https://cdn.nlark.com/yuque/0/2023/png/22935284/1696989606785-e5101561-d3a7-432e-b38b-f26f25b69c34.png#averageHue=%23c6ccb3&clientId=u080925fc-7a7b-4&from=paste&id=nKVLB&originHeight=979&originWidth=1849&originalType=url&ratio=1&rotation=0&showTitle=false&size=851368&status=done&style=none&taskId=uc39330e9-4af0-4627-90af-cdefbb0c301&title=)

Cell Array ：存储单元由于存在刷新、行列驱动等电路，因此节省空闲存储单元能够有效节省因存储单元造成的功耗的损失；

数字逻辑及刷新电路：数字逻辑主要实现状态机的正常运转、地址解析，以及刷新控制。该单元的耗电量主要工作在 DDR 的活跃状态，且这部分功耗固定，且不容易被控制；

PHY 电路：功耗的主要来源，PHY 电路集成了大量模拟电路，如 ZQ 校准电路、上下拉电阻、ODT 等 VTT 网络电路；

![](https://cdn.nlark.com/yuque/0/2023/png/22935284/1696861390103-37d1c191-8448-4183-9fbf-8866dcf79d84.png#averageHue=%23ecf5f4&height=485&id=StWC0&originHeight=957&originWidth=1514&originalType=binary&ratio=1&rotation=0&showTitle=false&status=done&style=none&title=&width=768)

DDR 低功耗设计的两个重要方向：**整机供电拆分和电压降低收益**。

![image.png](https://cdn.nlark.com/yuque/0/2023/png/22935284/1696989563150-fb4e2ccd-41e1-423d-b357-7c269ce2ccf0.png#averageHue=%23f0efee&clientId=u080925fc-7a7b-4&from=paste&height=141&id=iFSjL&originHeight=183&originWidth=1051&originalType=url&ratio=1&rotation=0&showTitle=false&size=37224&status=done&style=none&taskId=u2e0e5ca9-e46f-4e99-a605-2a169fd40ce&title=&width=810)

工作频率和带宽获取：

![image.png](https://cdn.nlark.com/yuque/0/2023/png/22935284/1696989618864-0ad4c555-bfee-40ef-9a7d-a82598f3eb09.png#averageHue=%23e7e7e7&clientId=u080925fc-7a7b-4&from=paste&id=mcavG&originHeight=253&originWidth=817&originalType=url&ratio=1&rotation=0&showTitle=false&size=27754&status=done&style=none&taskId=u6e13f5ed-b13f-4051-adc5-9b6ac5d9ee8&title=)

分析不同场景下 DDR 频率的占比：

hynix vs 镁光 DDR5X 功耗

![image.png](https://cdn.nlark.com/yuque/0/2023/png/22935284/1696989644756-174de911-ff0c-49d8-ac41-c1e26d1de822.png#averageHue=%23fdfcfb&clientId=u080925fc-7a7b-4&from=paste&id=xSJ2a&originHeight=548&originWidth=971&originalType=url&ratio=1&rotation=0&showTitle=false&size=30618&status=done&style=none&taskId=u9c958ccd-a3ac-4b6d-9896-de4c56d9f85&title=)

![image.png](https://cdn.nlark.com/yuque/0/2023/png/22935284/1696989644755-afa9857d-51a6-4752-803c-a3a85f464286.png#averageHue=%23e5e3e1&clientId=u080925fc-7a7b-4&from=paste&id=dZ4xZ&originHeight=283&originWidth=1479&originalType=url&ratio=1&rotation=0&showTitle=false&size=44181&status=done&style=none&taskId=u94268e95-1750-4069-94cc-e4a25220873&title=)

#### 3.12.1.1 Power Gating

Power Gating 技术可降低部分场景漏电。

![image.png](https://cdn.nlark.com/yuque/0/2023/png/22935284/1696989563180-b3f0a209-c20a-4128-b3e3-5a60504d1e37.png#averageHue=%23f9f6f5&clientId=u080925fc-7a7b-4&from=paste&id=gZwp3&originHeight=455&originWidth=1267&originalType=url&ratio=1&rotation=0&showTitle=false&size=88579&status=done&style=none&taskId=uc042b83f-4813-41a8-bf19-f3f41e8cc3a&title=)

#### 3.12.1.2 制程优化

减小 Cell 单元尺寸。
![image.png](https://cdn.nlark.com/yuque/0/2023/png/22935284/1696989563180-8b50c07b-66e2-4a9a-baab-cb6d6ffb8c4a.png#averageHue=%23e7e7e7&clientId=u080925fc-7a7b-4&from=paste&height=260&id=qxiSp&originHeight=349&originWidth=737&originalType=url&ratio=1&rotation=0&showTitle=false&size=73872&status=done&style=none&taskId=u2f495f58-615d-4258-9cb6-12dfbcf6eb1&title=&width=548)

如下图所示为 LPDDR 制程演化历史，特别需要注意的是 LPDDR 的制程不是常规的数字表示方法，而是用 1X、1Y 和 1Z 表示。

![](https://cdn.nlark.com/yuque/0/2023/png/22935284/1696861393299-7d368909-107d-4bba-9a89-6bf298b59cf6.png#averageHue=%23cacdca&height=353&id=hisC2&originHeight=445&originWidth=1352&originalType=binary&ratio=1&rotation=0&showTitle=false&status=done&style=none&title=&width=1073)

#### 3.12.1.3 DVFS

DVFS: Dynamic Voltage and Frequency Scaling，动态电压频率调节。根据芯片所运行的子系统对计算或带宽能力的不同需求，动态调节芯片或总线的运行频率和电压（不需要高性能时，降低电压和频率，以降低功耗；在需要高性能时，提高电压和频率，以提高性能），从而达到兼顾性能而又节能的目的。

DVFSRC 是一个硬件模块，用于采集软硬件的所有请求，并转化为最小工作电压和 DRAM 频率，最终决策合适的 DDR 频率和电压来满足这些请求。

![image.png](https://cdn.nlark.com/yuque/0/2023/png/22935284/1696989583391-9b9f0bfe-a3ac-4254-a5b9-117f7d08b43b.png#averageHue=%23919191&clientId=u080925fc-7a7b-4&from=paste&id=ka61c&originHeight=545&originWidth=1024&originalType=url&ratio=1&rotation=0&showTitle=false&size=84934&status=done&style=none&taskId=ua7532a49-6676-4e18-be9f-02a103f6b09&title=)

DVFS core switches between VDD2L and VDD2H。目前 DVFS 支持调频调压策略主要包含如下几种：

- ondemand：系统负载小时以低频率运行，系统负载提高时按需提高频率
- conservative：跟 ondemand 方式类似， 不同之处在于提高频率时渐进提高
- performance：cpu 按照支持的最高频率运行
- powersave：cpu 以支持的最低频率运行
- userspace：使用用户在/sys 节点 scaling_setspeed 设置的频率运行。

在调整频率和电压时，要特别注意调整的顺序，当频率由高到低调整时，应该先降频率，再降电压；相反，当升高频率时，应该先升电压，再升频率； 

![image.png](https://cdn.nlark.com/yuque/0/2023/png/22935284/1696989583454-2d930ff1-66d6-4a94-9e2f-b361fe699268.png#averageHue=%23eeeded&clientId=u080925fc-7a7b-4&from=paste&id=I53ZU&originHeight=517&originWidth=1280&originalType=url&ratio=1&rotation=0&showTitle=false&size=222540&status=done&style=none&taskId=u4864d76f-cf71-4cf3-ba0d-ee7aa299208&title=)

![image.png](https://cdn.nlark.com/yuque/0/2023/png/22935284/1696989583509-46056240-8be5-4df2-87ae-96f226208077.png#averageHue=%23ececeb&clientId=u080925fc-7a7b-4&from=paste&id=DqBxR&originHeight=674&originWidth=1280&originalType=url&ratio=1&rotation=0&showTitle=false&size=377301&status=done&style=none&taskId=u563b664f-91dd-4c51-b036-839a45d1db5&title=)

![image.png](https://cdn.nlark.com/yuque/0/2023/png/22935284/1696989583408-dc57aed0-4ff0-4b7a-a2de-fb22896c3baa.png#averageHue=%23e9e7e5&clientId=u080925fc-7a7b-4&from=paste&id=SUjMU&originHeight=156&originWidth=1280&originalType=url&ratio=1&rotation=0&showTitle=false&size=110068&status=done&style=none&taskId=u5ee52dbf-94ff-4c51-b95e-a52f8743c71&title=)

### 3.12.2 性能影响因子

理想情况下，频率与位宽固定后，带宽也就不可更改，在内存的工作周期内，不可能总处于数据传输的状态，因为要有命令、寻址等必要的过程。但这些操作占用的时间越短，内存工作的效率越高，性能也就越好。非数据传输时间的主要组成部分就是各种延迟与潜伏期。衡量 DDR 性能的指标主要是 Latency，主要包括 tRCD、CL 和 tRP。

![image.png](https://cdn.nlark.com/yuque/0/2023/png/22935284/1696989644828-427bd4d8-3dd4-4a47-9155-3f9a2ec0c943.png#averageHue=%23f89832&clientId=u080925fc-7a7b-4&from=paste&id=Zv2bW&originHeight=580&originWidth=704&originalType=url&ratio=1&rotation=0&showTitle=false&size=239935&status=done&style=none&taskId=u70a78e49-8dd7-4844-97e5-ab1b1c29f76&title=)

要特别注意 NOC，它对性能的影响非常明显。在 L3 得不到的数据，需要先去总线申请，再去 SLC 和 DDR 中找。

![image.png](https://cdn.nlark.com/yuque/0/2023/png/22935284/1696989644840-008e3c50-86f8-4020-b120-a17b81fd85f1.png#averageHue=%234d4f4e&clientId=u080925fc-7a7b-4&from=paste&id=wt2Wi&originHeight=660&originWidth=821&originalType=url&ratio=1&rotation=0&showTitle=false&size=245674&status=done&style=none&taskId=ud465fc15-9730-4cb2-be81-6aff2d1d508&title=)

#### 3.12.2.1 PH/PFH/PM

PH : 要寻址的行与 Bank 是空闲的。也就是说该 Bank 的所有行是关闭的，此时可直接发送行有效命令，数据读取前的总耗时为 tRCD+CL，这种情况我们称之为**页命中 （PH，Page Hit）**。<br />PFH : 要寻址的行正好是现有的工作行，也就是说要寻址的行已经处于选通有效状态，此时可直接发送列寻址命令，数据读取前的总耗时仅为 CL，这就是所谓的背靠背 （Back to Back）寻址，我们称之为**页快速命中（PFH，Page Fast Hit）或页直接命中（PDH，Page Direct Hit）**。<br />PM : 要寻址的行所在的 Bank 中已经有一个行处于活动状态（未关闭），这种现象就被称作寻址冲突，此时就必须要进行预充电来关闭工作行，再对新行发送行有效命令。结果，总耗时就是 tRP+tRCD+CL，这种情况我们称之为**页错失 （PM，Page Miss）**。<br />显然，PFH 是最理想的寻址情况，PM 则是最糟糕的寻址情况。上述三种情况发生的机率各自简称为 PHR —— PH Rate、PFHR —— PFH Rate、PMR —— PM Rate。因此，系统设计人员都尽量想提高 PHR 与 PFHR，同时减少 PMR，以达到提高内存工作效率的目的。

从重要性来论，优先优化的顺序也是 CL → tRCD → tRP，因为 CL 的遇到的机会最多，tRCD 其次，tRP 如果页面交错管理的好，大多不受影响。

**增加 PHR 的方法**<br />显然，这与预充电管理策略有着直接的关系，目前有两种方法来尽量提高 PHR。自动预充电技术就是其中之一，它自动的在每次行操作之后进行预充电，从而减少了日后对同一 Bank 不同行寻址时发生冲突的可能性。但是，如果要在当前行工作完成后马上打开同一 Bank 的另一行工作时，仍然存在 tRP 的延迟。怎么办？ 此时就需要 Bank 交错预充电了。<br />早期非常令人关注的 VIA 4 路交错式内存控制，就是在一个 Bank 工作时，对另一个 Bank 进行预充电或者寻址（如果要寻址的 Bank 是关闭的)。这样，预充电与数据的传输交错执行，当访问下一个 Bank 时，tRP 已过，就可以直接进入行有效状态了，如果配合得理想，那么就可以实现无间隔的 Bank 交错读/写（一般的，交错操作都会用到自动预充电）,这是比 PFH 更好的情况，但它只出现在后续的数据不在同一页面的时时候。当时 VIA 声称可以跨 Rank 进行 16 路内存交错，并以 LRU（Least Recently Used，近期最少使用）算法进行 交错预充电/寻址管理。<br />![](https://cdn.nlark.com/yuque/0/2023/gif/22935284/1698651285007-a7fb8164-9ae3-42a3-bdf7-02d18ef485b1.gif#averageHue=%23eae9e9&clientId=u24f34a98-ffbd-4&from=paste&id=u21a14f5c&originHeight=315&originWidth=550&originalType=url&ratio=1&rotation=0&showTitle=false&status=done&style=none&taskId=ubc95ee20-368c-4074-ac2b-1fd40f67268&title=)<br /> Bank 交错自动预充电 / 读取时序图： Bank 0 与 Bank 3 实现了无间隔交错读取，避免了 tRP 与 tRCD 对性能的影响 ，是最理想的状态

**增加 PFHR 的方法**<br />无论是自动预充电还是交错工作的方法都无法消除同行（页面）寻址时 tRCD 所带来的延迟。要解决这个问题，就要尽量让一个工作行在进行预充电前尽可能多的接收工作命令，以达到背靠背的效果，此时就只剩下 CL 所造成的读取延迟了（写入时没有延迟）。<br />如何做到这一点呢？现在我们就又接触到 tRAS 这个参数，在 BIOS 中所设置的 tRAS 是指行有效至预充电的最短周期，在内存规范中定义为 tRAS（min），过了这个周期后就可以发出预充电指令。对于 SDRAM 和 DDR SDRAM 而言，一般是预充电命令至少要在行有效命令 5 个时钟周期之后发出，最长间隔视芯片而异（目前的 DDR SDRAM 标准一般基本在 70000ns 左右），否则工作行的数据将有丢失的危险。那么这也就意味着一个工作行从有效（选通）开始，可以有 70000ns 的持续工作时间而不用进行预充电。显然，只要北桥芯片不发出预充电（包括允许自动预充电）的命令，行打开的状态就会一直保持。在此期间的对该行的任何读写操作也就不会有 tRCD 的延迟。可见，如果北桥芯片在能同时打开的行（页）越多，那么 PFHR 也就越大。需要强调的是，这里的同时打开不是指对多行同时寻址（那是不可能的），而是指多行同时处于选通状态。我们可以看到一些 SDRAM 芯片组的资料中会指出可以同时打开多少个页的指标，这可以说是决定其内存性能的一个重要因素。<br />但是，可同时打开的页数也是有限制的。从 SDRAM 的寻址原理讲，同一 L-Bank 中不可能有两个打开的行（读出放大器只能为一行服务），这就限制了可同时打开的页面总数。以 SDRAM 有 4 个 L-Bank，北桥最多支持 8 个 P-Bank（4 条 DIMM）为例，理论上最多只能有 32 个页面能同时处于打开的状态。而如果只有一个 P-Bank，那么就只剩下 4 个页面，因为有几个 L-Bank 才能有同时打开几个行而互不干扰 。Intel 845 的 MHC 虽然可以支持 24 个打开的页面，那也是指 6 个 P-Bank 的情况下（845MCH 只支持 6 个 P-Bank）。可见 845 已经将同时打开页数发挥到了极致。<br />不过，同时打开页数多了，也对存取策略提出了一定的要求。理论上，要尽量多地使用已打开的页来保证最短的延迟周期，只有在数据不存在（读取时）或页存满了（写入时）再考虑打开新的指定页，这也就是变向的连续读 / 写。而打开新页时就必须要关闭一个打开的页，如果此时打开的页面已是北桥所支持的最大值但还不到理论极限的话 （如果已经达到极限，就关闭有冲突的 L-Bank 内的页面即可），就需要一个替换策略，一般都是用 LRU 算法来进行，这与 VIA 的交错控制大同小异。<br />回到正题，虽然 tRAS 代表的是最小的行有效至预充电期限，但一般的，北桥芯片一般都会在这个期限后第一时间发出预充电指令（自动预充电时，会在 tRAS 之后自动执行预充电命令），只有在与其他操作相冲突时预充电操作才被延后（比如，DDR SDRAM 标准中规定，在读取命令发出后不能立即发出预充电指令）。因此，tRAS 的长短一直是内存优化发烧友所争论的话题，在最近一两年，由于这个参数在 BIOS 选项中越来越普及，所以也逐渐被用户所关注。其实，在 SDRAM 时代就没有对这个参数有刻意的设定，在 DDR SDRAM 的官方组织 JEDEC 的相关标准中，也没有把其列为必须标明的性能参数 （CL、tRCD、tRP 才是），tRAS 应该是某些主板厂商炒作出来的，并且在主板说明书上也注明越短越好。<br />其实，缩小 tRAS 的本意在于，尽量压缩行打开状态下的时间，以减少同 L-Bank 下对其他行进行寻址时的冲突，从内存的本身来讲，这是完全正确的做法，符合内存性能优化的原则，但如果放到整体的内存系统中，伴随着主板芯片组内存页面控制管理能力的提升，这种做法可能就不见得是完全正确的，在下文中我们会继续分析 tRAS 的不同长短设置对内存性能所带来的影响。

#### 3.12.2.1 BL

BL 越长，对于连续的大数据量传输很有好处，但是对零散的数据，BL 太长反而会造成总线周期的浪费，虽然能通过一些命令来进行终止，但也占用了控制资源。以 Rank 位宽 64bit 为例 ，BL=4 时，一个突发操作能传输 32 字节的数据，为了满足 Cache Line 的容量需求，还得多发一次，如果是 BL=8，一次就可以满足需要，不用再次发出读取指令。而对于 2KB 的数据 ，BL=4 的设置意味着要每隔 4 个周期发送新的列地址，并重复 63 次。而对于 BL=256，一次突发就可完成，并且不需要中途再进行控制，但如果仅传输 64 字节，就需要额外的命令来中止 BL=256 的传输。而额外的命令越多，越占用内存子系统的控制资源，从而降低总体的控制效率，从这可以看出 BL 对性能的影响因素。

## 3.13 DDR 封装拓扑及布局布线

### 3.13.1 DDR4 引脚分布

![image.png](https://cdn.nlark.com/yuque/0/2023/png/22935284/1696989909099-651f6ccb-2f54-42fb-9b6c-1100099f309b.png#averageHue=%2371f734&clientId=u33443cfc-91a7-4&from=paste&id=uPQT0&originHeight=945&originWidth=631&originalType=url&ratio=1&rotation=0&showTitle=false&size=99472&status=done&style=none&taskId=u109df456-3979-407c-84ad-2a90476df3a&title=)
<a name="cj7lI"></a>

### 3.13.2、DDR5 引脚分布

![image.png](https://cdn.nlark.com/yuque/0/2024/png/22935284/1713258085289-2fef331d-bbcb-49d2-9c3c-c7d7dde9efae.png#averageHue=%23faf9f8&clientId=u6c012d3c-3157-4&from=paste&height=924&id=u7d8700be&originHeight=924&originWidth=845&originalType=binary&ratio=1&rotation=0&showTitle=false&size=172693&status=done&style=none&taskId=uc0682c25-8376-4354-9f7a-5f8e7e1ddd1&title=&width=845)
<a name="nO50E"></a>

### 3.13.3 DDR 封装

一般一个 Package 包含一个 Die（这里指一个内存颗粒），即 SDP，此时容量还是为一个内存颗粒的容量大小。而一个 Package 包含 2 个 Die，即 DDP（Dual-Die Package）的封装方式，此时容量就是 2 个内存颗粒的容量。

![](https://cdn.nlark.com/yuque/0/2023/png/22935284/1696861410525-df462cd2-e048-4553-bba5-fdc29dd20b45.png#averageHue=%23f3f2f2&id=oHnN2&originHeight=155&originWidth=549&originalType=binary&ratio=1&rotation=0&showTitle=false&status=done&style=none&title=)

DDP 这种封装方式，也可以理解为将**两个内存颗粒并联，扩展位宽**，比如两个 8 位的内存颗粒，采用 DDP 的封装方式，那么整体上就可以看做是一个 16 位的内存颗粒。

### 3.13.4 DDR 拓扑结构

1、DDR 的布线拓扑分为 T 型和 fly-by 结构。DDR3 之前频率较低，都采用 T 型；

![](https://cdn.nlark.com/yuque/0/2023/png/22935284/1696861409427-66de261b-f247-48c2-a5a3-092e4a8f601e.png#averageHue=%23b5b8ae&height=374&id=EY3QC&originHeight=444&originWidth=1348&originalType=binary&ratio=1&rotation=0&showTitle=false&status=done&style=none&title=&width=1135)

2、DDR3 超过 1GHz 时钟后，T 型结构的眼图较差，采用 fly-by 既可以节省布局空间，还可以改善信号完整性；

3、在 4 die 以下，T 型和 fly-by 差别不大，超过 4 die 时，fly-by 的优势比较明显；

4、两者的区别在 Command/Address/Clock 是 fly-by 结构，尾端会串联一个端接电阻。

5、针对单 rank，DQ 和 DQS 还是端到端（例如 Device 1 的数据输出到数据总线的 DATA[0:7] 信号上，Device 2 的数据输出到数据总线的 DATA[8:15] 上。这样的组织方式下，访问 16 bits 的数据就只需要一个访问周期就可以完成，而不需要分解为两个 8 bits 的访问周期。）；而针对双 rank，DQ 也可以做 fly-by。对于每一个 DRAM，其 Data Skew 不同，因此需要做 Write Leveling 来 de-skew 每个 DRAM 的 skew。

![](https://cdn.nlark.com/yuque/0/2023/png/22935284/1696861410040-3c55b7ab-2ffb-418b-bba6-599a3eb1c490.png#averageHue=%238e8e77&id=SqEN9&originHeight=744&originWidth=1049&originalType=binary&ratio=1&rotation=0&showTitle=false&status=done&style=none&title=)

多个 DRAM Devices 共享控制和数据总线，DRAM Controller 通过 Chip Select 分时单独访问各个 DRAM Devices。此外，在其中一个 Device 进入刷新周期时，DRAM Controller 可以按照一定的调度算法，优先执行其他 Device 上的访问请求，提高系统整体内存访问性能。 

#### 3.13.4.1 深度级联与宽度级联

深度级联：例如两个 RANK 的连接方式。此时两个设备连接到相同的地址和数据总线上，但需要两个 CS 信号来分别给每个设备寻址。深度主要是指 DQ 信号是共用的。<br />![image.png](https://cdn.nlark.com/yuque/0/2023/png/22935284/1697523213636-aa7ee6b0-cd86-4f50-a7d0-2e380be3fc30.png#averageHue=%23fdfef8&clientId=u88f81057-d1b4-4&from=paste&height=398&id=TiVaR&originHeight=398&originWidth=401&originalType=binary&ratio=1&rotation=0&showTitle=false&size=165779&status=done&style=none&taskId=u3a896f25-4a9a-4122-bc22-f9c673739da&title=&width=401)<br />宽度级联：假设需要一个 8Gb 内存，并且芯片接口是 x8。可以选择一个 8Gb x8 设备或两个 4Gb x4 设备，并在 PCB 上以“宽度级联”方式连接它们。在宽度级联的情况下，两个 DRAM 都连接到相同的芯片选择、地址和命令总线上，但使用不同的数据总线部分（DQ 和 DQS）。<br />![image.png](https://cdn.nlark.com/yuque/0/2023/png/22935284/1697523258678-33da3350-3586-400e-839f-f4ed4ef567ca.png#averageHue=%23f9fdf9&clientId=u88f81057-d1b4-4&from=paste&height=380&id=pEPn6&originHeight=380&originWidth=352&originalType=binary&ratio=1&rotation=0&showTitle=false&size=146430&status=done&style=none&taskId=u6cadfe53-b8e9-4faa-a270-54b95f109f3&title=&width=352)
<a name="NYUD9"></a>

### 3.13.5 PCB 设计建议（参考 INNO）

1、单 DDR 颗粒信号分组：
- A 通道低 8 位：包括 A_DM0、A_DQS0/DQSB0、A_DQ[0-7]、A_WCK0/B0；
- A 通道高 8 位：包括 A_DM1、A_DQS1/DQSB1、A_DQ[8-15]、A_WCK1/B1；
- B 通道低 8 位：包括 B_DM0、B_DQS0/DQSB0、B_DQ[0-7]、B_WCK0/B0；
- B 通道高 8 位：包括 B_DM1、B_DQS1/DQSB1、B_DQ[8-15]、B_WCK1/B1；
- 地址控制组：包括 CA[0-5]、CKE0/1、CKB/CK、CSB0/1、ODT0。

2、阻抗要求： 
- CA/CKE/CSB/ODT：单端 50R+-10%（45R 更好）；
- DQ/DM：单端 45R+-10%；
- DQS/WCK/CK：差分 85R+-10%；
- 其它单端：单端 50R+-10%。

3、布线长度约束：
DQ/DM/DQS/WCK <= 2 inches
CA/CS/CK <= 3 inches

4、CA/CKE/CSB/ODT 存在通道复用的情况，要采用 flyby 的拓扑结构。建议从 PHY 到 CHA 的长度不大于 2inches，从 CHA 到 CHB 的长度不大于 0.28inches。

5、布线间距约束：
DQ/DM/DQS/WCK >= 4W；
CA/CS/CK >= 3W；

6、远端串扰要求：
数据 Byte 组：-30dB@2.13GHz；
地址控制组：-30dB@1.06GHz；

7、Skew 约束（PKG+PCB）：
- Differential DQS/DQSB：+/-3mil
- Differential WCK/WCKB：+/-3mil
- Differential CK/CKB：+/-3mil
- DQ to DQS/DQSB skew for same Byte：+/-120mil
- A 通道：CKE/CS/CA/ODT to CK/CKB：+/-30mil
- B 通道：CKE/CS/CA/ODT to CK/CKB：+/-30mil
- A 通道：DQS to CK/CKB：+/-600mil
- B 通道：DQS to CK/CKB：+/-600mil

DQ/DM/DQS 信号走线需参考 GND 网络，AC(Address Command)、CLK、MISC(alert reset) 信号走线需参考 VDDQ 或 GND 网络，走线与参考平面边缘间距应小于 3H，保证参考平面完整，保证没有跨平面分割的情况；

PCB 布局时，需要把 DDR 颗粒尽量靠近 DDR 控制器放置。每个电源管脚需要放置一个滤波电容，整个电源上需要有 10uF 以上大电容放在电源入口的位置上。电源最好使用独立的层铺到管脚上去。

串联匹配的电阻最好放在源端，如果是双向信号，那么要统一放在同一端。如果是一驱多的 DDR 匹配结构，VTT 上拉电阻需要放在最远端，注意芯片的排布需要平衡。

PCB 布线时，单端走线走 50ohm，差分走线走 100ohm 阻抗。

控制和地址线及 DQS 线和时钟等长，DQ 数据线和同组的 DQS 线等长。

同一组信号最好在同一层布线。尽量减少过孔的数目。

### 3.13.6 EMI 问题

DDR 由于其速度快，访问频繁，所以在许多设计中需要考虑其对外的干扰性，在设计时需要注意一下几点：

1. 原理图有性能指标要求的，易受干扰的电路模块和信号，如模拟信号，射频信号，时钟信号等，防止 DDR 对其干扰，影响指标。
2. DDR 的电源不要与其它易受干扰的电源模块使用同一电源，如必须使用同一电源，注意使用电感、磁珠或电容进行滤波隔离处理。
3. 在时钟及 DQS 信号线上，预留一些可以增加的串联电阻和并联电容的位置，在 EMI 超出标准时，在信号完整性允许的范围内增大串联电阻或对地电容，使其信号上升延变缓，减少对外的辐射。
4. 进行屏蔽处理，使用金属外壳的屏蔽结构，屏蔽对外辐射。
4. 注意保持地的完整性。

---

# 四 LPDDR SDRAM 介绍

## 4.1 LPDDR SDRAM 分类

Low Power DDR。LPDDR 是 JEDEC 面向低功耗内存而指定的通信标准。LPDDR 在 DDR 基础上演化而来，DDR 的性能要高于同代的 LPDDR 的性能。

LPDDR 和 DDR 之间的关系很密切，简单来讲，LPDDR 是在 DDR 基础上演化而来的，LPDDR2 是在 DDR2 基础上演化而来的，依次类推。从第四代开始，两者开始走上不一样的发展道路，**DDR 主要是通过提高核心频率来提升性能，而 LPDDR 继续选择提高 Prefetch 预读取位数**。以 LPDDR4 和 DDR4 来举例，LPDDR4 是用两个 16 位通道组成 32 位总线，而 DDR4 却具备原生 64 位通道，LPDDR4 的 Prefetch 预读取位数为 16bit，而 DDR4 为 8bit，这在实际运算过程中，DDR4 的性能利用率会更高，但 LPDDR 却可以用更低的功耗来获取更高的理论性能。

如下图所示为 LPDDR 到 LPDDR5 的参数对比。重点介绍 LPDDR4 到 LPDDR5 的变化。相较于 LPDDR4 标准，LPDDR5 的 I/O 速度从 3200MT/s 提升到 6400MT/s，速度直接翻番。如果匹配高端智能机常见的 64bit BUS，每秒可以传送 51.2GB 数据；如果匹配 PC 的 128bit BUS，每秒可突破 100GB 数据。

![](https://cdn.nlark.com/yuque/0/2023/png/22935284/1696861392805-c3f9e62e-a532-445e-a9b3-ad17a0aa8fb3.png#averageHue=%23e3e4e3&height=262&id=siVrU&originHeight=370&originWidth=952&originalType=binary&ratio=1&rotation=0&showTitle=false&status=done&style=none&title=&width=673)

![](https://cdn.nlark.com/yuque/0/2023/png/22935284/1696861393718-1d4b9533-79ce-4c20-821a-30bc27a16be3.png#averageHue=%23ebeded&id=f7MMi&originHeight=598&originWidth=677&originalType=binary&ratio=1&rotation=0&showTitle=false&status=done&style=none&title=)

### 4.1.1 LPDDR4/4X

![image.png](https://cdn.nlark.com/yuque/0/2023/png/22935284/1696991056748-d4b79230-6232-45d7-bf1f-14e10d81189b.png#averageHue=%23eeeeed&clientId=u1041eab2-3de1-4&from=paste&height=624&id=uaa3409fb&originHeight=800&originWidth=832&originalType=url&ratio=1&rotation=0&showTitle=false&size=83517&status=done&style=none&taskId=u0b99bd93-34e3-480f-a67d-0c5fa05a4e4&title=&width=649)

LPDDR4X 仅支持 Single Bank Group，而 LPDDR5 支持多 Bank Group，相当于数据传输由单车道变为多车道，提升了数据传输带宽，LPDDR5 的数据传输数率从 4266Mbps 提升到 5500Mbps，传输带宽从 34GB/s 提升到 44GB/s（带宽是数据传输速率的 8 倍，8 个颗粒）。

### 4.1.2 LPDDR5/5X

LPDDR5 和 LPDDR4X 的区别如下所示：

![image.png](https://cdn.nlark.com/yuque/0/2023/png/22935284/1696991069919-5f6c5019-4d85-450c-9be7-7b7d6e0fda6c.png#averageHue=%23f1f0ef&clientId=u1041eab2-3de1-4&from=paste&height=306&id=u4efe52cb&originHeight=397&originWidth=1024&originalType=url&ratio=1&rotation=0&showTitle=false&size=127789&status=done&style=none&taskId=u83ba883e-1baa-41bf-9413-71bf14580fb&title=&width=789)

![image.png](https://cdn.nlark.com/yuque/0/2023/png/22935284/1696991069919-a735d56b-6e31-4f86-b237-2fb257a5fae6.png#averageHue=%23f2f1f0&clientId=u1041eab2-3de1-4&from=paste&height=726&id=uf53d4c1f&originHeight=903&originWidth=803&originalType=url&ratio=1&rotation=0&showTitle=false&size=80229&status=done&style=none&taskId=ud954375b-ab68-47c3-b371-977fc8e6e3b&title=&width=646)

## 4.2 LPDDR 数据/控制/地址/时钟信号

### 4.2.1 LPDDR4/4X

![image.png](https://cdn.nlark.com/yuque/0/2023/png/22935284/1697702829998-efc5de2a-cf52-4ee8-a599-8306957ce066.png#averageHue=%23dedcdb&clientId=uacf613cc-07aa-4&from=paste&height=697&id=u3c6b70c1&originHeight=753&originWidth=944&originalType=binary&ratio=1&rotation=0&showTitle=false&size=250662&status=done&style=none&taskId=ua8187c05-004a-4e08-a1a1-c50ae094c32&title=&width=874)

![image.png](https://cdn.nlark.com/yuque/0/2023/png/22935284/1697702853206-d381cf97-b005-4860-a1cc-d8f1f15c35d2.png#averageHue=%23dddbd9&clientId=uacf613cc-07aa-4&from=paste&height=560&id=u099141c9&originHeight=610&originWidth=945&originalType=binary&ratio=1&rotation=0&showTitle=false&size=206167&status=done&style=none&taskId=u5c4a0629-cd78-4a02-ab72-1e8e779a145&title=&width=868)

![image.png](https://cdn.nlark.com/yuque/0/2023/png/22935284/1698301135034-33dda84e-571e-4413-892d-053b27bac59d.png#averageHue=%23ebe9e7&clientId=uaf0de5bd-5523-4&from=paste&height=459&id=u2387a3be&originHeight=459&originWidth=794&originalType=binary&ratio=1&rotation=0&showTitle=false&size=79706&status=done&style=none&taskId=u9672fe86-337a-44de-8c4b-3419e3ca64b&title=&width=794)

![image.png](https://cdn.nlark.com/yuque/0/2023/png/22935284/1698301197421-e9b389a9-f62f-47e0-b6cb-0c32bef651e8.png#averageHue=%23ebe9e6&clientId=uaf0de5bd-5523-4&from=paste&height=390&id=ufcea80bd&originHeight=390&originWidth=790&originalType=binary&ratio=1&rotation=0&showTitle=false&size=61717&status=done&style=none&taskId=u3c5f791f-844f-42e1-b21d-f770b3c94b7&title=&width=790)

![image.png](https://cdn.nlark.com/yuque/0/2023/png/22935284/1698301615687-66575492-cd9f-4a5b-a1a1-9b5b9369b48b.png#averageHue=%23efebe8&clientId=uaf0de5bd-5523-4&from=paste&height=722&id=ud857f56e&originHeight=722&originWidth=693&originalType=binary&ratio=1&rotation=0&showTitle=false&size=110211&status=done&style=none&taskId=u16828449-4ad0-4980-a04a-ac2408a6eed&title=&width=693)

![image.png](https://cdn.nlark.com/yuque/0/2023/png/22935284/1698301687277-b21cb627-6041-4957-bb19-1623c8e1c24c.png#averageHue=%23eeeae7&clientId=uaf0de5bd-5523-4&from=paste&height=706&id=u1493221a&originHeight=706&originWidth=695&originalType=binary&ratio=1&rotation=0&showTitle=false&size=112746&status=done&style=none&taskId=uf49bc382-f6ce-4fff-8cfe-a8b62e3d4e1&title=&width=695)

![image.png](https://cdn.nlark.com/yuque/0/2023/png/22935284/1698301878008-1ec55516-742a-4479-ae73-fc4fa6bc8e38.png#averageHue=%23e8e2dc&clientId=uaf0de5bd-5523-4&from=paste&height=150&id=uc6ba561c&originHeight=150&originWidth=691&originalType=binary&ratio=1&rotation=0&showTitle=false&size=25911&status=done&style=none&taskId=u605eef6d-7305-4cb1-adb8-36e0edde9fa&title=&width=691)

#### 4.2.1.1 DQ 交换准则

根据 64 位 DQ 数据划分两个 Group，L_Group_32 位包含 L_Channel_A(Slice0、Slice1)、U_Channel_B(Slice2、Slice3)，U_Group_32 位包含 L_Channel_A(Slice4、Slice5)、U_Channel_B(Slice6、Slice7)，每个 Group 可以整体交换，在每个 Group 内部 L_Channel_A 和 U_Channel_B 可以整体交换，每个 Channel 内部两个 slice 可以整体交换，每个 slice 内部 8bit DQ 数据可以相互交换线序。
![](https://cdn.nlark.com/yuque/0/2023/jpeg/22935284/1698112424014-6438998f-9973-4847-9aa3-e181ebb91bb9.jpeg)

LPDDR4 在 PCB 设计时可以按照以上规则调整线序，方便 PCB 的布线工作。需要声明，若 LPDDR4 的线序有所调整，则 PHY 的寄存器也需要重新配置，需更新相应固件。

### 4.2.3 LPDDR5/5X

![image.png](https://cdn.nlark.com/yuque/0/2024/png/22935284/1715062770526-6e402493-31d1-42da-83e5-2cff40608a88.png#averageHue=%23eee8e3&clientId=ub42d357b-5638-4&from=paste&height=603&id=u41e98a06&originHeight=603&originWidth=849&originalType=binary&ratio=1&rotation=0&showTitle=false&size=230878&status=done&style=none&taskId=ue1bb1dd5-5c9c-4a57-88d6-704a5c47cfb&title=&width=849)

## 4.3 LPDDR 供电 /上电时序

### 4.3.1 LPDDR4/4X

VDD1 : 1.8V

VDD2 : 1.1V

VDDQ : LPDDR4—1.1V LPDDR4X—0.6V

### 4.3.2 LPDDR5/5X

LPDDR5 内部有三组电压：VDD1/VDD2/VDDQ，其中 VDD2 又分为 VDD2H 和 VDD2L；

VDD1：capacitor Matrix/array/bank 电压，为 core 电路供电；

VDD2：给 cell 数字电路/大部分逻辑电路供电；

VDDQ：给 I/O 和 buffer 部分供电；

上电时序：VDD1 must ramp at the same time or earlier than VDD2. VDD2 must ramp at the same time or earlier than VDDQ. 

![image.png](https://cdn.nlark.com/yuque/0/2023/png/22935284/1696991036165-49706b0c-5ea4-4dcd-9d2f-b6ae69b709ce.png#averageHue=%23e8ded3&clientId=u1041eab2-3de1-4&from=paste&id=ImE6T&originHeight=247&originWidth=736&originalType=url&ratio=1&rotation=0&showTitle=false&size=44499&status=done&style=none&taskId=u61ec97f7-3e01-4390-b090-d392e5a5d39&title=)

相比 LPDDR4X，LPDDR5 的 VDD2H 由 1.1V 降至 1.05V，VDDQ 由 0.6V 降至 0.5V。由于新加的 DVFSC 和 DVFSQ 功能，在低速时切换至更低的 0.9V 和 0.3V 电压，进一步降低功耗。

![image.png](https://cdn.nlark.com/yuque/0/2023/png/22935284/1696991036150-13eabb9f-9169-4068-b362-d603cb1a8a88.png#averageHue=%23fbf9f6&clientId=u1041eab2-3de1-4&from=paste&id=vAaeR&originHeight=272&originWidth=430&originalType=url&ratio=1&rotation=0&showTitle=false&size=38127&status=done&style=none&taskId=u5bf1f7e4-9921-4544-8665-9a40188bcd3&title=)

![image.png](https://cdn.nlark.com/yuque/0/2023/png/22935284/1696991036170-42f62550-eb0d-4d88-bc3e-a5cb9b65148f.png#averageHue=%23cfdcaa&clientId=u1041eab2-3de1-4&from=paste&height=184&id=w4fHu&originHeight=243&originWidth=902&originalType=url&ratio=1&rotation=0&showTitle=false&size=34836&status=done&style=none&taskId=ubc6d270c-1209-4da7-a72e-0caede17988&title=&width=684)

![image.png](https://cdn.nlark.com/yuque/0/2023/png/22935284/1696991036294-8315a8a9-bf20-4560-81a9-530d73dfd304.png#averageHue=%23dfe0dd&clientId=u1041eab2-3de1-4&from=paste&height=409&id=Ofs3b&originHeight=469&originWidth=834&originalType=url&ratio=1&rotation=0&showTitle=false&size=402450&status=done&style=none&taskId=u006320a7-219a-49c5-9f0c-9688e1de73f&title=&width=728)

LPDDR4X 器件在高速工作时，时钟为 2133MHz，且需要一直保持在这样的状态，无形中会增加功耗。**而 LPDDR5 器件修改了这一设计，它的时钟只有 800MHz，在数据有读写操作时，会有一个 WCK 信号，这个 WCK 信号会达到最大工作频率，当读/写工作停止时，WCK 信号也会停止，从而降低功耗。** 

VREF=VDDQ/2(default)**：用于 RX 电平判决，在**training**时需要调整 VREF 值来得到对应的时间参数。LPDDR4 将**VREFCA(命令和地址)和 VREFDQ(数据)**分开，VREFCA=VDD2/2,VREFDQ=VDDQ/2，它将有效提高系统数据总线的信噪等级。

VTT=VDDQ/2：DDR 和控制器之间控制总线和数据总线上的端接电源 ，通过较小电阻并联端接，降低信号的反射。电流较大，怕干扰 VREF，需要和 VREF 隔离。VTT 用于片外端接。

**SOC DDR 相关供电：**
VDD_A_HV_EBI:参考电压，工作过程基本不耗电。
VDD_IO_EBI: PHY IO 电压，用于数据、时钟、地址等供电，与 DDR 端 VDDQ 共享 VREG_S3 BUCK。
VDD_A_PLL, VDD_A_EBI: DDR PHY 模拟部分电路供电， DDR PLL 模拟供电，耗电占比小。
VDD_D_EBI: PHY 数字部分供电，耗电占比小；当前和 CX 共供电。**
MX: System Level Cache 以及片内 SRAM 供电。

![image.png](https://cdn.nlark.com/yuque/0/2023/png/22935284/1696991046831-adc9fc04-6c1e-42f1-be63-ef88c0312ee9.png#averageHue=%23f7ba00&clientId=u1041eab2-3de1-4&from=paste&id=nXTZH&originHeight=586&originWidth=714&originalType=url&ratio=1&rotation=0&showTitle=false&size=73439&status=done&style=none&taskId=uea55fd41-d3ee-4e8a-90a6-f259aac87c5&title=)

## 4.4 LPDDR 封装拓扑及布局布线

高端平台采用 PoP 设计，对 SI 友好。MSM 通过 EBI 接口与 LPDDR 相连，最新平台都有两组 EBI 总线。

![image.png](https://cdn.nlark.com/yuque/0/2023/png/22935284/1696991025181-746222c8-fd86-48e7-b5d4-32f87bda1ce6.png#averageHue=%23efe9e0&clientId=u1041eab2-3de1-4&from=paste&height=271&id=NYI1h&originHeight=356&originWidth=1142&originalType=url&ratio=1&rotation=0&showTitle=false&size=70358&status=done&style=none&taskId=u4c91731b-994a-4360-ab09-03c026fe1a4&title=&width=869)

### 4.4.1 LPDDR4 引脚分布

A Standard LPDDR4 SDRAM die features either one or two 16-bit channels where each 16-bit channel supports its own CA bus.

![image.png](https://cdn.nlark.com/yuque/0/2023/png/22935284/1698313852650-49cec8a6-824f-41af-8f5c-8696eeac4b83.png#averageHue=%2329e800&clientId=u2439aa52-da09-4&from=paste&height=940&id=D2Lff&originHeight=940&originWidth=758&originalType=binary&ratio=1&rotation=0&showTitle=false&size=150394&status=done&style=none&taskId=u3e6e1bd5-77e8-4c20-a8f7-a98f4c216c3&title=&width=758)

### 4.4.2 LPDDR5 引脚分布

![image.png](https://cdn.nlark.com/yuque/0/2024/png/22935284/1713258199003-7de430b7-ed16-4b2d-a8c3-b4a032e4fa1f.png#averageHue=%2300d000&clientId=u6c012d3c-3157-4&from=paste&height=898&id=u68f2127a&originHeight=898&originWidth=1038&originalType=binary&ratio=1&rotation=0&showTitle=false&size=150544&status=done&style=none&taskId=ufa21f609-19e9-4642-b390-c5e1cca4165&title=&width=1038)

---

# 五 GDDR SDRAM 介绍

## 5.1 GDDR SDRAM 分类

GDDR，即 Graphic DDR，主要用在显卡设备上。相对于 DDR，GDDR 具有更高的性能、更低的功耗、更少的发热，以满足显卡设备的计算需求。

![](https://cdn.nlark.com/yuque/0/2024/jpeg/22935284/1713249739554-d20e5cd2-22a7-48d0-953d-ea6035697aec.jpeg#averageHue=%23efefef&clientId=ucaafbd18-126b-4&from=paste&id=mXqyW&originHeight=686&originWidth=640&originalType=url&ratio=1&rotation=0&showTitle=false&status=done&style=none&taskId=uc837960a-4c91-4d08-b0e6-a6b1a55ee99&title=)

## 5.2 GDDR 数据/控制/地址/时钟信号

## 5.3 GDDR 供电 /上电时序

## 5.4 GDDR 封装拓扑及布局布线

---

# 六 DDR 工作流程介绍

启动流程：上电->解复位->初始化->ZQ Calibration-> Leveling->IDLE(READY)

读：IDLE->行激活->读数据(1 次或多次突发)->预充电->IDLE

写：IDLE->行激活->写数据(1 次或多次突发)->预充电->IDLE

刷新：IDLE->Refresh->IDLE

自刷新的进入与退出：IDLE->Self Refresh->IDLE

定期校正：IDLE->ZQ Calibration->IDLE，一般外部温度或电压改变时操作

动态更改配置：IDLE->MRS/MPR->IDLE

![image.png](https://cdn.nlark.com/yuque/0/2023/png/22935284/1696989896879-5ff1a81d-3322-46df-b8c0-ebb325a564fc.png#averageHue=%23fcfcfc&clientId=u33443cfc-91a7-4&from=paste&id=iCMQk&originHeight=882&originWidth=887&originalType=url&ratio=1&rotation=0&showTitle=false&size=172569&status=done&style=none&taskId=u14b864f7-6bb9-418b-ba9a-79264da8db2&title=)

![image.png](https://cdn.nlark.com/yuque/0/2023/png/22935284/1696989896856-3128b114-4ab6-4ca7-be6c-eedf6a3cc033.png#averageHue=%23fcfcfc&clientId=u33443cfc-91a7-4&from=paste&id=quibz&originHeight=540&originWidth=861&originalType=url&ratio=1&rotation=0&showTitle=false&size=73157&status=done&style=none&taskId=u9a4345da-cfec-4a9d-b6d5-ceb65920572&title=)

DDR4 SDRAM 状态机

![image.png](https://cdn.nlark.com/yuque/0/2023/png/22935284/1697012362513-72ec1efb-7759-4110-85db-3e88b0d6afd5.png#averageHue=%23f0f0f0&clientId=ubee3ae6b-a866-4&from=paste&height=877&id=srt4G&originHeight=877&originWidth=807&originalType=binary&ratio=1&rotation=0&showTitle=false&size=209292&status=done&style=none&taskId=u80bc7805-6924-40fb-b9da-52c62bdd07c&title=&width=807)

DDR5 SDRAM 状态机

## 6.1 Reset and Initialization

POWER UP and Initialization 所需要的流程如下所示：<br />Apply power (RESET_n and TEN are recommended to be maintained below 0.2 x VDD; all other inputs may be undefined).RESET_n needs to be maintained below 0.2 x VDD for minimum **200us** with stable power and TEN needs to be maintained below 0.2 x VDD for minimum **700us** with stable power. <br />CKE is pulled “Low” anytime before RESET_n being de-asserted (min. time **10ns**). The power voltage ramp time between 300mV to VDD min must be no greater than 200ms; and during the ramp, VDD ≥VDDQ and (VDD-VDDQ) < 0.3volts. <br />VPP must ramp at the **same time or earlier** than VDD and VPP must be equal to or higher than VDD at all times.<br />![image.png](https://cdn.nlark.com/yuque/0/2023/png/22935284/1697533347625-198dd321-54a9-4aeb-94ff-c828c9cf0225.png#averageHue=%23ededed&clientId=u887b7529-dcb8-4&from=paste&height=812&id=u50863108&originHeight=893&originWidth=1129&originalType=binary&ratio=1&rotation=0&showTitle=false&size=180377&status=done&style=none&taskId=u6db40258-e535-41bd-8142-0bfe1cbd85e&title=&width=1026)<br />![image.png](https://cdn.nlark.com/yuque/0/2023/png/22935284/1697533227729-121d0395-1f23-448b-a07a-6249184e8b49.png#averageHue=%23ededed&clientId=u887b7529-dcb8-4&from=paste&height=787&id=u70712aea&originHeight=829&originWidth=1133&originalType=binary&ratio=1&rotation=0&showTitle=false&size=165828&status=done&style=none&taskId=ub026c63d-7725-4851-8faa-d6aa80e168a&title=&width=1075)


## 6.2 ZQ Calibration

ZQ 信号在 DDR3 时代开始引入，ZQ 引脚放置一个 240Ω±1% 的高精度电阻到地，而且这个电阻不能省略。DRAM 将其作为参考电阻，用来校准 DRAM 内部的 240Ω 电阻。因为芯片内部的 240Ω 电阻是由 CMOS 构成，由于 CMOS 的天然特性，造成该电阻会随着 PTV(制程，温度和电压)变化，因此必须对其进行校准，以获得更好的信号质量。

校准的前提是，我们认为该外部参考电阻不会随着环境变化，在任何条件下都是标准的 240Ω。DRAM 内部对每个 240Ω 电阻进行校准时都会共用该外部参考电阻，因此每个电阻是分开进行校准，在时间上不能重叠。

![image.png](https://cdn.nlark.com/yuque/0/2023/png/22935284/1697536059910-6d51f85b-a7cd-45a2-834d-39904bb6269e.png#averageHue=%23f4f4f4&clientId=u887b7529-dcb8-4&from=paste&height=904&id=uKDIW&originHeight=904&originWidth=551&originalType=binary&ratio=1&rotation=0&showTitle=false&size=78398&status=done&style=none&taskId=ube7dcea3-bf9b-4c55-83dd-0ec44502113&title=&width=551)

如下图所示为 ZQ 校准电路，其中左侧方框为校准控制模块，内部包含 ADC, 比较器，择多滤波器(majority filter)。图中 VDDQ/2 作为参考电压，由 DRAM 内部产生。图中最右侧为一个近似电阻(approximation register，是 polyresistor), 比 240Ω 稍大。和该 240Ω+电阻并联的有 5 个 P Channel device，通过控制其导通个数，来使其最终等效电阻最终等于 240Ω。

![image.png](https://cdn.nlark.com/yuque/0/2023/png/22935284/1697542283509-689ad408-27f3-47a3-b3c4-bc96c4dd2c51.png#averageHue=%23f9f8f8&clientId=u887b7529-dcb8-4&from=paste&height=322&id=u8c537c8c&originHeight=521&originWidth=1280&originalType=url&ratio=1&rotation=0&showTitle=false&size=239319&status=done&style=none&taskId=u29e4d1f9-6a12-4aaf-a7ea-97cd1678da1&title=&width=790)

收到 ZQ 校准命令后，PUP 会被驱动为低电平，使和 VDDQ 连接的 PMOS 开关打开，校准控制模块通过调整 VOH[0:4], 来使不同的 P Channel device 导通。比较 VPULL-UP 和 VDDQ/2 的电压，当二者相等时，DQ 上下两侧的电阻相等，均为 240Ω，校准完成，记录下该电阻的 VOH[0:4]的值。

对每个上拉电阻（各 DQ 引脚，共用 1 个 ZQ 引脚的 240 欧电阻，时间上逐个）进行校准，记录下每个电阻对应的 VOH[0:4]值。

下拉电阻校准过程类似，不多赘述。不同的是和 240Ω+电阻并联的是 N Channel device。

关于 ZQ 校正有两个命令 ZQCL （ZQ CALIBRATION LONG )和 ZQ CALIBRATION SHORT (ZQCS)。ZQCL 主要用于系统上电初始化和器件复位，一次完整的 ZQCL 需要 512 个时钟周期，随后（初始化和复位之后），校准一次的时间要减少到 256 周期。ZQCS 在正常操作时跟踪连续的电压和温度变化，ZQCS 需要 64 个时钟周期。

## 6.3 DDR Training

### 6.3.1 DDR Training 作用

1、DDR 相对 SDRAM，采用上升下降双边沿采样，速率是 SDRAM 的两倍；

2、DDR 相对 SDRAM，多了 dqs 信号，原因是由于速度变快，时序难度加大，引入 dqs 就可以不考虑 dq 和 clk 之间的关系（原来是 clk 边沿采集 dq，现在每个 dqs 采集 dq）；

3、fly-by 拓扑结构导致每个 DRAM 的 data skew 不同， Training 是用来补偿来自板子和 DRAM 的延时；

4、通过加 delay 可以解决 data skew 问题，但要得到合适的 dqs delay 值就需要做 Write Leveling Training，使 dqs 和 clk 对齐。

### 6.3.2 Write Leveling 写平衡

从 DDR3 开始引入了 fly-by 结构，地址、控制和时钟信号的布线依次经过每一颗 SDRAM Die 芯片，但 DQ、DQS、DMI 仍可能是点对点的连接，这种布线方式就造成了 CLK 和 DQ/DQS 信号的偏移。

![](https://cdn.nlark.com/yuque/0/2023/png/22935284/1697020206416-4ccabedb-d290-499e-b763-09067e76ee3a.png#averageHue=%23d5ddaa&clientId=ubee3ae6b-a866-4&from=paste&height=355&id=vV9Wb&originHeight=559&originWidth=1017&originalType=url&ratio=1&rotation=0&showTitle=false&status=done&style=none&taskId=ucefd6c73-208f-476b-8016-c731a319069&title=&width=645)

为了解决 DQS 和 CLK 的 edge alignment 的问题，Write leveling 作为业界标准解决方案，来解决 PCB 布线造成的时序问题。**MC(Memory Controller)通过调节发出 DQS 的时间，来让各个 DIE 的 DQS 和 CLK 对齐，起到补偿 skew 的作用**。

Write Leveling Training 方法和基本过程：

通过将 MR1 寄存器 A7 设置为 1 进入 Write Leveling 模式，把 clk 的一个周期分成很多小份，用 DQS 的上升沿采样 CLK 信号的状态，然后将采样结果通过 DQ 引脚反馈给 MC，MC 根据收到的反馈结果让 DQS 不断加上 delay，调整 CLK 和 DQS 的关系，将这个过程不断重复。如果采样值为低则将所有的 DQ[n]保持低电平以通知 DDR controller tDQSS 相位关系还未满足。如果发现在某一个 DQS 上升沿，采样到的 CLK 电平变为了高，则认为此时 tDQSS 相位关系已经满足要求，则通过拉高 DQ[n]通知 DDR controller 一个 Write Leveling 成功，同时 DDR controller 会锁住这个相位差。这时从 DRAM 端看到的 CLK 和 DQS 都是边沿对齐的，Write Leveling 停止，该 delay 值就是 training 结果（t3 时采到的 clk 还是低电平，t4 时采到的就是为高电平）。

![image.png](https://cdn.nlark.com/yuque/0/2023/png/22935284/1697595135547-49934cae-1bd6-4a8f-bc44-fac455f947b1.png#averageHue=%23bbba5b&clientId=u887b7529-dcb8-4&from=paste&height=230&id=u65f65a3b&originHeight=243&originWidth=868&originalType=binary&ratio=1&rotation=0&showTitle=false&size=63387&status=done&style=none&taskId=uc4637020-d414-4b33-b18a-000bcc469ed&title=&width=822)

参见下图，写入均衡的修调过程：

t1：将 ODT 拉起，使能 on die termination；
t2：等待 tWLDQSEN 时间后（保证 DQS 管脚上的 ODT 已设置好），DDR 控制器将 DQS 置起；DDR memory 在 DQS 上升沿采样 CK 信号，发现 CK=0，则 DQ 保持为 0。
t3：DDR 控制器将 DQS 置起；DDR memory 在 DQS 上升沿采样 CK 信号，发现 CK=0，则 DQ 仍然保持为 0。
t4：DDR 控制器将 DQS 置起；DDR memory 在 DQS 上升沿采样 CK 信号，发现 CK=1，则等待一段时间后，DDR memory 将 dq 信号置起。

下面通过一个简单的例子来说明，颗粒 1 的 CLK 传输时间为 1ns，颗粒 2 的 CLK 传输时间为 2ns，颗粒 1 和颗粒 2 的 DQS 传输时间均为 1ns。如果不进行 Write Leveling 调整，颗粒 2 的 CLK 信号和 DQS 信号的飞行时间偏差为 1ns，没有对齐。经过调整后，控制器在发送颗粒 1 的 DQS 后，延时 1ns 再发送颗粒 2 的 DQS，则颗粒 1 在 1ns 后收到对齐的 DQS 和 CLK，颗粒 2 在 2ns 后收到对齐的 DQS 和 CLK，从而满足了时序关系。

![](https://cdn.nlark.com/yuque/0/2023/webp/22935284/1697687169006-6cfb20e5-5f0c-427d-828c-f1e479f7a725.webp#averageHue=%23f6f6f6&clientId=u5f5606b7-8f3d-4&from=paste&height=232&id=u9f2b0c03&originHeight=299&originWidth=644&originalType=url&ratio=1&rotation=0&showTitle=false&status=done&style=none&taskId=u0fe15f6e-a2b4-4cec-abb6-4d92656258d&title=&width=500)

布线要求：
1、只有使用了 fly-by 的情况下需使能 write leveling；
2、MC 只能对 DQS 信号做延迟，不能做超前处理，所以 CK 要大于 DQS 信号线的长度，否则将不能满足 tDQSS（DQS, DQS# rising edge to CK, CK# rising edge，在标准中要求为+/-0.25 tCK。tCK 为 CLK 时钟周期））。

![](https://cdn.nlark.com/yuque/0/2023/png/22935284/1697020206451-9e4ecf11-b7eb-4eb1-93cf-b835837825c6.png#averageHue=%23f2f2f2&clientId=ubee3ae6b-a866-4&from=paste&height=242&id=TwBlD&originHeight=157&originWidth=327&originalType=url&ratio=1&rotation=0&showTitle=false&status=done&style=none&taskId=u1fa05528-3478-445c-932c-105ab7eb0c6&title=&width=505)

### 6.3.3 DQS Centering

DQS Centering 分为 Read DQS Timing 和 Write DQS Timing 两种，目的是**保证 DQS 信号在 DQ data eye 的中间**。

Read DQS Timing<br />1、读数据时，CPU/MC 通过调整内部的 DLL 延迟锁定回路电路，延迟 DQS 使它在 DQ data eye 中间；<br />2、对于 Memory 的每个 CH 的每个 Rank，写一个 cacheline 的特定类型数据；<br />3、然后读回数据并标记基于现在的 ReadDQS timing 的情况 pass 或 fail；<br />4、增加 ReadDQSTiming delay，继续步骤 1，当且仅当出现连续 3 组 pass 的情况，取中间的一组数据并记录 ReadDqsTiming 平均值。<br /> <br />Write DQS Timing<br />1、写数据时，Read DQS Timing 已经找到，因为 Memory 没有 DLL 电路，所以只能通过调整 CPU/MC 端的 Write Data Timing 去配合 DQS 信号使 DQS 在 DQ data eye 的中间；<br />2、对于 Memory 的每个 CH 的每个 Rank，写一个 cacheline 的特定类型数据；<br />3、然后读回数据并标记基于现在的 Write Data timing 的情况 pass 或 fail；<br />4、增加 Write Data Timing Delay，继续步骤 1，直到找到最大可以 pass 的 Write DQS Timing delay。计算出他们的中点（平均值）并且设置相应的 Write DQ Delay timing 的值。<br />![](https://cdn.nlark.com/yuque/0/2023/png/22935284/1697020238814-f0df0643-6a70-4aa5-a943-a99906f75833.png#averageHue=%23e7e6e6&clientId=ubee3ae6b-a866-4&from=paste&height=319&id=bac3m&originHeight=256&originWidth=434&originalType=url&ratio=1&rotation=0&showTitle=false&status=done&style=none&taskId=u970fe734-2f26-4f15-81b6-33b93afafbd&title=&width=540)

## 6.4 Activate

它以 ACTIVATE 命令开始（ACT_n 和 CS_n 在一个时钟周期内变为低电平），ACTIVATE 命令用于打开（或激活）特定 Bank 中的一行，以便后续访问。X4/8 中的 BG0-BG1 和 X16 中的 BG0 用来选择 bankgroup；BA0-BA1 用来选择 bankgroup 内的 bank，A0-A17 上提供的地址用于选择 row。在对该 bank 发出 precharge 或 precharge all 命令之前，该 row 一直处于 active or open for access 状态。在打开同一 bank 不同 row 之前，必须要对该 bank 进行 precharge。

## 6.5 Bank / Row Active

在实际工作中，Bank 地址与相应的行地址是同时发出的，此时这个命令称之为“**行激活**”（Row Active）。在此之后，将发送列地址寻址命令与具体的操作命令（是读还是写），这两个命令也是同时发出的，所以一般都会以“读/写命令”来表示列寻址。在列地址被选中之后，将会触发数据传输。

![](https://cdn.nlark.com/yuque/0/2023/png/22935284/1697076545945-6f1fdb0b-7d57-4e46-9d9a-2c90d170fd25.png#averageHue=%23e8e5e5&clientId=u76956dc3-b696-4&from=paste&id=iySfB&originHeight=102&originWidth=702&originalType=url&ratio=1&rotation=0&showTitle=false&status=done&style=none&taskId=u69213549-a691-4d99-b737-6965ee694dc&title=)

在进行数据的读写前，Controller 需要先发送 Row Active Command，打开 DRAM Memory Array 中的指定的 Row。Row Active Command 的时序如下图所示：
![](https://cdn.nlark.com/yuque/0/2023/png/22935284/1697020868696-664d6f82-84f1-4eba-bc88-793f0f0b9405.png#averageHue=%23f5f2f0&clientId=ub1ae5802-66b9-4&from=paste&id=oexDb&originHeight=1060&originWidth=2080&originalType=url&ratio=1&rotation=0&showTitle=false&status=done&style=none&taskId=u215f3166-f1b6-4c82-aa4b-5294d7d2e60&title=)

Row Active Command 可以分为两个阶段：
1. Row Sense：Row Active Command 通过地址总线指明需要打开某一个 Bank 的某一个 Row。DRAM 在接收到该 Command 后，会打开该 Row 的 Wordline，将其存储的数据读取到 Sense Amplifiers 中。
2. Row Restore：由于 DRAM 的特性，Row 中的数据在被读取到 Sense Amplifiers 后，需要进行 Restore 的操作。Restore 操作可以和数据的读取同时进行。即在这个阶段，Controller 可能发送了 Read Command 进行数据读取。DRAM 接收到 Row Active Command 到完成 Row Restore 操作所需要的时间定义为 tRAS。Controller 在发出一个 Row Active Command 后，必须要等待 tRAS 时间后，才可以发起另一次的 Precharge 和 Row Access。

## 6.6 Read

Controller 发送 Row Active Command 并等待 tRCD 时间后，再发送 Column Read Command 进行数据读取。Read Command 将通过 A[12:0] 信号，发送需要读取的 Column 的地址给 SDRAM。然后 SDRAM 再将 Active Command 所选中的 Row 中，将对应 Column 的数据通过 DQ 引脚发送给 Host。

要从内存中读取数据需要提供地址，而要向它写入数据除了地址以外还要提供数据。这个由用户提供的地址通常被称为 "**逻辑地址**"。**逻辑地址在被提交给 DRAM 之前被翻译成物理地址。**

物理地址是由以下字段组成的：
• Bank Group
• Bank
• Row # 行
• Column # 列
这些单独的字段被用来识别内存中的确切位置，以便进行读出或写入。

读和写操作是以突发为导向的。它从一个选定的位置开始（由用户提供的地址指定），并继续进行 4 个或 8 个突发长度。与读或写命令同时注册的地址位用于选择突发操作的起始列位置。此步骤也称为 CAS - Column Address Strobe(CAS - 列地址选通)。

每个 bank 只有一组感应放大器。在对同一 bank 的不同行进行读/写之前，必须使用 PRECHARGE 命令解除当前打开的 row 的激活。PRECHARGE 相当于关闭柜子里当前的文件抽屉，它使感应放大器中的数据被写回 row 中。

可以使用 RDA（自动预充电读取）和 WRA（自动预充电写入）命令，而不是发出显式 PRECHARGE 命令来停用行。这些命令告诉 DRAM 在读取或写入操作完成后自动停用/预充电行。由于列地址只使用地址位 A0-A9，A10 在 CAS 期间是一个未使用的位，它被重载以指示自动预充电。

数据 Burst Length 为 8 时的 Column Read Command 时序如下图所示：

![](https://cdn.nlark.com/yuque/0/2023/png/22935284/1697020868938-754ba843-0694-43cf-8632-2358d5b765d3.png#averageHue=%23f2efec&clientId=ub1ae5802-66b9-4&from=paste&height=579&id=syxeu&originHeight=1198&originWidth=2080&originalType=url&ratio=1&rotation=0&showTitle=false&status=done&style=none&taskId=u801ee075-29ed-41bd-b0b2-b70a7f59c09&title=&width=1006)

Column Read Command 通过地址总线 A[0:9] 指明需要读取的 Column 的起始地址。DRAM 在接收到该 Command 后，会将数据从 Sense Amplifiers 中通过 IO 电路搬运到数据总线上。

DRAM 从接收到 Command 到第一组数据从数据总线上输出的时间称为 tCAS（CAS for Column Address Strobe），也称为 tCL（CL for CAS Latency），这一时间可以通过 mode register 进行配置，通常为 3~5 个时钟周期。

DRAM 在接收到 Column Read Command 的 tCAS 时间后，会通过数据总线，将 n 个 Column 的数据逐个发送给 Controller，其中 n 由 mode register 中的 burst length 决定，通常可以将 burst length 设定为 2、4 或者 8。

开始发送第一个 Column 数据，到最后一个 Column 数据的时间定义为 tBurst。

一个完整的 Burst Length 为 4 的 Read Cycle 如下图所示：

![](https://cdn.nlark.com/yuque/0/2023/png/22935284/1697020870636-bcd5d705-33bb-42c7-8c45-be0fe0773c04.png#averageHue=%23f1f0f0&clientId=ub1ae5802-66b9-4&from=paste&height=338&id=VuoNW&originHeight=656&originWidth=2080&originalType=url&ratio=1&rotation=0&showTitle=false&status=done&style=none&taskId=u29dfa4fd-5178-4795-aad1-b8551792b54&title=&width=1071)

一旦识别了 Bank Group 和 Bank，地址的 Row 部分将激活内存阵列中的一行。这一行称为 word line(字线) ，激活它会将数据从内存阵列读取到 Sense Amplifiers 感应放大器 中。然后，列地址读出加载到 Sense Amplifiers 感应放大器中的部分字。列的宽度称为 Bit Line(位线) 。

对 DDR SDRAM 的读和写操作是以**突发**为导向的。它从一个选定的位置开始（由用户提供的地址指定），并继续进行**4 个或 8 个突发长度**。

- 读取和写入操作是一个两步过程。它以 ACTIVATE 命令开始（ACT_n 和 CS_n 在一个时钟周期内变为低电平），然后是 RD 或 WR 命令。
- 与** ACTIVATE 命令同时注册的地址位用于选择要激活的 BankGroup、Bank 和 Row**（x4/8 中的 BG0-BG1 和 x16 中的 BG0 选择 bankgroup；BA0-BA1 选择 bank；A0-A17 选择 row）。此步骤也称为 RAS - Row Address Strobe(RAS - 行地址选通)
- 与**读或写命令同时注册的地址位用于选择突发操作的起始 Column 位置**。此步骤也称为 CAS - Column Address Strobe(CAS - 列地址选通)。
- 每个 bank 只有一组感应放大器。在**对同一 bank 的不同行进行读/写之前，必须使用 PRECHARGE 命令解除当前打开的 row 的激活**。PRECHARGE 相当于关闭柜子里当前的文件抽屉，它使感应放大器中的数据被写回 row 中。
- 可以使用 RDA（自动预充电读取）和 WRA（自动预充电写入）命令，而不是发出显式 PRECHARGE 命令来停用行。这些命令告诉 DRAM 在读取或写入操作完成后自动停用/预充电行。由于列地址只使用地址位 A0-A9，A10 在 CAS 期间是一个未使用的位，它被重载以指示自动预充电。

![](https://cdn.nlark.com/yuque/0/2023/png/22935284/1696861408836-94290768-a174-4c85-a4f7-6b7835ac7128.png#averageHue=%230e0e0e&id=ThkOI&originHeight=150&originWidth=889&originalType=binary&ratio=1&rotation=0&showTitle=false&status=done&style=none&title=)

上图显示了突发长度为 8（BL8）的 READ 操作的时序图：
• 第一步是一个 ACT 命令。这时地址总线上的值表示行地址。
• 在第二步，发出 RDA（自动预充电的读）。这时地址总线上的值是列地址。
• RDA 命令告诉 DRAM 在读取完成后自动 PRECHARGEs bank。

在读写 memory 之前要先通过 CS#选定相应的 PBANK(RANK), 接下来通过 BA0,BA1 选通 LBANK，接下来就是通过行有效(RAS#)和列有效(CAS#)选通具体的行和列，然后就可以进行读写了。但是因为行地址和列地址都是共用地址线，所以在 RAS#切换到 CAS#之间一定有一个间隔用于保证芯片存储阵列电子元件响应时间，这个时间叫做为 tRCD，即 RAS to CAS Delay（RAS 至 CAS 延迟）单位是时钟周期。
![](https://cdn.nlark.com/yuque/0/2023/png/22935284/1697019053115-64134aa8-331b-429c-889b-0e27146b8891.png#averageHue=%23f4f4f4&clientId=ubee3ae6b-a866-4&from=paste&id=LARKw&originHeight=370&originWidth=837&originalType=url&ratio=1&rotation=0&showTitle=false&status=done&style=none&taskId=u9a8f5cba-405e-4498-be69-40714e8b0db&title=)

在列地址确定了以后，只需要将数据通过 DQ 送给总线上即可。但是从 CAS#到真正的数据输出到总线其实还需要一段时间，被称为 CL/RL（CAS Latency，CAS 潜伏期）。为什么需要这个时间呢，主要是因为存储单元的需要一定的反应时间，所以不可能和 CAS 在同一个上升沿触发。另外电容容量很小，信号需要放大（S-AMP)识别以后才能送到总线上，这也需要一段时间被称为 tAC （Access Time from CLK，时钟触发后的访问时间）。tAC 的单位是 ns。

![](https://cdn.nlark.com/yuque/0/2023/png/22935284/1697019053159-52af5d07-80c9-4f07-8ea4-f5fe357cce7b.png#averageHue=%23efeeee&clientId=ubee3ae6b-a866-4&from=paste&id=opRHk&originHeight=435&originWidth=894&originalType=url&ratio=1&rotation=0&showTitle=false&status=done&style=none&taskId=u03f84bca-723d-43a4-a46d-dda7a5e7dc4&title=)

数据写入时也是在 tRCD 之后进行，但是并不需要 CL. 虽然数据可以和 CAS#同时送出，但是因为选通三极管与电容的充电必须要要有一段时间，所以真正的数据写入需要一定的周期，都会留出足够的写入/校正时间（tWR，Write Recovery Time），这个操作也被称作写回（Write Back）。tWR 至少占用一个时钟周期或再多一点。

![](https://cdn.nlark.com/yuque/0/2023/png/22935284/1697019053158-ff55e04c-6e5d-46dd-83af-33b0cddff74a.png#averageHue=%23eeeded&clientId=ubee3ae6b-a866-4&from=paste&height=477&id=m7cLh&originHeight=585&originWidth=856&originalType=url&ratio=1&rotation=0&showTitle=false&status=done&style=none&taskId=uc9aa868c-741e-428c-8e53-3e06c6c5afe&title=&width=698)

Burst Mode 是指同一行的相邻的存储单元连续进行传输的方式，连续传输的数量称之为突发长度 BL(Burst Length)。在未使用 Burst 的情况下，连续读写多个数据会导致内存控制资源被占用（要一直发读取 cmd 和列地址），在数据传送期间无法输入新的命令。

![](https://cdn.nlark.com/yuque/0/2023/png/22935284/1697019053202-c65322d0-7e60-47c8-968c-76bd56595824.png#averageHue=%23f0eeed&clientId=ubee3ae6b-a866-4&from=paste&id=myRF7&originHeight=512&originWidth=941&originalType=url&ratio=1&rotation=0&showTitle=false&status=done&style=none&taskId=u1353050e-d803-4445-b1f7-dff06ae9987&title=)

使用了突发传输模式以后只要指定列起始地址和 BL，内存就会自动依次读取后续相应数量的存储单元而且并不需要一直提供列地址。BL 有 1，2，4，8。

![](https://cdn.nlark.com/yuque/0/2023/png/22935284/1697019053159-49795ef4-0fea-43b0-b446-e147f78dca04.png#averageHue=%23f0eeed&clientId=ubee3ae6b-a866-4&from=paste&id=fDyJE&originHeight=476&originWidth=954&originalType=url&ratio=1&rotation=0&showTitle=false&status=done&style=none&taskId=u4b5d7cc0-1729-4326-883a-054ed905f0c&title=)

### 6.6.1 Read Command With Auto Precharge

DRAM 还可以支持 Auto Precharge 机制。在 Read Command 中的地址线 A10 设为 1 时，就可以触发 Auto Precharge。此时 DRAM 会在完成 Read Command 后的合适的时机，在内部自动执行 Precharge 操作。

Read Command With Auto Precharge 的时序如下图所示：

![](https://cdn.nlark.com/yuque/0/2023/png/22935284/1697020871090-05b50d60-ad89-4d98-af9a-29071c57dbf2.png#averageHue=%23f0f0ef&clientId=ub1ae5802-66b9-4&from=paste&height=423&id=J94un&originHeight=760&originWidth=2080&originalType=url&ratio=1&rotation=0&showTitle=false&status=done&style=none&taskId=ucd6e2f68-676a-4c8b-8300-057ed20756a&title=&width=1157)

Auto Precharge 机制的引入，可以降低 Controller 实现的复杂度，进而在功耗和性能上带来改善。

NOTE:Write Command 也支持 Auto Precharge 机制，参考下一小节的时序图。

##  6.7、Write

Write Command 将通过 A[12:0] 信号，发送需要写入的 Column 的地址给 SDRAM，同时通过 DQ[15:0] 将待写入的数据发送给 SDRAM。然后 SDRAM 将数据写入到 Actived Row 的指定 Column 中。<br />SDRAM 接收到最后一个数据到完成数据写入到 Memory 的时间定义为 tWR （Write Recovery）。

![](https://cdn.nlark.com/yuque/0/2023/png/22935284/1696861409057-b80fde95-ca9a-4bce-b95c-d3976a531354.png#averageHue=%230f0f0f&id=KRoey&originHeight=150&originWidth=1209&originalType=binary&ratio=1&rotation=0&showTitle=false&status=done&style=none&title=)

上图是 WRITE 操作的时序图：
• 第一步激活一行
• 然后发出 2 个 WRITE 命令。第一个寻址 COL，第二个寻址 COL+8。
• 第二个写入操作之前不需要 ACT，因为我们要写入的行已经在 Sense Amps(感应放大器)中被激活了。
• 还要注意的是，第一条命令是一个普通的 WR，所以它使行处于激活状态。第二条命令是 WRA，它在写完后取消了该行的激活。

Controller 发送 Row Active Command 并等待 tRCD 时间后，再发送 Column Write Command 进行数据写入。数据 Burst Length 为 8 时的 Column Write Command 时序如下图所示：
![](https://cdn.nlark.com/yuque/0/2023/png/22935284/1697020868567-6080cfb8-2e50-46d1-aa46-78cd4373d3f3.png#averageHue=%23f4f1ef&clientId=ub1ae5802-66b9-4&from=paste&height=505&id=nmKfF&originHeight=1064&originWidth=2080&originalType=url&ratio=1&rotation=0&showTitle=false&status=done&style=none&taskId=u8609e8c8-f95d-4681-8269-62d44a54abf&title=&width=988)

Column Write Command 通过地址总线 A[0:9] 指明需要写入数据的 Column 的起始地址。Controller 在发送完 Write Command 后，需要等待 tCWD （CWD for Column Write Delay） 时间后，才可以发送待写入的数据。tCWD 在一些描述中也称为 tCWL（CWL for Column Write Latency）

tCWD 在不同类型的 SDRAM 标准有所不同：

| Memory Type | tCWD           |
| ----------- | -------------- |
| SDRAM       | 0 cycles       |
| DDR SDRAM   | 1 cycle        |
| DDR2 SDRAM  | tCAS - 1 cycle |
| DDR3 SDRAM  | programmable   |

DRAM 接收完数据后，需要一定的时间将数据写入到 DRAM Cells 中，这个时间定义为 tWR（WR for Write Recovery）。

## 6.8 Precharge 预充电

在进行一次 Read 或 Write 操作前后，通常要执行 Precharge 操作。

**Precharge 操作是以 Bank 为单位进行的，可以单独对某一个 Bank 进行，也可以一次对所有 Bank 进行。**如果 A10 为高，那么 SDRAM 进行 All Bank Precharge 操作，如果 A10 为低，那么 SDRAM 根据 BA[1:0] 的值，对指定的 Bank 进行 Precharge 操作。

SDRAM 的寻址具有独占性，**每次切换到同一个 bank 的不同的行时就要将原来的工作行关闭，重新发送行和列地址。L-BANK 关闭现有行打开新的行的过程被称作为了预充电。实际上，预充电是一种对工作行中所有存储体进行数据重写，并对行地址进行复位，同时释放 S-AMP，以准备新行的工作。**通常在发出预充电之后还需要一段时间才允许发送 RAS#打开新的工作行，这个时间被称为 tRP(precharge command period,预充电有效周期)，单位是时钟周期数。

在 Controller 发送 Row Active Command 访问一个具体的 Row 前， Controller 需要发送 Precharge Command 对该 Row 所在的 Bank 进行 Precharge 操作。下面的时序图描述了 Controller 访问一个 Row 后，执行 Precharge，然后再访问另一个 Row 的流程。

![](https://cdn.nlark.com/yuque/0/2023/png/22935284/1697020868717-87c4115a-dc21-466a-b8c5-d17c33e423b0.png#averageHue=%23f6f4f2&clientId=ub1ae5802-66b9-4&from=paste&height=633&id=uLbGb&originHeight=1132&originWidth=2080&originalType=url&ratio=1&rotation=0&showTitle=false&status=done&style=none&taskId=u6a562368-c7a5-4a41-9278-1fae00a6c26&title=&width=1163)

Controller 在发送一个 Row Active Command 后，需要等待 tRC（RC for Row Cycle）时间后，才能发送第二个 Row Active Command 进行另一个 Row 的访问。从时序图上我们可以看到，tRC = tRAS + tRP，tRC 时间决定了访问 DRAM 不同 Row 的性能。在实际的产品中，通常会通过降低 tRC 耗时或者在一个 Row Cycle 执行尽可能多数据读写等方式来优化性能。

在一个 Row Cycle 中，发送 Row Active Command 打开一个 Row 后，Controller 可以发起多个 Read 或者 Write Command 进行一个 Row 内的数据访问。这种情况下，由于不用进行 Row 切换，数据访问的性能会比需要切换 Row 的情况好。在一些产品上，MC 会利用这一特性，对 CPU 发起的内存访问进行调度，在不影响数据有效性的情况下，将同一个 Row 上的数据访问汇聚到一直起执行，以提供整体访问性能。

## 6.9 Refresh 刷新

**DRAM 的 Storage Cell 中的电荷会随着时间慢慢减少，为了保证其存储的信息不丢失，需要周期性的对其进行刷新操作，也就是对数据进行重写。**Controller 每隔一段时间就需要发送一个 Row Refresh Command 给 DRAM，进行 Row 刷新操作。DRAM 在接收到 Row Refresh Command 后，会根据内部 Refresh Counter 的值，对所有 Bank 的一个或者多个 Row 进行刷新操作。<br />刷新操作与预充电中重写的操作一样，都是用 S-AMP 先读再写。但为什么有预充电操作还要进行刷新呢？因为预充电是对一个或所有 L-Bank 中的工作 Row 操作，并且是不定期的，而刷新则是有固定的周期，依次对所有 Row 进行操作，以保留那些久久没经历重写的存储体中的数据。但与所有 L-Bank 预充电不同的是，这里的 Row 是指所有 L-Bank 中地址相同的 Row，而预充电中各 L-Bank 中的工作 Row 地址并不是一定是相同的。<br />下图为 DRAM Row Refresh Command 的时序图。

![](https://cdn.nlark.com/yuque/0/2023/png/22935284/1697020870454-9e2176ed-7006-47b4-a16c-fc25d9a824d2.png#averageHue=%23f5f3f1&clientId=ub1ae5802-66b9-4&from=paste&id=oqPUS&originHeight=1200&originWidth=2080&originalType=url&ratio=1&rotation=0&showTitle=false&status=done&style=none&taskId=u48d09670-5143-47d6-8858-ce3b45e9592&title=)

**DRAM 完成刷新操作所需的时间定义为 tRFC（RFC for Refresh Cycle）**, 这个时间会随着 SDRAM Row 的数量的增加而变大。tRFC 包含两个部分的时间，一是完成刷新操作所需要的时间，由于 DRAM Refresh 是同时对所有 Bank 进行的，刷新操作会比单个 Row 的 Active + Precharge 操作需要更长的时间；tRFC 的另一部分时间则是为了降低平均功耗而引入的延时，DRAM Refresh 操作所消耗的电流会比单个 Row 的 Active + Precharge 操作要大的多，tRFC 中引入额外的时延可以限制 Refresh 操作的频率。<br />刷新分两种，自动刷新(Auto refresh)和自刷新(Self refresh)。

### 6.9.1 Auto Refresh

为了简化 SDRAM Controller 的设计，SDRAM 标准定义了 Auto-Refresh 机制，该机制要求 SDRAM Controller 在一个刷新周期内，发送 **8192(2^13)** 个 Auto-Refresh Command，即 AR， 给 SDRAM。SDRAM 每收到一个 AR，就进行 n 个 Row 的刷新操作，其中，**n = 总的 Row 数量 / 8192** ，其中的固定时间间隔为 **tREFI（REFI for Refresh Interval），8192 次刷新需要在 32ms 或者 64ms 时间内完成（存储体中电容的数据有效保存期上限是 64ms），具体取决于温度是否超过 85**。

![image.png](https://cdn.nlark.com/yuque/0/2023/png/22935284/1697708723410-172db70f-7f34-43fa-8162-03f7484676e0.png#averageHue=%23cbaf7c&clientId=u34526b3b-c1f6-4&from=paste&height=101&id=Xa32r&originHeight=113&originWidth=931&originalType=binary&ratio=1&rotation=0&showTitle=false&size=41151&status=done&style=none&taskId=u7c9d92f8-03c8-415b-aa5a-cfd2b03854e&title=&width=831)

此外，SDRAM 内部维护一个 refresh counter，每完成一次刷新操作，就将计数器更新为下一次需要进行刷新操作的 Row。

由于 AR 会占用总线，阻塞正常的数据请求，同时 SDRAM 在执行 refresh 操作是很费电，所以在 SDRAM 的标准中，还提供了一些优化的措施，例如 DRAM Controller 可以最多延时 8 个 tREFI 后，再一起把 8 个 AR 同时发出。

### 6.9.2 Self Refresh

Host 还可以让 SDRAM 进入 Self-Refresh 模式。在该模式下，Host 不能对 SDRAM 进行读写操作，SDRAM 内部自行进行刷新操作保证数据的完整。通常在设备进入待机状态时，Host 会让 SDRAM 进入 Self-Refresh 模式，以节省功耗。

![|600](https://cdn.nlark.com/yuque/0/2023/png/22935284/1696861407952-e18ccdd8-6434-448b-b799-eb451f0ea90f.png#averageHue=%23ece259&height=313&id=FzVZ4&originHeight=513&originWidth=1134&originalType=binary&ratio=1&rotation=0&showTitle=false&status=done&style=none&title=&width=691)

高温时电荷泄放比较快，刷新率也要相应提高。可以根据温度传感器的值来调整自刷新率。

## 6.10 LPM

Deep sleep mode( 深度睡眠，刷新需要 CS 有电平切换，息屏待机可进入该模式降低待机功耗， DDR5 新特性)

SR power down(所有数据 时钟 控制线信号稳定低电平、 ODT 关闭，息屏待机)

Idle power down(Power down 无刷新)

Active power down(Active 下的低功耗状态)

Deep power down(DPD 模式，关闭 Cell 供电，数据丢失，但寄存器设定保留，退出需要初始化

![](https://cdn.nlark.com/yuque/0/2023/png/22935284/1697020805787-ec5984ce-074d-4bff-b03c-34f0b231b4e3.png#averageHue=%23ebebea&clientId=ub1ae5802-66b9-4&from=paste&id=u944370c5&originHeight=1234&originWidth=2080&originalType=url&ratio=1&rotation=0&showTitle=false&status=done&style=none&taskId=ud40d0dfa-62f5-4614-af7b-9a5dba5a5e6&title=)

如上图所示，SDRAM 的相关操作在内部大概可以分为以下的几个阶段：

1. Command transport and decode 在这个阶段，Host 端会通过 Command Bus 和 Address Bus 将具体的 Command 以及相应参数传递给 SDRAM。SDRAM 接收并解析 Command，接着驱动内部模块进行相应的操作。
2. In bank data movement 在这个阶段，SDRAM 主要是将 Memory Array 中的数据从 DRAM Cells 中读出到 Sense Amplifiers，或者将数据从 Sense Amplifiers 写入到 DRAM Cells。
3. In device data movement 这个阶段中，数据将通过 IO 电路缓存到 Read Latchs 或者通过 IO 电路和 Write Drivers 更新到 Sense Amplifiers。
4. System data transport 在这个阶段，进行读数据操作时，SDRAM 会将数据输出到数据总线上，进行写数据操作时，则是 Host 端的 Controller 将数据输出到总线上。

在上述的四个阶段中，每个阶段都会有一定的耗时，例如数据从 DRAM Cells 搬运到 Read Latchs 的操作需要一定的时间，因此在一个具体的操作需要按照一定时序进行。

同时，由于内部的一些部件可能会被多个操作使用，例如读数据和写数据都需要用到部分 IO 电路，因此多个不同的操作通常不能同时进行，也需要遵守一定的时序

此外，某些操作会消耗很大的电流，为了满足 SDRAM 设计上的功耗指标，可能会限制某一些操作的执行频率。

基于上面的几点限制，SDRAM Controller 在发出 Command 时，需要遵守一定的时序和规则，这些时序和规则由相应的 SDRAM 标准定义。在后续的小节中，我们将对各个 Command 的时序进行详细的介绍。

后续的小节中，我们将通过下图类似的时序图，来描述各个 Command 的详细时序。

![](https://cdn.nlark.com/yuque/0/2023/png/22935284/1697020952123-9207c92f-d9f4-4c85-8c90-8218471e9a8c.png#averageHue=%23f3f3f3&clientId=ub1ae5802-66b9-4&from=paste&id=u7b47da9d&originHeight=546&originWidth=2080&originalType=url&ratio=1&rotation=0&showTitle=false&status=done&style=none&taskId=u38e61ede-0afd-422b-91eb-1a9329e944e&title=)

上图中，Clock 信号是由 SDRAM Controller 发出的，用于和 DRAM 之间的同步。在 DDRx 中，Clock 信号是一组差分信号，在本文中为了简化描述，将只画出其中的 Positive Clock。

Controller 与 DRAM 之间的交互，都是以 Controller 发起一个 Command 开始的。从 Controller 发出一个 Command 到 DRAM 接收并解析该 Command 所需要的时间定义为 tCMD，不同类型的 Command 的 tCMD 都是相同的。

DRAM 在成功解析 Command 后，就会根据 Command 在内部进行相应的操作。从 Controller 发出 Command 到 DRAM 执行完 Command 所对应的操作所需要的时间定义为 tParam。不同类型的 Command 的 tParam 可能不一样，相同 Command 的 tParam 由于 Command 参数的不同也可能会不一样。

## 6.11 Additive Latency

在 DDR2 中，又引入了 Additive Latency 机制，即 AL。通过 AL 机制，Controller 可以在发送完 Active Command 后紧接着就发送 Read 或者 Write Command，而后 DRAM 会在合适的时机（延时 tAL 时间）执行 Read 或者 Write Command。时序如下图所示：

![](https://cdn.nlark.com/yuque/0/2023/png/22935284/1697020871189-fb0aeb46-50c8-4e3d-9d73-b15ca3fce4e4.png#averageHue=%23f1f0f0&clientId=ub1ae5802-66b9-4&from=paste&id=u28449ce2&originHeight=828&originWidth=2080&originalType=url&ratio=1&rotation=0&showTitle=false&status=done&style=none&taskId=u6e36979a-8f94-4b20-8a56-dd41d6633f5&title=)

![](https://cdn.nlark.com/yuque/0/2023/png/22935284/1697020871677-851da91a-4470-4de5-9fe2-52118230dff8.png#averageHue=%23f1f0f0&clientId=ub1ae5802-66b9-4&from=paste&id=u79da0b65&originHeight=828&originWidth=2080&originalType=url&ratio=1&rotation=0&showTitle=false&status=done&style=none&taskId=u8c4e10f1-c400-4b4f-b2db-ad5623eeda1&title=)

Additive Latency 机制同样是降低了 Controller 实现的复杂度，在功耗和性能上带来改善。

## 6.12 DRAM Timing 设定

上述的 DRAM Timing 中的一部分参数可以编程设定，例如 tCAS、tAL、Burst Length 等。这些参数通常是在 Host 初始化时，通过 Controller 发起 Load Mode Register Command 写入到 DRAM 的 Mode Register 中。DRAM 完成初始化后，就会按照设定的参数运行。Host 与 SDRAM 之间的交互都是由 Host 以 Command 的形式发起的。一个 Command 由多个信号组合而成，下面表格中描述了主要的 Command： 

![](https://cdn.nlark.com/yuque/0/2023/png/22935284/1696861406135-e015067a-4237-458a-ad4a-fc085f15afba.png#averageHue=%23f8f7f7&id=MeKfo&originHeight=430&originWidth=678&originalType=binary&ratio=1&rotation=0&showTitle=false&status=done&style=none&title=)

![](https://cdn.nlark.com/yuque/0/2023/png/22935284/1696861406373-7e87b33e-3f31-4002-bcf0-88dff694e8d7.png#averageHue=%23fafaf9&id=RNbiI&originHeight=342&originWidth=869&originalType=binary&ratio=1&rotation=0&showTitle=false&status=done&style=none&title=)

- Active Active Command 会通过 BA[1:0] 和 A[12:0] 信号，选中指定 Bank 中的一个 Row，并打开该 Row 的 wordline。在进行 Read 或者 Write 前，都需要先执行 Active Command。
- Read Read Command 将通过 A[12:0] 信号，发送需要读取的 Column 的地址给 SDRAM。然后 SDRAM 再将 Active Command 所选中的 Row 中，将对应 Column 的数据通过 DQ[15:0] 发送给 Host。 Host 端发送 Read Command，到 SDRAM 将数据发送到总线上的需要的时钟周期个数定义为 CL。
- Write Write Command 将通过 A[12:0] 信号，发送需要写入的 Column 的地址给 SDRAM，同时通过 DQ[15:0] 将待写入的数据发送给 SDRAM。然后 SDRAM 将数据写入到 Actived Row 的指定 Column 中。SDRAM 接收到最后一个数据到完成数据写入到 Memory 的时间定义为 tWR （Write Recovery）。
- Precharge 在进行下一次的 Read 或者 Write 操作前必须要先执行 Precharge 操作。Precharge 操作是以 Bank 为单位进行的，可以单独对某一个 Bank 进行，也可以一次对所有 Bank 进行。如果 A10 为高，那么 SDRAM 进行 All Bank Precharge 操作，如果 A10 为低，那么 SDRAM 根据 BA[1:0] 的值，对指定的 Bank 进行 Precharge 操作。 SDRAM 完成 Precharge 操作需要的时间定义为 tPR。
- Auto-Refresh DRAM 的 Storage Cell 中的电荷会随着时间慢慢减少，为了保证其存储的信息不丢失，需要周期性的对其进行刷新操作。 SDRAM 的刷新是按 Row 进行，标准中定义了在一个刷新周期内（常温下 64ms，高温下 32ms）需要完成一次所有 Row 的刷新操作。 为了简化 SDRAM Controller 的设计，SDRAM 标准定义了 Auto-Refresh 机制，该机制要求 SDRAM Controller 在一个刷新周期内，发送 8192 个 Auto-Refresh Command，即 AR， 给 SDRAM。 SDRAM 每收到一个 AR，就进行 n 个 Row 的刷新操作，其中，n = 总的 Row 数量 / 8192 。此外，SDRAM 内部维护一个刷新计数器，每完成一次刷新操作，就将计数器更新为下一次需要进行刷新操作的 Row。 一般情况下，SDRAM Controller 会周期性的发送 AR，每两个 AR 直接的时间间隔定义为 tREFI = 64ms / 8192 = 7.8 us。 SDRAM 完成一次刷新操作所需要的时间定义为 tRFC, 这个时间会随着 SDRAM Row 的数量的增加而变大。 由于 AR 会占用总线，阻塞正常的数据请求，同时 SDRAM 在执行 refresh 操作是很费电，所以在 SDRAM 的标准中，还提供了一些优化的措施，例如 DRAM Controller 可以最多延时 8 个 tREFI 后，再一起把 8 个 AR 同时发出。
- Self-Refresh Host 还可以让 SDRAM 进入 Self-Refresh 模式，降低功耗。在该模式下，Host 不能对 SDRAM 进行读写操作，SDRAM 内部自行进行刷新操作保证数据的完整。通常在设备进入待机状态时，Host 会让 SDRAM 进入 Self-Refresh 模式，以节省功耗。

---

# 七 MRS 介绍

MRS : Mode Register Set，模式寄存器设置。通过编程来实现，MR 没有缺省值，所以它必须在上电或复位后被完全初始化，这样才能使得 DDR 正常工作。

## 7.1 MR0

## 7.2 MR1

---

# 八 DDR 测试介绍

1、 注意示波器探头和示波器带宽能够满足测试要求，测试点要注意选到尽量靠近信号的接收端；

2、 由于 DDR 信令比较复杂，为了能快速测试、调试和解决信号上的问题，最常用的是通过眼图分析来帮助检查 DDR 信号是否满足电压、定时和抖动方面的要求；

3、 触发模式的设置有几种，首先可以利用前导宽度触发器分离读/写信号。根据 JEDEC 规范，读前导的宽度为 0.9 到 1.1 个时钟周期，而写前导的宽度规定为大于 0.35 个时钟周期，没有上限。第二种触发方式是利用更大的信号幅度触发方法分离读/写信号。通常，读/写信号的信号幅度是不同的，因此我们可以通过在更大的信号幅度上触发示波器来实现两者的分离；

4、 测试中要注意信号的幅度，时钟的频率，差分时钟的交叉点，上升沿是否单调，过冲等。

## 8.1 一致性测试

---
# 九 DDR 异常分析

![image.png](https://cdn.nlark.com/yuque/0/2023/png/22935284/1696991179921-852c495f-a2ea-4b13-8fe7-0d58c4c2b0db.png#averageHue=%23ebf6f4&clientId=uecf66a0e-c566-4&from=paste&height=440&id=u054421b1&originHeight=440&originWidth=1392&originalType=binary&ratio=1&rotation=0&showTitle=false&size=78300&status=done&style=none&taskId=ub65f6ae8-c4d5-4a8a-952b-eb797d95822&title=&width=1392)

![image.png](https://cdn.nlark.com/yuque/0/2023/png/22935284/1696991190597-1ca10558-5e29-4125-b7e0-777ff241b2c7.png#averageHue=%23ecf6f5&clientId=uecf66a0e-c566-4&from=paste&height=575&id=u0cfede7c&originHeight=575&originWidth=1396&originalType=binary&ratio=1&rotation=0&showTitle=false&size=92069&status=done&style=none&taskId=u22038d57-02f7-430e-8895-77bd4c0c386&title=&width=1396)

![image.png](https://cdn.nlark.com/yuque/0/2023/png/22935284/1696991203303-6545f408-5fa2-4870-bf3e-6c385059c342.png#averageHue=%23ecf6f4&clientId=uecf66a0e-c566-4&from=paste&height=326&id=u851186e0&originHeight=326&originWidth=1393&originalType=binary&ratio=1&rotation=0&showTitle=false&size=54705&status=done&style=none&taskId=uf2b9d067-b78c-4af3-80d5-1790e48cf1e&title=&width=1393)

![image.png](https://cdn.nlark.com/yuque/0/2023/png/22935284/1696991223873-392cc82d-2420-4ead-a3fc-289030a16e90.png#averageHue=%23d5ecf7&clientId=uecf66a0e-c566-4&from=paste&id=u3af9404b&originHeight=1080&originWidth=1920&originalType=url&ratio=1&rotation=0&showTitle=false&size=408176&status=done&style=none&taskId=u4b488e26-b22e-40f7-a460-808c7c34e69&title=)

![image.png](https://cdn.nlark.com/yuque/0/2023/png/22935284/1696991223924-9e26932b-e36d-4e90-a9ec-dcfd085c8a2b.png#averageHue=%23cbcbd0&clientId=uecf66a0e-c566-4&from=paste&id=u0df1996f&originHeight=1080&originWidth=1920&originalType=url&ratio=1&rotation=0&showTitle=false&size=493730&status=done&style=none&taskId=u75fc81fb-793b-45ca-976a-56d68f73030&title=)

![image.png](https://cdn.nlark.com/yuque/0/2023/png/22935284/1696991223866-bbb06339-28f0-4c9d-aa15-192a3ff11fd5.png#averageHue=%23e1e0e4&clientId=uecf66a0e-c566-4&from=paste&id=u076d0a08&originHeight=1080&originWidth=1920&originalType=url&ratio=1&rotation=0&showTitle=false&size=236292&status=done&style=none&taskId=ub85a78f3-3b2b-4c94-9c80-18ab9df9d6c&title=)

![image.png](https://cdn.nlark.com/yuque/0/2023/png/22935284/1696991224303-9a02769c-069d-4ea8-be2e-d6dd16514d4a.png#averageHue=%23abaaac&clientId=uecf66a0e-c566-4&from=paste&id=ud71143c3&originHeight=1080&originWidth=1920&originalType=url&ratio=1&rotation=0&showTitle=false&size=287446&status=done&style=none&taskId=u0d1eaf75-eaec-48f2-937e-2d20dee8b45&title=)

![image.png](https://cdn.nlark.com/yuque/0/2023/png/22935284/1696991224363-e8d50522-ffc3-4c13-897d-16a0f6763d2f.png#averageHue=%23878e9d&clientId=uecf66a0e-c566-4&from=paste&id=ue7c309bf&originHeight=1080&originWidth=1920&originalType=url&ratio=1&rotation=0&showTitle=false&size=347089&status=done&style=none&taskId=u79659176-9be5-466a-ba2a-6c135abf2d0&title=)
