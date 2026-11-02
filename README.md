# Analysis and Optimisation of Memory Bank Conflicts in a Dual-Bus Architecture

### FPGA-Based Memory Banking, Conflict Detection and Performance Analysis

<p align="center">
  <b>Target Platform:</b> Sipeed Tang Nano 9K (Gowin GW1NR-9) <br>
  <b>HDL:</b> Verilog <br>
  <b>Development Environment:</b> Gowin EDA
</p>

---

## 1. Project Overview

Memory access conflicts are a major source of performance degradation in systems where multiple requests compete for limited memory resources.

This project investigates the design and optimisation of a dual-bus memory architecture implemented on an FPGA.

The architecture consists of two independent memory banks, each supporting a dedicated read port and write port. Separate read and write paths allow compatible operations to execute concurrently, while an arbitration mechanism manages conflicting requests.

When multiple requests compete for the same memory resource, the arbitration unit determines which request is serviced. Unserved requests are preserved in a FIFO-based request buffer and processed when the required resources become available.

A BUSY signalling mechanism provides backpressure to the CPU when the request-handling resources cannot accept additional requests.

The project also investigates different memory access patterns and evaluates their impact on memory performance.

### Primary Objectives

- Design a dual-bus memory architecture using two memory banks.
- Implement independent read and write paths.
- Detect structural conflicts and same-address conflicts.
- Implement rotating-priority arbitration.
- Preserve pending requests using FIFO buffering.
- Implement CPU backpressure using a BUSY signal.
- Measure memory stalls and effective throughput.
- Analyse the impact of different address mapping and buffering strategies.
- Prototype and evaluate the architecture using the Tang Nano 9K FPGA.

---

## 2. System Architecture

The architecture contains the following major components:

1. CPU request interface
2. Address decoder
3. Conflict detection unit
4. Arbitration controller
5. Request FIFO
6. Bank 0 memory
7. Bank 1 memory
8. Read data path
9. Write data path
10. BUSY controller
11. Performance monitoring unit

### High-Level Architecture

```mermaid
flowchart TB

    CPU["CPU / Request Generator"]

    DEC["Address Decoder"]
    CON["Conflict Detection Unit"]
    ARB["Rotating Priority Arbiter"]

    FIFO["Pending Request FIFO"]
    BUSY["BUSY Controller"]

    subgraph MEM["Dual-Bank Memory Subsystem"]

        subgraph B0["Memory Bank 0"]
            B0R["Read Port"]
            B0W["Write Port"]
            B0M["Bank 0 Memory"]
            B0R --> B0M
            B0W --> B0M
        end

        subgraph B1["Memory Bank 1"]
            B1R["Read Port"]
            B1W["Write Port"]
            B1M["Bank 1 Memory"]
            B1R --> B1M
            B1W --> B1M
        end
    end

    READ["Read Data Bus"]
    WRITE["Write Data Bus"]

    PERF["Performance Counters"]

    CPU --> DEC
    DEC --> CON
    CON --> ARB

    ARB --> B0
    ARB --> B1

    ARB --> FIFO
    FIFO --> ARB

    ARB --> BUSY
    BUSY --> CPU

    B0M --> READ
    B1M --> READ

    WRITE --> B0M
    WRITE --> B1M

    ARB --> PERF
    FIFO --> PERF
    CON --> PERF
