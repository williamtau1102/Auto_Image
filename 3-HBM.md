<a name="e81Q3"></a>
# 参考资料 
1、JESD238A

# 一 HBM 概述 

HBM 全称是High Bandwidth Memory DRAM，高带宽内存。它是一种新型的CPU/GPU内存芯片，**将很多个DDR芯片堆叠在一起后和GPU封装在一起**，实现大容量、高位宽的DDR组合阵列。和传统的DRAM相比，HBM在**数据处理速度和性能方面**都具有显著优势

从2013年开始，国际电子元件工业联合会（JEDEC）先后制定了3代、多个系列版本的高带宽存储器（**HBM、HBM2、HBM2E、HBM3**）标准。2022年1月28日，JEDEC正式发布了JESD238 HBM DRAM（HBM3）标准，技术指标较现有的HBM2和HBM2E标准有巨大的提升，**芯片单个引脚速率达到 6.4 Gbit/s，实现了超过1TB/s的总带宽，支持16-Hi堆栈，堆栈容量达到64GB**，为新一代高带宽内存确定了发展方向。HBM通过TSV（硅通孔技术，Through -Silicon-Via，通过在芯片和芯片之间、晶圆和晶圆之间**制作垂直导通**，实现芯片之间互连的最新技术）和微凸块技术将3D垂直堆叠的DRAM芯片相互连接，突破了现有的性能限制，大大提高了存储容量，同时可提供很高的访存带宽。

高速、高带宽HBM堆栈没有以外部互连线的方式与信号处理器芯片连接，而是通过中间介质层紧凑而快速地连接，同时HBM内部的不同DRAM采用TSV实现信号纵向连接，HBM具备的特性几乎与片内集成的RAM存储器一样。

![image.png](https://cdn.nlark.com/yuque/0/2023/jpeg/22935284/1696991106178-6952e544-4e6e-4389-ac12-5ecabdc60bb7.jpeg#averageHue=%23cfcaae&clientId=uf3f6a2fc-82a9-4&from=paste&id=u1cae8b65&originHeight=444&originWidth=640&originalType=url&ratio=1&rotation=0&showTitle=false&size=51618&status=done&style=none&taskId=u2cf94fd5-1e35-46e1-ae04-63784fe2d28&title=)

更低功耗：由于采用了TSV和微凸块技术，DRAM裸片与处理器间实现了较短的信号传输路径以及较低的单引脚I/O速度和I/O电压，使HBM具备更好的内存功耗能效特性。以DDR3存储器归一化单引脚I/O带宽功耗比为基准，HBM2的I/O功耗比明显低于DDR3、DDR4和GDDR5存储器，相对于GDDR5存储器，HBM2的单引脚I/O带宽功耗比数值降低42%。

更小体积：在系统集成方面，HBM将原本在PCB板上的DDR内存颗粒和CPU芯片一起全部集成到SiP里，因此HBM在节省产品空间方面也更具优势。相对于GDDR5存储器，HBM2节省了94%的芯片面积。

The HBM3 DRAM uses a **wide-interface architecture** to achieve high-speed, low power operation. Each channel interface maintains a **64 bit data bus operating** at double data rate (DDR). 

# 二 HBM3 介绍

Each DRAM Stack containing 4 DRAM dies, each die supporting 4 channels.

Each channel provides access to an independent set of DRAM banks. Channels are independently clocked, and need not be synchronous.

![image.png](https://cdn.nlark.com/yuque/0/2023/png/22935284/1696991106186-7bddc698-3a49-492d-87d8-b910b976e9d3.png#averageHue=%238cc64d&clientId=uf3f6a2fc-82a9-4&from=paste&height=441&id=uac661e26&originHeight=620&originWidth=1024&originalType=url&ratio=1&rotation=0&showTitle=false&size=57549&status=done&style=none&taskId=u5769a352-ddc2-4b81-b95b-94703044f0f&title=&width=728)

## 2.1 HBM3 信号引脚介绍 

![image.png](https://cdn.nlark.com/yuque/0/2023/png/22935284/1697703824507-2d0c3210-b74c-4e97-a765-d5448a8e15a5.png#averageHue=%23e6e6e6&clientId=ufb403e09-8930-4&from=paste&height=907&id=u851279b9&originHeight=907&originWidth=751&originalType=binary&ratio=1&rotation=0&showTitle=false&size=216115&status=done&style=none&taskId=u48decdef-383e-4063-99d2-d71a7746146&title=&width=751)

![image.png](https://cdn.nlark.com/yuque/0/2023/png/22935284/1697703858510-4a6420bf-9ac8-467d-a39e-f85621fa46cb.png#averageHue=%23eaeaea&clientId=ufb403e09-8930-4&from=paste&height=847&id=ud31527ef&originHeight=847&originWidth=751&originalType=binary&ratio=1&rotation=0&showTitle=false&size=163547&status=done&style=none&taskId=ueb40e381-37be-4d9d-bfb1-eaa779faba8&title=&width=751)![image.png](https://cdn.nlark.com/yuque/0/2023/png/22935284/1696991106190-ae41582b-86ad-4cf9-b334-16158c071e25.png#averageHue=%23f9f9f9&clientId=uf3f6a2fc-82a9-4&from=paste&id=u3f95181f&originHeight=357&originWidth=412&originalType=url&ratio=1&rotation=0&showTitle=false&size=27055&status=done&style=none&taskId=ue09eccd5-2573-455d-8221-5f1b26ee1e6&title=)

## 2.2 HBM3 供电介绍 

VDDC : Core Supply Voltage
VDDQ : I/O Supply Voltage
VDDQL : Supply Voltage for TX Driver Output Stage
VPP : Pump Voltage

![image.png](https://cdn.nlark.com/yuque/0/2023/png/22935284/1696991106193-3fa87fe1-8134-4750-8fa4-9e0faa56152c.png#averageHue=%23ebebea&clientId=uf3f6a2fc-82a9-4&from=paste&id=uf99e5f1b&originHeight=278&originWidth=778&originalType=url&ratio=1&rotation=0&showTitle=false&size=51651&status=done&style=none&taskId=udf1b7f43-3bcf-4bd6-852e-c7d5e54f6d0&title=)

## 2.3 IEEE1500 

The IEEE Standard 1500 compliant test access port provides a direct test connection between a host and the HBM3 DRAM. 

# 三、HBM2 介绍
