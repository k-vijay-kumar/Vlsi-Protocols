# SPI (Serial Peripheral Interface)

SPI is a synchronous, four-wire, full-duplex, point-to-multi-point serial communication interface capable of very high-speed data transmission.

## 1. Physical Interface
SPI typically requires **four physical wires**:
* **SCLK (Serial Clock):** Synchronizing clock output from the Master.
* **MOSI (Master Out Slave In):** Data line driven by the Master to send data to the Slave.
* **MISO (Master In Slave Out):** Data line driven by the Slave to send data to the Master.
* **CS / SS (Chip Select / Slave Select):** Active-low line driven by the Master to initiate communication with a specific slave device.

```text
+-------------------+                    +-------------------+
|      Master       |------ SCLK ------->|       Slave       |
|                   |------ MOSI ------->|                   |
|                   |<----- MISO --------|                   |
|                   |------ CS ---------->|                   |
+-------------------+                    +-------------------+
```

## 2. Multi-Slave Configuration
For systems with multiple slaves, the master must provide dedicated **CS** lines for each individual slave device, or chain them together in a daisy-chain configuration.

## 3. Clock Polarity (CPOL) and Clock Phase (CPHA)
SPI supports four distinct operating modes configured by setting CPOL and CPHA, which define when data changes and when it is sampled:
* **CPOL = 0:** Clock idles LOW.
* **CPOL = 1:** Clock idles HIGH.
* **CPHA = 0:** Data is sampled on the *first* clock edge (leading edge).
* **CPHA = 1:** Data is sampled on the *second* clock edge (trailing edge).

| SPI Mode | CPOL | CPHA | Sample Edge | Shift Edge |
| :--- | :--- | :--- | :--- | :--- |
| **Mode 0** | 0 | 0 | Rising | Falling |
| **Mode 1** | 0 | 1 | Falling | Rising |
| **Mode 2** | 1 | 0 | Falling | Rising |
| **Mode 3** | 1 | 1 | Rising | Falling |

## 4. Data Frame Structure
* **No Start/Stop Overheads:** Unlike UART or I2C, SPI does not use start, stop, or address overheads within the data stream.
* **Stream Structure:** As long as the `CS` line is held **LOW**, data is continuously shifted out on MOSI and shifted in on MISO simultaneously on every clock cycle. The data packet size is arbitrary but typically handled in multiples of 8 bits (e.g., 8-bit or 16-bit shifts).

Write (Mode 0):
       ___                                                                                                                       ___
CS        \_____________________________________________________________________________________________________________________/
             ___     ___     ___     ___     ___     ___     ___     ___     ___     ___     ___     ___     ___     ___     ___     ___
SCLK   _____/   \___/   \___/   \___/   \___/   \___/   \___/   \___/   \___/   \___/   \___/   \___/   \___/   \___/   \___/   \___/   \___
            ^   \   ^   \   ^   \   ^   \   ^   \   ^   \   ^   \   ^   \   ^   \   ^   \   ^   \   ^   \   ^   \   ^   \   ^   \   ^   \

            |   |   |   |   |   |   |   |   |   |   |   |   |   |   |   |   |   |   |   |   |   |   |   |   |   |   |   |   |   |   |   |
CLK EDGE:   1   F1  2   F2  3   F3  4   F4  5   F5  6   F6  7   F7  8   F8  9   F9  10  F10 11  F11 12  F12 13  F13 14  F14 15  F15 16  F16
            
            [1 to 16 = RISING EDGES (Slave Samples)]         [F1 to F16 = FALLING EDGES (Master Shifts New Bits)]

       _______ _______ _______ _______ _______ _______ _______ _______ _______ _______ _______ _______ _______ _______ _______ _______ _____
MOSI   XXXXXXX╳  A7   ╳  A6   ╳  A5   ╳  A4   ╳  A3   ╳  A2   ╳  A1   ╳  A0   ╳  D7   ╳  D6   ╳  D5   ╳  D4   ╳  D3   ╳  D2   ╳  D1   ╳  D0   ╳XXXX
              ^       ^       ^       ^       ^       ^       ^       ^       ^       ^       ^       ^       ^       ^       ^       ^

              |       |       |       |       |       |       |       |       |       |       |       |       |       |       |       |
            [Slave samples stable bits here]                        [Slave samples stable write data bits here]
       _____________________________________________________________________________________________________________________________________
MISO   XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX_____________________________________
       (Slave line sits Idle / High-Z / Ignored by Master during a Write)
       
       |<----------------------- BYTE 1: REGISTER ADDRESS ---------------------->|<------------------------- BYTE 2: WRITE DATA ---------------------->|


Read (Mode 0):
       ___                                                                                                                       ___
CS        \_____________________________________________________________________________________________________________________/
             ___     ___     ___     ___     ___     ___     ___     ___     ___     ___     ___     ___     ___     ___     ___     ___
SCLK   _____/   \___/   \___/   \___/   \___/   \___/   \___/   \___/   \___/   \___/   \___/   \___/   \___/   \___/   \___/   \___/   \___
            ^   \   ^   \   ^   \   ^   \   ^   \   ^   \   ^   \   ^   \   ^   \   ^   \   ^   \   ^   \   ^   \   ^   \   ^   \   ^   \

            |   |   |   |   |   |   |   |   |   |   |   |   |   |   |   |   |   |   |   |   |   |   |   |   |   |   |   |   |   |   |   |
CLK EDGE:   1   F1  2   F2  3   F3  4   F4  5   F5  6   F6  7   F7  8   F8  9   F9  10  F10 11  F11 12  F12 13  F13 14  F14 15  F15 16  F16

       _______ _______ _______ _______ _______ _______ _______ _______ _____________________________________________________________________
MOSI   XXXXXXX╳  A7   ╳  A6   ╳  A5   ╳  A4   ╳  A3   ╳  A2   ╳  A1   ╳  A0   ╳                   [ DUMMY BITS / ALL ZEROS ]                        XXXX
              ^       ^       ^       ^       ^       ^       ^       ^

              |       |       |       |       |       |       |       |       (Master drives zeros just to keep generating SCLK cycles)
            [Slave samples target register address bits here]
                                                                     _________ _______ _______ _______ _______ _______ _______ _______ _____
MISO   XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX╳  D7   ╳  D6   ╳  D5   ╳  D4   ╳  D3   ╳  D2   ╳  D1   ╳  D0   ╳XXXX
       (Slave line sits Idle/Hi-Z during Address phase)               ^       ^       ^       ^       ^       ^       ^       ^

                                                                      |       |       |       |       |       |       |       |
                                                                    [Master samples stable read data bits driven by Slave here]
                                                                      |
                                            [CRITICAL INTERFACE BOUNDARY]:
                                            - At Rising Edge 8: Slave clocks in final bit 'A0'.
                                            - At Falling Edge 8 (F8): Slave instantly decodes address, opens register,
                                              and drives bit 'D7' onto the MISO line.
                                            - At Rising Edge 9: Master cleanly captures stable 'D7' bit.

       |<----------------------- BYTE 1: REGISTER ADDRESS ---------------------->|<---------------------- BYTE 2: DECODED READ DATA ------------------->|



## 5. VLSI & RTL Design Challenges
* **Hold Time Violations:** Because SPI can operate at extremely high frequencies (100MHz+), timing alignment between SCLK and data shifting (especially on the MISO path coming back from an external chip) must be tightly managed with proper constraints.
* **Shift Register Implentation:** Designing robust parallel-in serial-out (PISO) and serial-in parallel-out (SIPO) blocks that cleanly interface with the primary system bus clock domain.