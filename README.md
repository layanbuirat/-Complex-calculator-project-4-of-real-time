# PIC16F877A Division Calculator System  
**Master-Slave Architecture with High-Precision Arithmetic**
<div align="center">
  <img src="view of project.gif" width="800" alt="Landing Page Demo">
</div>


## 📌 Table of Contents
- [Project Overview](#-project-overview)
- [Key Features](#-key-features)
- [Hardware Architecture](#-hardware-architecture)
- [Communication Protocol](#-communication-protocol)
- [Software Design](#-software-design)
- [Memory Utilization](#-memory-utilization)
- [Installation](#-installation)
- [Usage Guide](#-usage-guide)
- [Performance Metrics](#-performance-metrics)
- [Development Tools](#-development-tools)
- [Future Enhancements](#-future-enhancements)

## 🚀 Project Overview
This system implements a high-precision floating-point division calculator using two PIC16F877A microcontrollers in a master-slave configuration:
- **Master Controller**: Handles user interface with 16x2 LCD and input buttons
- **Slave Co-Processor**: Dedicated arithmetic unit for division operations

Precision: 6 integer + 6 fractional digits (12 significant figures)

## ✨ Key Features
- **Dual-MCU Architecture**  
  - Clear separation of I/O and computation tasks
  - Parallel processing capability

- **Interactive Interface**  
  - Single-button control system
  - Visual cursor positioning
  - Double-tap detection for navigation

- **Advanced Arithmetic**  
  - Long division algorithm in BCD format
  - 12-digit result precision
  - Optimized subtraction routines

## 🔧 Hardware Architecture
### Master Controller
| Port | Function                     | I/O |
|------|------------------------------|-----|
| RB0  | User input button            | IN  |
| RB1  | ACK to slave                 | OUT |
| RB2  | Data ready from slave        | IN  |
| RB5  | ACK from slave               | IN  |
| PORTC| 4-bit data bus to slave      | BID |
| PORTD| LCD control + clock signal   | OUT |

### Slave Controller
| Port | Function                     | I/O |
|------|------------------------------|-----|
| RB1  | ACK to master                | OUT |
| RB5  | Data ready signal            | OUT |
| PORTC| 4-bit data bus to master     | BID |

## 📡 Communication Protocol
### Master → Slave Transmission
```mermaid
sequenceDiagram
    Master->>Slave: Set 4-bit data on PORTC
    Master->>Slave: Clock pulse (PORTD.0)
    Slave->>Master: ACK via RB5
    loop Until all data sent
        Master->>Slave: Next nibble
    end

Slave → Master Transmission
![PIC16F877A Microcontroller](https://github.com/layanbuirat/-Complex-calculator-project-4-of-real-time/blob/main/slave.png)
sequenceDiagram
    Slave->>Master: Set 4-bit result on PORTC
    Slave->>Master: Clock pulse (RB1)
    Master->>Slave: ACK via RB2
    loop Until all results sent
        Slave->>Master: Next nibble
    end

💾 Software Design
Master State Machine

enum States {
    INIT,          // System initialization
    NUM1_INPUT,    // First number entry
    NUM2_INPUT,    // Second number entry
    SEND_DATA,     // Transmit to slave
    RECV_DATA,     // Receive results
    DISP_RESULT    // Show final output
};

Slave Core Routines
Routine	Cycles	Description
CONV_BIN_DEC	25	Binary to BCD conversion
COMP_INT	1200+	Integer division
COMP_FRAC	1800+	Fractional division
SUB_BCD	85	BCD subtraction with borrow
📊 Memory Utilization
Master Controller
Memory Type	Usage	Percentage
Program	847 words	10%
Data	0x20-0x59	25%
Slave Controller
Memory Type	Usage	Percentage
Program	543 words	6%
Data	0x20-0x67	35%
🔌 Installation
Hardware Setup:

bash
# Connect master and slave as per schematic
VDD ---- 5V regulated supply
GND ---- Common ground
PORTC -- 4-wire data bus
Programming:

bash
# Using MPLAB X IDE
mplab_ide --program master.hex
mplab_ide --program CO.hex
🎮 Usage Guide
Power on system

Input sequence:

text
[Single Press] - Increment current digit
[Double Press] - Move cursor
Calculation flow:

text
Enter NUM1 → Enter NUM2 → View Result
⚡ Performance Metrics
Metric	Value
Worst-case calculation	520ms
Input response time	<10ms
Data transfer rate	1ms/nibble
Power consumption	42mA @5V
🛠️ Development Tools
MPLAB X IDE v5.50

XC8 Compiler v2.32

PICKit 3 Programmer

Proteus 8.13 (Simulation)

🔮 Future Enhancements
Error handling for:

Division by zero

Overflow conditions

Additional operations:

math
\sqrt{x}, x^y, log(x)
Scientific notation display

Serial debugging interface

📂 Project Structure

/Division_Calculator
│
├── /Firmware
│   ├── Master
│   │   ├── main.asm
│   │   └── LCD_driver.inc
│   │
│   └── Slave
│       ├── math_engine.asm
│       └── comm_protocol.inc
│
├── /Hardware
│   ├── Schematic.pdf
│   └── BOM.csv
│
└── /Documentation
    ├── Technical_Manual.md
    └── API_Reference.md
