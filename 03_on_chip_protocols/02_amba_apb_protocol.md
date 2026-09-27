# AMBA APB (Advanced Peripheral Bus) Protocol Specification


## 1. Architectural Role
APB is a low-power, low-bandwidth, non-pipelined interface optimized for minimal power consumption and reduced design complexity. In a System-on-Chip (SoC), it is primarily used to connect the central interconnect to peripheral configuration registers, timers, UART, and slow I/O blocks.

---

## 2. Core Interface Signals
All APB signals are synchronous to the rising edge of the system clock.

*   **PCLK:** System clock. All transactions are timed against its rising edge.
*   **PRESETn:** Active-low system reset signal.
*   **PADDR:** Peripheral Address Bus (up to 32 bits wide).
*   **PWRITE:** Transfer Direction. High indicates a Write cycle, Low indicates a Read cycle.
*   **PSELx:** Peripheral Select. A dedicated select line from the address decoder to each target slave.
*   **PENABLE:** Strobe/Enable signal. Used to time the actual data phase transition.
*   **PWDATA:** Write Data Bus. Driven by the Master during Write operations.
*   **PRDATA:** Read Data Bus. Driven by the Selected Slave during Read operations.
*   **PREADY:** Slave Ready. Driven by the Slave to extend a transaction (insert wait states).
*   **PSLVERR:** Slave Error. Optional signal indicating a transaction failure (e.g., bad address write).

---

## 3. APB Operational Finite State Machine (FSM)

The APB interface operates on a highly deterministic 3-state execution engine:

```text
       ┌──────────────┐
       │              │
       │    IDLE      │◄────────────────────────┐
       │              │                         │
       └──────┬───────┘                         │
              │                                 │
              │ Transfer Required               │ No further
              │ (PSEL = 1)                      │ transfers
              ▼                                 │
       ┌──────────────┐                         │
       │              │                         │
       │    SETUP     │                         │
       │              │                         │
       └──────┬───────┘                         │
              │                                 │
              │ Next Clock Edge                 │
              │ (PENABLE = 1)                   │
              ▼                                 │
       ┌──────────────┐                         │
       │              │─────────────────────────┘
       │    ACCESS    │ PREADY = 1
       │              │
       └──────────────┘
              ▲
              │ PREADY = 0
              │ (Wait State)
              └─┘
```

### State Definitions
1.  **IDLE:** The default reset state. No peripheral activity. `PSEL` and `PENABLE` are held low.
2.  **SETUP:** Entered when a transfer is requested. The Master drives the valid address `PADDR`, asserts direction `PWRITE`, and asserts `PSEL`. This state lasts exactly one `PCLK` cycle.
3.  **ACCESS:** Entered on the next rising clock edge. The Master asserts `PENABLE`. The Slave must sample write data or provide read data during this state. If the Slave pulls `PREADY` low, the FSM freezes in the ACCESS state to extend the data cycle. Once `PREADY` is high, the transaction finishes, and the state exits to IDLE or goes back to SETUP if a new transfer is back-to-back.

---

## 4. Hardware Verification & Implementation Checklist
*   **No Pipelining:** Addresses must remain stable during both the SETUP and ACCESS phases.
*   **Power Optimization:** Ensure `PWDATA` paths are gated when `PSEL` is low to reduce dynamic switching power across long routing distances.