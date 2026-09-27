# AMBA AHB (Advanced High-performance Bus) Protocol Specification

## 1. Architectural Role
AHB is a high-performance system bus designed for high-frequency operations. It serves as the primary backbone linking high-bandwidth on-chip modules such as internal SRAM memory blocks, Direct Memory Access (DMA) engines, and legacy core processor matrices before the widespread adoption of AXI.

---

## 2. Core Interface Signals
*   **HCLK:** System Clock.
*   **HRESETn:** Active-low system reset.
*   **HADDR:** System Address Bus.
*   **HTRANS:** Transfer Type. Indicates if the current cycle is IDLE, BUSY, NONSEQUENTIAL (start of burst), or SEQUENTIAL (continuation of burst).
*   **HWRITE:** Write control (1 = Write, 0 = Read).
*   **HSIZE:** Transfer Size (indicates widths: Byte, Halfword, Word, up to 1024 bits).
*   **HBURST:** Burst Type (Single, 4-beat, 8-beat, 16-beat, incrementing or wrapping).
*   **HWDATA:** Write Data Bus. Driven by Master.
*   **HRDATA:** Read Data Bus. Driven by Slave.
*   **HREADY:** Transfer Done. Driven by Slave to stall or advance the pipeline.
*   **HRESP:** Transfer Response Status (OKAY, ERROR).

---

## 3. The Pipelined Address and Data Phase Concept
The defining hardware feature of AHB is its decoupled **Address Phase** and **Data Phase**. This configuration allows transactions to overlap, meaning the address phase for transaction `N+1` occurs simultaneously with the data phase of transaction `N`.

### Text Pipelining Waveform
```text
           __    __    __    __    __    __
HCLK    __/  \__/  \__/  \__/  \__/  \__/  \__
        ______ _____________________________
HADDR   XXXXXX╳ Addr A ╳ Addr B ╳ Addr C ╳
        ───────┼───────┼───────┼───────┼────
HWDATA/        │XXXXXXX╳ Data A ╳ Data B ╳ Data C
HRDATA         │       ╳       ╳       ╳
               │       │       │       │

               |<-Address Phase A      |
                       |<-Data Phase A |
                       |<-Address Phase B
```

### Pipeline Execution Analysis
1.  **Cycle 1:** Master drives `Addr A` onto the address bus.
2.  **Cycle 2:** Slave samples `Addr A`. Simultaneously, the Master drives `Addr B` onto the address bus while the data bus processes `Data A` (Read or Write).
3.  **Cycle 3:** If the slave holds `HREADY` high, `Data A` finishes instantly. The data bus transitions to process `Data B`, while the address bus advances to broadcast `Addr C`.

---

## 4. High-Frequency RTL Challenges
*   **HREADY Feedback Loop:** The `HREADY` signal is a shared bidirectional pipeline control line. If a slave stalls its data phase by dropping `HREADY`, it automatically forces the master to extend its current address output phase for the subsequent transaction, preventing setup/hold slips across the fabric.
*   **Data Bus Multiplexing:** Unlike point-to-point buses, standard AHB relies on explicit central multiplexers to route data lines from multiple slaves back to masters based on decoded addresses.