# Proiect-TSC

## Descriere generală

InkTime este un smartwatch construit în jurul microcontroller-ului nRF52840, utilizând un afișaj E-Paper. Proiectul este dezvoltat în Autodesk Fusion: de la concepția schemelor și rutarea unui PCB pe 4 straturi, până la integrarea mecanică finală în carcasă.

Principalele caracteristici:
- Procesare: nRF52840 (Cortex-M4F) cu conectivitate Bluetooth 5.0 integrată
- Afișaj: Ecran E-Paper (SPI)
- Senzori: Accelerometru BMA423 pentru pedometru și gesturi
- Baterie LiPo cu circuit de încărcare dedicat (BQ25180).
- Regulator DC/DC (RT6160) pentru o tensiune stabilă de 3.3V
- Fuel Gauge (MAX17048) pentru monitorizarea nivelului bateriei
- Driver haptic (DRV2605) pentru feedback
- Conectivitate USB-C pentru încărcare
- 3 butoane fizice pentru interacțiune

---

## Diagrama bloc

```text
                         +-------------------+
                         |    Baterie LiPo   |
                         +--------+----------+
                                  |
                         +--------v----------+
                         |  LiPo Charger     |  <-- USB-C (5V / VBUS)
                         |  BQ25180YBGR      |
                         +--------+----------+
                                  | VBAT (3.7-4.2V)
                         +--------v----------+
                         |  Fuel Gauge       |  <-- I2C --> nRF52840
                         |  MAX17048G+T10    |
                         +--------+----------+
                                  |
                         +--------v----------+
                         |  DC/DC Regulator  |  <-- I2C --> nRF52840
                         |  RT6160AWSC       |
                         +--------+----------+
                                  | 3V3
          +-----------------------+-----------------------+
          |                       |                       |
+---------v---------+   +---------v---------+   +---------v---------+
|   nRF52840 (U1)   |   |  ESD Protection   |   |   Butoane (x3)    |
|   Microcontroller |   |  USBLC6-2SC6Y     |   |   EVP-AKE31A      |
+---------+---------+   +-------------------+   +-------------------+
          |
    +-----+------+------+------+
    |            |      |      |
+---v---+  +----v--+  +-v---+ +-v---------+
| SPI   |  |  I2C  |  | SWD | | GPIO/ANT  |
+---+---+  +---+---+  +-----+ +-----------+
    |          |
+---v---+   +--+--+--+--+
|E-Paper|   | IMU | Haptic|
|Display|   |BMA423|DRV2605|
+-------+   +-------+-----+
```

---
## Bill of Materials (BOM)

| Cantitate | Componenta | Descriere | Referinta | 
|:---------:|:----------:|:---------:|:---------:|
| 1 | nRF52840 | Microcontroller ARM Cortex-M4, BT 5.0 | U1 | 
| 1 | BQ25180YBGR | LiPo Charger IC, 8-DSBGA | IC1 | 
| 1 | DRV2605YZFR | Haptic Driver ERM/LRA, 9-BGA | IC2 |
| 1 | BMA423 | IMU Accelerometru triaxial 12-bit | IC3 |
| 1 | MAX17048G+T10 | Fuel Gauge 1-Cell ModelGauge | U3 | 
| 1 | RT6160AWSC | Buck-Boost DC/DC Regulator I2C, 15-WL-CSP | IC9 | 
| 1 | USBLC6-2SC6Y | ESD Protection TVS Diode, SOT-23-6 | D3 | 
| 1 | KH-TYPE-C-16P | Conector USB-C 16 pini | J4 |
| 1 | 503480-2400 | Conector FPC/FFC 0.5mm 24 circuite | J1 | 
| 1 | DMG2305UX-7 | P-Channel MOSFET 20V/4.2A SOT-23 | Q2 | 
| 1 | SI1308EDL-T1-GE3 | N-Channel MOSFET 30V/1.5A SC-70 | Q3 | 
| 3 | MBR0530 | Dioda Schottky 30V/500mA SOD-123 | D1, D2, D5 | 
| 3 | EVP-AKE31A | Buton tactil SMD ultra-subtire | SW_DN, SW_ENT, SW_UP | 
| 1 | 2450AT18B100E | Antena 2.45GHz SMD | ANT1 | 
| 1 | FTC252012SR47MBCA | Inductor SMD 0.47uH 2016 | L7 | 
| 1 | 744043680 | Inductor SMD 68uH WE-TPC | L5 |
| 1 | TC2030-IDC | Conector Tag-Connect 6 pini SWD | J2 |
| 1 | X1 (32MHz) | Crystal 32MHz 2016 SMD | X1 |
| 1 | X2 (32.768kHz) | Crystal 32.768kHz 3215 SMD | X2 | 
| ~50 | Condensatoare SMD | 0201/0402, diverse valori (1pF - 22uF) | Cx | 
| ~15 | Rezistoare SMD | 0201, diverse valori (2.2Ω - 10kΩ) | Rx |
| ~4 | Inductoare SMD | 0402, diverse valori (3.9nH - 15nH, 10uH) | Lx |
| 14 | Test Pad TP20R | Test pad-uri SMD 2mm | TP_* |
| 1 | Solder Jumper SJ | Jumper SMD | SJ1 |

---

## Funcționalitate hardware

#### Componente și roluri
Sistemul este coordonat de nRF52840, care gestionează atât logica internă, stările de sleep pentru economisirea energiei, cât și comunicarea Bluetooth.

* Alimentare și Power Management: Bateria Li-Po este conectată direct la test pad-urile plăcii pentru a economisi spațiu pe verticală. Încărcarea este gestionată de BQ25180. Tensiunea bateriei este monitorizată precis de Fuel Gauge-ul MAX17048, iar alimentarea componentelor logice la 3.3V este asigurată de convertorul DC/DC RT6160 (ales în detrimentul unui LDO pentru un randament energetic superior).

* Interfața cu utilizatorul: Afișajul E-Paper este controlat prin SPI pentru o viteză mare de transfer, consumând energie doar la actualizarea imaginii. Input-ul se face prin 3 butoane tactile cu debouncing software, iar feedback-ul este asigurat de driverul haptic DRV2605 conectat la un actuator de tip shaker.

* Senzori: Accelerometrul BMA423 este legat pe magistrala I2C și este folosit pentru funcții precum ridicarea mâinii pentru aprindere (wake-on-wrist).

#### Interfețe de comunicație
* I2C: LiPo Charger, DC/DC, IMU, Fuel Gauge, Haptic Driver. (Folosește rezistențe de pull-up externe pentru stabilitate).

* SPI: E-Paper Display Connector (viteză mare de transmisie a datelor grafice).

---

## Pinii nRF52840

| Pin nRF52840 | Semnal | Modul conectat | Interfata | Descriere |
| :--------- | :--------- | :--------- | :--------- | :--------- |
| P0.02/AIN0 | SCK | E-Paper Display | SPI | Clock serial SPI |
| P0.03/AIN1 | MOSI | E-Paper Display | SPI | Date catre display (Master Out Slave In) |
| P0.05/AIN3 | CS | E-Paper Display | SPI | Chip Select display |
| P0.15 | EPD_RST | E-Paper Display | GPIO | Reset display |
| P0.16 | EPD_DC | E-Paper Display | GPIO | Data/Command select |
| P0.17 | EPD_BUSY | E-Paper Display | GPIO | Stare ocupata display |
| P0.06 | SCL | IMU, LiPo Charger, DC/DC, Fuel Gauge, Haptic | I2C | Clock linie I2C |
| P0.07 | SDA | IMU, LiPo Charger, DC/DC, Fuel Gauge, Haptic | I2C | Date linie I2C |
| P0.08 | IMU_INT2 | BMA423 | GPIO | Intrerupere 2 accelerometru |
| P1.08 | IMU_INT1 | BMA423 | GPIO | Intrerupere 1 accelerometru |
| P0.10/NFC2 | FUELGAUGE_ALT | MAX17048 | GPIO | Alarma nivel scazut baterie |
| P0.11 | CHARGER_PG | BQ25180 | GPIO | Semnal Power Good charger |
| P0.12 | HAPTIC_EN | DRV2605 | GPIO | Enable driver haptic |
| P0.13 | BTN_UP | SW_UP | GPIO | Buton sus |
| P0.14 | BTN_ENT | SW_ENT | GPIO | Buton enter/confirmare |
| P1.02 | BTN_DN | SW_DN | GPIO | Buton jos |
| D+ / D- | USB_DP / DM | USB-C / ESD | USB | Date USB diferentiale |
| VBUS | 5V USB | LiPo Charger | - | Alimentare 5V din USB |
| SWDIO / CLK | SWDIO / CLK| TC2030-IDC (J2) | SWD | Date programare/debug |
| SWO | SWO | TC2030-IDC (J2) | SWD | Trace output |
| P0.18/RESET | RESET | TC2030-IDC (J2) | GPIO | Reset hardware microcontroller |
| ANT | RF | 2450AT18B100E | RF | Antena Bluetooth 2.4GHz |
| XC1, XC2 | XTAL_32M | X1 (32 MHz) | Clock | Sursa de ceas principala |
| XL1, XL2 | XTAL_32K | X2 (32.768 kHz) | Clock | Sursa de ceas RTC (low power) |

---
## Design PCB si Modelare Mecanică 3D


### Stackup PCB — 4 straturi
Placa a fost realizată pe 4 straturi pentru a asigura o integritate a semnalului excelentă și o rutare compactă:

* Top (L1): Rutare semnal + componente SMD.

* L2: Plan de masă (GND solid) care oferă un return path scurt pentru toate semnalele.

* L3: Rutare secundară — acest strat a fost utilizat în principal pentru rezolvarea manuală a Airwire-urilor rămase în urma procesului de autorutare, asigurând finalizarea tuturor conexiunilor complexe.

* Bottom (L4): Rutare semnal secundar.

### Decizii de design notabile
* Decupaj Antenă (RF): Antena ceramică este poziționată pe marginea plăcii. Sub aceasta s-a realizat un decupaj complet al substratului pe toate planurile (ground clearance) pentru performanță radio optimă.

* Via-in-Pad: Pentru componentele BGA dense, a fost utilizată tehnologia via-in-pad pentru a salva spațiu, rutând semnalele direct din pad-uri.

* Pad-uri de Test: Au fost adăugate 14 test pad-uri pentru semnalele vitale (3V3, VBAT, SWD, I2C), pentru lipirea directă a bateriei.