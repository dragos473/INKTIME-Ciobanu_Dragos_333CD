# InkTime – Smartwatch Open Source

InkTime este un smartwatch open-source construit in jurul microcontroller-ului nRF52840. Are display e-paper de 1.54", feedback haptic, si incarcare prin USB-C.

## Diagrama Bloc

```
                         +-----------+
        USB-C ──VBUS──>──| BQ25180   |──VBAT──>──[ Baterie LiPo ]
        (J3)             | (Charger) |              |
          |              +-----------+               |
          |                   |                      |
          |                   v                      v
          |              +-----------+         +-----------+
          |              | RT6160A   |<──VREG──| MAX17048  |
          |              | (DC/DC)   |         | (Fuel     |
          |              +-----------+         |  Gauge)   |
          |                   |                +-----------+
          |                 3V3                      |
          |                   |                   ALERT
          v                   v                      |
     +--------------------------------------------+  |
     |               nRF52840                      |<-+
     |                                             |
     |   SPI ──> Display E-Paper (conector FPC J2) |
     |   I2C ──> BMA423 (IMU)                      |
     |   I2C ──> DRV2605 (Haptic Driver)           |
     |   I2C ──> BQ25180 / RT6160A / MAX17048      |
     |   GPIO ──> 3x Butoane (UP/ENTER/DOWN)       |
     |   USB ──> USB-C (D+/D-)                     |
     |   RF ──> Antena 2.4GHz (BLE)                |
     +---------------------------------------------+
```

## Descriere Hardware

### Microcontroller – nRF52840

nRF52840 e IC-ul principal al proiectului. E un SoC cu procesor Cortex-M4F, are 1MB flash si 256kB RAM si merge pana la 64MHz. Suporta BLE 5.0 si USB 2.0. Toate perifericele de pe placa comunica cu el prin I2C, SPI sau GPIO.

### Alimentare

Lantul de alimentare arata asa: USB-C (VBUS, 5V) → BQ25180 (incarcator LiPo) → baterie LiPo (VBAT) → RT6160A (convertor DC/DC buck-boost) → linia de 3V3.

### Display E-Paper

Display-ul se conecteaza printr-un conector FPC cu 24 de pini. Circuitul de drive pentru e-paper foloseste un boost converter ca sa genereze tensiunile mari (PREVGH si PREVGL) de care are nevoie panoul. Un P-MOSFET (DMG2305UX) controlat de pinul P1.01 porneste/opreste alimentarea circutului de display, ca sa poata fi oprit complet cand nu se face refresh.

Semnalele SPI (SCK, MOSI, CS) si GPIO-urile de control (DC, RST, BUSY) merg direct la nRF52840. Fiecare linie de tensiune mare are condensatoare de decuplare de 1µF/50V.

### IMU – BMA423

BMA423 e un accelerometru triaxial folosit pentru numararea pasilor si detectarea gesturilor. Comunica pe I2C. Pinul SDO e legat la GND printr-un rezistor de 0Ω (R3), ceea ce seteaza adresa I2C la 0x18. CSB e legat la VDD ca sa selecteze modul I2C. INT1 si INT2 merg la nRF52840 pentru interrupt-uri.

### Haptic Driver – DRV2605

DRV2605 controleaza un motor LRA/ERM pentru feedback haptic. E controlat prin I2C si activat prin GPIO (P0.12 → EN). Pinul IN/TRIG e legat la GND (nu se foloseste in modul I2C). OUT+ si OUT- se conecteaza la motor prin test pad-uri.

### USB-C si Protectie ESD

Conectorul USB-C (KH-TYPE-C-16P) furnizeaza alimentare (VBUS) si date (D+, D-). Doua rezistoare de 5.1kΩ pe CC1 si CC2 identifica dispozitivul ca UFP (sink). USBLC6-2SC6Y asigura protectie ESD pe liniile de date. D+/D- se conecteaza la controller-ul USB intern al nRF52840.

### Butoane

Trei butoane tactile (EVP-AKE31A): UP (P0.13), ENTER (P0.14), DOWN (P1.02). Fiecare buton are un pull-up de 10kΩ la 3V3 si un condensator de debounce de 1µF la GND. Cand e apasat, butonul trage pinul la GND (active-low).

### Debug SWD

Un footprint TC2030-IDC (tag-connect) ofera acces SWD (SWDIO, SWDCLK, RESET, VCC, GND). Pe placa sunt si test pad-uri pentru SWO, SWDIO, SWDCLK, RESET, 3V3 si GND.

### Magistrala I2C

Toate perifericele I2C sunt pe acelasi bus, pe pinii P0.06 (SDA) si P0.07 (SCL). Rezistoarele de pull-up sunt de 3.3kΩ, conectate la 3V3.

| Dispozitiv | Adresa I2C | Functie |
|------------|------------|---------|
| BQ25180 | 0x6A | Incarcator baterie |
| RT6160A | 0x75 | Convertor DC/DC |
| MAX17048 | 0x36 | Fuel gauge |
| BMA423 | 0x18 | Accelerometru |
| DRV2605 | 0x5A | Driver haptic |
