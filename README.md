# STM32-Universal-Bootloader-Implementation
# STM32 Universal UART Bootloader

## 📜 Overview
This project is a **feature-rich, command-based bootloader for STM32 microcontrollers** (tested on STM32F103 series).  
It allows firmware updates, flash memory operations, read/write protection management, and execution control — all over a **UART interface**.  

Key features:
- Communicates with host via **UART**
- Supports **CRC verification** for data integrity
- Flash memory **erase** and **write** commands
- Jump to application (custom firmware execution)
- **Read protection** enable/disable
- MCU identification & bootloader version reporting
- Structured and extendable command handler

This bootloader is ideal for:
- Remote firmware updates
- Production device maintenance
- Embedded system prototyping

---

## ⚙️ Features

| Feature                  | Description |
|--------------------------|-------------|
| **Command-based Protocol** | Well-structured command IDs |
| **UART Interface**       | Configurable baud rate, 8N1 |
| **CRC32 Validation**     | Ensures reliable data transfer |
| **GET Commands**         | Query supported commands & version |
| **Flash Erase**          | Sector-wise or mass erase |
| **Flash Write**          | Program arbitrary memory locations |
| **Memory Read**          | Read and send MCU memory data |
| **Protection Management**| Enable/Disable read protection |
| **Jump to Application**  | Executes user firmware at given address |
| **MCU Identification**   | Reads and reports chip ID |

---

## 📂 Project Structure
```
.
├── Core/
│ ├── Inc/ # Header files
│ ├── Src/ # Source files
│ └── startup/ # Startup code
├── Drivers/ # HAL and CMSIS drivers
├── bootloader.c # Bootloader logic
├── bootloader.h # Bootloader definitions
├── main.c # Entry point
├── README.md # This documentation
└── ...
```

---

## 🧩 Bootloader Command Set

| Command ID | Command Name            | Description |
|------------|------------------------|-------------|
| `0x00`     | GET                     | List supported commands |
| `0x01`     | GET VERSION              | Report bootloader firmware version |
| `0x02`     | GET HELP                 | Help & usage information |
| `0x03`     | GET CHIP ID              | Return MCU ID |
| `0x04`     | READ PROTECTION STATUS   | Check flash read protection |
| `0x05`     | GO TO ADDRESS            | Jump to user application |
| `0x06`     | FLASH ERASE              | Erase flash sectors |
| `0x07`     | MEMORY WRITE             | Write data to flash |
| `0x08`     | ENABLE RDP LEVEL 1       | Enable read protection |
| `0x09`     | DISABLE RDP              | Disable read protection |

---

## 🔌 UART Communication Protocol

1. **Host sends:**
``` [COMMAND CODE] [PAYLOAD BYTES...] [CRC (4 bytes)] ```
2. **Bootloader verifies CRC**:
- If **valid**, sends ACK and executes command.
- If **invalid**, sends NACK.
3. **Responses** vary depending on command type.

Baud rate is configurable in `usart.c` (default **115200 bps**).

---

## 📊 Bootloader Flowchart (Text Form)
```
Start
 │
 ▼
System Initialization
 │
 ├─> HAL_Init()
 │
 ├─> Configure System Clock
 │
 ├─> Initialize Peripherals (GPIO, CRC, UART)
 │
 └─> Initialize Bootloader Context
 │
 ▼
Print Bootloader Welcome Message
 │
 ▼
Loop Forever
 │
 ├─> Wait for command from UART
 │
 ├─> If command received:
 │     │
 │     ├─> Validate Command CRC
 │     │
 │     ├─> If CRC invalid:
 │     │     └─> Send NACK
 │     │
 │     ├─> Else (CRC valid):
 │     │     ├─> Send ACK
 │     │     │
 │     │     ├─> Process Command:
 │     │     │
 │     │     ├─── "GET" Command:
 │     │     │        └─> Send Supported Command List
 │     │     │
 │     │     ├─── "GET VERSION" Command:
 │     │     │        └─> Send Bootloader Version
 │     │     │
 │     │     ├─── "GET HELP" Command:
 │     │     │        └─> Send Help Message
 │     │     │
 │     │     ├─── "GET CHIP ID" Command:
 │     │     │        └─> Send MCU ID
 │     │     │
 │     │     ├─── "READ PROTECTION STATUS":
 │     │     │        └─> Send ROP Status
 │     │     │
 │     │     ├─── "GO TO ADDRESS" Command:
 │     │     │        └─> Validate Address & Jump
 │     │     │
 │     │     ├─── "FLASH ERASE" Command:
 │     │     │        └─> Erase Specified Sectors
 │     │     │
 │     │     ├─── "MEMORY WRITE" Command:
 │     │     │        └─> Validate Address & Write Data
 │     │     │
 │     │     ├─── "ENABLE/DISABLE READ PROTECTION":
 │     │     │        └─> Change Flash Protection Level
 │     │     │
 │     │     └─── Unknown Command:
 │     │              └─> Send Error / NACK
 │     │
 │     └─> Return to Wait for Next Command
 │
 └─> (Loop repeats)
```

---

## 🛠️ Building & Flashing

### 1️⃣ Requirements
- **STM32CubeIDE** or **Keil uVision**
- STM32F103 or compatible MCU
- USB-to-UART adapter (for host communication)

### 2️⃣ Build
- Open the project in STM32CubeIDE
- Select the correct MCU target
- Build the firmware (`Ctrl+B`)

### 3️⃣ Flash Bootloader
- Use **ST-Link** or other programmer
- Set vector table offset to `0x08000000` (bootloader start)

---

## 🖥️ Using the Bootloader

A simple Python script can be used to send commands to the bootloader. Example:

```python
import serial, struct, binascii

ser = serial.Serial('COM5', 115200, timeout=1)

def send_command(cmd, payload=b''):
    crc = binascii.crc32(payload) & 0xffffffff
    ser.write(bytes([cmd]) + payload + struct.pack('<I', crc))
    response = ser.read(10)
    print("Response:", response)

send_command(0x00)  # GET command
```

🔐 Security Notes
CRC validation ensures basic data integrity, not encryption.

Read protection should be enabled for production devices.

Consider integrating AES encryption for secure firmware transfers.

📜 License
This project is licensed under the MIT License — feel free to modify and use in commercial or personal projects.

👨‍💻 Author
Developed by Mohamed Ayman

Embedded systems engineer & firmware developer

Contact: mohamedaymanworkspace@gmail.com


🚀 Future Improvements
USB DFU mode support

Encrypted firmware update protocol

Multi-interface bootloader (CAN, I2C, SPI)

OTA updates via wireless modules

