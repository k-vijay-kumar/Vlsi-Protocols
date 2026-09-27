# On-Chip Interconnect Protocols: Master Overview

---

## Evolution and Taxonomy of AMBA Buses

As SoC complexity grew from basic microcontrollers to advanced heterogeneous processors, on-chip communication split into hierarchical tiers based on power constraints, frequency requirements, and data bandwidth.

```text
               ┌────────────────────────────────────────┐
               │         AMBA Interconnect Matrix       │
               └───────────────────┬────────────────────┘
                                   │
         ┌─────────────────────────┼─────────────────────────┐
         ▼                         ▼                         ▼
┌─────────────────┐       ┌─────────────────┐       ┌─────────────────┐
│    AMBA APB     │       │    AMBA AHB     │       │    AMBA AXI     │
│  Peripheral Tier│       │  Main System Bus│       │ High-Performance│
└────────┬────────┘       └────────┬────────┘       └────────┬────────┘
         │                         │                         │
         ▼                         ▼                         ▼
  Low Power / Static       Pipelined / Shared       Multi-Channel / Matrix
  Register Config          SRAM & DMA Engines       Cores, NPU, & DDR Links
```

---

## On-Chip Protocol Comparison Matrix

The table below outlines the structural differences, protocol overheads, and design characteristics of each primary on-chip interconnect tier:

| Architectural Metric | AMBA APB | AMBA AHB | AMBA AXI |
| :--- | :--- | :--- | :--- |
| **Performance Tier** | Low Power / Control | Medium to High System Bus | High Performance / High Bandwidth |
| **Bus Type** | Non-pipelined | Pipelined Shared Bus | Point-to-Point Multi-Channel Matrix |
| **Pipelining Support** | No (Requires minimum 2 cycles) | Yes (Address and Data phases overlap) | Yes (Deeply pipelined split channels) |
| **Data Channels** | Single shared bidirectional data bus | Separate Read and Write data buses | 5 Independent unidirectional channels |
| **Handshaking Type** | Strobe signal (PENABLE) | Central Ready control (HREADY) | Two-wire interlocking (VALID / READY) |
| **Out-of-Order Support**| No | No | Yes (Using transactional ARID/AWID/RID tags) |
| **Concurrent R/W** | No | No | Yes (Read and Write paths are completely separate) |
| **Hardware Complexity**| Very Low (Simple 3-state FSM) | Medium (Shared routing with central multiplexers) | High (Requires sophisticated interconnect routers) |

---

## Choosing the Right On-Chip Interconnect

* **Use AMBA APB** when power budget and simplicity are your primary constraints and you are configuring static control registers, status registers, or low-bandwidth peripherals (e.g., configuration ports for timers, UART, I2C).
* **Use AMBA AHB** when you need simple pipelining for high-frequency operations across a shared system bus, particularly when connecting intermediate sequential components like internal block RAMs or single-master DMA pipelines.
* **Use AMBA AXI** when you are designing high-performance, high-bandwidth multi-core architectures that require massive parallel throughput, simultaneous read/write routing, outstanding transactions, and out-of-order execution support (e.g., linking CPU cores, NPUs, GPUs, and DDR controllers).

---

## Navigating the On-Chip Deep-Dives
Detailed hardware signaling, handshaking logic, and internal bus design challenges are split into separate tracking files:
1. [Advanced Peripheral Bus Specification](02_amba_apb_protocol.md)
2. [Advanced High-performance Bus Pipelining](03_amba_ahb_protocol.md)
3. [Advanced Extensible Interface Multi-Channel Design](04_amba_axi_protocol.md)