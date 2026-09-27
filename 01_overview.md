# VLSI Hardware Communication Protocols: Master Overview

We can broadly classify the protocols as On-Chip Interconnect (inside the Silicon/SoC) and Peripheral/External Interface (outside the chip) protocols. 

---

## 1. Taxonomy of VLSI Communication Protocols

```text
                     ┌────────────────────────────────────────┐
                     │       VLSI Hardware Protocols          │
                     └───────────────────┬────────────────────┘
                                         │
         ┌───────────────────────────────┴───────────────────────────────┐
         ▼                                                               ▼
┌────────────────────────────────┐                              ┌────────────────────────────────┐
│   On-Chip Interconnects        │                              │     Peripheral Interfaces      │
│   (System-on-Chip Bus Routing) │                              │     (Off-Chip & Board-Level)   │
└────────┬───────────────────────┘                              └────────┬───────────────────────┘
         ├─ APB (Low Power Regs)                                         ├─ UART (Asynchronous)
         ├─ AHB (Pipelined High Perf)                                    ├─ I2C  (Synchronous Shared Bus)
         └─ AXI (High-Bandwidth Channels)                                └─ SPI  (Full-Duplex Streaming)
```

---