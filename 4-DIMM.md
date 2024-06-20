
# 参考资料 

1. 04.20.28 - 288-Pin, 1.2 V (VDD), PC4-1600/PC4-1866/PC4-2133/PC4-2400/PC4-2666/PC4-2933/PC4-3200 DDR4 SDRAM Registered DIMM Design Specification

2. DDR5 Load Reduced (LRDIMM) and Registered Dual Inline Memory Module (RDIMM) Common Specification  JESD305

![image.png](https://cdn.nlark.com/yuque/0/2023/png/22935284/1696990960352-23e912dc-a5b8-4e54-9fc2-21e092e812b5.png#averageHue=%23eaeaea&clientId=uf1072fb8-f95f-4&from=paste&id=drYek&originHeight=324&originWidth=717&originalType=url&ratio=1&rotation=0&showTitle=false&size=77361&status=done&style=none&taskId=u779ed8b6-3c6d-4705-a83f-aad66a72dbd&title=)

![image.png|700](https://cdn.nlark.com/yuque/0/2023/png/22935284/1696990960351-3d43832f-5fa4-4104-88ca-d66568df2ab8.png#averageHue=%23e9e9e9&clientId=uf1072fb8-f95f-4&from=paste&id=jqKk8&originHeight=136&originWidth=618&originalType=url&ratio=1&rotation=0&showTitle=false&size=28669&status=done&style=none&taskId=u7c44fa11-7cb7-453c-b0f6-ea3620fcc4c&title=)

3. JESD82-513_v1.00-DDR RCD.pdf

# 一 DDR4 RDIMM 总结 

Registered Dual Inline Memory Module。主要用在服务器或工作站上，常见的有288Pin；

Fly-by topology . the clock, control, command, and address buses have been routed in a fly-by topology

![image.png](https://cdn.nlark.com/yuque/0/2023/png/22935284/1696990597860-5fd3eefc-0f64-4a3c-b61b-f5f5d95fd3a8.png#averageHue=%23f3f3f3&clientId=ud152c9a2-a529-4&from=paste&id=u54fc5cf7&originHeight=497&originWidth=809&originalType=url&ratio=1&rotation=0&showTitle=false&size=89454&status=done&style=none&taskId=u9de7ad82-ff48-46d9-bc31-d5f01392f0a&title=)

![image.png](https://cdn.nlark.com/yuque/0/2023/png/22935284/1696990597850-3551dfa0-8012-4d51-a7ef-0efb81f248d6.png#averageHue=%23e7e7e7&clientId=ud152c9a2-a529-4&from=paste&id=u13a644a4&originHeight=345&originWidth=803&originalType=url&ratio=1&rotation=0&showTitle=false&size=34957&status=done&style=none&taskId=u185165d6-f5ab-430f-ada3-ce6d3215b88&title=)

On-board I2C temperature sensor with integrated serial presence-detect (SPD) EEPROM； 

16 internal banks；Bank Grouping is applied；

Data transfer rates: PC4-3200, PC4-2933, 2666, PC4-2400, PC4-2133, PC4-1866, PC4-1600； 

![image.png](https://cdn.nlark.com/yuque/0/2023/png/22935284/1696990597884-5b8527a3-7f7f-4557-8e26-f26c0be3e10c.png#averageHue=%23e3e3e3&clientId=ud152c9a2-a529-4&from=paste&id=ub148bfc9&originHeight=355&originWidth=777&originalType=url&ratio=1&rotation=0&showTitle=false&size=71561&status=done&style=none&taskId=ubc34136f-ef1b-4d78-9fe4-cd7fdb5cd06&title=)

8 bit pre-fetch；Burst Length (BL) switch on-the-fly BL8 or BC4(Burst Chop)；

Temperature sensor with integrated SPD；

DBI (Data Bus Inversion) is supported(x8)； 

![image.png](https://cdn.nlark.com/yuque/0/2023/png/22935284/1696990597989-62ab2400-09e3-4dd5-9f71-2206defc6a28.png#averageHue=%23efefef&clientId=ud152c9a2-a529-4&from=paste&id=ue61843cb&originHeight=912&originWidth=809&originalType=url&ratio=1&rotation=0&showTitle=false&size=159041&status=done&style=none&taskId=u0b5fd347-8bcf-4781-8d0b-ff72d56c919&title=)

CK1_t/CK1_c : 见上图所示，**两信号在DIMM中通过电阻回环连接，并无实际意义**。

CK0_t, CK0_c terminated with 120 Ω ±5% resistor. <br />CK1_t, CK1_c terminated with 120 Ω ±5% resistor but not used.

![image.png](https://cdn.nlark.com/yuque/0/2023/png/22935284/1696990615843-78888965-fca4-4297-9af6-eb285fe91957.png#averageHue=%23f3f3f3&clientId=ud152c9a2-a529-4&from=paste&id=u54729b51&originHeight=824&originWidth=621&originalType=url&ratio=1&rotation=0&showTitle=false&size=131032&status=done&style=none&taskId=u0c18453a-d8a4-4a84-9b40-9fe21942ada&title=)![image.png](https://cdn.nlark.com/yuque/0/2023/png/22935284/1696990615813-1538d2e8-58f4-4c43-85d9-9fc77be520e2.png#averageHue=%23f1f1f1&clientId=ud152c9a2-a529-4&from=paste&id=u7febcd81&originHeight=735&originWidth=570&originalType=url&ratio=1&rotation=0&showTitle=false&size=119210&status=done&style=none&taskId=uc66673ab-f1fc-4c26-a9ec-9832febd915&title=)

<a name="H0fYe"></a>
## 1.1 供电总结 

VDD = 1.20V (NOM)；Module SDRAM core power supply

VPP = 2.5V (NOM)；DRAM activating power supply <br />VDDSPD = 2.5V (NOM)；Power supply used to power the I2C bus for SPD. 

VREFCA : Reference voltage for control, command, and address pins VDD/2.

VTT : Power supply for termination of address, command, and control VDD/2.
- VTT为DDR的地址线，控制线等[信号](https://www.hqchip.com/app/1072)提供上拉[电源](https://www.hqchip.com/app.html)，上拉[电阻](https://m.hqchip.com/app/485)是50Ω左右。VTT=1/2VDDQ，并且VTT要跟随VDDQ，因此需要专用的电源同时提供VDDQ和VTT。还有一个重要的原因，在Fly-by的拓扑中，VTT提[供电](https://www.hqchip.com/app/1720)流，增强DDR信号线的驱动能力。

- DDR的[接收器](https://m.hqchip.com/app/1847)是一个[比较器](https://m.hqchip.com/app/1737)，其中一端是[VR](https://m.elecfans.com/v/tag/3668/)EF，另一端是信号，例如地址线A2在有VTT上拉的时候，A2的信号在0和1.8V间跳动，当A2电压高于VTT时，[电流](https://www.elecfans.com/tags/%E7%94%B5%E6%B5%81/)流向VTT。当A2低于VTT时，VTT流向DDR。因此VTT需要有提供电流和吸收电流的能力，一般的[开关电源](https://www.hqchip.com/app/1703)不能作为VTT的提供者。此外，VTT电源相当于DDR接收器信号输入端的直流偏执，且这个偏执等于VREF，因此VTT的噪声要越小越好，否则当A2的状态为高阻态时，DDR接收器的比较器容易产生误触发。

- VTT相当于DDR接收器的直流偏执，其实如果没有VTT，这个直流偏执也存在，它在芯片的内部，提供电流的能力很弱。如果只有1个或2个DDR芯片，走Fly-by拓扑，那么不需要外部的VTT上拉。如果有2个以上的DDR芯片，则一定需要VTT上拉。

SDRAM I/O termination supply 

![image.png](https://cdn.nlark.com/yuque/0/2023/png/22935284/1696990615824-0a7b4ce8-5241-406c-80f9-1dbd9fbd2d5b.png#averageHue=%23e9e9e9&clientId=ud152c9a2-a529-4&from=paste&id=u9388725e&originHeight=645&originWidth=807&originalType=url&ratio=1&rotation=0&showTitle=false&size=155316&status=done&style=none&taskId=uadcb05f0-341b-4b4e-bb2b-b49a3b894e9&title=)

### 1.1.1 上下电时序

1. V_12 must ramp with VPP or prior to VPP such that it is available as a power source on the DIMMs which may have on DIMM voltage regulators.

2. VDDSPD is an independent power source which has no specific relationship to the other power sources. 

3. VTT has a specific relationship to VDD. That is VTT = VDD/2. 

4. The CK_t/CK_c input signals must be driven LOW (below the VIL(static) DDR4RCD01 parameter) throughout the VDD power ramp at least until the VDD supply voltage has settled to its final value. 

![image.png](https://cdn.nlark.com/yuque/0/2023/png/22935284/1696990635993-7dae9a5e-619f-426b-b5f3-e43f68294c69.png#averageHue=%23f0f0f0&clientId=ud152c9a2-a529-4&from=paste&id=udbaf3a01&originHeight=489&originWidth=804&originalType=url&ratio=1&rotation=0&showTitle=false&size=48145&status=done&style=none&taskId=u62bfdab8-c242-48f6-a2be-988795cf219&title=)

![image.png](https://cdn.nlark.com/yuque/0/2023/png/22935284/1696990636015-16e6b2e1-4cdf-40c7-a3b9-a0fa155713dd.png#averageHue=%23f2f2f2&clientId=ud152c9a2-a529-4&from=paste&id=u608182d8&originHeight=366&originWidth=802&originalType=url&ratio=1&rotation=0&showTitle=false&size=27054&status=done&style=none&taskId=u2b8cdae3-1311-4ce8-ab82-3eeeed335ca&title=)

## 1.2 数据/时钟/地址/控制信号总结

**DQ[63-0]** : Data input/output

**CBx** : check bit input/output

**C0, C1, C2(RDIMM/LRDIMM only)** : Chip ID input . These inputs are used only when devices are stacked; that is, 2H, 4H, and 8H stacks for x4 and x8 configurations using through-silicon vias (TSVs). **These pins are not used in the x16 configuration**. Some DDR4 modules support **a traditional DDP package, which uses CS1_n,CKE1, and ODT1 to control the second die.** All other stack configurations, such as a 4H or 8H,are assumed to be single-load (master/slave) type configurations where **C0, C1, and C2 are used as chip ID selects in conjunction with a single CS_n, CKE, and ODT. Chip ID is considered part of the command code**. 

![image.png](https://cdn.nlark.com/yuque/0/2023/png/22935284/1696990680155-9ba61a1f-e98c-4b60-b2b1-ff56b0e37e54.png#averageHue=%23e5e5e5&clientId=ud152c9a2-a529-4&from=paste&id=u8a72837f&originHeight=932&originWidth=744&originalType=url&ratio=1&rotation=0&showTitle=false&size=248846&status=done&style=none&taskId=u22b991ec-ba99-4318-a62d-e2d0a5a41df&title=)

![image.png](https://cdn.nlark.com/yuque/0/2023/png/22935284/1696990680352-33ffe4ea-8f75-4c2f-90db-3c496b88870b.png#averageHue=%23ebebeb&clientId=ud152c9a2-a529-4&from=paste&id=u77fd6c34&originHeight=869&originWidth=803&originalType=url&ratio=1&rotation=0&showTitle=false&size=207765&status=done&style=none&taskId=uc488f83b-e67b-44a1-a220-246d26ee54f&title=)
<a name="KzD3U"></a>
## 1.3 SPD-TSE

**SAx** : Serial address inputs: Used to configure the temperature sensor/SPD EEPROM address range on the I2C bus.

**SCL/SDA** : temperature sensor/SPD EEPROM: Used to synchronize communication to and from the temperature sensor/SPD EEPROM on the I2C bus. for SPD-TSE and register

**EVENT_n** : SPD signals a thermal event has occurred

![image.png](https://cdn.nlark.com/yuque/0/2023/png/22935284/1696990695593-e0facf59-63f6-46ee-8b88-91b9fa6c28e8.png#averageHue=%23f5f5f5&clientId=ud152c9a2-a529-4&from=paste&id=u9d7069eb&originHeight=278&originWidth=806&originalType=url&ratio=1&rotation=0&showTitle=false&size=30040&status=done&style=none&taskId=u9a2b49d5-f740-4ef6-a991-b216237ebc6&title=)

SPD（serial presence detect），即串行存在检测，是DIMM的相关描述信息。在每根内存条上，都有一份SPD数据，这份数据保存在一个可擦写的eeprom芯片中。

SPD数据记录了该内存的许多重要信息，诸如内存的芯片及模组厂商、工作频率、工作电压、速度、容量、电压与行、列地址带宽等参数。SPD数据一般都是在出厂前，由DIMM制造商根据内存芯片的实际性能写入到eeprom芯片中。SPD信息最重要的作用是作为一个身份标识，让主板能够识别到它。

不同类型的DDR，他们的SPD数据长度和格式可能会不一样. 

DDR2、DDR3的DIMM，对应的SPD数据长度为256B；

DDR4的DIMM，对应的SPD数据长度为512B；

DDR5的DIMM，对应的SPD数据长度为1024B。 

![image.png](https://cdn.nlark.com/yuque/0/2023/png/22935284/1696990695606-26f9cac8-d1a2-463a-9ef4-663a343c3f1c.png#averageHue=%23f1f1f1&clientId=ud152c9a2-a529-4&from=paste&id=u4f53c6b6&originHeight=226&originWidth=806&originalType=url&ratio=1&rotation=0&showTitle=false&size=42916&status=done&style=none&taskId=ue8eb9544-ac99-4178-ad88-3d5c289a6fd&title=)<br />![image.png](https://cdn.nlark.com/yuque/0/2023/png/22935284/1696990695617-8889c066-d78b-4c5e-aebf-c41e49c90630.png#averageHue=%23f4f4f4&clientId=ud152c9a2-a529-4&from=paste&id=u6f7d4e99&originHeight=439&originWidth=802&originalType=url&ratio=1&rotation=0&showTitle=false&size=63839&status=done&style=none&taskId=u57435354-1b59-4ed7-8e5a-d5c3efcc751&title=)<br />![image.png](https://cdn.nlark.com/yuque/0/2023/png/22935284/1696990695676-6710de32-dee7-42e0-a535-a6e612bc5241.png#averageHue=%23f3f3f3&clientId=ud152c9a2-a529-4&from=paste&id=ua70c3c3c&originHeight=935&originWidth=768&originalType=url&ratio=1&rotation=0&showTitle=false&size=161818&status=done&style=none&taskId=u9825091a-d786-435e-88ac-63e38a16f50&title=)<br />![image.png](https://cdn.nlark.com/yuque/0/2023/png/22935284/1696990695639-929154cb-579a-4b9f-9cdd-15f57ee077f8.png#averageHue=%23f0f0f0&clientId=ud152c9a2-a529-4&from=paste&id=u3475fbe8&originHeight=407&originWidth=768&originalType=url&ratio=1&rotation=0&showTitle=false&size=81601&status=done&style=none&taskId=u8d4d9205-541c-4505-bc3e-be2dea9f376&title=)
<a name="ux3CM"></a>
## 1.4 结构尺寸 

![image.png](https://cdn.nlark.com/yuque/0/2023/png/22935284/1696990705995-4bb3baa3-dd48-457c-8d59-fb4a2c46cc90.png#averageHue=%23eeeeee&clientId=ud152c9a2-a529-4&from=paste&id=u1613c120&originHeight=876&originWidth=790&originalType=url&ratio=1&rotation=0&showTitle=false&size=79905&status=done&style=none&taskId=u43bc6295-c829-48d1-aaea-abd48b558a1&title=)

<a name="ztXIW"></a>
# 二 DDR5 RDIMM/LRDIMM 总结

DIMM中包括 **SDRAM, RCD, Data Buffer, SPD Hub and Temperature Sensors** 这些组件。 

![image.png](https://cdn.nlark.com/yuque/0/2023/png/22935284/1696990960375-3c27455d-14ad-42be-8d82-ceeaa9d1d049.png#averageHue=%23f1f1f1&clientId=uf1072fb8-f95f-4&from=paste&id=ubf8e879b&originHeight=669&originWidth=791&originalType=url&ratio=1&rotation=0&showTitle=false&size=120452&status=done&style=none&taskId=u4fec856a-14c3-49f3-8fd5-a17919bb452&title=)

![image.png](https://cdn.nlark.com/yuque/0/2023/png/22935284/1696990960365-4fdd3c3f-5a1a-438d-821b-7b74d0752f6f.png#averageHue=%23ececec&clientId=uf1072fb8-f95f-4&from=paste&height=315&id=u558b8fb0&originHeight=269&originWidth=571&originalType=url&ratio=1&rotation=0&showTitle=false&size=34350&status=done&style=none&taskId=ud4af7d02-d2f2-4901-a096-dbafa22e229&title=&width=668)

![image.png|650](https://cdn.nlark.com/yuque/0/2023/png/22935284/1696990988616-daa2cbcd-7e2d-4e3e-aa38-d9c3053c7eed.png#averageHue=%23b39876&clientId=uf1072fb8-f95f-4&from=paste&height=381&id=pH1W7&originHeight=594&originWidth=1024&originalType=url&ratio=1&rotation=0&showTitle=false&size=118663&status=done&style=none&taskId=u741be698-37d0-425e-ac5d-8b0801b7cbb&title=&width=657)

## 2.1 SDRAM

## 2.2 RCD

全称是寄存器时钟驱动器（Registering Clock Driver），Jedec 中以 DDR5RCD03 指代。由于信号经过了插槽，又经过了一个又一个颗粒，质量会变差，可能会减弱或丢失部分信号。所以把 C/A 等信号经过 RCD 增强后再 drive 给各个颗粒。

Package options include a 240-ball Flip-Chip Fine-Pitch BGA (FCBGA) with 0.60 mm/0.70 mm ball pitch, 14 x 19 grid. Package size is 8.70 mm x 13.50 mm as defined in MO-330 Issue A.

![image.png](https://cdn.nlark.com/yuque/0/2024/png/22935284/1705385979330-935b57b7-bdcb-4881-ab0b-2e872c281626.png#averageHue=%23f0eded&clientId=u5fcfc024-d298-4&from=paste&height=556&id=uf7f3b345&originHeight=696&originWidth=497&originalType=binary&ratio=1&rotation=0&showTitle=false&size=129173&status=done&style=none&taskId=u7ca60f50-ec8f-4a51-b344-28e82c6c1ea&title=&width=397)<br />![image.png](https://cdn.nlark.com/yuque/0/2024/png/22935284/1705386516600-3ffb00fb-99b7-4608-9135-c178a4cffd97.png#averageHue=%23e1dfdd&clientId=u5fcfc024-d298-4&from=paste&height=780&id=uf1941c78&originHeight=780&originWidth=788&originalType=binary&ratio=1&rotation=0&showTitle=false&size=145627&status=done&style=none&taskId=u55eb3823-bcb3-45b0-acc5-c34bc21709e&title=&width=788)
The DDR5RCD03 is a registering clock driver used on DDR5 RDIMMs and LRDIMMs. **Its primary function is to buffer the Command/Address (CA) bus, chip selects, and clock between the host controller and the DRAMs.** It also creates a BCOM bus which controls the data buffers for LRDIMMs.<br />It  contains  two  separate  channels  which  have  some  common  logic  such  as  clocking,  but  otherwise  operate independently of each other. Each channel has a 7-bit double data rate CA bus input, a single parity input, two chip select inputs, and produces two copies of 14-bit single data rate CA bus outputs, and two copies of the chip select outputs. The RCD has a common clock input and PLL, but produces separate clock outputs to the DRAM channels.

![image.png](https://cdn.nlark.com/yuque/0/2024/png/22935284/1705386556675-1bf2cfef-81ba-4b15-99d3-583fde427b07.png#averageHue=%23eae7e4&clientId=u5fcfc024-d298-4&from=paste&height=751&id=uafee181f&originHeight=751&originWidth=751&originalType=binary&ratio=1&rotation=0&showTitle=false&size=253179&status=done&style=none&taskId=u5e9dd1bb-39b0-462a-9fbf-1da936a0ba6&title=&width=751)<br />![image.png](https://cdn.nlark.com/yuque/0/2024/png/22935284/1705386576773-41054e07-5a75-4777-bd24-a6aaca4a4777.png#averageHue=%23eceae6&clientId=u5fcfc024-d298-4&from=paste&height=222&id=u779ebdc2&originHeight=222&originWidth=753&originalType=binary&ratio=1&rotation=0&showTitle=false&size=73103&status=done&style=none&taskId=u30f96a86-5666-4f7c-8f88-aba6d52e546&title=&width=753)<br />![image.png](https://cdn.nlark.com/yuque/0/2024/png/22935284/1705386685933-fe5214dc-1c15-4b66-80a3-197815e9265f.png#averageHue=%23cfad84&clientId=u5fcfc024-d298-4&from=paste&height=204&id=ub10cfa2e&originHeight=204&originWidth=750&originalType=binary&ratio=1&rotation=0&showTitle=false&size=69516&status=done&style=none&taskId=u52eecf90-bcfe-4ffb-86be-330c95afecf&title=&width=750)<br />![image.png](https://cdn.nlark.com/yuque/0/2024/png/22935284/1705389483378-7cfb5551-67f7-4c68-b90f-9e019d264e51.png#averageHue=%23f8f7f7&clientId=u65a2f37f-af83-4&from=paste&height=320&id=u1233d038&originHeight=320&originWidth=640&originalType=binary&ratio=1&rotation=0&showTitle=false&size=35998&status=done&style=none&taskId=ub58c61e7-ff6b-4feb-a606-bfcbaa27a92&title=&width=640)

## 2.3 Data Buffer

更好地驱动 memory，提高带负载能力。

## 2.4 SPD Hub

SPD 全称是 Serial Presence Detect。

## 2.5 Temperature Sensors

## 2.6 PMIC

## 2.7 供电总结 

![image.png](https://cdn.nlark.com/yuque/0/2023/png/22935284/1696990960383-d64bca25-f990-4104-b7db-54cfbda126dc.png#averageHue=%23e7e7e7&clientId=uf1072fb8-f95f-4&from=paste&id=SEulh&originHeight=375&originWidth=725&originalType=url&ratio=1&rotation=0&showTitle=false&size=93017&status=done&style=none&taskId=u81692086-623a-48f7-8031-5df34ef8b8d&title=)

![image.png](https://cdn.nlark.com/yuque/0/2023/png/22935284/1696990961428-417704fa-c9f9-4cf8-8991-3df3b4f9321e.png#averageHue=%23f6f6f6&clientId=uf1072fb8-f95f-4&from=paste&id=Y4wKG&originHeight=565&originWidth=754&originalType=url&ratio=1&rotation=0&showTitle=false&size=47513&status=done&style=none&taskId=ue3292690-5956-4d78-97d2-f871bd42a03&title=)

<a name="usHk2"></a>
### 2.7.1、上下电时序

![image.png](https://cdn.nlark.com/yuque/0/2023/png/22935284/1696990961544-e0f4e268-a74b-496d-9a80-3ae8be93a3a0.png#averageHue=%23f0f0f0&clientId=uf1072fb8-f95f-4&from=paste&id=BmCgJ&originHeight=691&originWidth=792&originalType=url&ratio=1&rotation=0&showTitle=false&size=107943&status=done&style=none&taskId=u9539b358-45d1-4231-b09e-25815eab514&title=)<br />![image.png](https://cdn.nlark.com/yuque/0/2023/png/22935284/1696990961549-5599eb3c-1ffa-4d04-8f03-0feb8220bf22.png#averageHue=%23f4f4f4&clientId=uf1072fb8-f95f-4&from=paste&id=X8K36&originHeight=540&originWidth=746&originalType=url&ratio=1&rotation=0&showTitle=false&size=52506&status=done&style=none&taskId=u78a6a1d7-d980-464e-9c28-cbaef58cd4a&title=)

![image.png](https://cdn.nlark.com/yuque/0/2023/png/22935284/1696990961800-f22563e3-321f-486c-8be8-2aae80dbe4ce.png#averageHue=%23f0f0f0&clientId=uf1072fb8-f95f-4&from=paste&height=582&id=QUBFC&originHeight=654&originWidth=643&originalType=url&ratio=1&rotation=0&showTitle=false&size=98525&status=done&style=none&taskId=ufca26e5f-fd2c-4319-9d8f-9db779ae87a&title=&width=572)
<a name="oh1Ag"></a>
## 2.8 数据/时钟/地址/控制信号

DQ支持 64 或 72 位数据宽度。72 位 DIMM 称为**纠错码 (ECC) DIMM**，因为除支持 64 位数据外，还支持 8 位 ECC。服务器、云和数据中心应用通常使用基于 4 个 DRAM 的 72 位 ECC DIMM，从而获得更高密度的 DIMM 并支持更高的 RAS（可靠性、可用性、可维护性）功能。同时还能缩短此类应用在存储器发生故障时的停机时间。基于其他 8 bit 和 16 bit DRAM 的 DIMM 较为便宜，所以此类产品广泛用于在台式机和笔记本电脑中。

![image.png](https://cdn.nlark.com/yuque/0/2023/png/22935284/1696990973436-43799052-f6c5-4829-98bc-c6f3f8edbb4a.png#averageHue=%23eeeeee&clientId=uf1072fb8-f95f-4&from=paste&id=CmvZS&originHeight=701&originWidth=796&originalType=url&ratio=1&rotation=0&showTitle=false&size=138117&status=done&style=none&taskId=u7f7c517b-a2fe-4e1e-805c-138ff0b1092&title=)<br />![image.png](https://cdn.nlark.com/yuque/0/2023/png/22935284/1696990973498-e9e201a6-8d33-4772-8de2-103325c83880.png#averageHue=%23e8e8e8&clientId=uf1072fb8-f95f-4&from=paste&id=tbj9g&originHeight=939&originWidth=777&originalType=url&ratio=1&rotation=0&showTitle=false&size=238647&status=done&style=none&taskId=uc1fd9a2f-7d1b-4a71-8d00-9f04396995d&title=)<br />![image.png](https://cdn.nlark.com/yuque/0/2023/png/22935284/1696990973436-fd59a03e-34db-4423-ad80-5d13cf353e2a.png#averageHue=%23ececec&clientId=uf1072fb8-f95f-4&from=paste&id=u9wVb&originHeight=291&originWidth=775&originalType=url&ratio=1&rotation=0&showTitle=false&size=64197&status=done&style=none&taskId=ua2fbfcdd-81e5-4a9f-a38b-c218f34a0c7&title=)<br />![image.png](https://cdn.nlark.com/yuque/0/2023/png/22935284/1696990973492-f846011c-bf84-4c47-885f-943e497a59c3.png#averageHue=%23f4f4f4&clientId=uf1072fb8-f95f-4&from=paste&id=HCP3J&originHeight=942&originWidth=728&originalType=url&ratio=1&rotation=0&showTitle=false&size=147221&status=done&style=none&taskId=u6cd2460a-ef59-450e-982f-229bcd9fff9&title=)<br />![image.png](https://cdn.nlark.com/yuque/0/2023/png/22935284/1696990973489-75968728-8e09-4b61-b678-532ff56cb0aa.png#averageHue=%23f3f3f3&clientId=uf1072fb8-f95f-4&from=paste&id=m2tpT&originHeight=945&originWidth=722&originalType=url&ratio=1&rotation=0&showTitle=false&size=157608&status=done&style=none&taskId=u20e4eeee-757d-4efe-a5c6-760d455db6f&title=)

![image.png](https://cdn.nlark.com/yuque/0/2023/png/22935284/1696990974567-2071da02-6eca-41c6-92ea-bbe58e3be0b6.png#averageHue=%23f8f8f8&clientId=uf1072fb8-f95f-4&from=paste&id=XOBIX&originHeight=462&originWidth=787&originalType=url&ratio=1&rotation=0&showTitle=false&size=51769&status=done&style=none&taskId=u5486635e-bb3e-409e-aecc-c8fbf8d9a00&title=)<br />![image.png](https://cdn.nlark.com/yuque/0/2023/png/22935284/1696990975006-66290fcd-da63-429d-8e5d-229705700b23.png#averageHue=%23f7f7f7&clientId=uf1072fb8-f95f-4&from=paste&id=j2Ls1&originHeight=220&originWidth=788&originalType=url&ratio=1&rotation=0&showTitle=false&size=25475&status=done&style=none&taskId=u09cf2ac5-17db-4c0b-a24f-4698165bbd0&title=)

<a name="E5pDl"></a>
## 2.9、结构尺寸 

![image.png](https://cdn.nlark.com/yuque/0/2023/png/22935284/1696990988587-d37df24f-6154-4f2c-bbb6-734ef1a1da5d.png#averageHue=%23f6f0f0&clientId=uf1072fb8-f95f-4&from=paste&id=ua38bc830&originHeight=851&originWidth=388&originalType=url&ratio=1&rotation=0&showTitle=false&size=61686&status=done&style=none&taskId=u7f476a49-7cb6-422c-ac4b-eac52237e99&title=)

![image.png](https://cdn.nlark.com/yuque/0/2023/png/22935284/1696990988587-75bb6d7f-13e4-4ba7-aa61-7c1f879f4423.png#averageHue=%23f7f7f7&clientId=uf1072fb8-f95f-4&from=paste&id=u22cfd8bc&originHeight=394&originWidth=774&originalType=url&ratio=1&rotation=0&showTitle=false&size=28730&status=done&style=none&taskId=u4d38f6cd-ed0e-46e0-a2db-7684b245833&title=)

![image.png](https://cdn.nlark.com/yuque/0/2023/png/22935284/1696990988606-ffce8109-3040-4841-92c3-d064b087d90c.png#averageHue=%23e3e3e3&clientId=uf1072fb8-f95f-4&from=paste&id=ubdab2290&originHeight=600&originWidth=514&originalType=url&ratio=1&rotation=0&showTitle=false&size=61101&status=done&style=none&taskId=u58f03b8c-7438-4a3d-9bd5-d729ccadc32&title=)

<a name="lKzz1"></a>
# 三 DDR4 UDIMM 总结

Unbuffered Dual Inline Memory Module。主要用在PC上。 
<a name="o2eEG"></a>
## 3.1 供电总结 

<a name="xzrLg"></a>
## 3.2 数据/时钟/地址/控制信号总结 


<a name="OVdx1"></a>
# 四 DDR5 UDIMM 总结
<a name="Gcwv3"></a>
## 4.1 供电总结 

<a name="TxX1e"></a>
## 4.2 数据/时钟/地址/控制信号总结 


<a name="Zy4GF"></a>
## 4.3 PMIC
<a name="eMznI"></a>
### 4.3.1 RTQ5136 介绍
RTQ5136 是为 DDR5 SODIMM 和 UDIMM 开发的集成 PMIC ，特性如下：

      - VIN_Bulk 输入电压范围：4.25-5.5V，在 DIMM 上连接到 5V 电源网络；另外还有一个 VIN 的 5V 输入引脚；
      - 包括 3 个单相 converters（4A/4A/1A，SWA/SWB/SWC） 和 2 个 LDO（25mA VLDO_1.8V，20mA VLDO_1.0V）；
         - SWA/SWB/SWC 可以分别给 VDD-1.1V、VDDQ-1.1V 和 VPP(Charge Pump Power)-1.8V
         - VLDO_1.8V for SPD5/HUB & TS power；1.0V 给 SPD5/HUB & CK Buffer；
      - 支持 I2C/I3C Slave 控制；
      - 集成 DDR VR on DIMM Platform 顺序控制；
      - VR_EN 引脚用来控制电源开关，高电平打开，不能悬空。

<a name="RGtoQ"></a>
### 4.3.2 RTQ5132 介绍

RTQ5132 是为 DDR5 SODIMM 和 UDIMM 开发的集成 PMIC ，特性如下：

      - VIN_Bulk 输入电压范围：4.25-5.5V；
      - 包括 1 个可配置的双相或单相 converter SWA&SWB，1 个单相 converter SWC 和 2 个 LDO（25mA VLDO_1.8V，20mA VLDO_1.0V）；

<a name="AfQvo"></a>
# 五 SODIMM 总结

Small Outline DIMM。主要是笔记本电脑中使用，只有颗粒，面积比较小。<br />Mini-DIMM：DDR2时代新出现的模组类型，它是Registered DIMM的缩小版本，用于刀片式服务器等对体积要求苛刻的高端领域。 
<a name="rIlGz"></a>
# 六 DIMM 测试 

![image.png](https://cdn.nlark.com/yuque/0/2023/png/22935284/1696990988639-a1a4b221-8c8f-4f0c-9c67-63cb8b066296.png#averageHue=%23fcfcfc&clientId=uf1072fb8-f95f-4&from=paste&id=ua1089035&originHeight=464&originWidth=781&originalType=url&ratio=1&rotation=0&showTitle=false&size=110352&status=done&style=none&taskId=u8b977dd9-5aad-4a11-b11b-d3123a6fe53&title=)

注意：板上加SMA测试头时，需要另外增加如下图所示的SMA互联，用于解嵌，测试表层损耗。 

![image.png](https://cdn.nlark.com/yuque/0/2023/png/22935284/1696990989591-522c643a-eeed-4f00-871f-b90db5f77631.png#averageHue=%23f8f8f8&clientId=uf1072fb8-f95f-4&from=paste&id=ud55939f0&originHeight=446&originWidth=777&originalType=url&ratio=1&rotation=0&showTitle=false&size=104500&status=done&style=none&taskId=uac91aa86-3dd0-4d32-9a48-3e22cccea61&title=)

![image.png](https://cdn.nlark.com/yuque/0/2023/png/22935284/1696990989648-7b7e0318-dda7-4f29-a8d6-5310aa802802.png#averageHue=%23f5f5f5&clientId=uf1072fb8-f95f-4&from=paste&id=u2a1af9e8&originHeight=552&originWidth=689&originalType=url&ratio=1&rotation=0&showTitle=false&size=105493&status=done&style=none&taskId=u85e10fa2-606a-4f11-8f71-675c52355f2&title=)


