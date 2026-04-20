# InkWatch — E-Ink Smartwatch based on nRF52840
Tololoi Ilinca-Roxana, 331CB

A compact, low-power smartwatch built around the Nordic Semiconductor nRF52840 SoC and an e-ink display. The PCB measures 32 x 35 mm with rounded corners to fit a wristwatch enclosure. The design prioritizes ultra-low standby current, BLE connectivity, and an always-on e-paper display that retains its image without any power.

---

## Block Diagram

```
+---------------------+
|     LiPo Battery    |
+----------+----------+
           |
           v
+----------+----------+       +-------------------+
|   Charging IC        |       |   USB Type-C       |
|      TP4054          +<------+   (5V input)       |
+----------+----------+       +-------------------+
           |
           v
+----------+----------+
|   LDO 3.3V           |
|      XC6220          |
+----------+----------+
           |
           v
+----------+-------------------------------------------+
|                      nRF52840                        |
|   Cortex-M4F @ 64 MHz, 1MB Flash, 256KB RAM, BLE 5  |
+---+----------+----------+----------+----------------+
    |          |          |          |
    | SPI      | I2C      | GPIO     | USB
    |          |          |          |
    v          v          v          v
+-------+  +-------+  +-------+  +-----------+
| E-Ink |  | RTC   |  | Btns  |  |  USB CDC  |
| Display  | DS3231|  | x2    |  |  DFU      |
| 200x200  +-------+  +-------+  +-----------+
+-------+  +-------+
           | Haptic|
           | DRV   |
           | 2605L |
           +-------+

+-------+
| Flash |   <-- SPI (shared bus with E-Ink)
| W25Q16|
+-------+
```

---

## Bill of Materials (BOM)

| # | Component | Description | Footprint | Qty | JLC Part | Datasheet |
|---|-----------|-------------|-----------|-----|----------|-----------|
| 1 | nRF52840-QIAA | BLE SoC, Cortex-M4F, 1MB Flash | QFN-73 | 1 | [C190794](https://www.lcsc.com/product-detail/C190794.html) | [Link](https://infocenter.nordicsemi.com/pdf/nRF52840_PS_v1.7.pdf) |
| 2 | GDEW0154M10 | 1.54in E-Ink display, 200x200, SPI | FPC 24-pin | 1 | — | [Link](https://www.good-display.com/product/212.html) |
| 3 | DS3231SN | High-accuracy I2C RTC with TCXO | SOIC-16 | 1 | [C9877](https://www.lcsc.com/product-detail/C9877.html) | [Link](https://datasheets.maximintegrated.com/en/ds/DS3231.pdf) |
| 4 | W25Q16JVSSIQ | 16Mbit SPI NOR Flash | SOIC-8 | 1 | [C131025](https://www.lcsc.com/product-detail/C131025.html) | [Link](https://www.winbond.com/resource-files/w25q16jv%20spi%20revh%2004082019%20plus.pdf) |
| 5 | TP4054 | 500mA single-cell LiPo charger | SOT-23-5 | 1 | [C16581](https://www.lcsc.com/product-detail/C16581.html) | [Link](https://www.lcsc.com/datasheet/lcsc_datasheet_1810192335_TPUNITED-TP4054_C16581.pdf) |
| 6 | XC6220A331MR | 300mA LDO 3.3V regulator | SOT-25 | 1 | [C86534](https://www.lcsc.com/product-detail/C86534.html) | [Link](https://product.torexsemi.com/system/files/series/xc6220.pdf) |
| 7 | DRV2605L | Haptic motor driver, I2C | WSON-10 | 1 | [C167749](https://www.lcsc.com/product-detail/C167749.html) | [Link](https://www.ti.com/lit/ds/symlink/drv2605l.pdf) |
| 8 | PRTR5V0U2X | USB ESD protection, dual-rail | SOT-363 | 1 | [C12333](https://www.lcsc.com/product-detail/C12333.html) | [Link](https://assets.nexperia.com/documents/data-sheet/PRTR5V0U2X.pdf) |
| 9 | USB Type-C connector | USB 2.0 receptacle, mid-mount | SMD | 1 | [C165948](https://www.lcsc.com/product-detail/C165948.html) | — |
| 10 | LiPo 150mAh | 3.7V, 302525 form factor | JST-PH 2-pin | 1 | — | — |
| 11 | Tactile button 3x4mm | Side-press SMD, 4-pin | SMD | 2 | [C318884](https://www.lcsc.com/product-detail/C318884.html) | — |
| 12 | 100nF capacitor 0402 | Decoupling | 0402 | 20 | [C1525](https://www.lcsc.com/product-detail/C1525.html) | — |
| 13 | 10uF capacitor 0805 | Bulk decoupling | 0805 | 5 | [C15850](https://www.lcsc.com/product-detail/C15850.html) | — |
| 14 | 10k resistor 0402 | Pull-ups, voltage divider | 0402 | 15 | [C25744](https://www.lcsc.com/product-detail/C25744.html) | — |
| 15 | 32.768kHz crystal | RTC low-speed oscillator | 3215-2P | 1 | [C32346](https://www.lcsc.com/product-detail/C32346.html) | — |
| 16 | 32MHz crystal | nRF52840 HF oscillator | 3225-4P | 1 | [C9002](https://www.lcsc.com/product-detail/C9002.html) | — |

---

## Hardware Functionality

### Microcontroller — nRF52840

The central component is the Nordic Semiconductor nRF52840, an ARM Cortex-M4F SoC running at up to 64 MHz with 1 MB of internal flash and 256 KB of RAM. It integrates a Bluetooth 5.0 radio, USB 2.0 full-speed device controller, and a rich peripheral set that includes multiple SPI, I2C, UART, PWM, and ADC interfaces. The chip supports a wide supply range from 1.7 V to 5.5 V and multiple deep-sleep modes, making it a natural fit for a battery-powered wearable.

For this board, the nRF52840 runs in its QFN-73 package, clocked by an external 32 MHz crystal for the high-frequency domain and a 32.768 kHz crystal for the low-frequency RTC domain. The internal DC/DC converter is enabled in firmware to reduce dynamic power consumption during active BLE events.

### E-Ink Display

The 1.54-inch 200x200 pixel e-paper panel connects to the nRF52840 over a dedicated 4-wire SPI bus. E-ink technology consumes power only during refresh and holds its image indefinitely at zero standby current — exactly what a watch face display demands. The display interface uses six lines: SPI clock (SCK), MOSI, chip select (CS), data/command (DC), active-low reset (RST), and a BUSY output that the firmware polls before issuing new commands.

Full-screen refresh takes approximately 2 seconds. Panels that support partial refresh can update a region in under 300 ms, which is used for the seconds digit when enabled in firmware.

### Real-Time Clock — DS3231

The DS3231 is a precision I2C RTC with an integrated temperature-compensated crystal oscillator (TCXO), rated at plus or minus 2 ppm. It communicates over the shared I2C bus (SDA, SCL) and provides a square-wave or interrupt output (SQW/INT) on a dedicated GPIO interrupt pin of the nRF52840. Timekeeping continues from the main LiPo through a schottky diode, ensuring the clock does not lose time if the watch is powered off temporarily. The RTC is responsible for waking the nRF52840 from its deep sleep state once per minute to refresh the time display.

### SPI NOR Flash — W25Q16

The 2 MB external flash stores watch face bitmaps, fonts, historical step-count logs, BLE bonding keys, and the OTA firmware update staging area. It shares the SPI bus with the e-ink display but uses its own chip select line. In deep power-down mode the W25Q16 draws under 1 uA, contributing negligibly to the sleep budget.

### Power Management

Charging is performed by the TP4054, a linear single-cell LiPo charger. It accepts 5 V from the USB Type-C connector and charges the 150 mAh cell at a rate set by an external resistor (500 mA maximum). A STAT pin pulls low during charging and is routed to a GPIO pin that drives an indicator LED.

The system 3.3 V rail is generated by the XC6220 LDO, chosen for its 80 mV dropout at 100 mA and a quiescent current of only 1 uA. This is important because the LDO stays active even in deep sleep to power the DS3231 and the nRF52840 retention RAM. The nRF52840 internal DC/DC converter then steps the 3.3 V down further for its core logic.

Battery state of charge is estimated by reading the battery voltage through a resistor divider connected to the nRF52840's SAADC input. The firmware reports this as a BLE Battery Level characteristic and renders a battery icon on the display.

### Haptic Feedback — DRV2605L

The DRV2605L is an I2C haptic motor driver that controls a small ERM (eccentric rotating mass) vibration motor. It integrates a library of 123 licensed waveform effects, so the firmware simply writes a waveform index over I2C and the chip handles the timing and drive current autonomously. An enable GPIO completely removes power from the driver between events, keeping idle current near zero.

### Buttons

Two side-press tactile buttons are connected to GPIO input pins configured with 10 k pull-ups. One button handles navigation (next item / increment) and the other handles confirm or back. Debouncing is implemented in firmware with a 20 ms software timer.

### USB Interface

The USB Type-C connector routes D+ and D- directly to the nRF52840 internal USB PHY. An PRTR5V0U2X ESD protection array is placed immediately beside the connector footprint. In firmware, USB implements both a CDC serial interface (for debug output and log streaming) and a USB DFU class for firmware updates through the MCUboot bootloader.

---

## nRF52840 Pin Assignment

| Pin | Signal | Connected To | Interface | Notes |
|-----|--------|-------------|-----------|-------|
| P0.02 | AIN0 | Battery voltage divider | SAADC | Battery level measurement |
| P0.03 | BTN1 | Tactile button 1 | GPIO IN | 10k pull-up, active low |
| P0.04 | BTN2 | Tactile button 2 | GPIO IN | 10k pull-up, active low |
| P0.06 | UART_TX | Debug UART | UART | Optional, can be repurposed |
| P0.08 | UART_RX | Debug UART | UART | Optional |
| P0.11 | SDA | DS3231, DRV2605L | I2C | Shared I2C bus, 4.7k pull-up |
| P0.12 | SCL | DS3231, DRV2605L | I2C | Shared I2C bus, 4.7k pull-up |
| P0.13 | EINK_SCK | E-Ink display | SPI | Dedicated SPI peripheral |
| P0.14 | EINK_MOSI | E-Ink display | SPI | No MISO needed for display |
| P0.15 | EINK_CS | E-Ink display | SPI CS | Active low |
| P0.16 | EINK_DC | E-Ink display | GPIO OUT | High = data, Low = command |
| P0.17 | EINK_RST | E-Ink display | GPIO OUT | Active low reset |
| P0.18 | EINK_BUSY | E-Ink display | GPIO IN | Polled before each transaction |
| P0.19 | FLASH_SCK | W25Q16 | SPI | Shared SCK with EINK bus |
| P0.20 | FLASH_MOSI | W25Q16 | SPI | Shared MOSI with EINK bus |
| P0.21 | FLASH_MISO | W25Q16 | SPI | Only flash needs MISO |
| P0.22 | FLASH_CS | W25Q16 | SPI CS | Active low, separate from EINK_CS |
| P0.23 | HAPTIC_EN | DRV2605L enable | GPIO OUT | Powers down driver when low |
| P0.24 | RTC_INT | DS3231 SQW/INT | GPIO IN | Wake from System OFF sleep |
| P0.25 | CHG_STAT | TP4054 STAT | GPIO IN | Charging indicator logic |
| P1.09 | LED | Status LED | GPIO OUT | Optional charge/notification LED |
| D+ / D- | USB_DP / USB_DM | USB Type-C via ESD | USB 2.0 | Full-speed USB PHY |
| VDD | 3.3V | XC6220 output | Power | Main supply |
| VDDUSB | 3.3V | Shared rail | Power | USB PHY supply |
| GND | GND | Copper pour | — | Both layers, stitched |

The SPI bus serving the e-ink display and the NOR flash is physically shared (same SCK, MOSI, MISO traces) but each peripheral has an independent chip select driven by the nRF52840, preventing simultaneous transactions.

---

## Power Budget Estimate

| Operating State | Current | Duration per Minute | Charge per Minute |
|----------------|---------|--------------------|--------------------|
| Deep sleep (RTC alive, nRF off) | ~3 uA | 57 s | ~171 uA·s |
| CPU active (no radio) | ~2 mA | 0.5 s | ~1 mA·s |
| E-Ink full refresh | ~15 mA | 2 s | ~30 mA·s |
| BLE advertisement burst | ~5 mA | 0.5 s | ~2.5 mA·s |
| Average per minute | ~3.6 uA avg | — | ~34.7 uA·s |

With a 150 mAh (540 000 uA·s) cell, estimated battery life in time-display-only mode with one BLE sync per minute is approximately 35 to 40 days. Enabling continuous notification mirroring (BLE connected, scan interval 100 ms) reduces this to roughly 7 to 10 days.

---

## PCB Design Notes

The board is a 2-layer design at 32 x 35 mm with 4 mm corner radius, sized to fit a standard 34 mm watch case inner cavity. Design decisions:

- The nRF52840 sits near the center of the board to keep trace lengths short to all SPI and I2C peripherals.
- The 2.4 GHz antenna keep-out zone occupies the upper edge of the board. Both copper layers are cleared in this area and the zone extends 3 mm beyond the board edge as a trace antenna. No components are placed under the antenna.
- Every VDD supply pin on the nRF52840 has a 100 nF decoupling capacitor within 0.5 mm, with vias to the ground plane directly beside the capacitor pad.
- The TP4054 and XC6220 are grouped near the USB connector to minimize the length of high-current charging traces.
- The e-ink FPC connector is on the top edge of the PCB so the display can fold flat over the board.
- The LiPo cell mounts to the back of the case and connects through a JST-PH 2-pin connector on the bottom edge.


## Firmware Overview

The firmware targets the Zephyr RTOS with the nRF Connect SDK. Key components:

- **Display driver** — custom SPI driver implementing full and partial refresh sequences for the e-paper panel, with a framebuffer held in nRF52840 RAM
- **BLE stack** — Zephyr Bluetooth stack exposing a Current Time Service (CTS) profile, Battery Level Service, and a proprietary notification characteristic
- **RTC synchronization** — on each BLE connection, the companion app writes the current Unix timestamp to the CTS characteristic; between connections the DS3231 maintains time independently
- **Power management** — an application-level sleep manager issues `pm_state_force(PM_STATE_SOFT_OFF)` between minute ticks; the DS3231 SQW interrupt on P0.24 triggers a full reset/wakeup via the nRF52840 GPIO sense mechanism
- **MCUboot + USB DFU** — dual-slot bootloader supporting firmware updates over USB DFU without a hardware programmer
