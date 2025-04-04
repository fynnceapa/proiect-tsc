# Stan Andrei Razvan - Grupa 333CA
## Proiect Testarea Sistemelor de Calcul

Proiectul de față reunește mai multe componente hardware pentru a crea un Ebook Reader.

---

### 1. Schema bloc
![alt text](schema_bloc.jpg?raw=True, "Schema bloc")

### 2. Comunicare și Protocoale  

| Modul          | Interfață | Viteză           | Pini ESP32-C6               |
|----------------|-----------|------------------|-----------------------------|
| E-Paper        | SPI       | 10–40 MHz        | `EPD_CS`, MOSI, SCK         |
| BME688         | I2C       | 100–400 kHz      | SCL, SDA                    |
| DS3231 RTC     | I2C       | 100 kHz          | SCL, SDA                    |
| NOR Flash      | SPI/QSPI  | 133 MHz (QSPI)   | `FLASH_CS`, MOSI, SCK       |
| SD Card        | SPI       | 25 MHz           | `SS_SD`, MOSI, SCK          |

---

### 3. Placa de bază: Microcontroler ESP32-C6  
**Detalii tehnice principale:**  
- **Procesor:** RISC-V pe 32 de biți, compatibil cu Wi-Fi 6 (802.11ax) și Bluetooth 5  
- **Viteză de ceas:** până la 160 MHz  
- **Capacitate memorie:**  
  - RAM: 320 KB  
  - Flash intern de 512 KB, extensibil cu memorie NOR externă  
- **Conectivitate pini:** peste 30 GPIO, compatibili cu protocoale ca SPI, I2C, UART și USB 2.0  

---

### 4. Module și Interfețe  

#### **A. E-Paper Display**  
- **Model:** Probabil Waveshare sau similar (conform pinii EPD_*)  
- **Interfață:** SPI (4-wire: MOSI, SCK, MISO, EPD_CS)  
- **Pini dedicați:**  
  - `EPD_RST` (reset hardware)  
  - `EPD_DC` (comandă/date)  
  - `EPD_BUSY` (sincronizare)  
- **Consum:**  
  - ~100–200 mA la actualizare  
  - ~0 în stare statică (ideal pentru aplicații low-power)  

#### **B. Senzor Environmental: BME688**  
- **Funcționalități:** Măsoară temperatură, umiditate, presiune, VOC (compuși organici volatili)  
- **Interfață:** I2C (SCL, SDA)  
- **Specificații:**  
  - Rezoluție: ±0.5°C (temperatură), ±3% (umiditate)  
  - Consum: 2.1 µA în modul sleep, 3.6 mA în măsurare  

#### **C. RTC (Real-Time Clock): DS3231SN**  
- **Precizie:** ±2 ppm (±1 minut/an)  
- **Interfață:** I2C (SCL, SDA)  
- **Funcții:**  
  - Baterie backup (pin VBAT pentru mentinere timp)  
  - Alarmă și interrupt (`INT_RTC`)  

#### **D. Stocare Externă**  
- **NOR Flash:** W25Q512JVEIQ (64 MB)  
  - Interfață: SPI (`FLASH_CS`, MOSI, SCK, MISO)  
  - Viteză: Până la 133 MHz (QSPI)  
- **Slot SD Card:**  
  - Interfață: SPI (`SS_SD`, MOSI, SCK, MISO)  

#### **E. Power Management**  
- **Încărcător Baterie:** MCP73831 (Li-Po)  
  - Tensiune intrare: 5V (USB)  
  - Ieșire: 4.2V (baterie)  
- **LDO Regulator:** XC6220A331MR-G (3.3V pentru logică)  
- **Monitor Baterie:** MAX17048G+T10 (via I2C)  
  - Detectează nivelul bateriei (% și tensiune)  

---

### 5. Calcul Consum Energie  

- **Modul Activ (măsurători + afișaj):**  
  - ESP32-C6: ~80 mA (la 160 MHz)  
  - BME688: ~3.6 mA  
  - E-Paper: ~150 mA (la actualizare)  
  - **Total:** ~230 mA  

- **Modul Sleep (cu RTC activ):**  
  - ESP32-C6 în deep-sleep: ~5 µA  
  - DS3231: ~0.6 µA  
  - **Total:** ~6 µA  

---

### 6. Design Low-Power  

**Strategii:**  
- Folosirea deep-sleep cu trezire periodică (de la RTC sau senzor)  
- Dezactivarea SPI/I2C când nu sunt folosite  
- Alimentare selectivă a modulelor (ex.: afișajul este pornit doar la actualizare)  

**Autonomie estimată (cu baterie Li-Po 1000 mAh):**  
- 4–5 zile (cu actualizări la 10 minute)  

---

### Descriere detaliată a pinilor utilizați și a scopului lor  

#### **1. E-Paper Display**  
- `EPD_RST` (Reset): Pentru resetarea display-ului  
- `EPD_DC` (Data/Command): Selectează între modul de date și comenzi  
- `EPD_BUSY` (Busy): Semnalizează starea ocupat a display-ului  
- `EPD_CS` (Chip Select): Activează comunicarea SPI cu display-ul  
- `MOSI`, `SCK`, `MISO`: Pini SPI pentru transferul de date  

#### **2. NOR Flash Extern (W25Q512JVEIQ)**  
- `FLASH_CS` (Chip Select): Selectează memoria flash  
- `MOSI`, `SCK`, `MISO`: Pini SPI pentru citirea/scrierea datelor  

#### **3. RTC (DS3231SN)**  
- `SCL`, `SDA`: Pini I2C pentru comunicarea cu modulul RTC  
- `32KHZ`: Semnal de ceas pentru RTC  
- `INT_RTC` (Interrupt): Pentru notificări de alarmă sau evenimente  

#### **4. Senzorul Environmental (BME688)**  
- `SCL`, `SDA`: Pini I2C pentru citirea datelor de la senzor  

#### **5. SD Card**  
- `SS_SD` (Slave Select): Selectează cardul SD  
- `MOSI`, `SCK`, `MISO`: Pini SPI pentru transferul de date  

#### **6. Butoane (Reset, Boot, IO)**  
- `RESET`: Pentru resetarea microcontrolerului  
- `IO/BOOT`: Pentru intrarea în modul de boot  
- `IO/CHANGE`: Probabil pentru un buton de control suplimentar  

#### **7. USB și Comunicare Serială**  
- `USB_D+`, `USB_D-`: Pentru comunicarea USB  
- `TX`, `RX`: Pentru comunicarea UART (debug sau conectivitate)  

#### **8. Alimentare și GND**  
- `3V3`, `VBAT`, `GND`: Pentru alimentare și referințe de tensiune  

#### **9. Alti Pini**  
- `GPIO8`, `IO0-IO23`: Pini GPIO generali utilizați pentru diverse funcții

### **Bill of Materials (BOM) - DeskAssistant v19**

#### **Microcontrollers & Modules**
- **ESP32-C6-WROOM-1-N8** (WiFi/Bluetooth Module)
  - Mouser: [https://ro.mouser.com/ProductDetail/Espressif-Systems/ESP32-C6-WROOM-1-N8?qs=8Wlm6%252BaMh8ST02Gmwp74cw%3D%3D]()
  - Datasheet: [https://ro.mouser.com/datasheet/2/891/Espressif_ESP32_C6_WROOM_1__Datasheet_V0_1_PRELIMI-3239987.pdf]()

- **W25Q512JVEIQ** (64MB External NOR Flash)
  - Mouser: [https://ro.mouser.com/ProductDetail/Winbond/W25Q512JVEIQ?qs=l7cgNqFNU1jw6svr3at6tA%3D%3D]()
  - Datasheet: [https://ro.mouser.com/datasheet/2/949/Winbond_W25Q512JV_Datasheet-3240039.pdf]()

#### **Sensors**
- **BME688** (Environmental Sensor)
  - Mouser: [https://ro.mouser.com/ProductDetail/Bosch-Sensortec/Evaluation-Kit-Board-BME688?qs=QNEnbhJQKvZD%2Fs%2Ff0WWu0Q%3D%3D]()
  - Datasheet: [https://ro.mouser.com/datasheet/2/783/Bosch_sensortec_bme688_development_kit_flyer-3000177.pdf]()

- **DS3231SN#** (RTC Module)
  - Mouser: [https://ro.mouser.com/ProductDetail/Analog-Devices-Maxim-Integrated/DS3231SN?qs=1eQvB6Dk1vhUlr8%2FOrV0Fw%3D%3D]()
  - Datasheet: [https://ro.mouser.com/datasheet/2/609/DS3231-3421123.pdf]()

#### **Power Management**
- **MAX17048G+T10** (Battery Fuel Gauge)
  - Mouser: [https://eu.mouser.com/ProductDetail/Analog-Devices-Maxim-Integrated/MAX17048G+T10?qs=D7PJwyCwLAoGnnn8jEPRBQ%3D%3D&srsltid=AfmBOoq0Uf9CVLtvuYCdL-b56-Pbwotx2XvoNmNfB9vf0O63iEPsWucA]()
  - Datasheet: [https://eu.mouser.com/datasheet/2/609/MAX17048_MAX17049-3469099.pdf]()  

- **MCP73831** (Li-Po Battery Charger)
  - Mouser: [https://ro.mouser.com/ProductDetail/Microchip-Technology/MCP73831-2DCI-MC?qs=hH%252BOa0VZEiBQ%2FrptDRXKdg%3D%3D]()
  - Datasheet: [https://ro.mouser.com/datasheet/2/268/MCP73831_Family_Data_Sheet_DS20001984H-3441711.pdf]()

- **XC6220A331MR-G** (LDO Voltage Regulator 3.3V)
  - Mouser: [https://ro.mouser.com/ProductDetail/Torex-Semiconductor/XC6220A331MR-G?qs=AsjdqWjXhJ8ZSWznL1J0gg%3D%3D]()
  - Datasheet: [https://ro.mouser.com/datasheet/2/760/xc6220-3371556.pdf]()

#### **Connectors & Protection**
- **USBLC6-2SC6Y** (USB ESD Protection)
  - Mouser: [https://eu.mouser.com/ProductDetail/STMicroelectronics/USBLC6-2SC6Y?qs=gNDSiZmRJS%2FOgDexvXkdow%3D%3D&srsltid=AfmBOooGFMy49XJWMdHSBGK6U_FDXdeHD6KsINSiWZt-au9axwzb1pgh]()
  - Datasheet: [https://ro.mouser.com/datasheet/2/389/usblc6_2sc6y-1852505.pdf]()

- **FH34SRJ-24S-0.5SH** (24-pin Connector)
  - Mouser: [https://ro.mouser.com/ProductDetail/Hirose-Connector/FH34SRJ-24S-0.5SH50?qs=iyLo5FA4poC8fzWlavnA7A%3D%3D]()
  - Datasheet: [https://ro.mouser.com/datasheet/2/185/FH34SRJ_24S_0_5SH_50__CL0580_1255_6_50_2DDrawing_0-1615030.pdf]()

#### **Passive Components**
- **SD0805S020S1R0** (68μH Inductor)
  - Mouser: [https://ro.mouser.com/ProductDetail/KYOCERA-AVX/SD0805S020S1R0?qs=jCA%252BPfw4LHbpkAoSnwrdjw%3D%3D]()
  - Datasheet: [https://ro.mouser.com/datasheet/2/40/schottky-3165252.pdf]()

- **MBR0530** (Schottky Diode)
  - Mouser: [https://ro.mouser.com/ProductDetail/onsemi-Fairchild/MBR0530?qs=VOMQJJE%252BBNniwJKcE3T43Q%3D%3D&srsltid=AfmBOorGETz4X7561cljdZB-BtAN4aC8w0OtRadqMgVwKqojAjDrxsqV]()
  - Datasheet: [https://ro.mouser.com/datasheet/2/308/MBR0530_D-1810985.pdf]()
