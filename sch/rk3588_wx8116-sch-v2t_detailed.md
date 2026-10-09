# RK3588 + WX8116 原理图详细电气连接说明

> 源文件：`sch/rk3588_wx8116-sch-v2t.pdf`
>
> 源文件 SHA：`32068ef1a075b5196d98ac0bc5214aa81c2212cb`
>
> 文件大小：1,370,947 bytes；PDF 共 34 页。
>
> 本文依据 PDF 逐页阅读结果整理，重点记录**器件、引脚、网络名、上下游器件、电源和接口之间的电气连接**。PDF 是原理图图形文件而不是 KiCad/Altium/OrCAD 原始工程，因此本文不能替代 CAD 原生 netlist；对 PDF 中无法可靠辨认的极小文字不做猜测。

## 0. 总体结构

这份 PDF 实际是一个较大的复合原理图资料包，包含 RK3588/WX8116 主系统、WX8128/WX5124/PHY 相关参考设计、USB/PCIe/以太网/SFP、电源、CPLD、SATA/串口扩展等页面。页面标题和工程页码并不完全一致，因此分析时以 **PDF 页码 1~34** 为准。

核心电气层次可归纳为：

`12V 输入 → 5V / 3V3 主电源 → RK3588/SOM + WX8116/PHY + 外设`；

`RK3588/SOM → HDMI / USB / M.2 PCIe / eMMC / Micro-SD / GPIO / 调试 UART`；

`WX8116 → 8GE/QSGMII/SGMII SerDes → PHY/磁性器件/RJ45/SFP`；

`DDR3 → RK/WX 主控 DDR 接口 + 0.75V VTT + 1.5V IO`；

`CPLD → PHY reset/link LED/watchdog/状态信号`；

`SATA PM → SATA TX/RX + 1.2V/3.3V/5V 电源 + RS232/RS485 扩展`。

---

# 1. 电源网络总表

## 1.1 12V 主输入

主要网络：`VDD_12V_MAIN`、`VDD_12V`、`VDD12V_1`、`VDD12V_2`。

关键连接：

- `VDD_12V_MAIN` 是板级 DC/DC 主输入。
- 页面 9：`VDD_12V_MAIN → U137 MP1497DJ → L27 → VDD_5V_MAIN`，输出约 5V，设计值 4.992V，标注最大约 3A。
- 页面 9：`VDD_12V_MAIN → U139 MP1497DJ → L29 → VDD_3V3_MAIN`，输出约 3.3V，标注最大约 3A。
- 页面 31：`VDD12V_1`、`VDD12V_2` 分别进入两路输入滤波/保护，再产生 `power1_in`、`power2_in`。
- 页面 5：`VDD_12V` 经 Q23/Q20 等电源路径进入 `VDD_12V_MAIN`/`VCC_CAP` 等节点，并带电源检测、复位和保护逻辑。

## 1.2 5V

`VDD_5V_MAIN` 主要供给：

- HDMI +5V：`VDD_5V_HDMI`
- USB HUB / USB Host / USB-C VBUS
- RTC/部分外设
- SATA 电源路径
- 其它局部 LDO/DC-DC 输入

页面 9 的 12V→5V 主电源链：

`VDD_12V_MAIN → U137(MP1497DJ) VIN → SW → L27 → VDD_5V_MAIN`。

U137 周边：`C996/C997/C998` 输入去耦，`R867/R870` EN/SS，`R865/R868` FB 分压，`R515/R516` 等栅/补偿网络，`C999/C1000/C994` 输出滤波。

## 1.3 3.3V 主电源

页面 9：

`VDD_12V_MAIN → U139(MP1497DJ) → L29 → VDD_3V3_MAIN`。

输出反馈网络包含 `R873/R877/R879` 等，输出端有 `C1014/C1022/C1015` 等去耦。

`VDD_3V3_MAIN` 是全板最重要的数字 IO 电源之一，连接到：RK3588/SOM、CPLD、USB、HDMI 控制、按键、LED、RTC、调试、PHY 控制、M.2 控制逻辑等。

## 1.4 RK/WX 核心电源

主要网络：

- `VDD_1V1` / `VDD_1V1_KD5128`：核心 1.1V
- `VDD_2V5`：模拟/SerDes 2.5V
- `VDD_3V3_KD5128`：WX/PHY 系列 3.3V
- `VDDIO_3V3`
- `VDDHV_LDO`
- `VDDHV_TX_ABCD`
- `VDDLV_GPHY`
- `VDDPOST_PLL`
- `AVDL_1V1`
- `SERDES_AVDH`

页面 12 对 WX8116 大量 VCCK/VDDIO/VDDHV/GND 引脚进行集中去耦。原则上：

`VDD_1V1_KD5128 → 磁珠/大容量 + 100nF 阵列 → WX8116 VCCK`；

`VDD_3V3_KD5128 → 分组磁珠 → VDDIO_3V3 / VDDHV_LDO / SERDES_AVDH 等`；

各高速模拟/PLL 电源均采用独立磁珠 + 10uF/1uF/100nF 分级去耦。

---

# 2. RK3588/SOM 连接

## 2.1 SOM 电源和管理

页面 4 的 SOM CONNECTOR：

- `VDD_DCIN` / `VDD_12V_MAIN`：SOM 电源输入。
- `PWRON_L`
- `PMIC_EXT_EN_OUT`
- `RESET_L`
- `VDC_MODE`
- `BOOT_SARADCIN0`
- `UART2_TX_M0_DEBUG` / `UART2_RX_M0_DEBUG`
- `GPIO2_C1/MIPI_CAM2_RESET_L`
- `GPIO2_C2/MIPI_CAM2_PDN_L`
- MIPI CSI0/CSI1 差分对
- PCIe2.0/PCIe3.0 差分对
- GPIO / UART / I2C / SPI / USB 控制信号

页面 7/4 的 CON1B/CON1D 进一步给出 RK3588 SOM 到外设的完整端口名，包括：

- `TYPEC0/TYPEC1` USB3 TX/RX/SBU
- `USB20_HOST0/1_DP/DM`
- `HDMI0/HDMI1_TX*`
- `HDMI_RX_*`
- `SDMMC0_D[0..3]`、`SDMMC0_CMD`、`SD_CLK`
- `PCIE20_1_TX/RX/REFCLK`
- `PCIE30_PORT0/1_TX/RX/REFCLK`
- `GPIO4_B*` M.2 reset/CLKREQ 等

## 2.2 eMMC

页面 6：eMMC 使用 `FEMDRW016G-88A43` 类 16GB 器件。

连接关系：

- RK/WX 主控 `eMMC_DATA0..3` → `R107/R108/R109/R110 22R` → eMMC DAT0..DAT3。
- `eMMC_CMD` → `R125 22R` → eMMC CMD。
- `eMMC_CLK` → eMMC CLK。
- `EMMC_RESETn` → eMMC RST_n。
- `eMMC_VCCQ` 经 `FB22/FB23 BLM18EG101TN1D` 等磁珠供电。
- eMMC VCC/VCCQ 周边有 `10uF + 100nF` 去耦阵列。
- `EMMC_RESETn` 由 `R106 10K` 上拉到 `VDD_3V3`。
- 页面注释明确：`INT[1:0] is must`；`INT2 is reserved`；`SPI0_CS1 TO CPLD`；`SPI0_CS2/CS3 reserved`；`UART1 USE TO POE`。

## 2.3 SPI NOR Flash

页面 6：`GD25B256DFIGR / GD25Q256D`。

主要信号：

- `Flash_CS0`
- `Flash_CLK`
- `Flash_MOSI`
- `Flash_MISO`
- `nRESET_FLASH`

RK/WX 主控到 Flash 之间使用约 `22R` 串联电阻；Flash 供电为 `VDD_3V3`，并有 `10K` 类上拉/复位网络。

## 2.4 DDR3

页面 8：`U64` DDR3 与 `U7B`/主控 DDR 接口。

地址线：

`DDR3_A0..A15` ↔ `DDR_ADDR[0..15]`。

Bank：

`DDR3_BA0..BA2` ↔ `DDR_BA[0..2]`。

控制：

- `DDR3_RESETN`
- `DDR3_CS_N`
- `DDR3_CAS_N`
- `DDR3_RAS_N`
- `DDR3_WE_N`
- `DDR3_CKE`
- `DDR3_ODT`
- `DDR3_CK_P/N`

数据：

`DDR3_DQ0..DQ15` ↔ `DDR_DQ[0..15]`。

DQS：

`DDR3_DQS_0P/N`、`DDR3_DQS_1P/N`。

DM：

`DDR3_DM0/DM1`。

终端/参考：

- `DDR3_ZQ` → 约 `240R`。
- `DDR3_ODT` 等控制线存在 `4.7K` 类偏置。
- `DDR3_CK_P/N` 经 `R118/R119 40.2R` 等端接。
- 地址/控制组可看到大量 `40.2R` NC/可选串阻。

DDR 电源：

- `DDR_VDDQ`
- `VCC15IO_DDR`
- `DDR_VTT`
- `0V75-DDR_1`
- `0V75-DDR_2`
- `VDDA3V3_LDO_DDRPLL`

`U29 TPS51200DRCR/TPL51200` 负责 DDR VTT，输入 `DDR_VDDQ`，输出 `DDR_VTT`，并有 `REFIN/REFOUT/VOSNS/PGOOD/EN` 等连接；VTT 输出有多颗 22uF/100nF 去耦。

## 2.5 HDMI OUT

页面 1：

RK3588 HDMI0 差分信号首先经过两组 `RCLAMP0524P`：

- `HDMI0_TX2P_PORT/N`
- `HDMI0_TX0P_PORT/N`
- `HDMI0_TX3P_PORT/N`
- `HDMI0_TX1P_PORT/N`

分别进入 `U174/U175` 后形成：

- `HDMI0_HTX2P/N`
- `HDMI0_HTX0P/N`
- `HDMI0_HTX3P/N`
- `HDMI0_HTX1P/N`

然后进入 HDMI 连接器 `CON15`。

DDC/CEC：

- `HDMI0_SCL` / `HDMI0_SDA` → `R1081/R1082 4.7K` 上拉，并经 Q24/Q25 电平转换。
- `HDMI0_CEC` → Q12 等开关/电平网络。
- `HDMI0_HPD1` → Q10/Q11/Q12 等检测/控制路径。
- HDMI 连接器 `+5V` 使用 `VDD_5V_HDMI`。
- `R1074/R1075 4.7K`、`R1076/R1077 10K` 等构成 HPD/DDC 相关偏置。

页面 4/7 还给出 `HDMI1_TX*`、`HDMITX1_SCL/SDA/CEC`、`HDMI1_TX_ON_H` 等 SOM 引脚。

---

# 3. WX8116 / 以太网高速部分

## 3.1 WX8116 GMAC

页面 10/17：WX8116/WX5124 的 MAC 侧信号包括：

- `R0_RXD[0..3]`
- `R0_RX_CTL`
- `R0_RX_CLK`
- `R0_TXD[0..3]`
- `R0_TX_CTL`
- `R0_TX_CLK`
- `R1_RXD[0..3]`
- `R1_RX_CTL`
- `R1_RX_CLK`
- `R1_TXD[0..3]`
- `R1_TX_CTL`
- `R1_TX_CLK`
- `MII_MDC`
- `MII_MDIO`
- `PHY_INT0/1`
- `PHY_RST_0/1`
- `PHY_25M_0/1`

MAC→PHY 控制/数据线上使用 `22R` 串阻；PHY 地址相关使用 `10K`；参考时钟使用 `0R/200R` 等配置。

## 3.2 WX8116 SerDes

WX8116 的 SerDes 通过 `SG0..SG3` 与 QSGMII/SERDES 信号连接。

主要网络：

- `S0_Q2_TXP/N`
- `S0_S0_TXP/N`
- `S1_QSGMII0_TXP/N`
- `S1_QSGMII1_TXP/N`
- `S0_QSGMII0_TXP/N`
- `S0_QSGMII1_TXP/N`
- 对应 RX 对

页面 10 给出 SerDes0~7 的 AC 耦合电容，典型为 `100nF_25V`；页面 17 给出另一组 `10nF_25V` AC 耦合。

## 3.3 PHY 电源

WX8116/WX5124 的 PHY 模块使用：

- `PHY_3V3`
- `VDD_2V5`
- `VDD_1V1`
- `VCC25A_SERDES*`
- `VCC11A_SERDES*`
- `VCC33A_GPHY*`
- `VCC11A_GPHY*`
- `VCC25A_PLL`
- `VCC11K_PLL`

每组模拟电源采用磁珠隔离，例如 `BLM15AG102SN1D` / `BLM18EG101TN1D`，之后配置 22uF、10uF、1uF、100nF 等多级去耦。

## 3.4 RJ45/磁性器件

页面 19/26：多组 `HST-72021DXR` 磁性器件连接 PHY 的 8GE 差分对。

典型链路：

`WX/WX PHY RTxNGEAP/AN/BP/BN/CP/CN/DP/DN → 75R 串/端接 → HST-72021DXR → GE_*_P/N → RJ45`。

每组中心抽头/差分线配有：

- `75R_1%` 端接电阻
- `100nF` 电容
- `1nF/5kV` 隔离电容
- `SRV05-4` TVS 阵列

TVS 的 IO1/IO2/IO3/IO4 接对应四对差分线，GND/REF 接保护参考地。

## 3.5 SFP

页面 21：SFP2/SFP3。

SFP2：

- `SFP2_TX+/-`
- `SFP2_RX+/-`
- `SFP2_SDA`
- `SFP2_SCL`
- `SFP2_ONLINE`
- `SFP2_LOS`
- `FX2_TX_DISABLE`

SFP3：

- `SFP3_TX+/-`
- `SFP3_RX+/-`
- `SFP3_SDA`
- `SFP3_SCL`
- `SFP3_ONLINE`
- `SFP3_LOS`
- `FX3_TX_DISABLE`

SFP 电源使用 `VDD_3V3 → 磁珠 → FX*_VCCT/VCCR`，每路配 `22uF + 100nF`。

I2C：

`I2C1_SDA/SCL → U6 PCA9543ADR-SOIC14 → SFP2_SDA/SCL + SFP3_SDA/SCL`。

U6 地址标注为 `0x72`，主 I2C 经 `33R` 电阻连接。

---

# 4. CPLD / 系统管理

页面 16：CPLD `U2 EF2L15LG100B`。

主要连接：

- `CPLD_TDI/TDO/TMS/TCK`：JTAG
- `CPLD_3V3`：供电
- `CPLD_WDI`：看门狗输入
- `CPLD_CLKIN_25M` / `CPLD_25M`
- `MASTER0_CLK_25M`
- `EXT_INTR`
- `WDT_RST`
- `nRESET_FLASH`
- `PHY_INT0/PHY_INT1`
- `PHY_RST_0/PHY_RST_1`
- `P0_LINK..P11_LINK`
- `P0_SPEED..P11_SPEED`
- `NVME1_LED..NVME4_LED`
- `LED_LINKACT0..3_CPLD`
- `RK3588_RUN`
- `Hi3516_RUN`
- `RK3588_ST1/ST2`

CPLD JTAG 使用 `4.7K` 类上拉；大量外部 IO 通过 `22R/100R/330R` 等串阻连接。

LED 驱动：

- `P*_LINK/P*_SPEED` → `1K` 限流 → `VDD_3V3_LED`
- 状态灯使用 `XL-DZ304SYGD/4` 或 `19-217/G7C-AN1P2/3T` 类器件。

---

# 5. USB

## 5.1 USB UART 调试

页面 13：

`CH340_TXD/RXD` ↔ Q16/Q19 电平转换 ↔ `CH340_TXD_3V3/CH340_RXD_3V3` ↔ `U144 CH340E`。

USB：

`U144 UD+/UD- → R891/R892 22R → MICRO_USB_DP/DM`。

USB 连接器：`CON2 KH-TYPE-C-16P`。

CC：

- `CC1 → R893 5.1K`
- `CC2 → R894 5.1K`

USB ESD 使用 `U145 LC0504F`。

UART 选择：

`U18 CH442E` 根据 `UART_SEL` 在 `UART0` 与 `UART2 DEBUG` 路径之间切换。

## 5.2 USB2 HUB

页面 18：`U149` USB HUB，连接：

- `USB20_HOST0_DP/DM`
- `HUB_USB1_DP/DM`
- `HUB_USB2_DP/DM`
- `HUB_USB3_DP/DM`
- `HUB_USB4_DP/DM`

Hub 电源：`VDD_3V3_MAIN`，输出/USB 侧另有 `VDD_5V_HUB`。

USB 高速线按照约 90Ω 差分阻抗布线，并在接口附近使用 ESD/共模/滤波器。

## 5.3 USB3 Host

页面 20：两路 Host。

Host1：

`TYPEC1_OTG_DP/DM`、`TYPEC1_SSRX1P/N`、`TYPEC1_SSTX1P/N` → `U157 LC0504F` → `CON5`。

Host2：

`USB20_HOST1_DP/DM`、`USB2_HUB2_DP/DM`、`USB2_HUB2_RXP/N`、`USB2_HUB2_TXP/N` → `U159 LC0504F` → `CON6`。

两路 VBUS 分别由：

- `U155 SY6280AAC → VDD_5V_USB1`
- `U156 SY6280AAC → VDD_5V_USB2`

控制：

- `GPIO4_B0/USB3_TYPEC1_PWREN`
- `GPIO3_A5/USB3_2_PWREN`

---

# 6. USB-C OTG

页面 18：

`VDD_5V_MAIN → U150 SY6280AAC → VDD_5V_VBUS`。

USB2：

`TYPEC0_OTG_DP/DM → TYPEC_DP/TYPEC_DN`。

USB3：

- `TYPEC0_SSTX1P/N`
- `TYPEC0_SSRX1P/N`
- `TYPEC0_SSTX2P/N`
- `TYPEC0_SSRX2P/N`

CC 控制：

`U30 FUSB302MPX` 连接 `TYPEC_CC1/CC2`，由 `I2C6_SCL_M0/I2C6_SDA_M0` 控制，并输出 `GPIO0_D3/CC_INT_L`。

ESD：

- `U151 RCLAMP0524P`：USB-C 高速差分保护
- `U153/U154 RCLAMP0524P`：其它高速对保护

VBUS 有 `22uF/100nF` 去耦，CC 引脚有 5.1K/10K 等配置。

---

# 7. Micro-SD

页面 15：`CON3` Micro-SD。

RK3588/SOM → SD：

- `SDMMC0_D0 → SD_D0`
- `SDMMC0_D1 → SD_D1`
- `SDMMC0_D2 → SD_D2`
- `SDMMC0_D3 → SD_D3`
- `SDMMC0_CMD → SD_CMD`
- `SD_CLK → SD_CLK`
- `GPIO0_A4/SDMMC_DET_L → SD_DET`

供电：`VCC3V3_SD_S0`。

上拉：`R903 10K`，其余 `R904..R909` 标注 DNP，可选数据线/命令线偏置。

ESD：`U147/U148 LC0504F`。

---

# 8. M.2 4G/5G

页面 24：`CON9 M.2 KEY B`。

PCIe：

- `PCIE20_1_RXP/N`
- `PCIE20_1_TXP/N`
- `PCIE20_1_REFCLKP/N`
- `PCIE20X1_2_PERSTN_M0`
- `PCIE20X1_2_CLKREQN_M0`
- `PCIE20X1_2_WAKEN_M0`

USB：

- `HUB_USB2_DP/DM`

SIM：

- `SIM1_RST`
- `SIM1_CLK`
- `SIM1_DAT`
- `VDD_SIM1`

电源：

`VDD_5V_MAIN → U167 MIC29302S/TR → VDD_3V7_KEYB`，输出约 3.68V。

SIM 保护/滤波使用 `U166 LC0504F`。

图中明确标注 PCIe3.0/PCIe2.0 差分阻抗要求和 5G 模块上电时序要求。

---

# 9. M.2 NVMe

页面 27/28 有 4 组 M.2 KEY-M：`CON11/CON12/CON13/CON14`。

## KEY-M1

- `PCIE30_REFCLK_A_P/N`
- `PCIE30_PORT0_TX0P/N`
- `PCIE30_PORT0_RX0P/N`
- `GPIO4_B5/M2_A_CLKREQ_L`
- `GPIO4_B6/M2_A_PERST_L`
- `PMIC_EXT_EN_OUT`
- `NVME1_LED`
- `VDD_3V3_KEYM1`

3.3V 电源：`VDD_12V_MAIN → U170 SY8113IADC → VDD_3V3_KEYM1`。

## KEY-M2

- `PCIE30_REFCLK_B_P/N`
- `PCIE30_PORT0_TX1P/N`
- `PCIE30_PORT0_RX1P/N`
- `GPIO4_B4/M2_B_PERST_L`
- `NVME2_LED`
- `VDD_3V3_KEYM2`

电源：`U171 SY8113IADC → VDD_3V3_KEYM2`。

## KEY-M3

- `PCIE30_REFCLK_C_P/N`
- `PCIE30_PORT1_TX2P/N`
- `PCIE30_PORT1_RX2P/N`
- `GPIO4_B3/M2_C_PERST_L`
- `NVME3_LED`
- `VDD_3V3_KEYM3`

电源：`U172 SY8113IADC → VDD_3V3_KEYM3`。

## KEY-M4

- `PCIE30_REFCLK_D_P/N`
- `PCIE30_PORT1_TX3P/N`
- `PCIE30_PORT1_RX3P/N`
- `GPIO4_A2/M2_D_PERST_L`
- `NVME4_LED`
- `VDD_3V3_KEYM4`

电源：`U173 SY8113IADC → VDD_3V3_KEYM4`。

### PCIe 重要设计备注

原图明确指出：当前三组控制信号设计**不满足 PCIe3.0 x2 要求**。要求：

- `PCIE30X2_CLKREQn`
- `PCIE30X2_WAKEn`
- `PCIE30X2_PERSTn`
- `PCIE30X1_CLKREQn`
- `PCIE30X1_WAKEn`
- `PCIE30X1_PERSTn`

必须按相同 `_M0/_M1/_M2` 功能组配置，不能随意用 GPIO 替代 CLKREQ/WAKEn；PERSTn 在特定模式可以由 GPIO 替代，但若作为功能引脚必须与对应 CLKREQ/WAKEn 保持相同 Mx 组。

原图还特别注明 `R429` 位置错误，应为 `B19/PCIE30X1_WAKEn_M2/3V3` 的上拉电阻。这属于应保留在硬件审查清单中的明确设计问题。

---

# 10. 100MHz PCIe 时钟

页面 25：两颗 `PI6C557-05BLEX`：`U168/U169`。

输入：

- `P0_XI/P0_XO`
- `P1_XI/P1_XO`
- 外部 `X322525MSB4SI`

输出：

U168：

- `PCIE30_REFCLK_A_P/N`
- `PCIE30_REFCLK_B_P/N`
- `PCIE30_PORT0_REFCLKP/N_IN`

U169：

- `PCIE30_REFCLK_C_P/N`
- `PCIE30_REFCLK_D_P/N`
- `PCIE30_PORT1_REFCLKP/N_IN`

HCSL 输出端有 `49.9R` 电阻；差分线上有 `33R` 串阻；电源通过 `FB70/FB71` 与 `VDD_3V3_CLKA/B` 隔离。

图中标注 PCIe3.0 时钟差分阻抗约 `100Ω ±10%`。

---

# 11. WiFi / Bluetooth / GPS

页面 23：

## GPS

`U164 AT2659`：

- RF：`RFIN → L31 → 天线网络`
- `RFOUT → L32 → GPS 模块 ANT-RF`
- `VCC/VCC_RF`
- `SDA/SCL`
- `RXD/TXD`
- `VBAT → VBAT_BD`
- `ON/OFF`
- `1PPS`

GPS 模块 `U163 ATGM336H-5N31` 连接 `VDD_3V3_MOD`、`PER_RESET`、`HUB_USB1_DP/DM`、`VBAT_BD`、UART/I2C 和天线。

## WiFi

`VDD_3V3_MAIN → FB68 → VDD_3V3_WIFI`。

WiFi 模块 `U163/BL-R8188EU2`：

- `HUB_USB1_DP/DM`
- `VDD_3V3`
- `ANT-RF`

天线端：`R957 0R`，预留 `L33/L34` 5.6nH，`D62 PESD2442U005` ESD。

## 模块电源

`U165` 从 `VDD_5V_MAIN` 产生 `VDD_3V3_MOD`，输入输出均配置多颗 100nF/10uF。

---

# 12. RTC / 温度 / Watchdog

页面 14：

## RTC

`U36 GM8563ESA_NC`：

- `Y2 X321532768KGD2SI`
- `C627/C628 12.5pF`
- `I2C1_SCL/SDA`
- `RTC_INT`
- `VDD_3V3`
- 电池 `BAT3`

RTC 地址：写 `A2h`，读 `A3h`。

## 温度

`U38 SD5075-SOP8-50-157_NC`：

- `I2C1_SDA`
- `I2C1_SCL`
- `ALARM → SD5075_INT0`
- 地址 `0x48`
- `VDD_3V3`

## Watchdog

`U3 SGM706B-TXS8G/TR`：

- `MR ← SYS_MR`
- `WDI ← CPLD_WDI`
- `WDO → WATCHDOG_RST`
- `PFO/PFI` 电源监测
- `PER_RESET` 作为系统复位链的一部分

页面 11 的 `U142 TP74LVC1G07C5` 将 `RESET_L` 转换为 `PER_RESET`，并由 `R880 10K` 上拉。

---

# 13. 按键 / 启动 / LED

页面 11：

- `PWRON_L`：SOM 电源开关逻辑。
- `RECOVERY`：恢复键输入，带 `C1027 1uF`、`D58`。
- `BOOT_SARADCIN0`：启动 ADC 输入，串 `R882 100R`，并由 `D57` 保护。
- `GPIO3_B7`：按键/状态相关。
- `UART_SEL`：调试 UART 选择。

电源键 `SW2` 接 `PWRON_L`，并配 `D55 DW03D-B-S` 和 `C1024 10nF`。

---

# 14. 风扇

页面 15：

`VDD_12V_MAIN → FB63 → Q4 CJ3401 → J26/J55 风扇电源`。

控制：

`GPIO3_A7/PWM8_M0 → R914 100R → Q5 CJ3400 → Q4 gate`。

偏置：`R912 1K`、`R913 20K`、`R1187 1K` 等。

因此风扇是低侧 MOS 控制/开关结构，12V 通过 FB63 后进入负载。

---

# 15. 12V 输入保护/电源检测/复位

页面 5：

- `Q23 NCE60P10K`、`Q20 AO3401A`：电源路径 MOS。
- `U187/U183 BCM857BS`：电源检测/控制。
- `U186 LMV331W5-7`：`V_DECT` 电压比较。
- `U185 TPS3808G01DBVR-TP`：`RESET` 监控。
- `U13 CJ7805`：`VCC_COM_5V` 线性稳压。
- `D68 BAT54C`、`D69/D70 RB521S30`：电源 OR/隔离/复位路径。
- `RESET → R1173 0R → RESET_P`。

这一级把输入电源状态与系统 `RESET/RESET_P/PER_RESET` 连接起来。

---

# 16. SATA / 存储扩展

PDF 第 34 页为 Rockchip `P28_SATA` 页面，属于较独立的 SATA/控制扩展。

主要高速信号：

- `SATA30_2_TXP/N`
- `SATA30_2_RXP/N`
- `SATAPM_TXP0/N0`
- `SATAPM_RXP0/N0`
- `SATAPM_TXP1/N1`
- `SATAPM_RXP1/N1`
- `SATAPM_TXP2/N2`
- `SATAPM_RXP2/N2`
- `SATAPM_TXP3/N3`
- `SATAPM_RXP3/N3`

SATA PM 控制器 `U8300 JMB575` 周边：

- `VDD1V2_SATAPM`
- `VCC3V3_SATAPM`
- `SATAPM_REXT`
- `SATAPM_RSTn`
- `SATAPM_GPIO0/1/2/3/4/5`
- `SATAPM_TESTn`
- `ZGPIO*`

SPI Flash：

`U8301 W25X40CLNIG/SOIC-8`，连接 `SATAPM_GPIO14/2/3/4/0/1` 等控制信号。

SATA PM 1.2V：

`VDD_5V_MAIN → U8303 SY8089AAC / DC-DC → VDD1V2_SATAPM`。

SATA PM 5V：

`VDD_12V_MAIN → U8 MP1497DJ-LF-Z → L40 → VDD_5V_SATA`，输出设计值约 4.992V。

SATA 差分对均串接 `10nF` AC 耦合电容，例如 `C8318/C8321/C8322/C8323` 等。

---

# 17. RS232 / RS485

PDF 第 34 页：

## RS485 UART0

`MCU3_RXD/TXD → U21 SIT3485ISO → RS485_A0/RS485_B0`。

接口保护：

- `D10 P6SMB6.8CA`
- `R8332/R8333 10R`
- `R129 120R` 终端

## RS485 UART1

`MCU4_RXD/TXD → U8305 SIT3485ISO → RS485_A1/RS485_B1`。

对应也有 `120R` 终端、10R 串阻和 `P6SMB6.8CA` TVS。

## RS232

`U22 SP3232EEA`：

- `MCU1_TXD/RXD → RS232_TX1/RX1`
- `MCU2_TXD/RXD → RS232_TX2/RX2`

U22 周边使用 `10uF/100nF` 电荷泵/去耦电容。

## 隔离

`U8304 JMB575`/SATA PM 相关电源与 `VDD_5V_ISO`、`GNDI` 构成隔离电源域；`B0505S-1WR3L` 提供 5V 隔离电源。

---

# 18. 每页功能索引

| PDF页 | 主要内容 | 关键网络/器件 |
|---:|---|---|
| 1 | HDMI OUT | HDMI0_TX*, HPD, CEC, SCL/SDA, U174/U175, CON15 |
| 2 | 系统框图/标题 | WX8128、25MHz、DDR3、eMMC、SerDes |
| 3 | WX5124/PHY | GMAC、SerDes、PHY 电源、磁性器件 |
| 4 | RK3588 SOM | CON1A/C、PCIe、MIPI、GPIO、HDMI、USB |
| 5 | 电源检测/复位 | Q20/Q23、U183/U187、U185、U186、RESET |
| 6 | WX8116/eMMC/Flash | SPI NOR、eMMC、strap、GPIO、DDR/控制 |
| 7 | SOM CON0B/D | USB/MIPI/HDMI/SD/GPIO |
| 8 | DDR3 | DDR3、VTT、VDDQ、地址/数据/控制 |
| 9 | POWER INPUT | 12V→5V、12V→3V3，U137/U139 |
| 10 | SerDes/PHY | WX8116 GPHY、QSGMII、SerDes |
| 11 | KEY/BOOT/LED | PWRON、RECOVERY、BOOT ADC、LED |
| 12 | WX8116 POWER | 1.1V/3.3V/2.5V/模拟电源及大量去耦 |
| 13 | DEBUG UART/RTC/WD | CH340E、USB-C、RTC、电池、HC32、Watchdog |
| 14 | CLOCK/I2C | 25MHz、RTC、温度、Watchdog |
| 15 | Micro SD/FAN/JTAG | SD、风扇 MOS、ARM JTAG |
| 16 | CPLD | EF2L15、JTAG、PHY/LED/状态/watchdog |
| 17 | WX5124/PHY | GMAC/QSGMII/PHY 电源 |
| 18 | USB HUB/OTG | USB HUB、USB-C、FUSB302、ESD |
| 19 | GE0~GE7 | 8路磁性器件、75R、TVS |
| 20 | USB3 HOST | 两路 USB3 Host、电源开关、ESD |
| 21 | SFP | SFP2/SFP3、PCA9543、I2C、TVS |
| 22 | PWR | 1V1、1V5、3V3、2V5 电源生成 |
| 23 | GPS/WIFI | ATGM336H、AT2659、WiFi 模块、天线 |
| 24 | M.2 4G/5G | PCIe2、USB2、SIM、3.7V 电源 |
| 25 | 100MHz CLK | PI6C557、PCIe REFCLK A~D |
| 26 | GE8~GE11 | 磁性器件/TVS |
| 27 | M.2 NVMe | KEY-M1/M2、PCIe3、3.3V DC/DC |
| 28 | M.2 NVMe | KEY-M3/M4、PCIe3、3.3V DC/DC |
| 29 | RJ45 | P0~P11 link/speed、RTx8~11、LED |
| 30 | Fix holes | 机械孔/机壳接地电容 |
| 31 | PWR IN | 两路 12V 输入、TVS、LC、power1/2_in |
| 32 | Design notes | 8128 GPIO/上电/SerDes/eMMC 注意事项 |
| 33 | PORT3002 | NPI0、WX5021、3002 PHY、SerDes、电源 |
| 34 | SATA | JMB575、SATA PM、电源、SPI、RS232/RS485 |

---

# 19. 关键 Net → 器件/接口反向索引

## `VDD_3V3_MAIN`

主要连接：RK3588/SOM IO、电源监控、CPLD、USB、HDMI 控制、RTC、M.2 控制、WiFi、GPS、SFP、各种 LED/上拉。

## `VDD_5V_MAIN`

主要连接：HDMI 5V、USB VBUS、USB HUB、USB Host、SATA、GPS/WiFi 电源输入、局部 LDO。

## `VDD_1V1 / VDD_1V1_KD5128`

主要连接：WX8116/WX5124 `VCCK`、PHY/SerDes 核心模拟域。

## `VDD_2V5`

主要连接：PHY/SerDes PLL/模拟电源。

## `PHY_3V3`

主要连接：PHY IO、GPHY 模拟电源、PHY 参考/控制电路。

## `PER_RESET`

连接：`RESET_L → U142 → PER_RESET`，并分发至 USB HUB、M.2、GPS、CPLD、PHY 等外设。

## `I2C1_SCL/SDA`

连接：RTC `U36`、温度 `U38`、SFP I2C 复用器 `U6` 等。

## `I2C6_SCL_M0/SDA_M0`

连接：FUSB302 USB-C、CPLD/外部模块等。

## `PCIE30_REFCLK_A/B/C/D`

分别服务四组 M.2/PCIe 通道，由 `U168/U169 PI6C557-05BLEX` 产生 HCSL 100MHz 时钟。

---

# 20. 原图明确指出的硬件设计注意事项

1. **PCIe M.2 控制信号存在模式/功能复用约束**；CLKREQn/WAKEn 不能随意换成 GPIO。
2. **M.2 NVMe 页面明确指出 R429 位置错误**，应作为 `PCIE30X1_WAKEn_M2/3V3` 的上拉。
3. 原图明确指出当前三组 PCIe 控制信号设计不满足 PCIe3.0 x2 要求。
4. PCIe3.0 差分阻抗标注约 `85Ω ±10%`；PCIe 时钟约 `100Ω ±10%`。
5. USB3 差分阻抗标注约 `90Ω`；USB2 也要求约 `90Ω`。
6. HDMI 页面明确要求信号从左向右布线，并对 ESD/HPD/DDC/CEC 有专门处理。
7. DDR3 采用大量 100nF/1uF/10uF/22uF 本地去耦以及 VTT 终端。
8. WX8116/WX5124 模拟电源大量采用磁珠隔离，不能简单视为同一个 3.3V 电源域。
9. 8GE/多路 PHY 外部接口均配置磁性器件、75R 和高压隔离/TVS 保护。
10. PDF 第 32 页明确要求：`8128 上电时序，先 3.3V 再 1.1V`。
11. PDF 第 32 页注明：不使用 GMII/eMMC 时对应管脚悬空；内部 QSGMII 转 4×SGMII 时只能用 `S1_Q1`。

---

# 21. 关于“完整电气连接”的边界

本文已经按 PDF 全部 34 页建立了功能、器件、主要网络和跨页连接索引。对于**可以从 PDF 清楚辨认的网络**，尽量保留了原图网络名和器件编号；对于大量重复的 100nF 去耦阵列，采用“网络 + 器件组”方式记录，避免把相同结构错误地解释成不同网络。

如果下一步需要达到真正的 **CAD 级完整 netlist**（例如：`R107.1 → eMMC_DAT0 → U64.A? → ...`，逐个 pin、逐个器件脚位列出），需要原始 Altium/KiCad/OrCAD 工程或导出的 netlist/BOM。仅凭 PDF 可以做非常详细的人工连接追踪，但不能保证 PDF 中所有微小引脚号和重叠文字 100% 无误。
