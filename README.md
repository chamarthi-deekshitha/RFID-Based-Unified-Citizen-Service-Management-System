# RFID-Based Unified Citizen Service Management System

> An embedded-C RFID access and citizen-service terminal for the NXP LPC21xx ARM7 family.

The system reads an RFID card over UART, authenticates the card ID, stores persistent state in an SPI EEPROM, and presents citizen or officer services through a 20×4 LCD and 4×4 keypad. It is designed and demonstrated on real hardware—not as a simulator.
## Hardware Architecture
<p align="center">
<img width="1600" height="878" alt="hardware-overview" src="https://github.com/user-attachments/assets/66013e49-59ca-4f7d-8750-bffbe081be12" />

" alt="RFID citizen service system on an LPC2148 development board" width="820">
</p>

## Highlights

- **RFID authentication** — valid citizen, officer, and unrecognized card flows with LCD, LED, buzzer, and UART feedback.
- **Citizen dashboard** — PAN-style details, ATM balance/withdrawal/deposit flow, voting status, and driving-licence information.
- **Officer controls** — voting reset, RTC adjustment, and driving-licence-expiry maintenance.
- **Persistent data** — account balances, voting flags, and PINs are retained in a 25LC512-compatible SPI EEPROM.
- **Responsive UI** — keypad input uses debounce handling and timeout-aware scanning; RFID reception is interrupt-driven.

## System architecture

<p align="center">
 <img width="561" height="441" alt="system-architecture" src="https://github.com/user-attachments/assets/d8035e71-5e69-47ab-8386-b1b945458def" />

</p>

| Block | Role |
| --- | --- |
| LPC21xx ARM7 MCU | Runs the embedded-C firmware, UI, authentication, and peripheral control. |
| RFID reader | Sends card frames to UART0 at 9,600 baud. |
| 4×4 keypad | Inputs menu options, PINs, amounts, and date/time values. |
| 20×4 HD44780 LCD | Displays prompts, authentication status, menus, and citizen records. |
| 25LC512 EEPROM | Preserves balances, votes, and PIN data across power cycles. |
| LEDs and buzzer | Give immediate success, failure, and officer-access feedback. |
| RTC | Maintains date and time, with officer-controlled editing. |

## Demonstrated workflow

<p align="center">
 <img width="1600" height="858" alt="scan-ready" src="https://github.com/user-attachments/assets/dcd682b1-f09b-4aaf-be0d-ae41e1e2446f" /
  <img width="1280" height="573" alt="carddetails" src="https://github.com/user-attachments/assets/9ab9e00a-6348-429b-a363-22bfdd9d67c1" />


  </p>

<p align="center">
  <img src="docs/images/card-valid.jpg" alt="LCD confirming a valid card" width="48%">
</p>

1. Power on the system; it shows the RFID scan prompt.
2. Present a registered citizen card to open the citizen dashboard.
3. Use the keypad to view records or use the ATM, voting, and licence services.
4. Present the officer card for administrative controls.
5. Unknown cards are rejected with the red LED and buzzer.

## LCD interface

<p align="center">
  <img width="1280" height="960" alt="lcd-menu" src="https://github.com/user-attachments/assets/88be96b6-39e5-458b-844a-d2119d32b8b4" />

  <img src="docs/images/record-view.jpg" alt="Citizen PAN record on LCD" width="31%">
  <img src="docs/images/admin-menu.jpg" alt="ATM operations menu on LCD" width="31%">
</p>

<p align="center">
  <img src="docs/images/pin-entry.jpg" alt="PIN entry screen on LCD" width="31%">
  <img src="docs/images/lcd-time-settings.jpg" alt="Date and time configuration screen on LCD" width="31%">
  <img src="docs/images/officer-card.jpg" alt="Officer RFID card confirmation screen on LCD" width="31%">
</p>

<p align="center">
  <img src="docs/images/settings-menu.jpg" alt="System time and licence settings menu on LCD" width="31%">
</p>

## RFID cards

<p align="center">
  <img src="docs/images/rfid-card-anatomy.jpg" alt="RFID card showing its antenna and integrated circuit" width="38%">
  <img src="docs/images/rfid-cards.jpg" alt="RFID cards used for the project" width="38%">
</p>

The firmware expects RFID frames delimited by **STX** (`0x02`) and **ETX** (`0x03`) and compares the first eight received ID characters with its configured records. Use only authorised test cards when modifying the card database.

## Wiring reference

This table is derived from the firmware in this repository—not copied from the reference project. Verify every connection against the labels on your specific development board before applying power.

| Peripheral | LPC21xx pins / interface | Firmware location |
| --- | --- | --- |
| RFID reader | UART0: **P0.0 / TxD0**, **P0.1 / RxD0**, 9,600 baud | `uart.c` |
| SPI EEPROM | SPI0: **P0.4 / SCK0**, **P0.5 / MISO0**, **P0.6 / MOSI0**; **P0.7** chip select | `spi.c`, `spi_eeprom.c` |
| LCD data bus | **P0.8–P0.15** (D0–D7) | `lcd_defines.h` |
| LCD control | **P0.16 / RS**, **P0.17 / RW**, **P0.18 / EN** | `lcd_defines.h` |
| Status outputs | **P0.19** buzzer, **P0.20** red LED, **P0.21** green LED | `Rfid_test.c` |
| 4×4 keypad | **P1.16–P1.19** rows, **P1.20–P1.23** columns | `kpm_defines.h` |
| Officer/RTC-edit interrupt | **P0.30 / EINT3**, falling edge | `Rfid_test.c` |

> **Configured target:** this repository is configured for the **NXP LPC2148** (ARM7TDMI, 512 KB flash, 32 KB RAM), matching the photographed development board. Before flashing, still confirm that your board carries an LPC2148 and that its ISP/programmer settings match your hardware.

## Build and flash

### Requirements

- Keil µVision / MDK with ARM7 and LPC21xx device support
- An LPC21xx board, RFID reader, 20×4 HD44780-compatible LCD, 4×4 matrix keypad, SPI EEPROM, LEDs, and buzzer
- A compatible programmer or the board's ISP bootloader connection

### Steps

1. Clone or download this repository.
2. Open `RFID_PROJECT.uvproj` in Keil µVision.
3. Confirm that **Target 1** is set to LPC2148.
4. Confirm the connections in the wiring table, then build **Target 1**.
5. Flash the generated `RFID_PROJECT.hex` using your board's supported programming method.
6. Power the hardware and test a known citizen or officer card.

## Source layout

| File | Responsibility |
| --- | --- |
| `Rfid_test.c` | Application entry point, initialisation, RFID authentication, indicators, and interrupt setup. |
| `user_menu.c` | Citizen/officer menus, PIN flow, voting, ATM operations, driving-licence data, and RTC editing. |
| `uart.c` | UART0 setup and interrupt-driven RFID frame reception. |
| `spi.c`, `spi_eeprom.c` | SPI0 transport and 25LC512 EEPROM read/write operations. |
| `lcd.c` | HD44780 LCD driver. |
| `keypad.c` | 4×4 keypad scanning and debounce handling. |
| `delay.c` | Timing helpers. |

## Important notes

- The sample card IDs, names, balances, and PIN values are demonstration data. Replace them before any real deployment.
- This project is appropriate for education and laboratory demonstration. It is **not** a production-grade identity, banking, or voting system: card IDs and PIN storage need stronger security for real-world use.
- Do not commit generated Keil binaries or per-user IDE settings. The included `.gitignore` already excludes them.

## Media

All images in `docs/images/` are project documentation supplied by the repository owner. They are included to show the actual prototype and interface states.
