# OpenBook Project for Systems Testing
# Stan Andrei Razvan (Group 333CA)  

## Block Diagram
![Block Diagram](schema_bloc.jpg?raw=True "System Block Diagram")  

## Core Components  

### 1. ESP32-C6 Microcontroller (Main Board)  
- **Processor:** 32-bit RISC-V @ 160MHz  
- **Wireless:** Wi-Fi 6 (802.11ax) + Bluetooth 5  
- **Memory:**  
  - 320KB RAM  
  - 512KB internal Flash (expandable via external NOR Flash)  
- **I/O:** 30+ GPIOs supporting SPI, I2C, UART, USB 2.0  

### 2. Peripheral Modules  

| Module          | Interface | Speed       | Key ESP32-C6 Pins       |  
|-----------------|-----------|-------------|-------------------------|  
| E-Paper Display | SPI       | 10-40 MHz   | `EPD_CS`, MOSI, SCK     |  
| BME688 Sensor   | I2C       | 400 kHz     | SCL, SDA                |  
| DS3231 RTC      | I2C       | 100 kHz     | SCL, SDA                |  
| NOR Flash       | SPI/QSPI  | 133 MHz     | `FLASH_CS`, MOSI, SCK   |  
| SD Card         | SPI       | 25 MHz      | `SS_SD`, MOSI, SCK      |  

## Module Specifications  

### A. E-Paper Display  
- **Type:** SPI-based (e.g., Waveshare)  
- **Key Pins:**  
  - `EPD_RST`: Hardware reset  
  - `EPD_DC`: Data/command control  
  - `EPD_BUSY`: Status monitoring  
- **Power:**  
  - 150mA (active refresh)  
  - 0mA (static display)  

### B. BME688 Environmental Sensor  
- **Metrics:** Temperature, humidity, pressure, VOC  
- **Accuracy:** ±0.5°C, ±3% RH  
- **Power:** 3.6mA (active), 2.1µA (sleep)  

### C. DS3231 Real-Time Clock  
- **Precision:** ±2ppm (±1min/year)  
- **Features:** Battery backup (`VBAT`), alarm interrupts (`INT_RTC`)  

### D. Storage Systems  
- **NOR Flash (W25Q512JVEIQ):**  
  - 64MB capacity  
  - QSPI @ 133MHz  
- **SD Card:**  
  - SPI interface @ 25MHz  

### E. Power Management  
- **Charger:** MCP73831 (Li-Po, 5V USB → 4.2V)  
- **LDO Regulator:** XC6220A331MR-G (3.3V logic)  
- **Battery Monitor:** MAX17048G+T10 (I2C)  

## Power Analysis  
| Mode             | Current Draw | Runtime* |  
|------------------|--------------|----------|  
| Active Operation | ~230mA       | ~4h      |  
| Deep Sleep       | ~6µA         | ~2 years |  

*With 1000mAh Li-Po battery  

## Low-Power Design Strategies  
- Deep-sleep mode with RTC wake-up  
- Selective peripheral power control  
- SPI/I2C bus deactivation when idle  

## Pinout Summary  

| Function            | Key Pins                 | Purpose                          |  
|---------------------|--------------------------|----------------------------------|  
| **E-Paper**         | `EPD_CS`, `EPD_DC`       | Display control                  |  
| **NOR Flash**       | `FLASH_CS`               | External memory access           |  
| **RTC**            | `INT_RTC`, `32KHZ`       | Timekeeping & alarms             |  
| **BME688**         | `SCL`, `SDA`             | Environmental sensing            |  
| **Power**          | `VBAT`, `3V3`            | Battery & regulated power        |  

## Key Features  
- Energy-efficient e-ink display  
- Environmental monitoring  
- Reliable timekeeping with battery backup  
- Expandable storage (64MB Flash + SD card)  
- Optimized for long battery life  
