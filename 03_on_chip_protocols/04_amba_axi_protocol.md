# AMBA AXI (Advanced Extensible Interface) Protocol Specification

## 1. Architectural Role
AXI is a high-bandwidth, high-frequency, point-to-point interconnect protocol designed for complex SoC systems. It completely decouples addresses and data payloads into five independent transaction channels, enabling concurrent out-of-order execution, read/write asymmetry, and maximum throughput for multi-core processors, GPUs, NPUs, and memory controllers.

---

## 2. Decoupled Five-Channel Architecture

AXI segregates all data operations into 5 dedicated, unidirectional hardware signal groups:

```text
                  ┌────────────────────────────────────────┐
                  │              AXI MASTER                │
                  └───────────────────┬────────────────────┘
       Write Address Channel (AW)     │     Read Address Channel (AR)
       ───► AWADDR, AWVALID, AWREADY ─┤     ───► ARADDR, ARVALID, ARREADY
                                      │
       Write Data Channel (W)         │     Read Data Channel (R)
       ───► WDATA, WVALID, WREADY ────┤     ◄─── RDATA, RVALID, RREADY
                                      │
       Write Response Channel (B)     │
       ◄─── BRESP, BVALID, BREADY ────┘
                  ┌───────────────────┴────────────────────┐
                  │              AXI SLAVE                 │
                  └────────────────────────────────────────┘
```

### Channel Definitions
1.  **Write Address Channel (AW):** Transmits address, burst type, size, and transaction IDs for write operations.
2.  **Write Data Channel (W):** Moves active write payloads from master to slave. Includes byte-strobe filters (`WSTRB`).
3.  **Write Response Channel (B):** Slave transmits write completion status tokens (`BRESP`) back to the master to guarantee data integrity.
4.  **Read Address Channel (AR):** Transmits address, burst boundaries, and configuration IDs for read operations.
5.  **Read Data Channel (R):** Moves read data payloads and status tokens (`RRESP`) from slave back to master.

---

## 3. The VALID / READY Interconnect Handshake
Every single channel across the AXI protocol utilizes an identical two-wire interlocking handshaking handshake mechanism to establish flow control:

*   **VALID:** Asserted by the **Source** when valid address, data, or control information is present on the bus.
*   **READY:** Asserted by the **Destination** when it is fully capable of accepting the incoming information.

A data transaction occurs on the exact rising clock edge where **both VALID and READY are high**.

### Three Valid/Ready Interlocking Cases

```text
CASE 1: VALID asserts before READY
             __    __    __
HCLK      __/  \__/  \__/  \__
          ___________
DATA      X___VALID__╳________ (Source drives data)
                _____
READY     _____/     \________ (Destination acknowledges late)
                     ^ Data Handshake Happens Here

CASE 2: READY asserts before VALID
             __    __    __
HCLK      __/  \__/  \__/  \__
                _____
DATA      _____/ VALID\_______ (Source drives data late)
          ___________
READY     X___READY__╳________ (Destination is open early)
                     ^ Data Handshake Happens Here

CASE 3: Simultaneous assertion
             __    __    __
HCLK      __/  \__/  \__/  \__
                _____
DATA      _____/ VALID\_______
                _____
READY     _____/ READY\_______
                     ^ Data Handshake Happens Here
```

### Strict Architectural Interlocking Rules
*   A Source **must never** wait for `READY` to go high before asserting `VALID`. Doing so creates an unresolvable combinational logic deadlock condition.
*   Once `VALID` is asserted, the data or address lines **must remain completely stable** until the handshake completes.
*   A Destination is allowed to wait for `VALID` to assert before turning on its `READY` response line.

---

## 4. Advanced High-Performance RTL Features
*   **Outstanding Transactions:** A master can issue multiple read addresses sequentially without waiting for the first data payload to finish.
*   **Out-of-Order Execution:** Using transaction IDs (`AWID`, `ARID`, `RID`), the system can return data from a fast internal memory block before finishing a slow request from an external DDR controller, even if the slow request was made first.