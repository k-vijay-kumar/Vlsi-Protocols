# Peripheral Communication Protocols Overview

This document provides a high-level overview of the primary peripheral communication protocols used in **VLSI design** and embedded systems: **UART**, **I2C**, and **SPI**. These interfaces bridge the gap between a System-on-Chip (SoC) and external devices such as sensors, displays, and memory modules.

## Summary Comparison

| Metric | UART | I2C | SPI |
| :--- | :--- | :--- | :--- |
| **Type** | Asynchronous | Synchronous | Synchronous |
| **Duplex** | Full Duplex | Half Duplex | Full Duplex |
| **Topology** | Point-to-Point (1:1) | Multi-Master / Multi-Slave | Single-Master / Multi-Slave |
| **Pin Count** | 2 Pins (TX, RX) | 2 Pins (SDA, SCL) | 4+ Pins (SCLK, MOSI, MISO, CS) |
| **Speed** | Slow (Typical < 1 Mbps) | Medium (100 kbps - 3.4 Mbps) | Fast (10 Mbps - 100+ Mbps) |
| **Addressing** | None | Hardware Addressing (7 or 10-bit) | Physical Chip Select Lines |

---

## Choosing the Right Protocol

* **Use UART** when you need simple, low-cost, point-to-point communication over longer distances without worrying about a shared clock line (e.g., debug consoles, simple wireless modules).
* **Use I2C** when system real estate is limited and you need to connect multiple lower-speed devices (sensors, EEPROMs) using the minimal number of pins.
* **Use SPI** when high-speed, high-throughput, continuous data streams are required (e.g., SD cards, displays, fast ADCs).

---

## Navigating the Protocol Deep-Dives
Detailed protocol breakdowns, frame structures, and RTL design challenges are split into separate files:
1. [Universal Asynchronous Receiver-Transmitter](02_uart_protocol.md)
2. [Inter-Integrated Circuit](03_i2c_protocol.md)
3. [Serial Peripheral Interface](04_spi_protocol.md)