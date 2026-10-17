# Multi-Architecture Embedded Systems Design & VSM Simulation Portfolio (8051 & PIC16F)

# 多架构嵌入式系统设计与虚拟系统建模 (VSM) 仿真项目集 (8051 & PIC16F)

[English](#english-documentation) | [中文说明](#中文说明)

---

<a name="english-documentation"></a>

## English Documentation

### 1. Project Overview & Academic Context

- **Aim:** To bridge theoretical microcontroller architecture with practical Electronic Design Automation (EDA) workflows through bare-metal C firmware development and mixed-mode SPICE hardware simulation.
  - **Course / Context:** Microcontroller Principles and Embedded Systems Design

  - **Institution:** Example University

  - **Student Name:** Ssamula Phillip Peter

  - **Semester:** 6th Semester

  - **Grade:** A / 96%

  - **Project Type:** Individual Academic Project

  
- **What It Does:** Implements and validates four complete embedded control and sensing systems across two distinct microcontroller architectures (CISC Intel 8051 and RISC Microchip PIC16F):
    1. **Real-Time Digital Clock (`AT89C52` + `DS1302`):** Synchronous serial RTC timekeeping with 8-digit 7-segment dynamic display multiplexing.
    2. **Closed-Loop Environmental Monitoring & Alarm System (`AT89C52` + `DHT11`):** 1-Wire digital telemetry acquisition, UART serial transmission, interrupt-driven UI navigation, and autonomous active-low threshold alarms.
    3. **Ultrasonic Time-of-Flight (ToF) Ranging System (`AT89C52` + `SRF04`):** Microsecond-precision echo pulse capture using 16-bit hardware timers and oscilloscope signal verification.
    4. **Sensored 3-Phase BLDC Motor PWM Controller (`PIC16F877`):** 6-step electronic commutation decoding 120° Hall-effect sensor feedback with ADC-driven software PWM speed regulation.

### 2. Toolchain & My Role

- **Software & EDA Tools:**
    * **Arm Keil µVision4 IDE (C51 Compiler):** C firmware authoring, register-level disassembly debugging, and Intel `.hex` generation for 8051 targets.
    * **Microchip MPLAB X IDE (v6.30) & MPLAB XC8 Compiler:** Bare-metal firmware compilation and configuration-bit programming for PIC16F targets.
    * **Labcenter Electronics Proteus Design Suite 9 Professional (ISIS & VSM):** Schematic capture, mixed-mode SPICE circuit simulation, virtual UART terminal monitoring, and 4-channel digital oscilloscope analysis.
- **My Role (Individual Contribution):** 
    * Designed and executed the complete simulation verification workflow, firmware integration, hardware-software debugging, and technical documentation across all modules.
    * **Self-Directed Cross-Architecture Migration:** Identified a critical toolchain limitation when transitioning from 8051 modules to the capstone PIC16F877 BLDC motor controller (as Keil C51 only supports 8051 instruction sets). Independently self-taught **Microchip MPLAB X IDE and the XC8 compiler**, configured PIC hardware pragmas and data-direction registers (`TRIS`, `ADCON`), and implemented the 6-step commutation and software PWM algorithms.

### 3. Methodology & System Architecture

#### Module 1 & 2: RTC Digital Clock & EDA Co-Design (`AT89C52` + `DS1302`)

- **Hardware:** Interfaced the `AT89C52` (12 MHz clock) with a `DS1302` trickle-charge RTC IC (`RST`, `SCLK`, `I/O`), a `74HC138` 3-to-8 line decoder for digit selection, and a `74HC573` octal latch driving an 8-digit common-cathode 7-segment display (`7SEG-MPX8-CC-BLUE`).
- **Firmware:** Implemented bit-banged synchronous serial read/write routines (`ds1302write`, `ds1302read`) and bidirectional BCD-to-Decimal conversion:
    $$\text{Decimal} = \left\lfloor\frac{\text{BCD}}{16}\right\rfloor \times 10 + (\text{BCD} \ \& \ \text{0x0F})$$

#### Module 3: Environmental Monitoring & Threshold Alarm (`AT89C52` + `DHT11`)

- **Hardware:** Driven by an **11.0592 MHz crystal oscillator** to ensure zero-error baud rate division for RS-232 UART telemetry. Interfaced a `DHT11` sensor on `P1.7`, an `LM016L` (LCD1602) on `P0.0–P0.7` (with an external `RESPACK-8` pull-up network required by Port 0's open-drain structure), 4 tactile switches (`P3.2–P3.5`), and 4 active-low alarm LEDs (`P2.0–P2.3` via $100\ \Omega$ current-limiting resistors).
- **Firmware:** Decodes the 40-bit 1-Wire packet (disabling global interrupts `EA = 0` during critical microsecond timing windows). Configures `Timer0` in 16-bit Mode 1 (`TH0 = 0xEE`, `TL0 = 0x00`) to trigger a **5 ms periodic interrupt** (`interrupt 1`) for non-blocking key scanning (`KeyScan()`) and closed-loop bang-bang threshold evaluation (`CheckLimits()`).

#### Module 4: Ultrasonic Time-of-Flight Distance Ranging (`AT89C52` + `SRF04`)

- **Hardware:** Interfaced an `SRF04` ultrasonic transducer (`TRIG` on `P2.1`, `ECHO` on `P2.0`), an `LM016L` LCD, and a 4-Channel Digital Oscilloscope (Channel A monitoring the trigger pulse; Channel B capturing the variable echo pulse width).
- **Firmware:** Generates a $10\ \mu\text{s}$ trigger pulse (`HCSR04_trigSendOnePulse()`) to initiate an 8-cycle 40 kHz acoustic burst. `Timer0` (16-bit Mode 1) measures the exact high-level duration $t$ of the `ECHO` pin (`TH0`/`TL0`), computing spatial distance via acoustic Time-of-Flight ($v \approx 340\text{ m/s}$):
    $$d = \frac{v \times t}{2}$$

#### Module 5: Sensored 3-Phase BLDC Motor PWM Controller (`PIC16F877`)

- **Hardware:** Centered on a `PIC16F877` (20 MHz HS crystal). Three internal Hall-effect position sensors ($120^\circ$ electrical spacing) feed rotor state into `PORTE` (`RE0–RE2`). Six commutation lines (`PORTC` `RC0–RC5`) drive `TC4469` gate driver ICs connected to a 3-phase inverter bridge (`IRF5305` P-channel high-side MOSFETs `Q1–Q3` and `IRLI3705N` N-channel low-side MOSFETs `Q4–Q6`). Speed reference is provided by potentiometer `RV1` on analog pin `AN0` (`RA0`), with direction switch `SW1` on `RA1`.
- **Firmware:** Masks 3-bit Hall sensor inputs (`PORTE & 0B00000111`) against lookup tables (`FwdTable` / `RevTable`, treating `000` and `111` as sensor fault states). Implements **computational software PWM** using free-running `Timer0` (1:256 prescaler) and 8-bit left-justified ADC readings (`ADRESH`):
    $$V_{\text{avg}} = D \times V_{\text{bus}} = \left(\frac{T_{\text{on}}}{T_{\text{on}} + T_{\text{off}}}\right) V_{\text{bus}}$$
    During the PWM off-phase (`TMR0 + ADC_Result <= 0xFF`), the firmware applies `BLDC_StopMask` (`0B11010101`) to chop high-side MOSFET drive signals while preserving low-side commutation paths.

### 4. Measured Simulation Results

| Module                     | MCU Target              | Test Condition / Stimulus                                                                                      | Measured Output / Telemetry                                                                                    | Verification Method                                                               |
|:-------------------------- |:----------------------- |:-------------------------------------------------------------------------------------------------------------- |:-------------------------------------------------------------------------------------------------------------- |:--------------------------------------------------------------------------------- |
| **1 & 2. RTC Clock**       | `AT89C52` (12 MHz)      | Initial preset `22:59:55`, continuous run                                                                      | Smooth rollover from `22-59-57` to `23-00-35`; zero compilation errors (`code=2090` bytes)                     | Keil Register Disassembly + Proteus 8-Digit 7-Seg Display                         |
| **3. Temp/Humidity Alarm** | `AT89C52` (11.0592 MHz) | **Baseline:** $22^\circ\text{C}$, $30\%\text{ RH}$ (Limits: $20\text{–}40^\circ\text{C}$, $30\text{–}60\%$)    | LCD: `hum:30 temp:22`; UART: `hello world\r\n`; `P2.0–P2.3 = 1` (All 4 LEDs OFF)                               | Proteus VSM + Virtual UART Terminal                                               |
| **3. Temp/Humidity Alarm** | `AT89C52` (11.0592 MHz) | **Over-Temp Fault:** Stepped `DHT11` to $41^\circ\text{C}$ ($> 40^\circ\text{C}$ limit)                        | LCD updates to `humi:30 temp:41`; `P2.0` driven LOW (`reduce_temp = 0`), illuminating **LED1**                 | Proteus VSM Active-Low Actuator Check                                             |
| **4. Ultrasonic ToF**      | `AT89C52`               | **State 1 (Baseline):** Default obstacle distance<br>**State 2 (Adjusted):** Sensor echo parameter set to `82` | **State 1:** LCD reads `distanc : 59.177`<br>**State 2:** Ch B pulse widens; LCD updates to `distanc : 80.869` | Proteus 4-Channel Digital Oscilloscope (Ch A: $10\ \mu\text{s}$ TRIG, Ch B: ECHO) |
| **5. BLDC Motor PWM**      | `PIC16F877` (20 MHz)    | **Mid-Range PWM:** Potentiometer `RV1` adjusted<br>**Brake Actuation:** Brake switch engaged                   | Stable rotation at **+296 RPM**; rapid inertial deceleration to **+1.13 / 0 RPM** upon braking                 | Proteus `MOTOR-BLDCM` Real-Time RPM Telemetry                                     |

### 5. Engineering Limitations & Future Improvements

1. **EDA Simulation Physics Limitation (BLDC Reversal):** While the reverse commutation lookup table (`RevTable`) in the PIC16F877 firmware is mathematically valid (swapping high-side and low-side phase windings), toggling direction switch `SW1` (`RA1`) during active rotation causes the virtual motor model to stall. This was isolated as a known limitation in the Proteus VSM `MOTOR-BLDCM` SPICE model regarding dynamic commutation reversal.
2. **Blocking Echo Polling (Ultrasonic Module):** The `DistCount()` routine currently uses blocking `while` loops to wait for the `ECHO` pin transition. In a multi-tasking system, this could cause CPU starvation if an echo is lost; migrating `ECHO` capture to external hardware interrupts (`INT0`/`INT1`) with timer overflow protection is recommended for physical deployment.
3. **Software PWM Resolution:** The `PIC16F877` software PWM loop relies on `TMR0` polling. Hardware CCP (Capture/Compare/PWM) modules combined with dead-time insertion hardware would further prevent shoot-through currents in physical high-power MOSFET bridges.

---

<a name="中文说明"></a>

## 中文说明 (Chinese Documentation)

### 1. 项目概述与课程背景

- **项目目标：** 通过底层 C 语言固件开发与混合模式 SPICE 硬件仿真，将微控制器（单片机）理论架构与电子设计自动化 (EDA) 工程工作流紧密结合。
- **课程背景：** 单片机原理与嵌入式系统设计
  院校：示例大学
  学生姓名：SSAMULA PHILLIP PETER
  学期：第 6 学期
  成绩：A / 96 %
  项目类型：个人学术项目
- **核心功能：** 跨越两种不同的微控制器架构（CISC 架构的 Intel 8051 与 RISC 架构的 Microchip PIC16F），完成了四个完整嵌入式传感与机电控制系统的设计、编译与仿真验证：
    1. **基于 51 单片机的电子时钟系统 (`AT89C52` + `DS1302`)：** 实现同步串行 RTC 时钟通信、BCD 码转换及 8 位七段数码管动态扫描显示。
    2. **基于 51 单片机的温湿度检测报警系统 (`AT89C52` + `DHT11`)：** 实现单总线 (1-Wire) 数字传感采集、UART 串口通信、定时器中断按键扫描交互及越限声光（LED）闭环报警。
    3. **基于 51 单片机的超声波测距系统 (`AT89C52` + `SRF04`)：** 利用 16 位硬件定时器捕获微秒级回波脉宽，结合虚拟四通道数字示波器验证飞行时间 (ToF) 测距算法。
    4. **带位置传感器的直流无刷电机 (BLDC) PWM 控制仿真 (`PIC16F877`)：** 解码 $120^\circ$ 霍尔位置传感器反馈以实现六步电子换向，并结合 ADC 采样与软件 PWM 算法实现电机无级调速与制动控制。

### 2. 开发工具链与个人职责

- **软件与 EDA 工具：**
    * **Arm Keil µVision4 IDE (C51 编译器)：** 用于 8051 架构的 C 语言固件编写、寄存器/反汇编级调试及可执行 `.hex` 文件生成。
    * **Microchip MPLAB X IDE (v6.30) & MPLAB XC8 编译器：** 用于 PIC16F 架构的底层固件开发、配置位 (`#pragma config`) 设定及编译。
    * **Labcenter Electronics Proteus Design Suite 9 Professional：** 用于硬件原理图绘制 (ISIS)、混合模式 SPICE 虚拟系统建模 (VSM)、虚拟串口终端及四通道数字示波器信号分析。
- **个人角色与核心贡献：**
    * 独立完成全部实验模块的软硬件联调、代码剖析、仿真测试、故障排查及英文技术报告撰写。
    * **跨架构工具链自主迁移：** 在进行第五部分（直流无刷电机 PWM 控制）课题时，针对目标芯片由 8051 (`AT89C52`) 切换至 Microchip `PIC16F877` 架构、导致 Keil C51 无法编译的工程瓶颈，**自主学习并掌握了 MPLAB X IDE 与 XC8 编译器**，独立完成了 PIC 寄存器配置 (`TRIS`, `ADCON`, `OPTION_REG`)、六步换向查表法及软件 PWM 调速代码的编译与联调。

### 3. 核心方法与系统设计

- **DS1302 时钟与数码管动态复用：** 通过模拟 SPI 时序读写 `DS1302` 寄存器（含写保护位 `0x8E` 控制），完成 BCD 码与十进制互转，并通过 `74HC138` 译码器与 `74HC573` 锁存器实现无闪烁的 8 位数码管动态扫描。
- **DHT11 采集与 5ms 定时器中断架构：** 采用 **11.0592 MHz 晶振**确保 UART 波特率零误差；在 `DHT11` 40-bit 单总线读取期间临时关闭全局中断 (`EA = 0`) 保证严格时序；配置 `Timer0` 为 16 位模式 (`TH0 = 0xEE, TL0 = 0x00`)，每 **5 ms** 触发一次中断以执行非阻塞按键扫描 (`KeyScan()`) 与上下限阈值判断 (`CheckLimits()`)。
- **超声波 ToF 硬件计时测距：** 单片机发送 $10\ \mu\text{s}$ 高电平触发脉冲 (`TRIG`)，通过 16 位 `Timer0` 精确记录 `ECHO` 引脚高电平持续时间 $t$，依据声速公式 $d = \frac{v \times t}{2}$ ($v \approx 340\text{ m/s}$) 实时解算浮点距离并通过 `sprintf` 格式化输出至 `LCD1602`。
- **PIC16F877 无刷电机六步换向与软件 PWM：** 读取 `PORTE` (`RE0–RE2`) 的 3 位霍尔传感器状态（屏蔽 `000` 和 `111` 非法故障态），通过查表法 (`FwdTable`/`RevTable`) 从 `PORTC` 输出 6 路控制信号，经 `TC4469` 驱动由 `IRF5305` (P-MOS) 与 `IRLI3705N` (N-MOS) 组成的三相全桥逆变电路。通过 `AN0` 采集调速电位器 `RV1` 的 8 位 ADC 值 (`ADRESH`)，与自由运行的 `Timer0` (`TMR0`) 比较，利用 `BLDC_StopMask` (`0B11010101`) 对高端 MOS 管进行斩波调制。

### 4. 仿真测试结果汇总

| 实验模块                  | 目标主控芯片                  | 测试条件与激励输入                                                                                          | 实测输出与遥测数据                                                                                   | 验证手段                                  |
|:--------------------- |:----------------------- |:-------------------------------------------------------------------------------------------------- |:------------------------------------------------------------------------------------------- |:------------------------------------- |
| **1 & 2. 电子时钟系统**     | `AT89C52` (12 MHz)      | 初始时间设为 `22:59:55`，连续运行                                                                             | 数码管稳定显示并从 `22-59-57` 平滑进位至 `23-00-35`；零编译错误 (`code=2090` 字节)                                | Keil 反汇编寄存器调试 + Proteus 8位数码管         |
| **3. 温湿度报警系统**        | `AT89C52` (11.0592 MHz) | **常态测试：** $22^\circ\text{C}$, $30\%\text{ RH}$ (阈值：$20\text{–}40^\circ\text{C}$, $30\text{–}60\%$) | LCD 显示 `hum:30 temp:22`；串口输出 `hello world\r\n`；`P2.0–P2.3 = 1`（4路报警灯全灭）                     | Proteus VSM + 虚拟串口终端                  |
| **3. 温湿度报警系统**        | `AT89C52` (11.0592 MHz) | **超温触发：** 手动调节 `DHT11` 温度至 $41^\circ\text{C}$ ($> 40^\circ\text{C}$)                               | LCD 实时更新为 `humi:30 temp:41`；`P2.0` 输出低电平 (`reduce_temp = 0`)，**LED1 降温报警灯点亮**               | Proteus VSM 低电平有效负载验证                 |
| **4. 超声波测距系统**        | `AT89C52`               | **状态 1：** 初始默认障碍物距离<br>**状态 2：** 传感器距离参数手动调至 `82`                                                  | **状态 1：** LCD 显示 `distanc : 59.177`<br>**状态 2：** 示波器 Ch B 脉宽明显展宽，LCD 更新为 `distanc : 80.869` | Proteus 四通道数字示波器 (Ch A: 触发, Ch B: 回波) |
| **5. BLDC 电机 PWM 控制** | `PIC16F877` (20 MHz)    | **PWM 调速：** 调节 `RV1` 电位器<br>**紧急制动：** 触发左上角制动开关                                                    | 电机稳定运行于 **+296 RPM**，随电位器平滑加速；制动触发后转速迅速衰减至 **+1.13 / 0 RPM**                                | Proteus `MOTOR-BLDCM` 实时转速遥测          |

### 5. 工程局限性分析与改进方向

1. **EDA 仿真引擎物理模型限制（BLDC 换向）：** 尽管 C 代码中的反转换向表 (`RevTable`) 在数学与电气逻辑上完全正确，但在 Proteus 仿真中拨动换向开关 `SW1` (`RA1`) 会导致电机失步停转。经排查确认，这是当前版本 Proteus VSM 的 `MOTOR-BLDCM` 模型不支持动态反转物理仿真所致。
2. **超声波回波轮询阻塞问题：** `DistCount()` 函数目前采用阻塞式 `while` 循环等待 `ECHO` 电平翻转。在实际工程中建议改为外部中断 (`INT0`/`INT1`) 配合定时器溢出中断保护，以防止回波丢失导致系统死锁。
