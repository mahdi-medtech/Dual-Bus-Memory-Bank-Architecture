# Detailed Architectural Specification

## 1. System objective
Design a two-bank memory subsystem using separate read and write request/data paths. The system permits one read and one write to a bank in the same cycle when they address different locations, while detecting same-type port contention and same-address hazards.

## 2. Bank and port model
Each bank exposes:
- One read port: at most one read request per cycle.
- One write port: at most one write request per cycle.

A read and a write to different addresses in the same bank may proceed concurrently. Two reads to the same bank contend for its read port. Two writes to the same bank contend for its write port.

## 3. Conflict rules
1. Read/read to the same bank in the same cycle: conflict.
2. Write/write to the same bank in the same cycle: conflict.
3. Any two operations targeting the same address in the same bank: conflict, including read/write.
4. Read/write to different addresses in the same bank: permitted by the stated port model.

## 4. Request arbitration
A rotating-priority arbiter chooses between competing eligible requests. The priority pointer advances after a grant, preventing a fixed requester from always winning. The losing request is stalled and retained, not discarded.

## 5. Pending request storage
A FIFO stores the pending request descriptor. At minimum, the descriptor must preserve operation type, bank selection, address, and write data for writes. If the implementation supports multiple request sources, include source ID/tag as well. Requests are dequeued only when they can be granted and accepted by the target bank.

**Design detail to finalise:** whether there is one global FIFO or independent queues per bank and/or operation type. The diagram currently shows a logical pending-request FIFO, not a fixed RTL queue topology.

## 6. CPU backpressure
The capacity monitor observes request holding-register occupancy, FIFO fullness and the ability to accept/track new requests. When the configured full condition is reached, BUSY is asserted. The CPU must stop issuing additional requests while BUSY is high. BUSY is deasserted when capacity becomes available and outstanding requests can continue draining.

The CPU-side protocol must define whether a request presented in the same cycle BUSY rises is accepted or rejected. This must be unambiguous in RTL and testbench.

## 7. Address decode
The address decoder derives bank selection and the bank-local address. Earlier project discussion proposed A0 as the bank-select bit and a 15-bit address with 16-bit data; confirm these in the final interface specification before hard-coding the diagram or RTL widths.

## 8. Instrumentation
Suggested counters:
- Total clock cycles.
- Accepted requests.
- Completed requests.
- Stall cycles.
- Read/read conflicts.
- Write/write conflicts.
- Same-address conflicts.
- Grant wait cycles / grant latency.

Counters should be defined precisely (event count versus cycle count) and validated against directed testbench cases.

## 9. Verification scenarios
- One read and one write to different addresses in the same bank.
- Two reads to the same bank.
- Two writes to the same bank.
- Read and write to the same address.
- Two requests to different banks.
- FIFO full and BUSY assertion.
- FIFO drain and BUSY deassertion.
- Sustained contention to verify rotating-priority fairness.
- Reset with outstanding requests.

## 10. Open implementation parameters
- Address width and data width.
- Bank-select bit and address mapping.
- FIFO topology and depth.
- Number of request holding registers.
- CPU request/accept handshake.
- Memory read latency and response ordering.
- Reset and simultaneous enqueue/dequeue behaviour.
