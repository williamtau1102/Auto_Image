
# 参考资料

[https://coolshell.cn/articles/20793.html](https://coolshell.cn/articles/20793.html)
[https://www.cnblogs.com/jokerjason/p/10711022.html](https://www.cnblogs.com/jokerjason/p/10711022.html)
[https://hio.oppo.com/app/module/detail?tkh_id=1536869&itm_id=5959&mod_id=121671](https://hio.oppo.com/app/module/detail?tkh_id=1536869&itm_id=5959&mod_id=121671) [https://www.processon.com/mindmap/63e44b2f052b4a2bae427456](https://www.processon.com/mindmap/63e44b2f052b4a2bae427456) 
<a name="yTOuh"></a>
# 一 MEMORY Roadmap

![image.png](https://cdn.nlark.com/yuque/0/2023/png/22935284/1695778729209-feb23d2d-480c-43b6-b994-a9c523f2a000.png#averageHue=%23f6e8cb&clientId=u9bd7bc36-280e-4&from=paste&id=u0eece3a4&originHeight=906&originWidth=1943&originalType=url&ratio=1&rotation=0&showTitle=false&size=565051&status=done&style=none&taskId=u1aa79132-258d-42cd-89f4-ec12fd86eef&title=)

RPMb：Replay Protected Memory Block ，重放保护内存块。是eMMC中的一个具有安全特性的分区。eMMC 在写入数据到 RPMB 时，会校验数据的合法性，只有指定的 Host 才能够写入，同时在读数据时，也提供了签名机制，保证 Host 读取到的数据是 RPMB 内部数据，而不是攻击者伪造的数据。 <br />RPMB 在实际应用中，通常用于存储一些有防止非法篡改需求的数据，例如手机上指纹支付相关的[公钥](https://so.csdn.net/so/search?q=%E5%85%AC%E9%92%A5&spm=1001.2101.3001.7020)、序列号等。RPMB 可以对写入操作进行鉴权，但是读取并不需要鉴权，任何人都可以进行读取操作，因此存储到 RPMB 的数据通常会进行加密后再存储。 

![image.png](https://cdn.nlark.com/yuque/0/2023/png/22935284/1695778729224-e4cfa196-3db1-4cf8-ac6a-a35274aebec0.png#averageHue=%2343ad7f&clientId=u9bd7bc36-280e-4&from=paste&id=u7f4df364&originHeight=849&originWidth=1632&originalType=url&ratio=1&rotation=0&showTitle=false&size=726767&status=done&style=none&taskId=ub459b8b5-03ed-4605-8eed-820f1a75b5d&title=)

![image.png](https://cdn.nlark.com/yuque/0/2023/png/22935284/1695778729064-cd79ed1e-1bf4-4e0f-9df6-54828105c6c9.png#averageHue=%23fcf2e2&clientId=u9bd7bc36-280e-4&from=paste&id=u90c5ba4c&originHeight=612&originWidth=1457&originalType=url&ratio=1&rotation=0&showTitle=false&size=271445&status=done&style=none&taskId=u4016798d-b692-42e5-8086-d3fffaa7e5a&title=)

![image.png](https://cdn.nlark.com/yuque/0/2023/png/22935284/1695778729607-a9503d46-3ab8-4c73-81ed-f0aa99fe98d7.png#averageHue=%239f9f98&clientId=u9bd7bc36-280e-4&from=paste&id=uc34e2069&originHeight=812&originWidth=1773&originalType=url&ratio=1&rotation=0&showTitle=false&size=1739553&status=done&style=none&taskId=u36d29182-acc2-457b-8cee-6a8d431d7ae&title=)

![image.png](https://cdn.nlark.com/yuque/0/2023/png/22935284/1695778729051-a37807f7-cb49-4f32-b409-89c94406d39d.png#averageHue=%2395c459&clientId=u9bd7bc36-280e-4&from=paste&height=510&id=u42ed01ab&originHeight=801&originWidth=1269&originalType=url&ratio=1&rotation=0&showTitle=false&size=280567&status=done&style=none&taskId=u093e053f-71b0-441c-b450-9171bc226ee&title=&width=808)

![image.png](https://cdn.nlark.com/yuque/0/2023/png/22935284/1695778730273-b9fe8b75-90d7-4998-90dc-c27983f1a605.png#averageHue=%2395c45a&clientId=u9bd7bc36-280e-4&from=paste&height=510&id=ued00a7db&originHeight=816&originWidth=1295&originalType=url&ratio=1&rotation=0&showTitle=false&size=301994&status=done&style=none&taskId=u57727b54-de3b-4ada-983c-c905734d6b7&title=&width=810)

JEDEC(Joint Electron Device Engineering Council，固态技术协会)，主要促进固体技术工业和其它相关活动的标准化，成立于1958年。包括**术语、定义、产品特征描述与操作、测试方法、生产支持功能、产品质量与可靠性、机械外形、固态存储器、DRAM、闪存卡及模块，以及射频识别RFID标识**等。 

LPDDR5 : JESD209-5
LPDDR6 :
UFS5.0 

产品可靠性测试标准完整大集合（JEDEC/IEC/SAE...）： [https://blog.csdn.net/Johnho130/article/details/117413710](https://blog.csdn.net/Johnho130/article/details/117413710)

# 二 数据存储原理

如下图所示为使用四个与非门组成的逻辑电路，i为输入信号，o为输出信号，s为设置信号。

s=1，i=0，a=1&b=0，c=1，o=0； <br />s=1，i=1，a=0&b=1，o=1； <br />s=0，i=0/i=1，a=1&b=1，o？？？

从而实现了一位数据的记忆功能。

![image.png](https://cdn.nlark.com/yuque/0/2023/png/22935284/1695778944779-a439650f-901e-4f61-9ce4-13fb28ce66ac.png#averageHue=%23f4f3f1&clientId=u258e3275-017c-4&from=paste&id=u4d2ba979&originHeight=352&originWidth=589&originalType=url&ratio=1&rotation=0&showTitle=false&size=58858&status=done&style=none&taskId=uf5034db9-e798-4083-bb91-4e5ebfa52d8&title=)<br />![image.png|525](https://cdn.nlark.com/yuque/0/2023/png/22935284/1695778944977-f0b1d2f4-fbdc-47d2-844c-8c57ef680ce2.png#averageHue=%23a539ab&clientId=u258e3275-017c-4&from=paste&height=465&id=uabf42d0e&originHeight=635&originWidth=737&originalType=url&ratio=1&rotation=0&showTitle=false&size=535013&status=done&style=none&taskId=u4ed17ebc-6a5f-4741-9906-484f5ad1da6&title=&width=540)

# 三 锁存器

![image.png|600](https://cdn.nlark.com/yuque/0/2023/png/22935284/1696991520793-c5adffdf-0a00-4fd0-8cba-4e7797d8d2e6.png#averageHue=%23656065&clientId=ufbda3028-bc8e-4&from=paste&height=346&id=u82243667&originHeight=675&originWidth=1062&originalType=url&ratio=1&rotation=0&showTitle=false&size=860419&status=done&style=none&taskId=u27f1e22b-c2a1-4036-9f19-82d2f120e90&title=&width=544)

<a name="L0HW2"></a>
# 四 寄存器

一组锁存器被称为寄存器Register，寄存器能存多少个bit叫位宽，如8位寄存器、16位寄存器和32位寄存器等。具体如下图所示，有允许写入信号、数据写入信号和数据读取信号三种。

实际操作时，先将允许写入信号置1，然后通过数据线写入8位数据，再将允许写入信号置0，然后8bit的值就存起来了。

![image.png|675](https://cdn.nlark.com/yuque/0/2023/png/22935284/1696991532193-6abd51b6-8242-4667-ae8c-762e3ff7e3de.png#averageHue=%23c755c6&clientId=ufbda3028-bc8e-4&from=paste&height=386&id=ub26f770f&originHeight=593&originWidth=1024&originalType=url&ratio=1&rotation=0&showTitle=false&size=767961&status=done&style=none&taskId=uc85fa415-80c2-4540-8c4c-45d336c4e9e&title=&width=666)

多个寄存器通过总线连接，能够存储更多的数据。再加上行列的选择，最终可以组成更大的内存。

![image.png](https://cdn.nlark.com/yuque/0/2023/png/22935284/1696991538155-3c2e3c01-fc00-40a5-aa3c-b0d48a421886.png#averageHue=%23111011&clientId=ufbda3028-bc8e-4&from=paste&id=u59ab795d&originHeight=452&originWidth=675&originalType=url&ratio=1&rotation=0&showTitle=false&size=113243&status=done&style=none&taskId=u0f2448dc-d849-4b12-8d61-a1027b25e01&title=)

# 五 DRAM vs SRAM

DRAM(Dynamic Random Access Memory)，DRAM每个bit存储需要用到的晶体管比SRAM少。它存在一个问题，它的电荷会随时间而衰减，通常每隔10个系统周期DRAM的内容就需要刷新(重写)。重写包括把数据读到memory控制器后再写回DRAM。当DRAM开始刷新时，数据将不能读写。

SRAM(Static Random Access Memory)，**SRAM存储一位用到的器件比DRAM要贵要多，它不需要进行不断刷新，SRAM主要用在Cache上**。SRAM不需要刷新，数据放置在SRAM里永远有效，通常用于需要高性能访问的地方。主要任务是从DRAM中拷贝数据和指令然后给CPU使用。当CPU需要访问特定数据时，它总是先检查访问速度较快的缓存，看看所需要的数据是否在那里，如果没有，那么它就要去访问速度较慢的DRAM以获取所需要的数据。CPU的速度要比存储的速度快很多。

![[Pasted image 20240618105654.png]]

![image.png](https://cdn.nlark.com/yuque/0/2023/png/22935284/1696991545206-2989aa6a-86b6-4da0-b20e-27252888d803.png#averageHue=%232a282a&clientId=ufbda3028-bc8e-4&from=paste&height=319&id=ue22fa0d7&originHeight=437&originWidth=945&originalType=url&ratio=1&rotation=0&showTitle=false&size=130727&status=done&style=none&taskId=u24ffd6ac-c1dc-40af-99dc-c38ef85c0a5&title=&width=690)

# 六 Cache

## 6.1 Memory Hierarchy 

Cache需要考虑性能与价格的折中设计。
![image.png](https://cdn.nlark.com/yuque/0/2023/png/22935284/1696991349543-f022bdf4-06f8-480c-a9de-a322831b3e60.png#averageHue=%23111011&clientId=u3cbe1a87-2571-4&from=paste&height=383&id=u98a16869&originHeight=452&originWidth=675&originalType=url&ratio=1&rotation=0&showTitle=false&size=113243&status=done&style=none&taskId=u14a0bf63-989e-421e-b8eb-77a43e2433a&title=&width=572)

寄存器是由D触发器构成。
![image.png](https://cdn.nlark.com/yuque/0/2023/png/22935284/1696991349604-e3901609-99ad-4cd1-97ee-fdf50f34c034.png#averageHue=%23879d8b&clientId=u3cbe1a87-2571-4&from=paste&id=udd4dce1a&originHeight=436&originWidth=1024&originalType=url&ratio=1&rotation=0&showTitle=false&size=222977&status=done&style=none&taskId=u19853dce-1429-4928-a416-c94fca082c4&title=)

Cache通常只保留有效数据的很小一部分。

![image.png](https://cdn.nlark.com/yuque/0/2023/png/22935284/1696991349526-726c3e68-2bc7-49c5-bbb7-0a4a95e0112c.png#averageHue=%23f5c306&clientId=u3cbe1a87-2571-4&from=paste&id=u3abe4bcc&originHeight=371&originWidth=729&originalType=url&ratio=1&rotation=0&showTitle=false&size=16578&status=done&style=none&taskId=u2158e140-258d-456f-ade9-a58d5da192a&title=)

如下图所示为骁龙888的Cache结构图：

![image.png](https://cdn.nlark.com/yuque/0/2023/png/22935284/1696991349537-7496fdb0-e5ef-461a-a822-0f96f5aa610c.png#averageHue=%23bfdb99&clientId=u3cbe1a87-2571-4&from=paste&id=uc39d2ee2&originHeight=444&originWidth=945&originalType=url&ratio=1&rotation=0&showTitle=false&size=58706&status=done&style=none&taskId=ua4183855-6b31-4069-9f79-24f6125007a&title=)

## 6.2 缓存分级

CPU的Cache有三种不同的等级：从一级到三级，离CPU越来越远，容量越来越大，但速度越来越慢。一般L1和L2容量是KB级别，L3是MB级别的。 
<a name="xML82"></a>
### 6.2.1 一级缓存L1 

也被称为主缓存，它在CPU内部，为单一CPU核专用，与处理器的速度相当，也是速度最快的缓存。包括指令缓存和数据缓存，而其它两种缓存不分指令和数据。

![image.png](https://cdn.nlark.com/yuque/0/2023/png/22935284/1696991364898-a1d49956-f7c7-48df-90a3-36985de506b7.png#averageHue=%23f0f0f0&clientId=u3cbe1a87-2571-4&from=paste&id=u0b921cc1&originHeight=328&originWidth=598&originalType=url&ratio=1&rotation=0&showTitle=false&size=115239&status=done&style=none&taskId=u58afb721-dd3a-496a-8c15-8c754b7f45b&title=)

### 6.2.2 二级缓存L2 

用于存储最近被处理器访问的数据，为CPU核专用，但这些数据还没有被一级缓存存储。如果CPU没有在一级缓存中找到想要的数据，那么它就会搜索二极缓存。 

### 6.2.3 三级缓存L3 

用于存储最近没有被二级缓存存储的数据。如果三级缓存中仍然没有CPU想要的数据，那么CPU就会访问DDR。三级缓存被称为共享缓存，它是被CPU之间的所有内核共享。

### 6.2.4 DSU 

全称是DynamIQ Shared Unit，包括 L3 memory system, control logic, and external interfaces to support a DynamIQ cluster。 

什么是DynamIQ?  DynamIQ是ARM一个新的底层solution，用于连接在一个芯片上的不同core。有了DynamIQ，我们可以将不同类型的core放到一个cluster中。比如，将性能高的core，和功耗低的core放进一个cluster。如果没有DynamIQ，我们是将其放在2个不同cluster中的。最常见 4个Cortex-A72 核与4个Cortex-A53核，或者4个Cortex-A53与另外的4个Cortex-A53核配对。把核心放在同一个cluster中能保证核和核之间更好的通信。

![image.png](https://cdn.nlark.com/yuque/0/2023/png/22935284/1696991364893-f8c48edd-0d0c-47a3-89ee-0dbbd79047d3.png#averageHue=%2381c002&clientId=u3cbe1a87-2571-4&from=paste&id=u35b26fac&originHeight=358&originWidth=697&originalType=url&ratio=1&rotation=0&showTitle=false&size=18670&status=done&style=none&taskId=uc243e6dd-99e6-4e7b-82f8-a6f5a86fe2d&title=)

### 6.2.5 System Cache 

**LLCC Last level cache controller**

**也称SLC/SLB(System Level Cache/Buffer)由CPUSS(CPU子系统)跟其它子系统(GPU、DSP等)共用

![image.png](https://cdn.nlark.com/yuque/0/2023/png/22935284/1696991364926-dcc56163-9c80-4224-9dfb-61033460aaea.png#averageHue=%23f8f5f5&clientId=u3cbe1a87-2571-4&from=paste&id=u739bb6d0&originHeight=480&originWidth=864&originalType=url&ratio=1&rotation=0&showTitle=false&size=116542&status=done&style=none&taskId=ub851bf8e-0c08-425b-9cf6-8b77ee149ac&title=)

数据的路径：从内存向上，先到L3，再到L2，再到L1，最后到寄存器进行CPU计算。Cache设计成三层的原因是： <br /> 一方面是物理速度，如果要更大的容量就需要更多的晶体管，除了芯片的体积会变大，更重要的是大量的晶体管会导致速度下降，因为访问速度和要访问的晶体管所在的位置成反比，当信号路径变长时，通信速度会变慢；

另一方面，多核技术中，数据的状态需要在多个CPU中进行同步，多级不同尺寸的缓存有利于提高整体的性能。 

## 6.3 局部性原则 

 缓存的空间是有限的，它只能缓存最新被CPU访问过的数据。 

**时间局部性：在最近的未来要用到的信息，很可能是现在正在使用的信息，因为程序中存在循环**。 

**空间局部性：在最近的未来要用到的信息，很可能与现在正在使用的信息在存储空间上是邻近的，因为指令通常是顺序存放**，顺序存放的数据也一般是以向量、数组形式簇聚地存储在一起的； 

因为如果将刚刚访问过的数据缓存在Cache中，那下次访问时，可以直接从Cache中取，其速度可以得到数量级的提高。 

LRU(Least Recently Used，最近最少使用)，是一种常见的页面置换算法。选择最近最久未使用的页面予以淘汰。 

## 6.4 缓存的命中/Cache Hit or Miss 

**如果想要的数据已经在缓存，就叫缓存命中(Cache hit)。**如果想要的数据没有在缓存中，则必须向下一级Cache发出请示，就叫缓存未命中(Cache miss)。此时，请求者必须等待，或者处理其它的工作，直到被请求的数据被找到。 

![image.png](https://cdn.nlark.com/yuque/0/2023/png/22935284/1696991376912-ee9db8d5-2c50-4482-bc67-2beb8359e009.png#averageHue=%23bec2c9&clientId=u3cbe1a87-2571-4&from=paste&id=u947e0c7a&originHeight=459&originWidth=883&originalType=url&ratio=1&rotation=0&showTitle=false&size=93153&status=done&style=none&taskId=u6d4f0652-bff1-47cf-882e-6c5b6b82ac3&title=)

Cache hit的比率取决于：1、Cache的大小；2、Cache存放数据的多少；3、离开参考点的位置；4、Cache的结构。Cache miss的花销非常高，大部分的花销用于从下一级cache中获取数据，即使是很少比例的cache miss都将对系统的性能产生显著的影响。 

因为效率的原因，cache通常100%用于保存数据。当cache miss发生时，新的数据必须补充到cache里用于替换cache里已经存在的数据，这个过程叫"move in"。替换发生时，必须为进来的数据寻找位置。数据的花销已经在Cache里出现了，根据cache的设计，cache里或许只有一个位置能存放新数据，或者有多个，如果有多个，对性能损害更少的位置将被选择。用于寻找位置的运算法则是：**旧的、很长时间不用的位置赋上最小值并选择它作为进来数据的新位置,这种算法叫LRU(least recently used)**。 

## 6.5 Cache写机制 

Cache写机制分为write through和write back两种。

Write through直写模式：在数据更新时，同时写入Cache和后端存储。此模式的优点是操作简单，缺点是因为数据修改需要同时写入存储，数据写入速度较慢。

Write back回写模式：在数据更新时只写入Cache，只有数据被替换出Cache时，被修改的Cache数据才会被写到后端存储。此模式的优点是数据写入速度快，因为不需要写存储；缺点是一旦更新后的数据未被写入存储时出现系统掉电的情况，数据将无法找回。

## 6.6 Cache一致性 

多个CPU对某个内存块同时读写，会引起冲突的问题，也被称为Cache一致性问题。Cache一致性出现的原因是在一个多处理器系统中，多个处理器核心都能够独立地执行计算机指令，从而有可能同时对某个内存块进行读写操作，并且由于我们之前提到的回写和直写的Cache策略，导致一个内存块同时可能有多个备份，有的已经写回到内存中，有的在不同的处理器核心的一级、二极Cache中。由于Cache缓存的原因，我们不知道数据写入的时序性，因而也不知道哪个备份是最新的。另外，假如有两个线程A和B共享一个变量，当线程A处理完一个数据之后，通过这个变量通知线程B，然后线程B对这个数据接着进行处理，如果两个线程运行在不同的处理器核心上，那么运行线程B的处理器就会不停地检查这个变量，而这个变量存储在本地的Cache中，因此就会发现这个值总也不会发生变化。 

为了正确性，一旦一个核心更新了内存中的内容，硬件就必须要保证其他的核心能够读到更新后的数据。目前大多数硬件采用的策略或协议是MESI或基于MESI的变种：
M代表更改（modified），表示缓存中的数据已经更改，在未来的某个时刻将会写入内存；
E代表排除（exclusive），表示缓存的数据只被当前的核心所缓存；
S代表共享（shared），表示缓存的数据还被其他核心缓存；
I代表无效（invalid），表示缓存中的数据已经失效，即其他核心更改了数据。 

## 6.7 Cache与主存的映射  

### 6.7.1 主存的分块制度 

每四个主存物理地址的数据组成为一个块，主存物理地址的最后两位为块内地址，前面就是主存块号。主存物理地址与Cache的映射就是以块为单位进行信息交换的，这也是空间局部性的来源——即前一次使用某一个块中的地址，后面很有可能会使用其相邻的地址。

![image.png](https://cdn.nlark.com/yuque/0/2023/png/22935284/1696991390255-daa49d95-9d9a-4181-b19d-5bd0a029bdd8.png#averageHue=%23f2ae1f&clientId=u3cbe1a87-2571-4&from=paste&id=u643cce7a&originHeight=540&originWidth=812&originalType=url&ratio=1&rotation=0&showTitle=false&size=189117&status=done&style=none&taskId=u74da7ff4-c578-4921-b437-8a532fe973d&title=)

Cache Line:**对CPU来说，它不会一个字节一个字节加载数据，一般来说都是要一块一块的加载的，对于这样的一块一块的数据单位，术语叫“Cache Line”**，一般来说，一个主流的CPU的Cache Line 是 64 Bytes（也有的CPU用32Bytes和128Bytes），64Bytes也就是16个32位的整型，这就是CPU从内存中捞数据上来的最小数据单位。比如L1有32KB，那么共有512个Cache Line。 

linux原生系统中查看cache line大小的指令：`cat /sys/devices/system/cpu/cpu1/cache/index0/coherency_line_size` 

缓存需要把内存里的数据放进来，英文叫**CPU Associativity**。Cache的数据放置的策略决定了内存中的数据块会拷贝到Cache中的哪个位置上，因为Cache的大小远远小于内存，所以，需要有一种地址关联的算法，能够让内存中的数据可以被映射到Cache中来。 

### 6.7.2 Cache与主存的映射 

Cache里的内容一定是主存的副本，是从主存中复制过来的，会把常用的数据放在Cache中。

全相联映射：将一个主存块存储到任意一个Cache行。主存的一个块直接拷贝到cache中的任意一行上

![image.png](https://cdn.nlark.com/yuque/0/2023/png/22935284/1696991390230-99c07c49-aeeb-4b5a-b545-5c00f7ff22e1.png#averageHue=%23f4f4f4&clientId=u3cbe1a87-2571-4&from=paste&id=ucce23c20&originHeight=326&originWidth=546&originalType=url&ratio=1&rotation=0&showTitle=false&size=55424&status=done&style=none&taskId=u9373fa8b-d2c2-43ce-b837-91429156c25&title=)

组相联映射：将一个主存块存储到唯一的一个Cache组中任意一个行。

将cache分成u组，每组v行，主存块存放到哪个组是固定的，至于存到该组哪一行是灵活的，即有如下函数关系：cache总行数m＝u×v 组号q＝j mod u

组间采用直接映射，组内为全相联，硬件较简单，速度较快，命中率较高 

![image.png](https://cdn.nlark.com/yuque/0/2023/png/22935284/1696991390235-c9290437-6233-44d0-8bb5-02515bedf19b.png#averageHue=%23f6f6f6&clientId=u3cbe1a87-2571-4&from=paste&id=ue56c3ab1&originHeight=449&originWidth=660&originalType=url&ratio=1&rotation=0&showTitle=false&size=98443&status=done&style=none&taskId=u8e333a7f-fe9f-4cef-9ac1-f0ce610ed8d&title=)

直接映射：将一个内存块存储到唯一的一个Cache行。多对一的映射关系，但一个内存块只能copy到cache的一个特定行位置上去。cache的行号i和主存的块号j有如下函数关系：i=j mod m（m为cache中的总行数）

硬件简单、容易实现；但命中率低，Cache的存储空间利用率低。

![image.png](https://cdn.nlark.com/yuque/0/2023/png/22935284/1696991390230-18f29f89-cc18-4cee-8137-6afe96ee6863.png#averageHue=%23f7f7f7&clientId=u3cbe1a87-2571-4&from=paste&id=ua6231643&originHeight=376&originWidth=520&originalType=url&ratio=1&rotation=0&showTitle=false&size=64773&status=done&style=none&taskId=uc514fb44-de96-4164-b0c6-5478884f587&title=)

![image.png](https://cdn.nlark.com/yuque/0/2023/png/22935284/1696991390271-2cecb9a7-d48d-4191-bb3c-6d390f934621.png#averageHue=%23fefdfb&clientId=u3cbe1a87-2571-4&from=paste&id=u659b1e2d&originHeight=467&originWidth=976&originalType=url&ratio=1&rotation=0&showTitle=false&size=142185&status=done&style=none&taskId=u1d55bab8-5438-4a35-8011-f88451bda90&title=)

当进来的数据的位置被找到以后，系统并不马上将新数据拷贝到这个位置。在进来的数据放置之前，当前的数据必须被移出或flushed，并将其备份到下一级的cache上。如果在下一级已经有些数据存在，则数据将被丢弃。然后新进来的数据才被拷贝到这个位置上。在move out结束之后，move in开始进行，被移动的数据的总量叫cache line。在CPU cache当中被移动的数据有一定的大小。 

## 6.8 Buffer和Cache 

实验说明Cache和Buffer在使用中的区别：

dd if=/dev/urandom of=/dev/nvme0n1p1 bs=1M count=2000   # 向指定磁盘写入指定大小的数据
vmstat 1 1000       # 以1s为间隔实时显示缓冲缓存的使用情况 
dd if=/dev/nvme0n1p1 of=/dev/null bs=1M count=10000   # 磁盘读数据

![image.png](https://cdn.nlark.com/yuque/0/2023/png/22935284/1696991428653-7402f81c-bcff-46d6-be90-5bf4af780a38.png#averageHue=%23222222&clientId=u3cbe1a87-2571-4&from=paste&id=u9f0febd9&originHeight=749&originWidth=776&originalType=url&ratio=1&rotation=0&showTitle=false&size=91397&status=done&style=none&taskId=u33b401a6-c722-4b66-8be3-dffda6e23e8&title=)
