# TSC
## Project: OpenBook  
**Author:** Teodor-Adrian Harea (333CA)  

### Overview  
A low-power embedded system integrating multiple modules for environmental data logging and visualization.  

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

## 4. Pinout Configuration  

### 4.1 Critical Interfaces  
| Function       | Pins Used               | Protocol |
|----------------|-------------------------|----------|
| Primary SPI    | GPIO8-11 (EPD/Flash)    | SPI      |
| Secondary SPI  | GPIO12-15 (SD Card)     | SPI      |
| I2C Bus        | GPIO2 (SDA), GPIO3 (SCL)| I2C      |

### 4.2 Special Pins  
- **System Control:**  
  - `RESET` (Hardware reset)  
  - `IO0` (Boot mode selector)  
- **Debugging:**  
  - `TX/RX` (UART console)  

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

<button onclick="navigator.clipboard.writeText(document.querySelector('pre').textContent)">Copy Documentation</button>

<style>
pre {
  background: #f1f3f4;
  padding: 16px;
  border-radius: 8px;
  border-left: 4px solid #4285F4;
  font-family: 'Roboto Mono', monospace;
}
button {
  background: #4285F4;
  color: white;
  border: none;
  padding: 8px 16px;
  border-radius: 4px;
  margin-top: 12px;
  cursor: pointer;
  transition: all 0.3s;
}
button:hover {
  background: #3367D6;
  box-shadow: 0 2px 6px rgba(0,0,0,0.2);
}
</style>
