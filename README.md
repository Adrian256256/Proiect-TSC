# TSC
## Project: OpenBook  
**Author:** Teodor-Adrian Harea (333CA)  

![schema_bloc](https://github.com/user-attachments/assets/77c92a13-ef4c-4942-9d59-9a1516e06ebd)
---


## 1. Core Components  

### 1.1 Microcontroller: ESP32-C6  
| Specification          | Details                                                                 |
|------------------------|-------------------------------------------------------------------------|
| Architecture           | 32-bit RISC-V (Wi-Fi 6 802.11ax + Bluetooth 5)                         |
| Clock Speed            | Up to 160 MHz                                                          |
| Memory                 | 320 KB SRAM + 512 KB internal flash (expandable via external NOR flash)|
| Interfaces             | 30+ GPIO (SPI×2, I2C×1, UART×2, USB 2.0)                              |

---

## 2. Peripheral Modules  

### 2.1 E-Paper Display (Waveshare)  
- **Interface:** 4-wire SPI (MOSI/SCK/MISO/CS)  
- **Control Pins:**  
  - `EPD_RST` (Hardware reset)  
  - `EPD_DC` (Data/command selector)  
  - `EPD_BUSY` (Status monitoring)  
- **Power Characteristics:**  
  - 150 mA (active refresh)  
  - 0 mA (static display)  

### 2.2 Environmental Sensor (BME688)  
- **Measurements:** Temp/Humidity/Pressure/VOC  
- **Interface:** I2C (100-400 kHz)  
- **Accuracy:** ±0.5°C, ±3% RH  
- **Power:** 3.6 mA (active), 2.1 µA (sleep)  

### 2.3 Precision RTC (DS3231SN)  
- **Accuracy:** ±2ppm (±1 min/year)  
- **Features:**  
  - Battery backup (VBAT pin)  
  - Alarm interrupts (INT_RTC)  

### 2.4 Storage Solutions  
| Component       | Capacity | Interface | Speed           |
|-----------------|----------|-----------|-----------------|
| NOR Flash       | 64 MB    | QSPI      | 133 MHz         |
| microSD Slot    | Expandable | SPI      | 25 MHz          |

---

## 3. Power Management  

### 3.1 Power Architecture  
- **LiPo Charger:** MCP73831 (5V USB → 4.2V battery)  
- **Voltage Regulation:**  
  - XC6220A331MR-G (3.3V LDO)  
- **Battery Monitoring:** MAX17048G (I2C fuel gauge)  

### 3.2 Power States  
| Mode              | Current Draw | Runtime* |
|-------------------|--------------|----------|
| Active (measure + display) | 230 mA | 4h       |
| Deep Sleep (RTC only)      | 6 µA   | 1+ month |

*Assuming 1000mAh battery

---

## 4. Detailed Pin Configuration  

### 4.1 E-Paper Display Control  
| Pin Name      | Type    | Functionality                          | ESP32-C6 Connection |
|--------------|---------|----------------------------------------|---------------------|
| `EPD_RST`    | Output  | Hardware reset (active low)            | GPIO18              |
| `EPD_DC`     | Output  | Data/Command selection (SPI mode)      | GPIO21              |
| `EPD_BUSY`   | Input   | Busy status (high during refresh)      | GPIO19              |
| `EPD_CS`     | Output  | SPI chip select (active low)           | GPIO5               |
| `MOSI/SCK`   | SPI     | Main display data/clock lines          | SPI1_HOST           |

### 4.2 Storage Interfaces  
#### NOR Flash (W25Q512JVEIQ)  
| Pin         | Function                      | ESP32 Pin |
|-------------|-------------------------------|-----------|
| `FLASH_CS`  | QSPI chip select              | GPIO17    |
| `IO0-IO3`   | Quad SPI data lines           | QSPI bus  |

#### microSD Card Slot  
| Pin       | Function                | ESP32 Pin |
|-----------|-------------------------|-----------|
| `SS_SD`   | SPI slave select        | GPIO4     |
| `CD`      | Card detect (optional)  | GPIO22    |

### 4.3 System Management  
| Component    | Critical Pins               | Purpose                             |
|--------------|-----------------------------|-------------------------------------|
| **RTC**      | `SCL`/`SDA` (I2C)           | Timekeeping (DS3231SN)              |
|              | `INT_RTC`                   | Wake-up alarm interrupt             |
| **Power**    | `VBAT`                      | RTC backup battery (3V)             |
|              | `CHG_STAT`                  | LiPo charger status LED             |

### 4.4 User Interface  
| Pin       | Type      | Functionality                  |
|-----------|-----------|--------------------------------|
| `PAGE_UP` | GPIO      | Tactile button (internal pullup)|
| `PAGE_DN` | GPIO      | With debounce circuit          |
| `PWR_HLD` | Output    | Maintains power when pressed   |

### 4.5 Debug Interfaces  
| Interface | Pins Used          | Baud Rate |
|-----------|--------------------|-----------|
| USB-C     | `DP`/`DM`          | 12 Mbps   |
| UART0     | `TX0`/`RX0`        | 115200    |

---

## 5. Optimization Strategies  

### 5.1 Low-Power Techniques  
- Dynamic clock scaling (160 MHz → 10 MHz when idle)  
- Peripheral power gating (MOSFET control for EPD/sensors)  
- SPI/I2C bus deactivation between transactions  

### 5.2 Data Logging Workflow  
1. Wake from deep sleep (RTC/alarm)  
2. Power sensors via GPIO  
3. Acquire measurements (BME688)  
4. Store to flash/SD  
5. Update EPD if threshold exceeded  
6. Return to deep sleep

### **Bill of Materials (BOM)**

| Category            | Component               | Description                          | Mouser Link | Datasheet Link |
|---------------------|-------------------------|--------------------------------------|-------------|----------------|
| **Microcontrollers & Modules** | ESP32-C6-WROOM-1-N8 | WiFi/Bluetooth Module | [Link](https://ro.mouser.com/ProductDetail/Espressif-Systems/ESP32-C6-WROOM-1-N8?qs=8Wlm6%252BaMh8ST02Gmwp74cw%3D%3D) | [Datasheet](https://ro.mouser.com/datasheet/2/891/Espressif_ESP32_C6_WROOM_1__Datasheet_V0_1_PRELIMI-3239987.pdf) |
| | W25Q512JVEIQ | 64MB External NOR Flash | [Link](https://ro.mouser.com/ProductDetail/Winbond/W25Q512JVEIQ?qs=l7cgNqFNU1jw6svr3at6tA%3D%3D) | [Datasheet](https://ro.mouser.com/datasheet/2/949/Winbond_W25Q512JV_Datasheet-3240039.pdf) |
| **Sensors** | BME688 | Environmental Sensor | [Link](https://ro.mouser.com/ProductDetail/Bosch-Sensortec/Evaluation-Kit-Board-BME688?qs=QNEnbhJQKvZD%2Fs%2Ff0WWu0Q%3D%3D) | [Datasheet](https://ro.mouser.com/datasheet/2/783/Bosch_sensortec_bme688_development_kit_flyer-3000177.pdf) |
| | DS3231SN# | RTC Module | [Link](https://ro.mouser.com/ProductDetail/Analog-Devices-Maxim-Integrated/DS3231SN?qs=1eQvB6Dk1vhUlr8%2FOrV0Fw%3D%3D) | [Datasheet](https://ro.mouser.com/datasheet/2/609/DS3231-3421123.pdf) |
| **Power Management** | MAX17048G+T10 | Battery Fuel Gauge | [Link](https://eu.mouser.com/ProductDetail/Analog-Devices-Maxim-Integrated/MAX17048G+T10?qs=D7PJwyCwLAoGnnn8jEPRBQ%3D%3D&srsltid=AfmBOoq0Uf9CVLtvuYCdL-b56-Pbwotx2XvoNmNfB9vf0O63iEPsWucA) | [Datasheet](https://eu.mouser.com/datasheet/2/609/MAX17048_MAX17049-3469099.pdf) |
| | MCP73831 | Li-Po Battery Charger | [Link](https://ro.mouser.com/ProductDetail/Microchip-Technology/MCP73831-2DCI-MC?qs=hH%252BOa0VZEiBQ%2FrptDRXKdg%3D%3D) | [Datasheet](https://ro.mouser.com/datasheet/2/268/MCP73831_Family_Data_Sheet_DS20001984H-3441711.pdf) |
| | XC6220A331MR-G | LDO Voltage Regulator 3.3V | [Link](https://ro.mouser.com/ProductDetail/Torex-Semiconductor/XC6220A331MR-G?qs=AsjdqWjXhJ8ZSWznL1J0gg%3D%3D) | [Datasheet](https://ro.mouser.com/datasheet/2/760/xc6220-3371556.pdf) |
| **Connectors & Protection** | USBLC6-2SC6Y | USB ESD Protection | [Link](https://eu.mouser.com/ProductDetail/STMicroelectronics/USBLC6-2SC6Y?qs=gNDSiZmRJS%2FOgDexvXkdow%3D%3D&srsltid=AfmBOooGFMy49XJWMdHSBGK6U_FDXdeHD6KsINSiWZt-au9axwzb1pgh) | [Datasheet](https://ro.mouser.com/datasheet/2/389/usblc6_2sc6y-1852505.pdf) |
| | FH34SRJ-24S-0.5SH | 24-pin Connector | [Link](https://ro.mouser.com/ProductDetail/Hirose-Connector/FH34SRJ-24S-0.5SH50?qs=iyLo5FA4poC8fzWlavnA7A%3D%3D) | [Datasheet](https://ro.mouser.com/datasheet/2/185/FH34SRJ_24S_0_5SH_50__CL0580_1255_6_50_2DDrawing_0-1615030.pdf) |
| **Passive Components** | SD0805S020S1R0 | 68μH Inductor | [Link](https://ro.mouser.com/ProductDetail/KYOCERA-AVX/SD0805S020S1R0?qs=jCA%252BPfw4LHbpkAoSnwrdjw%3D%3D) | [Datasheet](https://ro.mouser.com/datasheet/2/40/schottky-3165252.pdf) |
| | MBR0530 | Schottky Diode | [Link](https://ro.mouser.com/ProductDetail/onsemi-Fairchild/MBR0530?qs=VOMQJJE%252BBNniwJKcE3T43Q%3D%3D&srsltid=AfmBOorGETz4X7561cljdZB-BtAN4aC8w0OtRadqMgVwKqojAjDrxsqV) | [Datasheet](https://ro.mouser.com/datasheet/2/308/MBR0530_D-1810985.pdf) |
