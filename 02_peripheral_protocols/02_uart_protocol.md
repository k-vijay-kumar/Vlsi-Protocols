# UART (Universal Asynchronous Receiver-Transmitter)

UART is a foundational hardware communication protocol that transmits data serially without a shared synchronizing clock signal.

## 1. Physical Interface
UART requires a minimum of **two signal wires** (excluding a common ground):
* **TX (Transmit Data):** Output line for sending serial data.
* **RX (Receive Data):** Input line for capturing incoming serial data.

> **Note:** The connections cross over between the two devices: `Device A TX` connects to `Device B RX`, and `Device A RX` connects to `Device B TX`.

```text
+-------------------+                   +-------------------+
|     Device A      |   TX -------> RX  |     Device B      |
|                   |   RX <------- TX  |                   |
+-------------------+                   +-------------------+
```

## 2. Communication & Clocking Mechanism
* **Asynchronous Operation:** Because there is no shared clock line, both devices must agree on a **Baud Rate** (bits per second) *prior* to communication (e.g., 9600, 115200, 921600 bps).
* **Oversampling:** To sample the asynchronous incoming line reliably, the receiver runs an internal clock that is typically **16x** the negotiated baud rate. It detects the initial falling edge, waits 8 clock cycles to hit the mid-point of the start bit, and then samples every 16 cycles thereafter to read the data at the center of each bit period.

## 3. Data Frame Structure
When no data is being sent, the line sits at an idle state of logic **HIGH (1)**. A standard UART frame consists of:
1. **Start Bit (1 bit):** The line drops to logic **LOW (0)** for 1 bit period, alerting the receiver to start counting cycles.
2. **Data Payload (5 to 9 bits, typically 8 bits):** The actual byte transmitted, typically ordered Least Significant Bit (LSB) first.
3. **Parity Bit (Optional, 1 bit):** Provides basic error checking. Can be set to *Even* or *Odd* parity.
4. **Stop Bit (1 or 2 bits):** The line is pulled back to logic **HIGH (1)** to signal the end of the frame and reset to idle.

```text
      IDLE   | START |  D0  |  D1  | ... |  D7  | PARITY | STOP | IDLE
Line: -------+       +------+------+ ... +------+--------+------+-------
             |_______|                                   |
```

## 4. VLSI & RTL Design Challenges
* **Metastability:** The asynchronous `RX` line can transition exactly at the setup/hold window of the internal clock. Designers **must** pass `RX` through a 2-stage or 3-stage flip-flop synchronizer before processing.
* **Clock Drift:** If the transmitter and receiver internal clocks differ by more than ~4.5%, the mid-bit sampling drift will eventually lead to corrupt frames.
