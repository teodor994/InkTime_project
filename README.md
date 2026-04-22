Nume: Vica Teodor Andrei
Grupa: 331CB

# InkTime Project

Acest proiect constă în proiectarea unui sistem hardware bazat pe microcontroller-ul nRF52840, utilizând platforma Fusion 360 Electronics. În cadrul proiectului au fost realizate schema electrică și layout-ul PCB, incluzând plasarea componentelor și rutarea conexiunilor principale.

Scopul proiectului este dezvoltarea unei baze hardware funcționale care poate fi utilizată ulterior pentru aplicații embedded ce implică comunicație wireless și interfațare cu senzori.


# Diagrama bloc a proiectului

```text
                                +----------------------+
                                |       USB-C          |
                                |   alimentare / USB   |
                                +----------+-----------+
                                           |
                                           v
                                +----------------------+
                                |   ESD Protection     |
                                |    USBLC6-2SC6Y      |
                                +----------+-----------+
                                           |
                                           v
                                +----------------------+
                                |  Li-Po Charger       |
                                |    BQ25180YBGR       |
                                +----+------------+----+
                                     |            |
                                     |            +--------------------+
                                     |                                 |
                                     v                                 v
                           +-------------------+             +-------------------+
                           |    Li-Po Battery  |             |   Fuel Gauge      |
                           |     LP502030      |<-- I2C ---> |    MAX17048       |
                           +---------+---------+             +-------------------+
                                     |
                                     v
                           +-------------------+
                           |  Buck-Boost DC/DC |
                           |    RT6160AWSC     |
                           +---------+---------+
                                     |
                                   3V3 / VREG
                                     |
        +----------------------------+------------------------------+
        |                            |                              |
        v                            v                              v
+---------------+          +-------------------+          +-------------------+
|   nRF52840    |<--I2C--->|      BMA421       |<--I2C--->|     DRV2605       |
| BLE MCU       |          |       IMU         |          |   Haptic Driver   |
+-------+-------+          +-------------------+          +---------+---------+
        |                                                              |
        | SPI + GPIO                                                    v
        v                                                      +----------------+
+-------------------+                                          | Vibration Motor |
| 1.54" E-Paper     |                                          |     shaker      |
| via FPC connector |                                          +----------------+
+-------------------+
        |
        +--> etaj dedicat pentru EPD:
             MOSFET-uri + diode Schottky + inductoare + condensatori

```

## Bill of Materials (BOM)

| Componenta | Part number / valoare | Rol | Observatii |
| ---------- | --------------------- | --- | ---------- |
| MCU | NRF52840_QF | microcontroller principal | BLE + control general |
| Charger | BQ25180YBGR | incarcarea bateriei Li-Po | conectat la VBUS |
| DC/DC | RT6160AWSC | conversie VBAT -> 3V3 / VREG | alimentare sistem |
| Fuel Gauge | MAX17048G+T10 | monitorizare baterie | interfata I2C |
| IMU | BMA421 | accelerometru | interfata I2C |
| Haptic Driver | DRV2605YZFR | comanda vibratie | interfata I2C |
| USB ESD | USBLC6-2SC6Y | protectie ESD pentru USB | pe liniile USB |
| Antena | 2450AT18B100E | antena 2.4 GHz | BLE |
| USB-C | KH-TYPE-C-16P | alimentare / USB | conector exterior |
| SWD | TC2030-IDC | programare / debugging | Tag-Connect |
| Conector display | 503480-2400 | conector FPC pentru display | 24 pini |
| Cristal HF | 32MHz | clock principal | pentru nRF52840 |
| Cristal LF | 32.768kHz | RTC / low-power clock | pentru moduri low-power |
| Q1 | DMG2305UX-7 | MOSFET in etajul EPD | drive display |
| Q3 | SI1308EDL-T1-GE3 | MOSFET in etajul EPD | drive display |
| D2 / D4 / D5 | MBR0530 | diode Schottky | etaj EPD |
| L5 | 68uH | inductor pentru etajul EPD | boost display |
| L7 | FTC252012SR47MBCA | inductor pentru RT6160 | 0.47uH |
| Baterie | LP502030 | sursa principala | conectata la test pads |
| Display | 1.54" e-paper | afisaj principal | prin FPC |
| Butoane | EVP-AKE31A | input utilizator | 3 bucati |

Descriere hardware
1. Microcontroller

Microcontroller-ul principal al proiectului este nRF52840 (U1). Acesta este componenta centrala a sistemului si se ocupa de:

comunicatia BLE
controlul display-ului e-paper
comunicatia cu senzorii si circuitele de power prin I2C
citirea butoanelor
controlul driverului haptic
interfata de debug SWD

Pentru clock sunt folosite doua cristale:

X1 = 32MHz
X2 = 32.768kHz

Cristalul de 32MHz este folosit pentru functionarea principala a MCU-ului si pentru partea de radio, iar cristalul de 32.768kHz este folosit pentru RTC si modurile low-power.

2. Alimentare si management energetic

Arhitectura de alimentare este impartita in trei blocuri principale:

Charger - BQ25180YBGR

IC1 = BQ25180YBGR se ocupa de incarcarea bateriei Li-Po din USB-C. Intrarea de 5V vine din VBUS, iar iesirea merge spre baterie. Circuitul are si semnal de interrupt catre MCU pentru monitorizarea starii de incarcare.

Fuel Gauge - MAX17048

U3 = MAX17048G+T10 monitorizeaza tensiunea bateriei si comunica prin I2C cu nRF52840. Are si semnal de alerta catre MCU pentru praguri critice sau schimbari de stare.

DC/DC - RT6160AWSC

IC9 = RT6160AWSC este convertorul buck-boost care genereaza alimentarea stabila 3V3 / VREG pentru sistem. Acesta este necesar deoarece tensiunea bateriei variaza in functie de starea de incarcare, iar sistemul are nevoie de o alimentare stabila.

3. Bateria

Bateria folosita este de tip LP502030, 3.7V, aproximativ 250mAh. In proiect, bateria nu este conectata prin conector JST, ci direct la doua test pad-uri:

TP_VBAT
TP_BAT_GND

Aceasta solutie a fost aleasa pentru economie de spatiu, conform cerintei proiectului.

4. Display e-paper

Display-ul este conectat prin conectorul 503480-2400 si este controlat prin:

SCK
MOSI
EPD_CS
EPD_DC
EPD_RST
EPD_BUSY

Pe langa partea de comunicatie, display-ul are si un etaj separat de alimentare / drive realizat cu:

MOSFET-uri
diode Schottky
inductor
condensatori

Acest etaj este necesar pentru generarea tensiunilor speciale cerute de panoul e-paper.

5. IMU

IMU-ul folosit este BMA421 (IC3) si comunica prin I2C. Are doua semnale de interrupt:

IMU_INT1
IMU_INT2

Aceste semnale sunt conectate la MCU pentru detectarea rapida a evenimentelor de miscare si pentru trezirea sistemului din moduri low-power.

6. Haptic

Pentru feedback haptic este folosit DRV2605YZFR (IC2), care comanda motorul de vibratie. Comunicarea este facuta pe I2C, iar activarea poate fi controlata si printr-un semnal GPIO dedicat.

7. USB-C si protectie ESD

Conectorul J4 = KH-TYPE-C-16P este folosit pentru alimentare si optional pentru interfata USB. Protectia ESD este facuta cu USBLC6-2SC6Y, pentru a proteja liniile sensibile la descarcari electrostatice.

8. RF si antena

Antena este ANT1 = 2450AT18B100E si este conectata la pinul RF al nRF52840 printr-o retea de matching. Antena este pusa la marginea placii, iar sub ea am lasat zona fara plan de masa si fara trasee, conform regulilor de proiectare RF.


PCB si layout
Board specifications
dimensiunea placii: 46 mm x 35 mm
grosimea PCB-ului: 1 mm
componentele sunt plasate pe TOP
rutarea este facuta pe Top si Bottom
exista poligoane de GND pe Top si Bottom
Routing rules
traseele de semnal au fost rutate cu minimum 0.15 mm
traseele de putere au fost rutate cu 0.3 mm acolo unde spatiul a permis
in zonele foarte dense, unele segmente scurte au trebuit ingustate local
Ground planes

Au fost folosite planuri de masa pe placa, iar in zonele importante a fost aplicat si via stitching, mai ales in jurul zonei RF si al microcontroller-ului.

Antena si keepout

Antena este pusa la marginea PCB-ului, iar sub ea:

nu exista plan de masa
nu exista trasee
nu exista turnare de cupru

Acest lucru a fost facut pentru a nu afecta performanta RF.




