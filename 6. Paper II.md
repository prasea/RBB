# Unit 1: Computer Organization and Architecture

## 1.1. Basic Structures: General Principles of Computer Organization and Design

Understanding the distinction between **Computer Architecture** and **Computer Organization** is fundamental to designing robust banking systems.

* **Computer Architecture:** Refers to those attributes of a system visible to a programmer, or, put another way, those attributes that have a direct impact on the logical execution of a program. Examples include the instruction set, the number of bits used to represent various data types, input/output (I/O) mechanisms, and techniques for addressing memory.
* **Computer Organization:** Refers to the operational units and their interconnections that realize the architectural specifications. Examples include hardware details transparent to the programmer, such as control signals, interfaces between the computer and peripherals, and the memory technology used.

For instance, it is an architectural design issue whether a computer will have a multiply instruction. It is an organizational issue whether that instruction will be implemented by a special multiply unit or by a mechanism that makes repeated use of the system's add unit.

### Von Neumann Architecture vs. Harvard Architecture

Most commercial and banking server systems utilize variations of these foundational structural designs:

| Feature | Von Neumann Architecture | Harvard Architecture |
| --- | --- | --- |
| **Memory Structure** | Shared memory for both data and instructions. | Physical separation of memory spaces for data and instructions. |
| **Bus System** | Uses a single system bus (data and address) for both instruction fetch and data transfer. | Uses separate buses for instruction and data pathways. |
| **Execution Speed** | Slower, as execution is serialized due to sharing of the bus (**Von Neumann Bottleneck**). | Faster, as instruction fetch and data access can occur simultaneously. |
| **Hardware Complexity** | Simpler hardware design and cheaper implementation. | Complex bus architecture and separate memory controllers. |
| **Application** | General-purpose computing systems (PCs, enterprise servers). | DSPs (Digital Signal Processors), microcontrollers, and high-speed embedded routing hardware. |

### Structural Components of a Computer System

A computer is a complex system consisting of four major structural components:

1. **Central Processing Unit (CPU):** Controls the operation of the computer and performs its data processing functions.
2. **Main Memory:** Stores both program instructions and user data.
3. **I/O System:** Moves data between the computer and its external environment (such as storage networks, core banking terminals, or payment gateways).
4. **System Interconnections (Buses):** Some mechanism that provides for communication among the CPU, main memory, and I/O.

---

## 1.2. Processing Unit: Instruction Format, Arithmetic & Logical Instruction, Addressing Modes

The CPU contains a **Control Unit (CU)**, an **Arithmetic and Logic Unit (ALU)**, and a set of **Registers**. The execution of instructions is guided by the instruction cycle (Fetch, Decode, Execute).

### Instruction Formats

An instruction is represented as a sequence of bits, partitioned into fields. The most basic fields are:

* **Opcode (Operation Code):** Specifies the operation to be performed (e.g., ADD, SUB, LOAD).
* **Operand Reference(s):** Specifies the address or registry of the data upon which the operation is to be performed.

Instructions can be formatted based on the number of address references they contain:

1. **Three-Address Instructions:** Specify two source operands and one destination operand.

$$\text{ADD } R_1, R_2, R_3 \quad (R_1 \leftarrow R_2 + R_3)$$


2. **Two-Address Instructions:** One address acts as both a source and the destination.

$$\text{ADD } R_1, R_2 \quad (R_1 \leftarrow R_1 + R_2)$$


3. **One-Address Instructions:** Utilizes an implicit register called the **Accumulator (AC)** for one of the operands.

$$\text{ADD } M[X] \quad (AC \leftarrow AC + M[X])$$


4. **Zero-Address Instructions:** Utilizes a **Stack** structure. Operands are implicitly popped from the top of the stack, and the result is pushed back.

$$\text{ADD}$$



### Arithmetic and Logical Instructions

The ALU executes operations categorized into:

* **Arithmetic Operations:** Addition, subtraction, multiplication, division, increment, decrement, and negation. These operations handle signed and unsigned integers, as well as floating-point representations.
* **Logical Operations:** Bitwise AND, OR, NOT, XOR, and bit-shifting operations (logical shift, arithmetic shift, and circular rotate).
* *Banking Context:* Bitwise operations are heavily utilized in processing raw network packets, cryptographic hashing algorithms (like SHA-256 for secure online banking transactions), and database indexing masks.



### Addressing Modes

Addressing modes specify the rule for interpreting or modifying the address fields of the instruction before the operand is actually referenced. This provides flexibility to the programmer (pointers, array indexing, program relocation).

Let $A$ be the address field in the instruction, $R$ be the register field, and $EA$ be the **Effective Address** (the actual physical address of the operand in memory).

* **Immediate Addressing:** Operand is part of the instruction itself.

$$\text{Operand} = A$$


* **Direct Addressing:** Effective address is equal to the address field.

$$EA = A$$


* **Indirect Addressing:** The address field points to a memory location that contains the effective address of the operand.

$$EA = M[A]$$


* **Register Addressing:** Operand is stored in a register specified by the instruction.

$$EA = R$$


* **Register Indirect Addressing:** The register contains the address of the operand in memory.

$$EA = M[R]$$


* **Displacement Addressing:** Combines direct addressing and register indirect addressing.

$$EA = A + (R)$$


* *Relative Addressing:* $EA = A + \text{Program Counter (PC)}$
* *Base-Register Addressing:* $EA = A + \text{Base Register (B)}$
* *Indexed Addressing:* $EA = A + \text{Index Register (I)}$



---

## 1.3. Input/Output Organization

The processing speed of CPUs is orders of magnitude faster than peripheral devices. Effective I/O organization is vital to ensure that slow peripheral operations do not stall core banking application executions.

### I/O Programming & Interfacing Techniques

The communication between the system memory/CPU and the I/O module can be configured in two main ways:

```
  Memory-Mapped I/O                     Isolated I/O
+-----------------------+             +-----------------------+
|  Single Address Space |             | Memory Address Space  |
|                       |             | [0000 - FFFF]         |
|  [0000 - 7FFF] RAM    |             +-----------------------+
|                       |             | I/O Address Space     |
|  [8000 - FFFF] I/O    |             | [00 - FF] (IN/OUT)    |
+-----------------------+             +-----------------------+

```

1. **Memory-Mapped I/O:**
* The CPU treats I/O ports identically to memory locations.
* A single address space is shared between memory and I/O devices.
* Any memory access instruction (e.g., `LOAD`, `STORE`) can be used to read or write to an I/O device.
* *Advantage:* Standard instruction set is simplified; no special I/O instructions are needed.
* *Disadvantage:* Uses up valuable memory address space.


2. **Isolated (I/O-Mapped) I/O:**
* Memory and I/O devices have distinct, separate address spaces.
* The CPU uses specialized instructions (e.g., `IN`, `OUT`) to interact with I/O devices.
* *Advantage:* Keeps memory addresses preserved fully for system software and user programs.
* *Disadvantage:* Requires dedicated control lines and a more complex command structure in the CPU.



### Interrupt-Driven I/O and Basic Interrupt Handling Systems

Instead of wasting CPU cycles constantly polling an I/O module to check if it has finished its task (**Programmed I/O**), the CPU delegates the operation and continues executing other tasks. When the I/O module is ready, it issues an **Interrupt** signal to the CPU.

#### Step-by-Step Interrupt Handling Sequence:

1. The peripheral device sends an interrupt request signal to the system bus.
2. The CPU completes the execution of its current active instruction.
3. The CPU acknowledges the interrupt request.
4. The current state of the CPU (Registers, Program Status Word, and Program Counter) is pushed onto the system stack.
5. The CPU loads the program counter with the entry address of the **Interrupt Service Routine (ISR)** associated with the interrupting device.
6. The ISR executes to perform the requested data transfer or task.
7. Upon completion, the ISR executes a "Return from Interrupt" instruction, popping the saved CPU state off the stack, and allowing the original process to resume seamlessly.

---

## 1.4. Memory Systems

Computer systems employ a hierarchical structure to balance speed, capacity, and cost.

```
       / \         [ Fastest, Smallest, Highest Cost ]
      /   \        Registers
     /     \       Cache (L1, L2, L3)
    /       \      Main Memory (DRAM)
   /---------\     Solid State Drives / HDDs
  /           \    Magnetic Tape / Optical Storage [ Slowest, Largest, Cheapest ]
 /_____________\

```

### Internal Memory (Main Memory)

* **RAM (Random Access Memory):** Volatile storage.
* *SRAM (Static RAM):* Uses flip-flops to store data. Faster, more expensive, and less dense. Used for cache memory.
* *DRAM (Dynamic RAM):* Uses a transistor-capacitor pair. Slower, cheaper, and denser. Requires periodic refreshing to keep the charge intact. Used for main memory.


* **ROM (Read-Only Memory):** Non-volatile storage. Contains boot programs (BIOS/UEFI) to initialize banking servers.

### Cache Memory & Mapping Techniques

Cache memory is a small, ultra-fast SRAM chip designed to sit between the CPU and Main Memory to bridge the speed gap. It operates based on the **Principle of Locality of Reference** (Temporal and Spatial locality).

When the CPU requests data, it first checks the cache. A **Cache Hit** occurs if the data is present; otherwise, a **Cache Miss** occurs, and the data block must be retrieved from main memory and copied into the cache.

To map memory blocks into cache lines, three primary techniques are used:

#### 1. Direct Mapping

Each block of main memory maps to exactly one specific line in the cache.

* *Address division:* `[ Tag | Line/Index | Offset ]`
* *Formula:*

$$i = j \pmod c$$



where $i$ is the cache line index, $j$ is the main memory block number, and $c$ is the total number of lines in the cache.
* *Pros:* Extremely simple and inexpensive to implement.
* *Cons:* High conflict miss rate if two programs map to the same cache lines repeatedly (cache thrashing).

#### 2. Fully Associative Mapping

Any block of main memory can be loaded into any available cache line.

* *Address division:* `[ Tag | Offset ]`
* *Pros:* Highly flexible with a very low conflict miss rate.
* *Cons:* Highly complex and expensive. Requires parallel searching of all cache lines via content-addressable memory (CAM).

#### 3. Set-Associative Mapping

A hybrid of the two. The cache is divided into $v$ sets, and each set contains $k$ lines (known as a $k$-way set-associative cache). Memory block $j$ can be mapped to any line within set $i$.

* *Address division:* `[ Tag | Set Index | Offset ]`
* *Formula:*

$$i = j \pmod v$$


* *Pros:* Offers an optimal balance between cost, complexity, and hit ratio.

### Direct Memory Access (DMA)

For high-speed devices (such as core banking SAN/NAS storage controllers or gigabit fiber cards), transfer-by-transfer interrupt handling wastes immense CPU power. **DMA** solves this by letting an external hardware controller manage data transfers directly between memory and I/O devices without continuous CPU intervention.

```
+-----+         Bus Request         +----------------+
|     | <-------------------------- |                |
| CPU |                             | DMA Controller |
|     | --------------------------> |                |
+-----+         Bus Grant           +----------------+
                                            |
                                            v (Takes over control of System Bus)
                                    [ Memory <===> I/O Module ]

```

#### DMA Operation Steps:

1. The CPU programs the DMA controller with the operation type (Read/Write), I/O device address, memory start address, and the count of bytes to transfer.
2. The DMA controller requests control of the system bus from the CPU (Bus Request).
3. The CPU relinquishes control of the bus and replies with a "Bus Grant."
4. The DMA controller transfers the data blocks directly between the I/O device and memory.
5. Once the entire transfer is complete, the DMA controller issues an interrupt to the CPU to notify it of completion. The CPU then regains control of the system bus.

### External Memory

This includes non-volatile storage media used to back up database transactions, system logs, and general operating system files.

* **Solid State Drives (SSDs):** Utilize NAND flash memory. Provide microsecond access latencies, making them ideal for high-volume online transaction processing (OLTP) database engines in commercial banking.
* **Hard Disk Drives (HDDs):** Magnetic disks used for bulk storage.
* **RAID (Redundant Array of Independent Disks):** Combines multiple physical disks into a single logical unit to achieve data redundancy, performance improvement, or both. Essential for banking data centers to ensure zero-loss storage resilience.

---

## 1.5. Parallelism and Pipelining

To meet the real-time processing demands of thousands of simultaneous mobile banking transactions, modern processors execute tasks concurrently through parallelism and pipelining.

### Pipelining

Pipelining is an implementation technique where multiple instructions are overlapped in execution, similar to an assembly line.

A standard instruction pipeline might be divided into five stages:

1. **Instruction Fetch (IF):** Get instruction from memory.
2. **Instruction Decode (ID):** Decode instructions and read source registers.
3. **Execute (EX):** Perform arithmetic/logic operation or calculate memory address.
4. **Memory Access (MEM):** Read or write data from/to memory.
5. **Write Back (WB):** Write instruction result back to register file.

```
Clock Cycle:   1    2    3    4    5    6    7
Instr 1:      IF   ID   EX   MEM  WB
Instr 2:           IF   ID   EX   MEM  WB
Instr 3:                IF   ID   EX   MEM  WB

```

#### Pipeline Hazards:

Problems that prevent the next instruction in the instruction stream from executing in its designated clock cycle.

1. **Structural Hazards:** Hardware resource conflicts where two overlapping instructions require the same physical resource (e.g., a single memory port used for instruction fetch and memory data write-back at the exact same cycle).
2. **Data Hazards:** Occur when an instruction depends on the result of a previous instruction that is still moving through the pipeline.
* *Read After Write (RAW), Write After Read (WAR), Write After Write (WAW).*


3. **Control Hazards (Branch Hazards):** Arise from the pipelining of branches and other instructions that change the Program Counter (PC). The pipeline does not know what instruction to fetch next until the branch condition is resolved.

### Parallel Processing Architectures (Flynn's Taxonomy)

Flynn's classification organizes computer systems based on the number of concurrent instruction streams and data streams:

```
                  +-----------------------------------+
                  |           DATA STREAMS            |
                  |     Single             Multiple   |
+-----------------+-----------------------------------+
| I  Single       |      SISD                 SIMD    |
| N               |  (Traditional PC)    (GPU/Vector) |
| S               |                                   |
| T  Multiple     |      MISD                 MIMD    |
| R               |  (Fault-tolerant)  (Multi-core)   |
+-----------------+-----------------------------------+

```

* **SISD (Single Instruction, Single Data):** Standard uniprocessor systems executing one instruction on a single data element per cycle.
* **SIMD (Single Instruction, Multiple Data):** A single instruction controls multiple processing units operating on different data elements simultaneously. Ideal for vector math, graphics cards, and basic cryptographic operations.
* **MISD (Multiple Instruction, Single Data):** Multiple instructions operate on the same data stream. Rarely used; sometimes found in highly resilient aerospace systems or dedicated real-time parallel verification algorithms.
* **MIMD (Multiple Instruction, Multiple Data):** Multiple independent processors simultaneously executing different instructions on different data. This is the foundation of modern multi-core computing nodes running microservices and virtualized core banking platforms.

### Overlapped CPU and I/O Operation

To maximize CPU utilization, systems use overlapping CPU and I/O execution models. While the DMA controller handles high-throughput block I/O transfers, or while disk controllers retrieve data records, the CPU is Context Switched to execute other independent threads or banking transactions. This allows the system to achieve high concurrency and minimizes overall processing latency.

---

## 1.6. Review & Practice Questions

### Section A: Multiple-Choice Questions (MCQs) — Specially Designed for Paper I (Section B)



Choose the single best option. Write your answers in uppercase (A, B, C, D) as per exam rules.

#### Q1. Which of the following is a primary characteristic of Von Neumann architecture?

A. Separate memory pathways for instruction and data

B. Shared physical memory for both instructions and data

C. Only utilizes register-to-register instruction executions

D. Direct execution of instructions from external magnetic storage

#### Q2. What is the main cause of the "Von Neumann Bottleneck"?

A. Inability to execute logical shifts within the ALU

B. The delay caused by DRAM refresh cycles

C. Shared bus structure causing data and instruction fetches to serialize

D. Extreme physical distance between cache and registers

#### Q3. In which addressing mode is the operand value stored directly inside the instruction itself?

A. Register Indirect Addressing

B. Immediate Addressing

C. Direct Addressing

D. Base-Register Addressing

#### Q4. If an instruction has an address field value of `0x1005` and is configured with "Indirect Addressing", what is the Effective Address (EA)?

A. `0x1005`

B. The value stored inside register `1005`

C. The memory address contained in memory location `0x1005`

D. The value calculated by adding `0x1005` to the Program Counter

#### Q5. What distinguishes Memory-Mapped I/O from Isolated I/O?

A. Memory-Mapped I/O utilizes special `IN` and `OUT` assembly instructions.

B. Memory-Mapped I/O shares the same address space for both memory and I/O devices.

C. Isolated I/O reduces overall performance by using SRAM as direct I/O ports.

D. Memory-Mapped I/O is physically isolated from the main memory bus.

#### Q6. What is the first action the CPU performs upon receiving and acknowledging an interrupt request?

A. It instantly flushes the Cache Memory to maintain consistency.

B. It halts all future processing and triggers system reboot protocols.

C. It completes execution of the current instruction and pushes the CPU state onto the stack.

D. It transfers data from the hard disk directly to SRAM.

#### Q7. Why does SRAM find use as Cache Memory while DRAM is used for Main Memory?

A. SRAM is cheaper and supports larger densities than DRAM.

B. SRAM is volatile, whereas DRAM is a non-volatile memory system.

C. SRAM is faster and does not require periodic refresh cycles, unlike DRAM.

D. DRAM has lower latency and is physically integrated directly inside the CPU register file.

#### Q8. In a 2-way set-associative cache, memory block 17 will be mapped to which set index if the total number of sets is 8?

A. Set 0

B. Set 1

C. Set 2

D. Set 9

#### Q9. Which device takes over the control of the system bus to transfer block data directly to and from memory without constant CPU execution cycles?

A. Programmed I/O Controller

B. Cache Controller

C. DMA Controller

D. Micro-programmed Control Unit

#### Q10. What type of pipeline hazard occurs when hardware cannot support the simultaneous overlapping execution of two instructions requiring the same physical resource?

A. Data Hazard

B. Control Hazard

C. Structural Hazard

D. Branch Hazard

---

### Answers and Explanations for Section A (MCQs)

| Question No. | Correct Answer | Technical Explanation |
| --- | --- | --- |
| **Q1** | **B** | Von Neumann architecture is defined by having a unified memory space storing both program instructions and working data. |
| **Q2** | **C** | Because both instructions and data must travel over the same shared system bus, they cannot be accessed at the exact same moment. This serial bottleneck slows down maximum execution speeds. |
| **Q3** | **B** | Immediate addressing defines the operand directly inside the opcode instruction (e.g., `ADD 5`). |
| **Q4** | **C** | In Indirect addressing, the address field contains the address where the actual effective target address is located: $EA = M[A]$. |
| **Q5** | **B** | Memory-mapped I/O integrates I/O registers directly into the physical memory address space, allowing normal memory reference instructions to act on I/O ports. |
| **Q6** | **C** | To avoid corruption and permit a seamless return, the CPU completes its executing instruction, saves its current environment parameters (PC, Status Register) onto the stack, and then proceeds to the ISR. |
| **Q7** | **C** | SRAM uses transistor flip-flops, making it much faster than capacitor-based DRAM. Since DRAM leaks charge, it requires refreshing, which adds latency. SRAM is faster but more expensive, making it ideal for cache. |
| ** | Q8** | **B** |
| **Q9** | **C** | The Direct Memory Access (DMA) controller takes bus ownership to move data directly between high-speed peripherals and RAM, saving valuable CPU processing cycles. |
| **Q10** | **C** | Structural hazards occur when two instructions try to use the same hardware block (e.g., ALU or Memory ports) at the same time in the execution pipeline. |

---

### Section B: Subjective Questions (Practice Material for Paper II — Section A)



#### Q1. Explain the operational mechanism of Direct Memory Access (DMA). Why is DMA highly preferred for data storage and network transfers in core banking server environments? (Short Answer - 5 Marks)



##### Model Answer:

**Direct Memory Access (DMA)** is an advanced input/output technique where an external hardware module (the DMA Controller) takes over the system bus to manage high-speed data transfers directly between peripheral I/O devices and main memory. This occurs with minimal CPU intervention.

```
       [ CPU ]                       [ Main Memory ]
          |                                 ^
          | Setup parameters                |
          v                                 | Direct Data
    [ DMA Controller ] <====================+  Transfer
          |                                 |
          | Control signals                 v
          v                          [ Network/Disk ]
    [ I/O Device ]

```

##### Operational Mechanism of DMA:

1. **Initialization:** The CPU prepares the DMA controller by writing parameters to its registers: the direction of transfer (read/write), the starting memory address pointer, the physical I/O device address, and the total count of data blocks/bytes to be transferred.
2. **Bus Request & Grant:** When the I/O device is ready to send or receive data, the DMA controller issues a **Bus Request (BR)** signal to the CPU. The CPU completes its current bus cycle, suspends its bus operations, and replies with a **Bus Grant (BG)** signal.
3. **Data Transfer:** Once the DMA controller secures the system bus, it directly copies data from the I/O device buffer to memory (or vice-versa) in blocks. This bypasses the CPU's internal registers completely.
4. **Termination:** After the block transfer is complete, the DMA controller releases the system bus back to the CPU and asserts an **Interrupt** signal to notify the CPU that the transfer is successfully finished.

##### Importance in Core Banking Server Environments:

In modern core banking systems (like Finacle, Pumori, etc.), systems must process thousands of high-volume ledger records, transactions, and system logs every second.

* **Preventing CPU Starvation:** Without DMA, the CPU would have to execute a transfer instruction for every single byte of data (Programmed I/O). This would consume 100% of CPU cycles, stalling database transaction processing.
* **High-Speed Storage Operations:** Banks rely on high-speed Storage Area Networks (SANs) and Solid-State Storage. DMA allows these fast systems to dump massive database blocks directly into RAM at hardware line-speeds.
* **Network Packet Handling:** Gigabit internet routing interfaces and digital payment settlement gateways use DMA (often Bus Mastering DMA) to feed transaction queues straight into memory buffers, minimizing system latency.

---

#### Q2. Define Pipelining. Discuss the different types of pipeline hazards that occur during execution and suggest standard techniques to mitigate them. (Long Answer - 10 Marks)



##### Model Answer:

**Pipelining** is an advanced processor design implementation where multiple instructions are overlapped in execution. Rather than waiting for one instruction to completely pass through all phases of execution before starting the next, a pipelined CPU divides instruction execution into sequential stages. This allows a new instruction to enter the pipeline during every clock cycle.

If a pipeline has $k$ stages, then up to $k$ instructions can be in progress simultaneously. While it does not decrease the latency of any individual instruction, it significantly increases the overall **instruction throughput** of the CPU.

---

### Pipeline Hazards

Pipeline hazards are conditions in a processor that prevent the next instruction in the instruction stream from executing in its designated clock cycle. These hazards cause the pipeline to stall, introducing "bubbles" or dead cycles that reduce processing efficiency.

The three primary categories of hazards are:

#### 1. Structural Hazards

Structural hazards occur when the hardware cannot support all possible combinations of instructions in overlapped execution due to resource conflicts. This happens when two or more instructions in different stages of the pipeline need to use the exact same physical resource during the same clock cycle.

* *Example:* A system with a single shared memory port for both instructions and data. If one instruction is in the **Instruction Fetch (IF)** stage while an older instruction is in the **Memory Access (MEM)** stage, both will try to access the single memory unit at the same time, creating a conflict.

```
Cycle 1      Cycle 2      Cycle 3      Cycle 4 (Hazard)
[IF - Inst1] [ID - Inst1] [EX - Inst1] [MEM - Inst1] -> Accesses Memory
             [IF - Inst2] [ID - Inst2] [EX  - Inst2]
                          [IF - Inst3] [ID  - Inst3] -> Accesses Memory (Conflict!)

```

* *Mitigation Techniques:*
* **Harvard Architecture/Separated Cache:** Provide physically independent instruction caches (I-Cache) and data caches (D-Cache) so that fetches and data memory actions can run in parallel without conflicts.
* **Resource Duplication:** Duplicate functional hardware units (e.g., adding multiple ALUs or read/write ports to the Register File).



#### 2. Data Hazards

Data hazards arise when the pipeline overlaps the execution of instructions that depend on the results of previous instructions that are still moving through the pipeline. These hazards happen when an instruction's source operand is modified by a preceding instruction before the change takes effect in the registers.

* *Classification of Data Hazards:*
* **Read After Write (RAW):** Instruction B tries to read a register source before Instruction A writes to it. (This is a true physical dependency).
* **Write After Read (WAR):** Instruction B tries to write to a target register before it is read by Instruction A.
* **Write After Write (WAW):** Instruction B tries to write to a destination register before Instruction A writes to it.


* *Example:*

$$\text{Instruction 1: } R_1 \leftarrow R_2 + R_3$$


$$\text{Instruction 2: } R_4 \leftarrow R_1 - R_5 \quad (\text{RAW dependency on } R_1)$$


* *Mitigation Techniques:*
* **Operand Forwarding (Bypassing):** Extra datapath hardware routes the execution result directly from the output of the ALU (or memory stage) back to the input of the ALU for the next instruction, bypassing the formal register write-back step.
* **Pipeline Interlock (Stalling):** The control unit detects the dependency and inserts a "stall" (or "bubble") to pause the dependent instruction until the data becomes available.
* **Compiler Optimization (Instruction Scheduling):** The compiler reorders independent instructions to separate the dependent operations, filling the execution gaps without changing the program's output.



#### 3. Control Hazards (Branch Hazards)

Control hazards occur when the pipeline must make a decision based on the results of one instruction while other instructions are being fetched. This is primarily caused by branch instructions (conditional and unconditional loops, jumps, subroutine calls), where the next instruction address (Program Counter) depends on the outcome of the branch.

* *Example:* When a conditional branch is fetched in Cycle 1, the CPU does not know whether the branch condition will be met (taken or not taken) until the instruction reaches the **Execute (EX)** or **Memory (MEM)** stage in Cycle 3 or 4. In the meantime, the pipeline has already fetched the subsequent sequential instructions, which may need to be discarded if the branch is taken.
* *Mitigation Techniques:*
* **Stalling (Freeze Pipeline):** Hold the instruction fetch stage until the target address of the branch is calculated. This is simple but hurts performance.
* **Static Branch Prediction:** The hardware always assumes a fixed outcome (e.g., "Always Taken" or "Always Not Taken"). If the prediction is correct, execution continues at full speed. If wrong, the pipeline is flushed, and the incorrect instructions are discarded.
* **Dynamic Branch Prediction:** The CPU uses a **Branch Target Buffer (BTB)** to track the history of branches during execution. This history is used to predict the branch's path with high accuracy (often over 90% in modern systems).
* **Delayed Branching:** The compiler places an independent instruction immediately after the branch in a "branch delay slot." This instruction is executed regardless of whether the branch is taken, keeping the pipeline productive while the target address is calculated.



# Unit 2: Network and Communication

## 2.1. Introduction of Computer Network, Network Architecture, LAN Architecture

### Introduction to Computer Networks

A computer network is a digital telecommunications network that allows nodes (computing devices like core banking servers, automated teller machines (ATMs), and teller terminals) to share resources and exchange data using data links.

### Network Architecture

Network architecture defines the structural and logical layout of a network, specifying how devices are connected and how data is transmitted. The most dominant framework is the **OSI (Open Systems Interconnection) Reference Model** and the **TCP/IP Protocol Suite**.

#### The 7-Layer OSI Model vs. 4-Layer TCP/IP Model

| OSI Layer | Functions | Key Protocols / Standards | TCP/IP Layer |
| --- | --- | --- | --- |
| **7. Application** | Network services to end-user applications. | HTTP, HTTPS, FTP, SMTP, DNS, SNMP | Dynamic Application Layer |
| **6. Presentation** | Data representation, encryption, compression. | SSL/TLS, ASCII, JPEG |  |
| **5. Session** | Interhost communication, managing sessions. | NetBIOS, RPC, PPTP |  |
| **4. Transport** | End-to-end connections, reliability, flow control. | TCP, UDP | **Transport (Host-to-Host)** |
| **3. Network** | Logical addressing, routing, path determination. | IP (IPv4/IPv6), ICMP, OSPF, BGP | **Internet** |
| **2. Data Link** | Physical addressing (MAC), error/flow control over physical link. | Ethernet (802.3), Wi-Fi (802.11), Frame Relay | **Network Access (Link)** |
| **1. Physical** | Binary transmission, electrical/mechanical media specs. | Fiber Optics, RJ-45, Cat6 cables, bits |  |

### LAN Architecture

Local Area Network (LAN) architecture dictates how devices within a limited geographical area (e.g., an RBB branch office or data center) communicate.

* **IEEE 802.3 (Ethernet):** The standard tokenless, carrier-sense multiple access with collision detection (**CSMA/CD**) architecture used for wired LANs. Modern LANs operate in full-duplex mode over switches, effectively eliminating collisions.
* **IEEE 802.11 (Wireless LAN/Wi-Fi):** Uses Carrier Sense Multiple Access with Collision Avoidance (**CSMA/CA**) to handle wireless media access control.

---

## 2.2. Network Topologies, Components, and Models

### Network Topologies

Topology defines the geometric arrangement of links and nodes in a network.

```
   [Star]             [Mesh]             [Bus]
    /|\               o---o              o---o---o
   o-O-o             / \ / \             |   |   |
    \|/              o---o               =========

```

* **Star Topology:** All nodes connect to a central device (hub/switch). If the central device fails, the whole network goes down. However, an individual cable failure only affects one node. *Banking application:* Standard deployment layout for branch office teller machines.
* **Mesh Topology (Full/Partial):** Every node is connected to every other node (Full) or to multiple nodes (Partial).
* *Formula for Full Mesh links:*

$$\text{Links} = \frac{n(n-1)}{2}$$



where $n$ is the number of nodes.
* *Banking application:* Used in core data centers (DC) and Disaster Recovery (DR) site interconnections to guarantee high availability and fault tolerance.


* **Bus Topology:** Devices share a single backbone cable (bus). Uses terminators at the ends. High collision rate and difficult to troubleshoot.
* **Ring Topology:** Data travels in one direction (or two in dual-ring) from device to device. A single break disrupts the entire loop.

### Network Components

* **Repeater (Layer 1):** Regenerates electrical, optical, or wireless signals degraded by attenuation over long physical distances. It does not look at addresses.
* **Hub (Layer 1):** A multiport repeater. It broadcasts any incoming data packet to all other ports, creating a single large **collision domain** and **broadcast domain**. It operates in half-duplex mode.
* **Switch (Layer 2):** An intelligent device that inspects the destination **MAC address** of incoming frames and forwards them *only* to the specific port connected to the destination device. It breaks up collision domains per port but maintains a single broadcast domain.
* **Router (Layer 3):** A device that forwards data packets between different networks based on **IP addresses**. It reads network layer headers and uses routing tables to determine the optimal path. Routers break up both collision domains and broadcast domains.

### Client-Server vs. Peer-to-Peer (P2P) Models

* **Client-Server Model:** Centralized architecture where dedicated servers provide services, security, credentials, and data storage, while clients (workstations) request them. *Banking Context:* Core Banking Systems (CBS) operate strictly on this model to guarantee ACID properties, central audit logs, and security.
* **Peer-to-Peer (P2P) Model:** Decentralized architecture where every node (peer) acts as both a client and a server, sharing responsibilities and data directly without a central coordinator.

---

## 2.3. Basic E-mail and Internet Technology

### E-mail Architecture and Protocols

Electronic mail relies on a distributed client-server architecture utilizing distinct application layer protocols:

```
[ Sender Client ] ---SMTP---> [ Sender Mail Server ] ---SMTP---> [ Receiver Mail Server ]
                                                                       |
                                                                   POP3 / IMAP
                                                                       v
                                                               [ Receiver Client ]

```

* **SMTP (Simple Mail Transfer Protocol):** A push protocol used to send mail from a client to a server, or between mail servers. It defaults to TCP port 25 (or secure ports 465/587).
* **POP3 (Post Office Protocol v3):** A pull protocol that downloads emails from the server to the local client device and, by default, deletes them from the server. Operates on TCP port 110 (or secure port 995).
* **IMAP (Internet Message Access Protocol):** A pull protocol that allows clients to view and synchronize emails directly on the server without downloading and deleting them. Highly ideal for multi-device access. Operates on TCP port 143 (or secure port 993).

### Internet Technology Fundamentals

* **DNS (Domain Name System):** The "phonebook of the internet" that translates human-readable domain names (e.g., `[www.rbb.com](https://www.rbb.com).np`) into machine-readable IP addresses. Operates primarily over UDP port 53.
* **HTTP/HTTPS (Hypertext Transfer Protocol / Secure):** The foundational protocol of the World Wide Web. HTTPS utilizes **SSL/TLS** encryption over TCP port 443 to secure communication between a customer's browser and corporate web banking systems.

---

## 2.4. Network Utilities and IP Addressing

### Network Utilities

Network administrators use command-line utilities to troubleshoot connectivity issues:

* **ping:** Uses **ICMP (Internet Control Message Protocol)** Echo Request and Echo Reply messages to verify layer 3 connectivity and measure round-trip times to a destination node.
* **traceroute / tracert:** Traces the actual hop-by-hop path a packet takes to a destination by incrementally increasing the **Time-to-Live (TTL)** value of packets starting from 1.
* **nslookup / dig:** Queries DNS servers to discover IP addresses mapped to domain names or to verify DNS resource records.
* **netstat:** Displays active network connections, routing tables, interface statistics, and protocol states on the local system.

### IP Addressing (IPv4)

An IPv4 address is a 32-bit logical address broken down into four 8-bit octets, separated by dots (e.g., `192.168.10.1`). It consists of a **Network ID** and a **Host ID**, defined by a **Subnet Mask**.

#### Classful Routing Architecture

| Class | First Octet Range | Default Subnet Mask | Purpose / Application |
| --- | --- | --- | --- |
| **Class A** | 1 – 126 | `255.0.0.0` /8 | Large scale networks (16.7M hosts/net) |
| **Class B** | 128 – 191 | `255.255.0.0` /16 | Medium scale networks (65k hosts/net) |
| **Class C** | 192 – 223 | `255.255.255.0` /24 | Small scale networks (254 hosts/net) |
| **Class D** | 224 – 239 | N/A | Multicast Streams |
| **Class E** | 240 – 255 | N/A | Experimental / Research |

> *Note:* `127.0.0.0` to `127.255.255.255` is reserved strictly for local loopback testing.

#### Subnetting Calculation Formulas

Subnetting breaks a large network into smaller, efficient, and secure subnetworks.

* **Number of Subnets created:** $2^n$, where $n$ is the number of bits borrowed from the host field.
* **Number of usable Hosts per Subnet:**

$$\text{Usable Hosts} = 2^h - 2$$



where $h$ is the number of remaining host bits. We subtract 2 because the first address is the **Network Address** and the last address is the **Broadcast Address**.

---

## 2.5. Switching, Routing, Multiplexing, and Transport Protocols

### Basic Switching Techniques

1. **Circuit Switching:** Establishes a dedicated, continuous physical path between two nodes before data transmission begins (e.g., traditional PSTN telephone networks). Guarantees bandwidth but is highly inefficient for bursty computer data.
2. **Packet Switching:** Breaks data down into small, independent units called packets. Each packet contains destination routing headers and can travel through different physical paths across the network before being reassembled at the destination. Used by the internet and modern banking networks.

### Routing Techniques

* **Static Routing:** Network routes are manually configured by an administrator. It is highly secure and predictable but does not adapt to dynamic link failures.
* **Dynamic Routing:** Routers use dynamic routing protocols to discover networks, build routing tables, and automatically adapt to path failures.
* *Distance Vector (e.g., RIP):* Routes based on hop count.
* *Link State (e.g., OSPF):* Builds a complete topological map of the network using Dijkstra's algorithm to calculate the shortest path.



### Multiplexing and De-multiplexing

* **Multiplexing (MUX):** The process of combining multiple analog or digital input signals into a single shared transmission medium (e.g., FDM - Frequency Division, TDM - Time Division, WDM - Wavelength Division).
* **De-multiplexing (DEMUX):** The reverse process at the receiving end, separating the combined stream back into its original individual signals.

### Transport Layer Protocols: TCP vs. UDP

```
     [ TCP Header ]                         [ UDP Header ]
+-----------------------+              +-----------------------+
| Src Port  | Dst Port  |              | Src Port  | Dst Port  |
| Sequence Number       |              +-----------------------+
| Acknowledgment Number |              | Length    | Checksum  |
| Flags     | Window    |              +-----------------------+
+-----------------------+              (Lightweight, No ACKs)
(Connection-Oriented)

```

* **TCP (Transmission Control Protocol):**
* **Connection-Oriented:** Establishes a connection using a **3-Way Handshake** (`SYN` $\rightarrow$ `SYN-ACK` $\rightarrow$ `ACK`) before sending data.
* **Reliable:** Uses sequence numbers, acknowledgments (`ACK`), and retransmissions to ensure data arrives error-free and in order.
* *Application:* Core banking balances, database queries, web banking sessions.


* **UDP (User Datagram Protocol):**
* **Connectionless:** Sends packets (datagrams) directly without establishing a connection or tracking delivery.
* **Unreliable/Fast:** No acknowledgments, no flow control, minimal overhead.
* *Application:* Live video streaming, VoIP, DNS lookups, DHCP.



### Flow Control Mechanisms

Flow control prevents a fast sender from overwhelming a slow receiver with too much data.

1. **Stop-and-Wait:** The sender transmits one frame and must wait for an explicit acknowledgment (`ACK`) from the receiver before sending the next frame. Highly inefficient over high-latency links.
2. **Sliding Window Protocol:** The sender can transmit multiple frames up to a negotiated "Window Size" before requiring an acknowledgment. As the receiver sends `ACK`s for early frames, the window slides forward, allowing continuous transmission and maximum channel utilization.

---

## 2.6. Review & Practice Questions

### Section A: Multiple-Choice Questions (MCQs) — Specially Designed for Paper I (Section B)



Choose the single best option. Write your answers in uppercase (A, B, C, D) as per exam rules.

#### Q1. Which layer of the OSI model handles data encryption, compression, and translation?

A. Session Layer

B. Transport Layer

C. Application Layer

D. Presentation Layer

#### Q2. A network device that forwards data frames based strictly on Layer 2 MAC addresses is called a:

A. Router

B. Switch

C. Hub

D. Repeater

#### Q3. How many physical links are required to connect 10 banking nodes in a Full Mesh network topology?

A. 10

B. 90

C. 45

D. 20

#### Q4. Which protocol is used by a local mail client to synchronize folders and read email messages directly on the server without downloading and removing them from the mail host?

A. SMTP

B. POP3

C. IMAP

D. SNMP

#### Q5. What is the network address of the host IP `192.168.1.55` with a subnet mask of `255.255.255.240`?

A. `192.168.1.0`

B. `192.168.1.32`

C. `192.168.1.48`

D. `192.168.1.64`

#### Q6. Which utility uses the Time-To-Live (TTL) field of IP packets to determine the path data takes across a network?

A. ping

B. netstat

C. traceroute

D. nslookup

#### Q7. What are the sequence flags exchanged during a standard TCP connection establishment handshake?

A. `SYN` $\rightarrow$ `ACK` $\rightarrow$ `FIN`

B. `SYN` $\rightarrow$ `SYN-ACK` $\rightarrow$ `ACK`

C. `RST` $\rightarrow$ `SYN` $\rightarrow$ `ACK`

D. `DATA` $\rightarrow$ `ACK` $\rightarrow$ `FIN-ACK`

#### Q8. Which of the following applications would benefit most from using UDP instead of TCP?

A. Transferring a customer ledger CSV file via FTP

B. Real-time video conferencing for banking board meetings

C. Processing an online credit card balance modification

D. Accessing the bank's secure administrative interface via SSH

#### Q9. The process of splitting a single physical transmission medium into multiple logical channels to send separate data streams concurrently is called:

A. Demultiplexing

B. Routing

C. Switching

D. Multiplexing

#### Q10. What is the primary purpose of the Sliding Window protocol at the Transport layer?

A. To resolve IP address conflicts in the local network

B. To dynamically encrypt data packets

C. To handle flow control, preventing a fast sender from overwhelming a slower receiver

D. To map domain names to physical MAC addresses

---

### Answers and Explanations for Section A (MCQs)

| Question No. | Correct Answer | Technical Explanation |
| --- | --- | --- |
| **Q1** | **D** | The Presentation Layer ensures that data is readable by the Application Layer, managing formatting, compression, and encryption/decryption. |
| **Q2** | **B** | A Switch is a Layer 2 device that builds a MAC address table to forward frames directly to their intended destination ports. |
| **Q3** | **C** | Using the formula: $\frac{n(n-1)}{2} = \frac{10 \times 9}{2} = 45$ links. |
| **Q4** | **C** | IMAP allows two-way synchronization, keeping emails on the server for multi-device access. POP3 downloads and removes them by default. |
| **Q5** | **C** | A subnet mask of `.240` creates block increments of 16 ($256 - 240 = 16$). The subnets are `.0, .16, .32, .48, .64`. The IP `.55` falls in the `.48` subnet (`192.168.1.48` to `192.168.1.63`). |
| **Q6** | **C** | Traceroute sends packets with increasing TTL values (1, 2, 3...). Each router along the path drops the TTL to 0, returns an ICMP time-exceeded message, and reveals its IP address. |
| **Q7** | **B** | TCP establishes a reliable connection using a 3-way handshake: Client sends `SYN`, Server responds with `SYN-ACK`, and Client completes with `ACK`. |
| **Q8** | **B** | Real-time video conferencing prioritizes speed over absolute delivery verification. Dropping occasional frames is better than experiencing the latency caused by TCP retransmissions. |
| **Q9** | **D** | Multiplexing combines multiple independent signals into a single physical path. Demultiplexing separates them at the destination. |
| ** | Q10** | **C** |

---

### Section B: Subjective Questions (Practice Material for Paper II — Section A)



#### Q1. Compare TCP and UDP. Justify why a core banking transaction system relies on TCP, whereas DNS services operate primarily over UDP. (Short Answer - 5 Marks)



##### Model Answer:

**Transmission Control Protocol (TCP)** and **User Datagram Protocol (UDP)** are the two primary protocols operating at the Transport Layer of the network model, but they are optimized for different types of traffic.

##### Key Differences:

* **Connection Type:** TCP is connection-oriented, establishing a session via a 3-way handshake before transmitting data. UDP is connectionless and sends packets immediately without setup.
* **Reliability:** TCP guarantees data delivery through acknowledgments, sequencing, and automatic retransmissions of lost packets. UDP provides best-effort delivery with no tracking or validation mechanisms.
* **Overhead:** TCP has a larger header (minimum 20 bytes) and requires more processing power. UDP has a small header (8 bytes), making it lightweight and fast.

##### Architectural Justifications:

* **Core Banking Transaction Systems (Why TCP):** Financial transactions require absolute precision and data integrity. If a message updating an account balance is corrupted or lost, it can lead to financial errors and audit failures. TCP ensures that every transaction packet arrives securely, error-free, and in the correct order. If a packet is dropped due to network congestion, TCP automatically halts execution and retransmits the missing data.
* **DNS Services (Why UDP):** DNS queries are typically simple, single-packet requests followed by a single-packet response. Setting up a full TCP connection via a 3-way handshake for a single query would triple network traffic and add unnecessary latency. UDP allows the system to send a quick lookup request and receive an immediate answer. If a packet is lost, the application simply times out and tries again, which is much more efficient for high-volume lookups.

---

#### Q2. What is IP Subnetting? A bank branch requires a local network layout that can support 5 distinct departments, with each department requiring up to 25 usable host IP addresses. Given the base network address `192.168.50.0/24`, calculate the subnet mask, list the network and broadcast addresses for the subnets, and determine the range of usable host IPs. (Long Answer - 10 Marks)



##### Model Answer:

**IP Subnetting** is the practice of dividing a single, large network into multiple smaller, logically isolated subnetworks (subnets). This helps reduce broadcast traffic, optimize IP address utilization, and improve network security by allowing administrators to apply targeted firewall policies between departments.

---

### Subnet Design Calculations

The bank requires **5 distinct subnets**, with each subnet supporting up to **25 usable host IP addresses**. The base network is a Class C network: `192.168.50.0/24`.

#### 1. Determining Borrowed Subnet Bits ($n$)

To create at least 5 subnets, we use the formula $2^n \ge \text{required subnets}$:

* If we borrow $n = 2$ bits: $2^2 = 4$ subnets (Not enough)
* If we borrow $n = 3$ bits: $2^3 = 8$ subnets (Sufficient, provides 3 spare subnets for growth)

#### 2. Validating Host Capacity ($h$)

A Class C network has 8 host bits. After borrowing 3 bits for subnetting, we have $8 - 3 = 5$ host bits remaining ($h = 5$).

* Using the host formula: $\text{Usable Hosts} = 2^h - 2$
* $\text{Usable Hosts} = 2^5 - 2 = 32 - 2 = 30 \text{ hosts per subnet}$
* This fulfills the requirement for 25 usable hosts per department.

#### 3. Defining the Custom Subnet Mask

The original mask was `/24` (`255.255.255.0`). Adding the 3 borrowed bits gives a new prefix length of `/27` ($24 + 3$).

* Binary modification of the fourth octet: `11100000` = $128 + 64 + 32 = 224$.
* **Custom Subnet Mask:** `255.255.255.224`
* **Subnet Block Size (Increment):** $256 - 224 = 32$

---

### Subnet Allocation Table

The base network `192.168.50.0/24` is divided into `/27` subnets using a fixed increment of 32. Here is the allocation for the first 5 departments:

| Department / Subnet | Network Address | First Usable Host IP | Last Usable Host IP | Broadcast Address |
| --- | --- | --- | --- | --- |
| **Dept 1 (Subnet 0)** | `192.168.50.0` | `192.168.50.1` | `192.168.50.30` | `192.168.50.31` |
| **Dept 2 (Subnet 1)** | `192.168.50.32` | `192.168.50.33` | `192.168.50.62` | `192.168.50.63` |
| **Dept 3 (Subnet 2)** | `192.168.50.64` | `192.168.50.65` | `192.168.50.94` | `192.168.50.95` |
| **Dept 4 (Subnet 3)** | `192.168.50.96` | `192.168.50.97` | `192.168.50.126` | `192.168.50.127` |
| **Dept 5 (Subnet 4)** | `192.168.50.128` | `192.168.50.129` | `192.168.50.158` | `192.168.50.159` |

*(Note: Subnets 5, 6, and 7 spanning from `.160` to `.255` remain unassigned as spare capacity for future branch expansion).*



# Unit 3: Operating System

## 3.1. Fundamentals of Operating System (OS)

An **Operating System (OS)** is a core system software component that acts as an intermediary between a user/application program and the underlying computer hardware.

### Core Objectives of an OS

1. **Convenience:** Makes the computer system more user-friendly and convenient to use.
2. **Efficiency:** Allows resource utilization (CPU cycles, memory space, I/O devices) to happen efficiently.
3. **Abstraction:** Hides the complex, messy details of low-level hardware operations from developers and end-users.

### Core Architecture and Functions

* **Kernel:** The central, core component of the OS that remains permanently loaded in main memory (RAM). It directly interfaces with hardware and manages processes, memory, and devices.
* **Process Management:** Allocates CPU cycles to multiple execution threads, handles task scheduling, creation, termination, and context-switching.
* **Memory Management:** Dynamically tracks, allocates, and deallocates chunks of volatile primary storage (RAM) among running user processes.
* **Storage and File System Management:** Organizes files logically into directories and maps them onto non-volatile hardware media (SSDs, HDDs).
* **Device (I/O) Management:** Communicates with physical device controllers through software components called **Device Drivers**.
* **Protection and Security:** Enforces access control structures, keeping user processes isolated from one another and preventing unauthorized changes to system memory.

---

## 3.2. Processes and Threads

### Process vs. Thread

* **Process:** A program in execution. It is the heavy-weight unit of resource allocation, possessing its own independent address space, code segment, data segment, file descriptors, and security context.
* **Thread:** A lightweight unit of CPU execution nested within a process. Multiple threads share the parent process’s memory space, code, and resources, but each maintains its own independent **Program Counter (PC)**, **Registers**, and **Stack**.

### Architectural Frameworks

* **Symmetric Multiprocessing (SMP):** A hardware and software architecture where two or more identical processors connect to a single shared main memory and are managed by a single OS instance. All processors share equal responsibility for executing user programs and operating system threads.
* **Micro-kernel Architecture:** A structural paradigm that modularizes the OS kernel. It places only the bare minimum services in kernel space (such as low-level memory management, primitive inter-process communication, and basic thread scheduling). All other non-essential services (like file systems, networking stack, and device drivers) run in user space as isolated, modular servers. This enhances stability and fault tolerance; if a file system server crashes, it can be restarted without taking down the entire kernel.

### Concurrency, Mutual Exclusion, and Synchronization

When multiple execution paths interleave or run simultaneously, they may try to read and write to shared data variables.

* **Race Condition:** A critical flaw where multiple processes/threads concurrently read and write to the same data item, and the final outcome depends on the exact, unpredictable order in which those executions occur.
* **Critical Section:** The specific segment of code within a program that accesses shared resources (like database records or shared variables) and must not be concurrently accessed by more than one process.
* **Mutual Exclusion:** The absolute principle ensuring that if one process is executing inside its critical section, no other processes are permitted to enter their critical sections for that same shared resource.

```
Process A                        Process B
   │                                │
   ▼                                ▼
[ Tries to Enter ]               [ Tries to Enter ]
   │                                │
   ▼                                │
───ENTER CRITICAL SECTION───        │
   │  (Locks Resource)              │
   │                                │  Blocked! Waits for 
   │  [Executes Balance Update]     │  Process A to release
   │                                │  the lock...
───EXIT CRITICAL SECTION────        │
      (Unlocks Resource)            ▼
                               ───ENTER CRITICAL SECTION───

```

#### Synchronization Tools: Semaphores and Mutexes

* **Mutex (Mutual Exclusion Lock):** A locking mechanism used to synchronize access to a resource. It is an object owned exclusively by one thread at a time. If a thread wants a resource, it locks the mutex; when finished, it unlocks it.
* **Semaphore:** A synchronization variable initialized to an integer value. It uses two atomic, standard operations: `wait()` (or $P$) and `signal()` (or $V$).
* *Binary Semaphore:* Can only take values 0 and 1 (acts similarly to a Mutex).
* *Counting Semaphore:* Can take any positive integer value, representing the number of identical, concurrent resources available to the system.



---

## 3.3. Memory Management Strategies & Virtual Memory

Primary memory (RAM) is a scarce resource that must be dynamically managed to support concurrent banking application instances.

### Basic Memory Management Strategies

* **Contiguous Allocation:** Every program is allocated a single, unbroken block of memory locations. It often leads to fragmentation problems.
* **Paging:** A non-contiguous memory management scheme. It breaks down physical memory into fixed-size blocks called **Frames** and breaks logical memory down into blocks of the exact same size called **Pages**.
* A system hardware component called the **Memory Management Unit (MMU)** uses a **Page Table** to translate logical addresses generated by the CPU into physical frames. This design eliminates **External Fragmentation** entirely.



### Virtual Memory Management Techniques

Virtual memory separates the user's logical memory view from the actual physical memory space. This allows execution of programs that are much larger than the physical RAM available, loading only the required segments into memory at any given time.

* **Demand Paging:** Pages are only loaded into physical memory frames when they are explicitly requested by the running CPU instruction during execution, rather than loading the whole program all at once.
* **Page Fault:** A hardware interrupt triggered when a running process attempts to access a logical page that is not currently mapped into a physical RAM frame. The OS must intercept this, locate the page on secondary storage (swap space), swap it into a free frame, update the page table, and resume the instruction.
* **Thrashing:** A state of high inefficiency where the system spends significantly more time swapping pages in and out of secondary storage than it does executing actual instructions. This happens when the total working memory sets of all active processes exceed the physical RAM capacity.

#### Page Replacement Algorithms

When a page fault occurs and no physical RAM frames are free, the OS must choose an existing page to replace:

1. **FIFO (First-In, First-Out):** Replaces the oldest page in memory. It suffers from **Belady's Anomaly** (where adding more physical frames can paradoxically increase the number of page faults).
2. **LRU (Least Recently Used):** Replaces the page that has gone unused for the longest duration of time. It is highly efficient but requires hardware tracking support.
3. **Optimal Page Replacement (OPT):** Replaces the page that will not be used for the longest period in the future. It is impossible to implement in practice because it requires future knowledge; it is used primarily as a baseline for performance comparisons.

---

## 3.4. Security Threats to Operating Systems

Securing operating systems is a critical requirement in government banking networks to protect sensitive data and financial records.

### Authentication and Access Authorization

* **Authentication:** Verifies the identity of a user or system entity attempting to access the platform (e.g., passwords, smart cards, biometric scans, multi-factor tokens).
* **Access Authorization:** Defines what an authenticated user is permitted to do. The OS enforces this through access control frameworks:
* **Discretionary Access Control (DAC):** The owner of a resource determines who can access it.
* **Role-Based Access Control (RBAC):** Access permissions are tied to organizational roles rather than individual users, which aligns well with standard banking structures.



### System Flaws and Attacks

* **Buffer Overflow:** A security vulnerability that occurs when a process writes more data into a fixed-length memory buffer than it was designed to hold. The excess data overflows into adjacent memory spaces, potentially overwriting return pointers and allowing attackers to execute arbitrary malicious code with system-level privileges.
* **Malicious Software (Malware):**
* *Trojan Horse:* Malicious software disguised as a legitimate application.
* *Ransomware:* Encrypts the local storage system and demands payment for the decryption key.


* Denial of Service (DoS / DDoS):Floods the operating system's connection queues or computational resources, rendering it unavailable to legitimate bank customers.

---

## 3.5. Input/Output and Files

### Principles of I/O Hardware

I/O hardware generally consists of two main parts: the physical device itself (e.g., a disk drive or network interface) and the electronic component called the **Device Controller**. The controller manages the physical operation of the device and provides an interface to the OS.

### Principles of I/O Software

I/O software is typically structured hierarchically to shield the core OS from hardware complexities:

```
+---------------------------------------+
| User-Level I/O Software (Libraries)  |
+---------------------------------------+
| Device-Independent OS Software        |
+---------------------------------------+
| Device Drivers                        |
+---------------------------------------+
| Interrupt Handlers                    |
+---------------------------------------+
| Hardware (Controllers & Peripherals)  |
+---------------------------------------+

```

1. **Interrupt Handlers:** Low-level routines that handle device interrupts, unblocking drivers when an I/O operation completes.
2. **Device Drivers:** Device-specific modules containing the code needed to interact directly with a particular hardware controller.
3. **Device-Independent OS Software:** Handles general I/O functions common to all devices, such as buffering, caching, device naming, and access protection.
4. **User-Space I/O Software:** System libraries (e.g., standard I/O calls like `printf` or `scanf`) and spooling systems that let user applications interface with system calls.

---

## 3.6. Review & Practice Questions

### Section A: Multiple-Choice Questions (MCQs) — Specially Designed for Paper I (Section B)



Choose the single best option. Write your answers in uppercase (A, B, C, D) as per exam rules.

#### Q1. Which component of the operating system architecture remains permanently loaded in main memory throughout system operation?

A. Device Driver

B. Compiler

C. Kernel

D. Command Line Interpreter

#### Q2. What is the fundamental difference between a process and a thread?

A. Threads do not possess their own independent program counters.

B. Processes nested within the same application share their stack space.

C. Multiple threads belonging to the same process share code, data, and system resources.

D. A process is a lightweight execution unit, whereas a thread is heavyweight.

#### Q3. What system state describes two or more processes trapped in a condition where each is waiting for a resource that is held by another process?

A. Thrashing

B. Deadlock

C. Race Condition

D. Context Switch

#### Q4. Which memory mapping scheme divides physical memory into fixed-size blocks called frames and logical memory into blocks called pages?

A. Contiguous Allocation

B. Paging

C. Segmentation

D. Swapping

#### Q5. What anomaly describes a scenario where increasing the number of physical memory frames results in a higher number of page faults for a process?

A. Belady's Anomaly

B. Dijkstra's Anomaly

C. Von Neumann Anomaly

D. Pipeline Hazard Anomaly

#### Q6. A situation where an operating system spends significantly more time swapping memory pages in and out of disk storage than executing actual instructions is called:

A. Buffering

B. Context Switching

C. Cache Invalidation

D. Thrashing

#### Q7. Which access control technique binds access permissions to specific organizational duties rather than individual user identities?

A. Discretionary Access Control (DAC)

B. Mandatory Access Control (MAC)

C. Role-Based Access Control (RBAC)

D. Matrix-Based Access Control

#### Q8. What security exploit involves writing more data into a fixed-size memory storage area than it can hold, often corrupting return addresses?

A. Phishing

B. SQL Injection

C. Buffer Overflow

D. Trojan Insertion

#### Q9. In the standard I/O software hierarchy, which layer is responsible for providing generic functions like uniform buffering, caching, and device naming?

A. Interrupt Handlers

B. Device-Independent OS Software

C. Device Drivers

D. User-Level Libraries

#### Q10. A counting semaphore is initialized to 5. If 3 consecutive `wait()` operations are performed on it, what is the resulting value of the semaphore?

A. 5

B. 8

C. 2

D. 0

---

### Answers and Explanations for Section A (MCQs)

| Question No. | Correct Answer | Technical Explanation |
| --- | --- | --- |
| **Q1** | **C** | The kernel is the core software component that is loaded into memory during booting and remains active to manage hardware and system tasks. |
| **Q2** | **C** | Threads run inside a process and share its address space, resources, and data. However, each thread retains its own independent stack, registers, and program counter. |
| **Q3** | **B** | A deadlock occurs when a set of processes are permanently blocked because each holds a resource that another process needs, creating a circular wait condition. |
| **Q4** | **B** | Paging maps logical memory onto non-contiguous physical memory by dividing physical storage into frames and logical code into identical pages. |
| **Q5** | **A** | Belady's anomaly occurs in some page replacement algorithms, such as First-In, First-Out (FIFO), where increasing the frame allocation can lead to more page faults. |
| **Q6** | **D** | Thrashing occurs when physical RAM is insufficient to hold the active working memory sets of running programs, causing the OS to spend most its time swapping pages. |
| **Q7** | **C** | Role-Based Access Control (RBAC) simplifies management by assigning access permissions to predefined operational roles rather than specific individual accounts. |
| **Q8** | **C** | A buffer overflow occurs when data exceeds a buffer's capacity, spilling into adjacent memory and potentially overwriting control structures like return addresses. |
| **Q9** | **B** | Device-independent OS software provides a uniform interface to the upper layers, handling tasks like caching, buffering, and device block mapping. |
| **Q10** | **C** | Each `wait()` operation decrements the semaphore by 1. Starting at 5, three consecutive operations reduce the value to 2 ($5 - 1 - 1 - 1 = 2$). |

---

### Section B: Subjective Questions (Practice Material for Paper II — Section A)



#### Q1. Define a critical section. What are the three essential requirements that any software solution must fulfill to resolve the critical section problem? (Short Answer - 5 Marks)



##### Model Answer:

A **Critical Section** is a segment of code within a concurrent program where shared resources (such as global variables, memory arrays, or file logs) are accessed. If multiple threads or processes execute their critical sections concurrently, it can lead to data corruption or a race condition.

To prevent this, any valid solution to the critical section problem must satisfy three essential requirements:

1. **Mutual Exclusion:** If a process is executing in its critical section, no other processes can be allowed to execute in their critical sections for the same shared resource at that time.
2. **Progress:** If no process is currently executing in its critical section and there are processes that want to enter, only those processes that are not executing in their remainder sections can participate in deciding which process enters next. This selection cannot be postponed indefinitely, preventing deadlocks.
3. **Bounded Waiting:** There must be a limit or bound on the number of times other processes can enter their critical sections after a process has made a request to enter its own, before that request is granted. This prevents starvation, ensuring every process eventually gets access.

---

#### Q2. Explain the concept of Paging. Distinguish between a Page Fault and Thrashing, detailing how an Operating System handles a Page Fault condition. (Long Answer - 10 Marks)



##### Model Answer:

**Paging** is a memory management scheme that eliminates the need for contiguous allocation of physical main memory. It divides physical memory into fixed-size blocks called **Frames** and divides an application's logical address space into blocks of the exact same size called **Pages**.

When a process is executed, its pages are loaded into any available physical frames. The **Memory Management Unit (MMU)** uses a data structure called a **Page Table** to map these logical addresses to their corresponding physical addresses. This approach eliminates external fragmentation.

---

### Page Fault vs. Thrashing

* **Page Fault:** A specific hardware interrupt triggered by the MMU when a running process attempts to access a logical page that is not currently loaded into physical RAM. It is a normal, manageable event in demand-paged operating systems.
* **Thrashing:** A state of systemic failure where the processor spends more time swapping pages in and out of secondary disk storage than it does executing actual application instructions. This happens when the combined working sets of all active processes exceed the physical memory available, causing a continuous loop of page faults.

---

### How the Operating System Handles a Page Fault

When a process references a page marked as invalid in the page table, the following step-by-step sequence occurs:

```
[ CPU executes instruction ]
            │
            ▼
[ MMU checks Page Table ] ───(Valid: 1)───> [ Access Physical RAM Frame ]
            │
      (Invalid: 0)
            ▼
┌───────────────────────────────┐
│     1. TRAP TO KERNEL         │ -> OS takes control, saves process context
└───────────────────────────────┘
            │
            ▼
┌───────────────────────────────┐
│  2. LOCATE PAGE ON STORAGE    │ -> Finds target page in swap/disk space
└───────────────────────────────┘
            │
            ▼
┌───────────────────────────────┐
│   3. ALLOCATE FREE RAM FRAME  │ -> If full, runs replacement algorithm
└───────────────────────────────┘
            │
            ▼
┌───────────────────────────────┐
│  4. READ PAGE INTO FRAME      │ -> Schedules physical disk I/O read
└───────────────────────────────┘
            │
            ▼
┌───────────────────────────────┐
│     5. UPDATE PAGE TABLE      │ -> Sets present bit to 1, maps to frame
└───────────────────────────────┘
            │
            ▼
┌───────────────────────────────┐
│   6. RESUME CPU INSTRUCTION   │ -> Restores state, re-executes original instruction
└───────────────────────────────┘

```

1. **Trap to OS:** The hardware detects the missing page and issues a trap (interrupt) to the operating system kernel.
2. **Context Preservation:** The OS suspends the current process, saving its registers, program counter, and state onto the system stack.
3. **Disk Analysis:** The OS checks an internal table to verify if the memory reference was valid. If valid, it locates the page's position on the secondary storage swap disk.
4. **Fetch Page:** The OS schedules a physical disk read operation to bring the page into an available main memory frame. If no frames are free, a page replacement algorithm (like LRU) is run to evict an existing page.
5. **Update Map:** Once the disk read completes, the OS updates the process's page table by entering the physical frame address and changing the valid/invalid bit from 0 to 1.
6. **Resume Execution:** The OS restores the process's saved context and restarts the instruction that was interrupted by the page fault, allowing it to access the now-cached memory frame seamlessly.






# Unit 4: Database Management System Design

## 4.1. Fundamentals of DBMS: Data Models, Warehousing, and Mining

A **Database Management System (DBMS)** is a software system that enables users to define, create, maintain, and control access to a database. In banking, data integrity, security, and performance are absolute prerequisites.

### Database Models

* **Relational Model:** The dominant data model in traditional banking systems (e.g., Core Banking Systems like Finacle). Data is organized into two-dimensional tables (relations) consisting of rows (tuples) and columns (attributes). It enforces strict constraints (primary keys, foreign keys) to maintain relational integrity.
* **Object-Oriented & Object-Relational Models:** Combine database capabilities with object-oriented programming language capabilities (e.g., support for user-defined types, inheritance, and encapsulation).
* **NoSQL Models:** Document, Key-Value, Graph, and Wide-Column stores. These trade strict ACID guarantees for horizontal scalability and high availability, often used in bank log analysis, fraud detection paths, and unstructured customer data processing.

### Data Warehousing and Data Mining

```
+------------------+     ETL Process     +-------------------+     Analysis     +-----------------+
| Transactional DB | ──────────────────> |   Data Warehouse  | ────────────────>  |   Data Mining   |
| (OLTP: Finacle)  |   (Extract, Trans,  | (Historical OLAP) |  (Pattern/Fraud)   | (Predictive AI) |
+------------------+        Load)        +-------------------+                    +-----------------+

```

* **Data Warehouse (DWH):** A centralized, subject-oriented, integrated, time-variant, and non-volatile collection of data used to support management decision-making processes.
* *OLTP (Online Transaction Processing):* Optimized for day-to-day banking updates (e.g., cash deposits, transfers). Focuses on high concurrency and rapid writes.
* *OLAP (Online Analytical Processing):* Optimized for complex, historical read queries and business intelligence reporting (e.g., quarterly loan growth analysis).


* **Data Mining:** The process of discovering hidden, valid, and actionable patterns, anomalies, and correlations within large datasets.
* *Banking Applications:* Credit scoring models, money laundering pattern detection (AML tracking), and customer churn prediction.



---

## 4.2. Database Servers: Functions, Procedures, Triggers, and Rules

Modern database servers distribute computation by running pre-compiled program logic directly inside the database engine, reducing network overhead between the application server and storage layer.

### Core Database Objects

* **Stored Procedures:** Pre-compiled collections of SQL statements and procedural logic (control loops, variables) stored on the database server. They accept input parameters, return output parameters, and can perform structural updates. *Advantage:* Reduces network traffic and mitigates SQL injection risks.
* **User-Defined Functions (UDFs):** Similar to procedures but designed to compute and return a single value or a table directly. They cannot make permanent structural modifications to the database state (no `INSERT` or `UPDATE` inside standard functions).
* **Triggers:** Specialized database programs that execute automatically ("fire") in response to a specific event (such as `INSERT`, `UPDATE`, or `DELETE`) on a particular table.
* *Banking Application:* Maintaining audit trails. If a teller modifies a customer's balance field, a trigger automatically captures the old value, new value, timestamp, and teller ID, writing it directly into an unalterable `Audit_Log` table.


* **Rules:** Database objects bound to columns or data types to specify valid data ranges or formats, enforcing domain integrity (largely superseded by standard `CHECK` constraints).

---

## 4.3. Transaction Management and Concurrency Control

### The Concept of a Transaction

A transaction is a logical unit of database processing that includes a series of access operations (read, write, delete). To maintain structural consistency, every transaction must satisfy the **ACID Properties**:

* **Atomicity:** The "All or Nothing" rule. If any part of a transaction fails, the entire transaction is aborted and rolled back to its original state.
* **Consistency:** A transaction must transform the database from one valid state, satisfying all structural integrity constraints, to another valid state.
* **Isolation:** The execution of a transaction must be unnoticeable to other concurrent transactions. Uncommitted changes from one transaction cannot be read by another.
* **Durability:** Once a transaction is successfully committed, its changes are permanently written to non-volatile storage and will survive even in the event of a system crash.

### Concurrency Control Techniques

When multiple users execute transactions simultaneously, the database engine must prevent anomalies like **Lost Updates**, **Temporary Selects (Dirty Reads)**, and **Unrepeatable Reads**.

#### 1. Locking Mechanisms (Two-Phase Locking - 2PL)

Locks prevent multiple transactions from accessing the same data element simultaneously.

* *Shared Lock (S):* Acquired for read operations. Multiple transactions can hold shared locks on the same data item simultaneously.
* *Exclusive Lock (X):* Acquired for write operations. Only one transaction can hold an exclusive lock, blocking all other read or write actions.
* **Two-Phase Locking Protocol (2PL):** Guarantees serializability by enforcing that a transaction cannot acquire any new lock after it has released its first lock. It consists of two phases:
1. *Growing Phase:* The transaction may acquire locks but cannot release any.
2. *Shrinking Phase:* The transaction may release locks but cannot acquire any new ones.



#### 2. Timestamp Ordering (TO)

Every transaction is assigned a unique, monotonically increasing timestamp when it starts. The database ensures that concurrent executions mirror the chronological order of these timestamps. If a conflict is detected (e.g., an older transaction tries to read/write an item already modified by a newer transaction), the offending transaction is aborted and restarted.

---

## 4.4. Network Security: Cryptography, Signatures, Firewalls, and VPNs

Securing data transmission channels prevents unauthorized interception or manipulation of critical financial records.

### Cryptography

* **Symmetric Encryption:** Uses a single, shared secret key for both encryption and decryption (e.g., AES, 3DES). It is fast and efficient for bulk data encryption but presents key distribution challenges.
* **Asymmetric Encryption:** Uses a mathematically linked key pair: a **Public Key** (distributed openly) and a **Private Key** (kept secure by the owner). Examples include RSA and ECC. Data encrypted with a public key can only be decrypted by the corresponding private key.

### Digital Signatures and Digital Certificates

* **Digital Signature:** Provides **Authentication**, **Data Integrity**, and **Non-Repudiation**.
* *Mechanism:* The sender hashes a transaction message, encrypts the hash value using their own *Private Key*, and appends it to the message. The receiver decrypts the hash using the sender's *Public Key* and compares it against a newly computed hash of the received message.


* **Digital Certificate:** An electronic document issued by a trusted third party called a **Certificate Authority (CA)** (e.g., OCA in Nepal). It binds a public key to the true identity of its owner, preventing man-in-the-middle impersonation attacks.

### Network Defense Architecture

* **Firewalls:** System barriers that filter incoming and outgoing network traffic based on predefined security rules. They operate as packet filters (Layers 3/4) or Next-Generation/Application-Aware firewalls (Layer 7).
* **Virtual Private Network (VPN):** Establishes an encrypted tunnel across a public network (like the internet), allowing remote bank branches or off-site ATMs to securely access the central data center network. Common protocols include **IPsec** and **SSL/TLS**.

---

## 4.5. Crash Recovery

The database recovery manager ensures that database integrity is maintained even during hardware failures, software bugs, or power losses.

### Types of Failures

1. **Transaction Failure:** Local errors within an active transaction (e.g., logical validation errors, division by zero, or deadlocks causing the system to abort the process).
2. **System Crash:** Hardware or software bugs that cause the operating system or database engine to halt abruptly. Volatile memory (RAM) is lost, but non-volatile disk storage remains intact.
3. **Media Failure:** Physical disk damage, head crashes, or storage array failures that destroy parts of non-volatile memory.

### Recovery Techniques

Most recovery systems rely on a **Log File** written to stable storage using the **Write-Ahead Logging (WAL)** protocol: the log records must be flushed to non-volatile storage *before* the actual database frames are updated on disk.

#### Immediate vs. Deferred Update

* **Deferred Update (No-Undo/Redo):** The database defers actual physical disk writes until the transaction reaches its commit point. If a system crash occurs before commitment, no changes have reached the disk, so an `UNDO` operation is unnecessary. The system only needs to `REDO` committed transactions whose changes were not yet fully flushed from volatile memory.
* **Immediate Update (Undo/Redo):** Changes made by an active transaction are written to the physical disk immediately during execution. If the system crashes, the recovery manager must read the log file to `UNDO` the uncommitted operations of active transactions and `REDO` the operations of successfully committed transactions that did not finish flushing to disk.

#### Checkpoints

Reading through an entire log file from the beginning of time during a system recovery operation is highly inefficient. The database periodically writes a **Checkpoint**. During a checkpoint, the engine flushes all volatile log buffers and modified database pages (dirty blocks) from RAM down to the physical disk. During a crash recovery, the system only needs to process log entries recorded *after* the last verified checkpoint, saving time.

---

## 4.6. Review & Practice Questions

### Section A: Multiple-Choice Questions (MCQs) — Specially Designed for Paper I (Section B)



Choose the single best option. Write your answers in uppercase (A, B, C, D) as per exam rules.

#### Q1. Which property of a database transaction guarantees that its changes become permanent and will survive a subsequent system crash once it commits?

A. Atomicity

B. Consistency

C. Isolation

D. Durability

#### Q2. What transaction anomaly occurs when a transaction reads data modified by another concurrent transaction that has not yet committed?

A. Lost Update

B. Dirty Read (Temporary Select)

C. Unrepeatable Read

D. Phantom Read

#### Q3. In the Two-Phase Locking (2PL) protocol, what actions are permitted during the "Shrinking Phase"?

A. The transaction can acquire new locks and release existing locks.

B. The transaction can acquire new locks but cannot release any locks.

C. The transaction can release locks but cannot acquire any new locks.

D. The transaction cannot release or acquire any locks.

#### Q4. Which database object executes automatically in response to a structural database event like an `INSERT`, `UPDATE`, or `DELETE` statement?

A. Stored Procedure

B. Trigger

C. View

D. User-Defined Function

#### Q5. What is the primary operational focus of an Online Analytical Processing (OLAP) database system compared to an OLTP system?

A. Minimizing response times for high-volume, concurrent write operations

B. Optimizing complex read-heavy historical data analysis and business reporting

C. Handling live ATM balance adjustment updates

D. Enforcing strict primary key constraints on input forms

#### Q6. To generate a valid Digital Signature on a financial transaction message, the sender must encrypt the message hash value using their own:

A. Private Key

B. Public Key

C. Symmetric Shared Key

D. Certificate Authority Key

#### Q7. Which protocol requires that database log records are written to stable storage before the corresponding data blocks are modified on physical disk?

A. Two-Phase Commit Protocol

B. Write-Ahead Logging (WAL) Protocol

C. Deferred Update Protocol

D. Strict 2PL Protocol

#### Q8. What recovery action is required for an uncommitted transaction that was actively modifying data during an Immediate Update database crash?

A. REDO

B. COMMIT

C. UNDO

D. CHECKPOINT

#### Q9. What security benefit does a Digital Certificate provide in a public key cryptography framework?

A. It compresses long financial messages to speed up transmission.

B. It binds a public key to the true identity of its owner via a trusted third party.

C. It acts as an inline packet-filtering firewall for web servers.

D. It automatically generates a symmetric session key for encryption.

#### Q10. What optimization technique is used to limit the length of the log file that the database recovery manager must read during a system crash recovery?

A. Cascading Rollbacks

B. Hashing Indexes

C. Checkpointing

D. Symmetric Key Rotation

---

### Answers and Explanations for Section A (MCQs)

| Question No. | Correct Answer | Technical Explanation |
| --- | --- | --- |
| **Q1** | **D** | Durability ensures that committed changes are written to non-volatile storage and survive system failures. |
| **Q2** | **B** | A dirty read occurs when Transaction A reads modifications made by Transaction B before Transaction B commits. If Transaction B aborts, Transaction A is left with invalid data. |
| **Q3** | **C** | The Two-Phase Locking protocol states that once a transaction releases its first lock (entering the shrinking phase), it cannot acquire any new locks. |
| **Q4** | **B** | A trigger is a specialized procedural block that fires automatically when data modification events (`INSERT`, `UPDATE`, `DELETE`) occur on its target table. |
| **Q5** | **B** | OLAP systems are optimized for read-heavy, historical data analysis to support business intelligence, while OLTP handles high-volume, concurrent transactional writes. |
| **Q6** | **A** | A digital signature uses asymmetric encryption. The sender encrypts the message hash with their *private key*. Anyone can verify it using the sender's *public key*, ensuring non-repudiation. |
| **Q7** | **B** | The Write-Ahead Logging (WAL) protocol ensures that transaction logs are safely written to disk before the actual database pages are modified, preventing unrecoverable states during a crash. |
| **Q8** | **C** | In an Immediate Update scheme, changes from active transactions can reach the physical disk before commitment. If a crash occurs, the recovery manager must `UNDO` these uncommitted changes. |
| ** | Q9** | **B** |
| **Q10** | **C** | Checkpoints periodically flush dirty memory blocks to disk. During recovery, the system only needs to process log entries recorded *after* the last checkpoint. |

---

### Section B: Subjective Questions (Practice Material for Paper II — Section A)



#### Q1. Explain the ACID properties of a database transaction. Discuss why maintaining these properties is critical in a core banking system. (Short Answer - 5 Marks)



##### Model Answer:

A database transaction is a logical unit of execution that performs a sequence of data access and modification operations. To ensure data integrity, the DBMS must guarantee the **ACID Properties**:

1. **Atomicity:** Ensures that a transaction is treated as a single, indivisible unit of work. Either all operations succeed, or the entire transaction is rolled back, leaving the database unchanged.
2. **Consistency:** Guarantees that a transaction can only bring the database from one valid state to another, maintaining all schema rules, constraints, and business logic.
3. **Isolation:** Ensures that concurrently executing transactions do not interfere with one another. The uncommitted intermediate states of a transaction remain invisible to other concurrent operations.
4. **Durability:** Guarantees that once a transaction commits, its modifications are permanently saved in non-volatile storage and will not be lost, even during a total system crash.

##### Critical Importance in Core Banking Systems:

* *Atomicity:* In a fund transfer transaction, money is deducted from Account A and added to Account B. If the system crashes mid-way after deducting from Account A, atomicity ensures the deduction is rolled back, preventing funds from disappearing.
* *Consistency:* If a bank policy dictates that an account balance cannot fall below a minimum threshold, consistency checks will reject any transaction that violates this rule.
* *Isolation:* If two customers attempt to withdraw money from the same joint account simultaneously, isolation forces the database to serialize the checks, preventing both from successfully withdrawing the same balance.
* *Durability:* When a customer deposits money at an ATM, durability ensures that once the receipt is printed (indicating a committed transaction), the deposit record survives any subsequent power outages or server crashes.

---

#### Q2. Compare the Immediate Update and Deferred Update techniques used in database crash recovery. Explain how the recovery manager uses the log file to restore database consistency following a sudden system failure. (Long Answer - 10 Marks)



##### Model Answer:

Database crash recovery is the process of restoring a database to a consistent state after a system failure. The recovery manager tracks transaction activity using a **Log File** stored on stable memory, which operates on the **Write-Ahead Logging (WAL)** protocol. The two primary strategies for managing modifications are **Immediate Update** and **Deferred Update**.

---

### Comparison of Modification Techniques

| Functional Aspect | Immediate Update Technique (Undo/Redo) | Deferred Update Technique (No-Undo/Redo) |
| --- | --- | --- |
| **Disk Update Timing** | Data modifications are written directly to the physical disk while the transaction is still actively running. | Data updates are held in volatile memory buffers and only written to disk after the transaction successfully commits. |
| **Log Entry Content** | Must record both the **Old Value (Before Image)** and the **New Value (After Image)** of data items. | Only needs to record the **New Value (After Image)** of data items. |
| **Crash Action: Uncommitted Transactions** | Requires an **UNDO** operation to reverse changes written to disk by transactions that were interrupted before committing. | No action is required on disk data; the system simply ignores the log entries for uncommitted transactions. |
| **Crash Action: Committed Transactions** | Requires a **REDO** operation to ensure changes from committed transactions are written to disk if they were still in memory buffers during the crash. | Requires a **REDO** operation to apply all changes from the uncommitted memory buffers to the physical disk. |
| **I/O Overhead** | Higher write overhead during transaction execution due to frequent disk updates. | Lower overhead during execution, but causes an I/O surge at commit time. |

---

### Restoring Consistency Using the Log File After a System Failure

When a system crashes due to a power outage or software bug, volatile RAM is lost, but the log file on stable storage remains intact. Upon rebooting, the recovery manager scans the log file to identify two groups of transactions:

* **Active Transactions:** Transactions that started (`[Start_transaction, T]`) but did not record a commit or abort log entry before the crash.
* **Committed Transactions:** Transactions that recorded both a start and a successful commit log entry (`[Commit, T]`).

#### Step-by-Step Recovery Execution:

```
                  [ SYSTEM CRASH OCCURS ]
                             │
                             ▼
               [ Scan Log File from Checkpoint ]
                             │
            ┌────────────────┴────────────────┐
            ▼                                 ▼
   Active Transactions             Committed Transactions
   (No Commit Record)                (Has Commit Record)
            │                                 │
            ▼                                 ▼
┌──────────────────────┐           ┌──────────────────────┐
│    UNDO Operation    │           │    REDO Operation    │
│ (Rollback changes from)          │ (Apply new values to )
│ ( disk using Before )            │ ( disk using After  )
│ (      Images)       │           │ (      Images)       │
└──────────────────────┘           └──────────────────────┘
            │                                 │
            └────────────────┬────────────────┘
                             │
                             ▼
             [ Database Restored to Consistent State ]

```

##### 1. Processing Active Transactions (Undo Phase):

If the database was using the **Immediate Update** strategy, some changes from active, uncommitted transactions may have already been written to the physical disk. The recovery manager must perform an **UNDO** operation on these transactions. It reads the log backward, using the recorded "Before Images" to overwrite the disk changes and restore the data to its pre-transaction state. This maintains **Atomicity**.

##### 2. Processing Committed Transactions (Redo Phase):

For all transactions that successfully reached their commit point before the crash, the recovery manager must perform a **REDO** operation. It reads the log forward, applying the "After Images" to the physical disk blocks. This ensures that even if committed changes were still sitting in volatile memory buffers when the crash occurred, they are permanently written to disk, maintaining **Durability**.

##### 3. Utilizing Checkpoints to Optimize Recovery:

To avoid parsing the entire history of the log file, the recovery manager looks for the most recent **Checkpoint** record. Since checkpoints guarantee that all prior committed data blocks have been flushed to disk, the manager only needs to scan log entries recorded *after* that checkpoint, making the recovery process much more efficient.






# Unit 5: Computer Programming

## 5.1. Fundamental Issues in Language Design

Language design involves trade-offs between execution efficiency, compilation speed, programming expressiveness, and target platform security.

### Virtual Machines (VMs)

A Virtual Machine is a software implementation of a physical machine that executes programs like a real computer. In programming language design, process-level VMs create a layer of abstraction between the source program code and the underlying host operating system and hardware.

* **Intermediate Code Representation:** Languages like Java or C# do not compile directly into native machine code. Instead, they compile into an intermediate state (e.g., Java **Bytecode** or C# Common Intermediate Language - CIL).
* **Interpretation and JIT Compilation:** The Virtual Machine (such as the **Java Virtual Machine - JVM**) interprets this intermediate code at runtime. Modern VMs include a **Just-In-Time (JIT) Compiler** that monitors execution paths and compiles frequently executed bytecode loops into native machine code on the fly, optimizing execution performance.
* *Banking Advantage:* Provides platform independence ("Write Once, Run Anywhere") and safety boundaries, preventing direct memory access exploits on core application nodes.

### Code Generation

Code generation is the final phase of a compiler where the internal abstract syntax tree or intermediate representation is mapped into concrete machine instructions or assembly code.

* **Register Allocation:** The compiler decides which program variables should reside in the CPU's high-speed registers versus the slower system RAM. Optimized register allocation minimizes memory bus latency.
* **Instruction Selection:** The code generator chooses the most efficient combination of machine instructions supported by the target processor architecture to execute a logical code sequence.

### Loop Optimization

Loops are standard focal points for optimization because they represent the areas where programs spend the majority of their execution cycles.

* **Loop Unrolling:** Replicating the loop body to reduce the total number of branch evaluations and induction variable modifications, which decreases control hazards in pipelined CPUs.
* **Loop-Invariant Code Motion (Hoisting):** Detecting expressions inside a loop that produce the exact same result regardless of the loop iteration index and moving them outside the loop body.
* **Loop Fusion (Combining):** Merging two adjacent loops that iterate over the exact same range into a single loop, maximizing cache locality for data arrays.

---

## 5.2. Programming Paradigms & C/C++ Fundamentals

### Evolution of Programming Paradigms

1. **Procedural Programming:** Programs are organized as a linear, top-down sequence of procedure calls (functions). Data and the functions that mutate them are kept distinct and separate.
2. **Structural Programming:** A subset of procedural programming that enforces logical control flow structures (sequences, selections like `if/else`, and loops like `while/for`), explicitly discouraging unstructured jumps such as `goto`.
3. **Object-Oriented Programming (OOP):** A paradigm based on the concept of **Objects**, which encapsulate both data (attributes) and behavioral code (methods) together.

#### Core Pillars of OOP

* **Encapsulation:** Hiding internal object details and requiring all interactions to go through public interfaces.
* **Inheritance:** The mechanism by which a child class acquires the attributes and behaviors of a parent class, facilitating code reuse.
* **Polymorphism:** The ability of different classes to respond to the exact same method call in unique ways. This includes **Compile-Time** polymorphism (Function Overloading) and **Runtime** polymorphism (Method Overriding using Virtual Functions).
* **Abstraction:** Hiding complex implementation details and showing only the essential high-level operational features to the outside program.

### Fundamentals of C/C++ Programming

* **Memory Pointers:** C/C++ grants direct access to physical memory addresses via pointer variables (e.g., `int *ptr = &var;`).
* **Manual Storage Management:** C uses `malloc()` and `free()`, while C++ uses `new` and `delete`. This provides execution speed but carries the risk of memory leaks and segmentation faults if managed improperly.
* **Pointers vs. References:** A pointer is a variable that holds a memory address and can be changed to point elsewhere, while a reference acts as a permanent alias to an existing variable.

---

## 5.3. Java Programming for Software Development

Java is a strongly typed, object-oriented language designed with built-in memory safety and modularity, making it an ideal choice for enterprise core banking applications.

### Declaration and Type Safety

Java requires explicit variable declarations and strictly enforces compile-time type checking, reducing runtime type mismatches.

### Modularity and Project Structure

* **Packages:** Java uses packages to group related classes, interfaces, and sub-packages together, creating isolated namespaces that prevent naming collisions.
* **Java Platform Module System (JPMS):** Introduced in Java 9, this framework allows developers to explicitly declare which packages a module exports to the outside world and which external modules it requires, enhancing application maintainability.

### Storage Management & Garbage Collection (GC)

Unlike C/C++, Java prevents direct pointer manipulation. All class objects are dynamically allocated on the **Heap Memory Space**.

```
+-------------------------------------------------------------+
|                      HEAP MEMORY SPACE                      |
|  +------------------------+    +-------------------------+  |
|  |    Young Generation    |    |     Old Generation      |  |
|  |  [Eden] -> [S0] -> [S1]| ──>| (Long-lived Objects)    |  |
|  +------------------------+    +-------------------------+  |
+-------------------------------------------------------------+
       │                                       │
       ▼                                       ▼
 Minor GC Cleans Here                    Major GC Runs Here
 (Fast, frequent)                        (Stop-the-World pause)

```

* **Garbage Collection Mechanism:** A background daemon thread automatically tracks object references. When an object becomes unreachable (no active references point to it), it is marked for destruction and its memory is reclaimed.
* **Generational Hypothesis:** Most objects die young. Therefore, the Java heap is divided into:
1. *Young Generation (Eden, Survivor Spaces S0/S1):* Where new objects are created. Reclaimed frequently via quick **Minor GCs**.
2. *Old (Tenured) Generation:* Where objects that survive multiple rounds of minor GC are moved. Reclaimed less frequently via resource-intensive **Major GCs**.



---

## 5.4. System Analysis and Design (SAD)

### Software Development Life Cycle (SDLC) Models

The SDLC provides a structured framework for planning, creating, testing, and deploying information systems.

#### 1. Waterfall Model

A linear, sequential development model where each phase must be fully completed before the next phase begins. Phases include: Requirements $\rightarrow$ Design $\rightarrow$ Implementation $\rightarrow$ Testing $\rightarrow$ Maintenance.

* *Pros:* Simple to manage, clear documentation, works well when requirements are completely fixed and understood.
* *Cons:* Highly rigid; testing occurs late in the cycle, making modifications expensive.

#### 2. Prototyping Model

An iterative model where a simplified, working version of the software (a prototype) is quickly built and presented to users for feedback.

* *Pros:* Clarifies user requirements early and reduces misunderstandings.
* *Cons:* Can increase management overhead, and prototypes are sometimes mistakenly put directly into production.

#### 3. Spiral Model

A risk-driven, evolutionary model that splits the project into multiple loops (spirals). Each loop involves: Identification of Objectives $\rightarrow$ Risk Evaluation $\rightarrow$ Engineering/Development $\rightarrow$ Planning for the Next Phase.

* *Pros:* Excellent risk assessment, suitable for large enterprise projects.
* *Cons:* Expensive, complex to manage, highly reliant on expert risk analysts.

#### 4. Rapid Application Development (RAD)

An agile-style model that prioritizes rapid prototyping and quick, iterative feedback over long documentation and planning phases. It relies on CASE tools and component reuse.

### Structured Analysis and Design Tools

#### 1. Requirement Analysis

The foundational phase of gathering, sorting, and documenting what the software system must do. It differentiates between **Functional Requirements** (e.g., "The system must deduct a 10 Rs fee for interbank transfers") and **Non-Functional Requirements** (e.g., "The system must process transactions within 1.5 seconds").

#### 2. Context Diagram & Data Flow Diagrams (DFDs)

DFDs visually track how data moves through an information system.

* **Context Diagram (Level 0 DFD):** A high-level view that represents the entire system as a single process bubble, mapping out its relationships with external entities.

```
+----------------+      Valid Credentials       +---------------+      Account Balance      +-----------------+
| Bank Customer  | ───────────────────────────> |  Core Banking | ────────────────────────> | External Ledger |
| (Ext. Entity)  | <─────────────────────────── | System (Proc) | <──────────────────────── |  (Ext. Entity)  |
+----------------+      Transaction Receipt     +---------------+      Settlement Status    +-----------------+

```

* **Level 1 DFD:** Breaks down the single process bubble into core sub-processes (e.g., Authentication, Balance Inquiry, Ledger Update), showing the data stores involved.
* *DFD Symbols:* Process (Circle/Rounded Rectangle), Data Flow (Arrow), Data Store (Open-ended Rectangle), External Entity (Square/Rectangle).

#### 3. Entity-Relationship (E-R) Diagram

A conceptual data modeling tool used to map database structures. It consists of **Entities** (rectangles), **Attributes** (ellipses), and **Relationships** (diamonds) with defined cardinalities ($1:1, 1:N, M:N$).

#### 4. Unified Modeling Language (UML)

A standardized, graphical modeling language used to specify, visualize, and document object-oriented software designs.

* **Use Case Diagram:** Maps the system's functional requirements by showing the relationships between **Actors** and **Use Cases**.
* **Class Diagram:** A static structural diagram showing the system's classes, their attributes, methods, and inheritances.
* **Sequence Diagram:** An interaction diagram showing how processes operate with one another and in what chronological sequence messages are sent.

---

## 5.5. Review & Practice Questions

### Section A: Multiple-Choice Questions (MCQs) — Specially Designed for Paper I (Section B)



Choose the single best option. Write your answers in uppercase (A, B, C, D) as per exam rules.

#### Q1. Which component within the Java Virtual Machine (JVM) architecture compiles frequently executed bytecode loops into native machine instructions at runtime?

A. Class Loader

B. Just-In-Time (JIT) Compiler

C. Bytecode Verifier

D. Garbage Collector

#### Q2. Moving a mathematical expression that yields an identical computational output across all loop iterations completely outside the loop block is called:

A. Loop Unrolling

B. Loop Fusion

C. Loop-Invariant Code Motion

D. Induction Variable Reduction

#### Q3. Which Object-Oriented Programming (OOP) pillar involves hiding internal object states and requiring all interactions to happen through clear, public interface methods?

A. Inheritance

B. Polymorphism

C. Encapsulation

D. Structural Induction

#### Q4. What feature distinguishes a standard pointer variable from a reference variable in C++?

A. A reference variable can be reassigned to point to different storage blocks at any time.

B. A pointer variable directly holds a memory address and can be incremented using arithmetic.

C. Pointers do not allocate memory space, whereas references allocate dynamic heap segments.

D. References require explicit dereferencing using the asterisk symbol (`*`).

#### Q5. According to the generational hypothesis utilized by the Java Garbage Collector, objects that survive multiple rounds of minor GC are moved to which memory space?

A. Eden Space

B. Survivor Space S0

C. Old (Tenured) Generation

D. Stack Memory

#### Q6. Which Software Development Life Cycle (SDLC) model is explicitly structured around iterative risk evaluation at every phase of development?

A. Waterfall Model

B. Spiral Model

C. Rapid Application Development (RAD)

D. V-Model

#### Q7. A Level 0 Data Flow Diagram (DFD) is also universally recognized by software engineers as a:

A. Entity-Relationship Diagram

B. Context Diagram

C. Use Case Diagram

D. Sequence Deployment Map

#### Q8. In structured system design notation, what geometric shape represents an External Entity on a Data Flow Diagram?

A. Circle

B. Open-ended Rectangle

C. Square / Rectangle

D. Diamond

#### Q9. Which Unified Modeling Language (UML) interaction diagram is explicitly optimized to visualize how system objects communicate with each other over a chronological timeline?

A. Class Diagram

B. Sequence Diagram

C. Component Diagram

D. Deployment Diagram

#### Q10. What type of requirement specifies system performance attributes, such as requiring a core banking module to handle 500 concurrent logins per second?

A. Functional Requirement

B. Non-Functional Requirement

C. Relational Attribute

D. Structural Constraint

---

### Answers and Explanations for Section A (MCQs)

| Question No. | Correct Answer | Technical Explanation |
| --- | --- | --- |
| **Q1** | **B** | The JIT compiler optimizes performance by compiling frequently executed bytecode segments into native machine code at runtime. |
| **Q2** | **C** | Loop-invariant code motion (hoisting) identifies expressions inside loops whose results do not change across iterations and moves them outside the loop to save cycles. |
| **Q3** | **C** | Encapsulation wraps data and code together, restricting direct access to an object's internal state and exposing it only through public methods. |
| **Q4** | **B** | A pointer stores a memory address and supports pointer arithmetic, whereas a reference acts as a permanent alias for an existing variable. |
| **Q5** | **C** | Long-lived objects that survive multiple minor garbage collection cycles are promoted from the young generation to the old (tenured) generation. |
| **Q6** | **B** | The Spiral model is a risk-driven lifecycle model that evaluates potential project risks during each iterative loop. |
| **Q7** | **B** | A Level 0 DFD, or Context Diagram, treats the entire system as a single process to show its data flows with external entities. |
| **Q8** | **C** | In standard DFD notation, external entities are drawn as squares or rectangles, processes as circles, and data stores as open rectangles. |
| ** | Q9** | **B** |
| **Q10** | **B** | Non-functional requirements specify operational criteria like performance, scalability, and security, rather than specific system behaviors. |

---

### Section B: Subjective Questions (Practice Material for Paper II — Section B)



#### Q1. Explain the differences between Object-Oriented Programming (OOP) and Procedural Programming. Provide an architectural reason why OOP is preferred for core banking applications. (Short Answer - 5 Marks)



##### Model Answer:

**Procedural Programming** and **Object-Oriented Programming (OOP)** represent two distinct structural paradigms in software development.

##### Key Differences:

* **Core Focus:** Procedural programming organizes code around sequential actions and functions that transform data. OOP structures code around autonomous **Objects** that combine data and behaviors into unified components.
* **Data Vulnerability:** In procedural programming, global data is often exposed across the entire program and can be modified by any function. OOP uses access modifiers (such as `private` and `protected`) to secure data fields, restricting modification to the object's own methods.
* **Design Orientation:** Procedural code uses a top-down decomposition model, breaking a large problem into smaller functional subroutines. OOP follows a bottom-up design model, building reusable class components that interact with one another.

##### Why OOP is Preferred for Core Banking Applications:

Core banking software systems require strict access control, high reliability, and manageable code maintainability. OOP provides these capabilities through **Encapsulation** and **Polymorphism**.

* *Security through Encapsulation:* An account balance variable inside a class cannot be directly altered by external code. It can only be modified through validated public methods like `deposit()` or `withdraw()`, which enforces business rule checks before any balance update occurs.
* *Maintainability via Polymorphism:* A base `Loan` class can define a generic interface for interest calculation. Specialized subclasses, such as `HomeLoan`, `AutoLoan`, and `AgriculturalLoan`, can override this method to implement their specific rules. This allows system updates to be introduced by adding new subclasses without breaking the existing core framework.

---

#### Q2. Describe the Software Development Life Cycle (SDLC). Compare the Waterfall model and the Spiral model, outlining when a bank should select each model for software procurement. (Long Answer - 10 Marks)



##### Model Answer:

The **Software Development Life Cycle (SDLC)** is a structured framework used by software engineering teams to design, develop, test, and deploy high-quality information systems. It defines a series of sequential and iterative stages—spanning from initial feasibility study and requirement gathering through systems architecture design, active source-code programming, integration testing, final deployment, and long-term maintenance.

---

### Structural Comparison: Waterfall vs. Spiral Model

| Evaluative Aspect | The Classic Waterfall Model | The Evolutionary Spiral Model |
| --- | --- | --- |
| **Process Approach** | Linear and sequential. The project flows downward through rigid phases. | Iterative and evolutionary. The project repeats loops in an expanding spiral structure. |
| **Requirement Flexibility** | Assumes requirements are completely fixed, well-documented, and understood upfront. | Dynamic. Accommodates changing, evolutionary requirements over time. |
| **Risk Management** | Does not feature explicit risk management phases; risks are handled reactively. | Strongly **risk-driven**. Every iterative loop includes a dedicated quadrant for identifying and mitigating risks. |
| **Testing Integration** | Testing occurs late in the development cycle, after implementation is completed. | Testing and prototype evaluation occur during every iterative loop. |
| **Cost and Complexity** | Lower initial management complexity, making it easy to track milestones. | High cost and management complexity, requiring continuous expert risk analysis. |

---

### Selection Criteria for Banking Software Deployment

```
   [ Waterfall Procurement ]                   [ Spiral Development ]
┌──────────────────────────────┐           ┌──────────────────────────────┐
│  - Simple, Fixed Scope       │           │  - Massive Scale / Critical  │
│  - Regulatory Reports        │           │  - Core Banking Migration    │
│  - Clear Vendor Specifications│           │  - Unpredictable Requirements│
└──────────────────────────────┘           └──────────────────────────────┘

```

#### When a Bank Should Select the Waterfall Model:

A commercial bank should choose the Waterfall model for projects with well-defined, stable requirements that are unlikely to change during development.

* *Compliance Tooling:* Developing a standardized system to generate regulatory reporting sheets for Nepal Rastra Bank, where data inputs and output formats are clearly defined by law.
* *Off-the-shelf Procurement:* Purchasing an established, well-documented third-party module (such as a standard SMS alert gateway) where integration steps are predictable.

#### When a Bank Should Select the Spiral Model:

The bank should choose the Spiral model for large-scale, complex, and high-risk projects where requirements may evolve over time.

* *Core Banking System (CBS) Migration:* Migrating the entire central ledger network from an aging system to a modern platform (e.g., updating to a new version of Finacle). This type of transition carries significant operational and financial risks, requiring continuous prototyping, security validation, and risk analysis at every step.
* *Custom Omni-Channel Mobile Application Platforms:* Building a custom digital banking system that integrates mobile applications, web portals, and AI-driven fraud detection engines. The iterative nature of the Spiral model allows the bank to deploy functional prototypes, evaluate user feedback, identify security risks, and adapt to changing technical trends over the course of the project.


# Unit 6: Data Structure and Algorithm

## 6.1. Fundamentals of Data Structures & Abstract Data Types (ADTs)

A **Data Structure** is a systematic way of organizing, managing, and storing data in a computer's memory so that operations can be performed efficiently.

### Abstract Data Type (ADT)

An ADT is a mathematical model for a certain class of data structures that have similar behavior. It defines the **logical properties** of data and the **operations** allowed on that data *without* specifying the underlying implementation details.

* **Separation of Concerns:** An ADT specifies *what* operations can be performed (e.g., a Queue's `enqueue` and `dequeue`), while the concrete Data Structure defines *how* those operations are implemented in code (e.g., using a contiguous array or a linked list).

---

## 6.2. Linear Data Structures

Linear data structures organize data elements sequentially, where each element is connected to its immediate predecessor and successor.

```
Stack (LIFO)              Queue (FIFO)                  Linked List
   [ Top ]                  [ Rear ]                     [Head]
      │                         │                           │
      ▼                         ▼                           ▼
  ┌───────┐                 ┌───────┐                 ┌────┬───┐    ┌────┬───┐
  │ Item3 │                 │ Item3 │                 │Data│Ref│───>│Data│Nil│
  ├───────┤                 ├───────┤                 └────┴───┘    └────┴───┘
  │ Item2 │                 │ Item2 │
  ├───────┤                 ├───────┤
  │ Item1 │                 │ Item1 │
  └───────┘                 └───────┘
                                ▲
                                │
                             [Front]

```

### 1. Stacks

* **Principle:** LIFO (Last-In, First-Out).
* **Key Operations:** `push()` (adds to top) and `pop()` (removes from top). Both run in $O(1)$ time complexity.
* **Banking Use Case:** Backtracking logic, undo/redo logs, or parsing arithmetic expressions in transactional engines.

### 2. Queues

* **Principle:** FIFO (First-In, First-Out).
* **Key Operations:** `enqueue()` (inserts at rear) and `dequeue()` (removes from front). Both run in $O(1)$ time complexity.
* **Banking Use Case:** Managing customer service queues at branch counters, message brokering (RabbitMQ/Kafka) for processing online transaction notifications.

### 3. Priority Queues

* **Principle:** Each element is assigned an explicit priority. Elements are dequeued based on priority rather than arrival order.
* **Implementation:** Typically implemented using a binary heap, giving $O(\log n)$ insertion and deletion times.
* **Banking Use Case:** Real-time transaction processing networks where high-value settlement requests are prioritized over standard retail transactions.

### 4. Linked Lists

A collection of nodes containing data and references (pointers) to the next node in sequence.

* **Singly Linked List:** Each node contains a data field and a single `next` pointer.
* **Doubly Linked List:** Each node contains a `prev` pointer, a data field, and a `next` pointer, allowing traversal in both directions.
* **Key Comparison:** Unlike static arrays, linked lists do not require contiguous memory allocation, allowing for dynamic size changes. However, accessing an arbitrary element requires linear scanning ($O(n)$ time), whereas an array allows $O(1)$ direct index access.

---

## 6.3. Trees: Non-Linear Hierarchical Structures

A tree is a non-linear, hierarchical data structure consisting of nodes connected by edges.

### General Trees vs. Binary Trees

* **General Tree:** A tree where each node can have an infinite number of child nodes.
* **Binary Tree:** A structured tree where each node can have **at most two child nodes**, traditionally referred to as the left child and right child.

### Tree Traversal Strategies

Tree traversal is the process of visiting every node in a tree exactly once.

* **Pre-order (Depth-First):** Visit Root $\rightarrow$ Traverse Left Subtree $\rightarrow$ Traverse Right Subtree.
* **In-order (Depth-First):** Traverse Left Subtree $\rightarrow$ Visit Root $\rightarrow$ Traverse Right Subtree. *Note: In-order traversal of a Binary Search Tree (BST) visits nodes in ascending, sorted order.*
* **Post-order (Depth-First):** Traverse Left Subtree $\rightarrow$ Traverse Right Subtree $\rightarrow$ Visit Root.
* **Level-order (Breadth-First):** Visits nodes level-by-level from top to bottom, left to right. It is implemented using an auxiliary **Queue**.

### Self-Balancing Binary Search Trees (BST)

A standard BST can become unbalanced (skewed) over time, degrading search, insertion, and deletion times from $O(\log n)$ to $O(n)$. Balanced search trees solve this by restructuring themselves during modifications to maintain a height of $O(\log n)$.

* **AVL Trees:** A strictly height-balanced BST. The heights of the left and right subtrees of any node can differ by **at most 1** (Balance Factor $\in \{-1, 0, 1\}$). It uses rotations (Left, Right, Left-Right, Right-Left) to restore balance after insertions or deletions.
* **2-3 Trees:** A self-balancing search tree where internal nodes can have either two children (2-node, containing one data key) or three children (3-node, containing two data keys).
* **Red-Black Trees:** A balanced BST that uses a color property (Red or Black) on each node to ensure the tree remains approximately balanced. The longest path from the root to a leaf is no more than twice as long as the shortest path.
* *Rule 1:* Every node is either red or black.
* *Rule 2:* The root and all leaf nodes (NIL) are always black.
* *Rule 3:* Red nodes cannot have red children (no two consecutive red nodes).
* *Rule 4:* Every path from a node to any of its descendant NIL leaves must contain the exact same number of black nodes.



---

## 6.4. Indexing, Hashing, and Advanced Trees

### Indexing Methods

Database indexes speed up data retrieval. While BSTs work well in-memory, disk-based storage engines use structures with higher branching factors.

* **B-Trees / B+ Trees:** Balanced search trees optimized for secondary disk storage. Instead of two children, a single node can have hundreds of children (high fan-out).
* In a **B+ Tree**, all user data records are stored exclusively in the leaf nodes, which are linked together in a sequence. Internal nodes store only search keys and routing pointers. This design improves range query performance on disk.



### Hashing

Hashing maps key values directly to a fixed-size index array (hash table) using a mathematical function (hash function). This allows for $O(1)$ average-case search, insertion, and deletion times.

```
Key ───> [ Hash Function ] ───> Index ───> [ Hash Table Array ]
                                                ├──────────────┤
                                                │   Record A   │
                                                ├──────────────┤
                                                │   Record B   │ (Collision!)
                                                └──────────────┘

```

* **Collisions:** Occur when the hash function maps two distinct keys to the exact same array index.
* **Collision Resolution Techniques:**
1. *Chaining (Open Addressing/Closed Book):* Each index bucket of the hash table points to a linked list containing all elements that hash to that same index.
2. *Open Addressing (Closed Hashing):* All elements are stored directly inside the hash table array. If a collision occurs, the system probes alternative array positions using:
* **Linear Probing:** Checking the next sequential slot ($i+1, i+2, \dots$).
* **Quadratic Probing:** Checking slots using quadratic increments ($i+1^2, i+2^2, \dots$).
* **Double Hashing:** Using a second hash function to calculate the step size for probing.





### Specialized Trees

* **Suffix Trees:** A compressed trie containing all suffixes of a given text string. They allow for fast substring searches in $O(m)$ time, where $m$ is the length of the query string.

---

## 6.5. Algorithm Analysis & Complexities

Algorithms are analyzed to estimate how their run times and memory requirements scale with input size ($n$).

### Asymptotic Notations

* **Big-O Notation ($O$):** Defines the mathematical upper bound (worst-case execution performance).
* **Omega Notation ($\Omega$):** Defines the mathematical lower bound (best-case performance).
* **Theta Notation ($\Theta$):** Defines the tight bound (average-case performance).

### Analyzing Non-Recursive Algorithms

Typically involves summing up the execution costs of nested loop operations.

```cpp
// Example: Non-recursive matrix traversal
for (int i = 0; i < n; i++) {       // Runs n times
    for (int j = 0; j < n; j++) {   // Runs n times per outer loop
        sum += matrix[i][j];         // O(1) operation
    }
}
// Total Time Complexity: O(n * n * 1) = O(n^2)

```

### Analyzing Recursive Algorithms

Recursive execution times are written as recurrence relations, which can be solved using several techniques.

#### 1. The Master Theorem

Used to solve recurrences of the form:


$$T(n) = aT\left(\frac{n}{b}\right) + f(n)$$


where $a \geq 1$ represents the number of recursive subproblems, $b > 1$ is the factor by which the subproblem size is reduced, and $f(n)$ is the cost of dividing and merging the subproblems.

We compare $f(n)$ with $n^{\log_b a}$:

* **Case 1:** If $f(n) = O(n^{\log_b a - \epsilon})$ for some constant $\epsilon > 0$, then:

$$T(n) = \Theta(n^{\log_b a})$$


* **Case 2:** If $f(n) = \Theta(n^{\log_b a})$, then:

$$T(n) = \Theta(n^{\log_b a} \log n)$$


* **Case 3:** If $f(n) = \Omega(n^{\log_b a + \epsilon})$ for some constant $\epsilon > 0$, and if $a f(n/b) \leq c f(n)$ for some constant $c < 1$ (regularity condition), then:

$$T(n) = \Theta(f(n))$$



---

## 6.6. Core Algorithm Paradigms

| Algorithmic Paradigm | Strategy Details | Typical Banking Application |
| --- | --- | --- |
| **Divide-and-Conquer** | Breaks a problem down into independent subproblems, solves them recursively, and merges their solutions. | Multi-threaded balance updates, ledger search patterns. |
| **Dynamic Programming** | Solves complex problems by combining solutions to overlapping subproblems. It stores intermediate results in a table (memoization) to avoid redundant computations. | Portfolio optimization models, currency exchange rate routing. |
| **Greedy Methods** | Makes the locally optimal choice at each step with the goal of finding a global optimum. | Minimum coin-change dispensing algorithms at ATMs. |
| **Backtracking** | Explores candidate solutions systematically and abandons (backtracks) paths as soon as it determines they cannot lead to a valid final solution. | Vault security path access networks, fraud graph traversals. |

### Sorting Algorithms

* **Merge Sort (Divide-and-Conquer):** Splits the array in half, recursively sorts both halves, and merges them.
* *Complexity:* $O(n \log n)$ across all cases.


* **Quick Sort (Divide-and-Conquer):** Chooses a pivot element, partitions the array so smaller elements go left and larger go right, and recursively sorts the partitions.
* *Complexity:* $O(n \log n)$ average-case, but degrades to $O(n^2)$ worst-case if the pivot selection is poor (e.g., on already sorted data).


* **Heap Sort (Comparison-based):** Inserts elements into a binary heap and repeatedly extracts the maximum element.
* *Complexity:* $O(n \log n)$ across all cases.



---

## 6.7. Graph Algorithms

A graph $G = (V, E)$ consists of a set of vertices (nodes, $V$) and edges (connections, $E$).

### Graph Traversals

* **Breadth-First Search (BFS):** Explores all neighbor vertices at the current depth before moving to nodes at the next depth level. It is implemented using a **Queue** and has a time complexity of $O(V + E)$.
* **Depth-First Search (DFS):** Explores as deep as possible along each branch before backtracking. It is implemented using a **Stack** (or recursion) and has a time complexity of $O(V + E)$.

### Shortest Path Algorithms

* **Dijkstra's Algorithm:** Finds the shortest path from a single source node to all other nodes in a graph with non-negative edge weights.
* *Complexity:* $O((V + E) \log V)$ when using a binary heap priority queue.


* **Bellman-Ford Algorithm:** Finds the shortest path from a single source node to all other nodes, but supports negative edge weights and can detect negative cycles.
* *Complexity:* $O(V \cdot E)$.



### Minimum Spanning Trees (MST)

An MST is a subset of the edges of a weighted, undirected graph that connects all vertices together without any cycles, while minimizing the total edge weight.

```
      (A)                     (A)
     /   \                   /   
   6/     \5   ───>        6/     
   /   1   \               /   1  
 (B)-------(C)           (B)-------(C)
  Graph G (Weights)        MST (Total Weight = 7)

```

* **Kruskal's Algorithm:** A greedy algorithm that sorts all edges by weight and adds the cheapest edge to the tree, as long as it does not create a cycle. It uses a **Union-Find** data structure.
* **Prim's Algorithm:** A greedy algorithm that grows the tree one vertex at a time. It starts from an arbitrary root vertex and repeatedly adds the cheapest edge connecting a tree vertex to a non-tree vertex.

---

## 6.8. Review & Practice Questions

### Section A: Multiple-Choice Questions (MCQs) — Specially Designed for Paper I (Section B)



Choose the single best option. Write your answers in uppercase (A, B, C, D) as per exam rules.

#### Q1. Which data structure operates on the Last-In, First-Out (LIFO) principle?

A. Queue

B. Stack

C. Linked List

D. Binary Heap

#### Q2. In an AVL Tree, what is the maximum allowable difference between the heights of the left and right subtrees of any node?

A. 0

B. 1

C. 2

D. Any value, as long as the tree is binary

#### Q3. In which data structure are all user data records stored exclusively in the leaf nodes, while the internal nodes store only search keys and routing pointers?

A. Red-Black Tree

B. AVL Tree

C. B+ Tree

D. Binary Search Tree

#### Q4. If a hash function maps two distinct search keys to the exact same index in a hash table, this condition is called a:

A. Overflow

B. Thrashing

C. Collision

D. Secondary Cluster

#### Q5. What is the time complexity of a recursive algorithm represented by the recurrence $T(n) = 2T(n/2) + \Theta(n)$?

A. $\Theta(n)$

B. $\Theta(\log n)$

C. $\Theta(n \log n)$

D. $\Theta(n^2)$

#### Q6. Which sorting algorithm divides the array in half, recursively sorts both halves, and then merges them, maintaining an $O(n \log n)$ time complexity across all cases?

A. Quick Sort

B. Insertion Sort

C. Merge Sort

D. Bubble Sort

#### Q7. Which algorithmic design paradigm is explicitly based on saving the solutions of overlapping subproblems in a lookup table to avoid redundant calculations?

A. Greedy Methods

B. Divide-and-Conquer

C. Dynamic Programming

D. Backtracking

#### Q8. What auxiliary data structure is required to implement Breadth-First Search (BFS) in a graph traversal?

A. Stack

B. Queue

C. Binary Heap

D. Hash Map

#### Q9. Which shortest path algorithm can detect the presence of negative edge weight cycles in a directed graph?

A. Dijkstra's Algorithm

B. Prim's Algorithm

C. Bellman-Ford Algorithm

D. Kruskal's Algorithm

#### Q10. What is the average-case search time complexity in a well-designed Hash Table?

A. $O(1)$

B. $O(\log n)$

C. $O(n)$

D. $O(n \log n)$

---

### Answers and Explanations for Section A (MCQs)

| Question No. | Correct Answer | Technical Explanation |
| --- | --- | --- |
| **Q1** | **B** | A stack is a LIFO data structure where elements are added and removed from the same end. |
| **Q2** | **B** | The balance factor of any node in an AVL tree must be $-1$, $0$, or $1$, meaning the height difference between subtrees can be at most 1. |
| **Q3** | **C** | B+ Trees optimize leaf node sequence traversal by storing all data records in leaf nodes, while internal nodes store only search keys. |
| **Q4** | **C** | A collision occurs when two distinct keys produce the same hash value and index location. |
| **Q5** | **C** | Using the Master Theorem: $a=2, b=2, f(n) = \Theta(n)$. Since $n^{\log_b a} = n^{\log_2 2} = n^1$, this falls under Case 2, giving $T(n) = \Theta(n \log n)$. |
| **Q6** | **C** | Merge Sort maintains a guaranteed time complexity of $O(n \log n)$ in best, average, and worst cases. |
| **Q7** | **C** | Dynamic Programming uses memoization or tabulation to store solved subproblems and avoid duplicate computations. |
| ** | Q8** | **B** |
| **Q9** | **C** | The Bellman-Ford algorithm can handle negative edge weights and will signal a failure if a negative loop cycle is detected. |
| **Q10** | **A** | Hash tables provide $O(1)$ average-case lookup times by using a hash function to map keys directly to array indices. |

---

### Section B: Subjective Questions (Practice Material for Paper II — Section B)



#### Q1. Define a Binary Search Tree (BST). Explain the conditions under which a standard BST degrades to $O(n)$ performance, and describe how an AVL Tree solves this issue. (Short Answer - 5 Marks)



##### Model Answer:

A **Binary Search Tree (BST)** is a binary tree where each node conforms to the following structural properties:

* The key in the left child node is less than the key in its parent node.
* The key in the right child node is greater than the key in its parent node.
* The left and right subtrees must also be valid binary search trees.

##### Degradation to $O(n)$ Performance:

When keys are inserted into a standard BST in a pre-sorted or near-sorted order (e.g., $[1, 2, 3, 4, 5]$), the tree becomes skewed. It forms a linear, chain-like structure where each node has only one child. In this scenario, search, insertion, and deletion times degrade from the optimal $O(\log n)$ to a linear $O(n)$ time complexity, turning the tree into a linked list.

```
  Standard BST Skewed Degradation:
  (1)
     \
     (2)
        \
        (3)
           \
           (4)  --> Degrades search to O(n)

```

##### The AVL Tree Solution:

An **AVL Tree** is a self-balancing binary search tree. It maintains a balanced height by enforcing that the height difference between the left and right subtrees of any node (the balance factor) is **at most 1** (Balance Factor $\in \{-1, 0, 1\}$).

If an insertion or deletion causes the balance factor of any node to fall outside this range, the AVL tree performs one of four structural rotations:

* **Single Rotations:** Left-Left (LL) or Right-Right (RR).
* **Double Rotations:** Left-Right (LR) or Right-Left (RL).

These rotations restructure the nodes to restore balance, ensuring the tree's height remains $O(\log n)$ and guaranteeing $O(\log n)$ search, insertion, and deletion times.

---

#### Q2. Explain Dijkstra’s Algorithm for solving the single-source shortest path problem. Contrast its strategy with the Bellman-Ford algorithm, and outline how a bank can use graph algorithms for fraud network detection. (Long Answer - 10 Marks)



##### Model Answer:

**Dijkstra’s Algorithm** is a greedy algorithm used to find the shortest paths from a single source vertex to all other vertices in a weighted graph where all edge weights are non-negative.

##### Algorithmic Steps of Dijkstra's:

1. **Initialization:** Set the distance to the source node to 0 and the distance to all other vertices to infinity ($\infty$). Mark all vertices as unvisited. Initialize a min-priority queue with all vertices.
2. **Vertex Selection:** Extract the vertex $u$ with the minimum tentative distance from the priority queue and mark it as visited.
3. **Relaxation:** For each unvisited neighbor $v$ of the current vertex $u$, calculate the alternative path distance through $u$:

$$\text{Distance}[v] = \min(\text{Distance}[v], \text{Distance}[u] + \text{Weight}(u, v))$$


4. **Iteration:** Repeat steps 2 and 3 until the priority queue is empty.

---

### Comparison: Dijkstra's vs. Bellman-Ford

| Architectural Property | Dijkstra’s Algorithm | Bellman-Ford Algorithm |
| --- | --- | --- |
| **Design Paradigm** | Greedy Strategy. | Dynamic Programming. |
| **Edge Weight Support** | Supports **non-negative weights only**. It can fail if negative weights are present. | Supports **negative edge weights**. |
| **Negative Cycle Detection** | Cannot detect negative weight cycles. | Identifies and flags negative weight cycles. |
| **Time Complexity** | $O((V + E) \log V)$ when using a binary heap priority queue. | $O(V \cdot E)$. |
| **Use Case Focus** | Optimized for speed on graphs with non-negative weights. | Used when negative edge weights are possible or for distance-vector routing protocols. |

---

### Banking Application: Fraud Network Detection using Graph Algorithms

In modern digital banking, transactions can be modeled as a graph $G = (V, E)$, where the vertices ($V$) represent accounts, credit cards, or physical devices, and the edges ($E$) represent transaction flows, shared IP addresses, or phone numbers.

```
  [ Fraud Account A ] ───(Transfer)───> [ Mule Account B ]
          │                                     │
     (Shared IP)                          (Shared Phone)
          ▼                                     ▼
  [ Device Node C ] <───(Transfer)─── [ Fraud Account D ]

```

1. **Identifying Mule Networks using DFS/BFS:**
Fraud rings often route stolen funds through a chain of intermediate "mule" accounts to make tracking difficult. By running Breadth-First Search (BFS) or Depth-First Search (DFS) from a known compromised account, the bank can map out the flow of funds and identify linked candidate accounts in real time.
2. **Uncovering Collusion using Shortest Path Algorithms:**
Fraud rings may try to hide connections by routing funds through many intermediate nodes. Shortest path algorithms can identify if two seemingly unrelated accounts are connected by an unusually short chain of high-frequency transfers or shared devices, triggering compliance alerts.
3. **Detecting Cycle Structures for Money Laundering:**
Money laundering schemes often move funds in circular paths (e.g., $A \rightarrow B \rightarrow C \rightarrow A$) to disguise their origin. Running cycle detection algorithms (such as DFS-based back-edge detection) helps banks spot these circular transaction flows automatically.


# Unit 7: Emerging Technologies and Cybersecurity

## 7.1. Basics of Cloud Computing & Characteristics

**Cloud Computing** is a model for enabling ubiquitous, convenient, on-demand network access to a shared pool of configurable computing resources (e.g., networks, servers, storage, applications, and services) that can be rapidly provisioned and released with minimal management effort or service provider interaction.

### Five Essential Characteristics of Cloud Computing

1. **On-Demand Self-Service:** A consumer can unilaterally provision computing capabilities, such as server time and network storage, as needed automatically without requiring human interaction with each service provider.
2. **Broad Network Access:** Capabilities are available over the network and accessed through standard mechanisms that promote use by heterogeneous thin or thick client platforms (e.g., mobile phones, tablets, laptops, and workstations).
3. **Resource Pooling:** The provider’s computing resources are pooled to serve multiple consumers using a multi-tenant model, with different physical and virtual resources dynamically assigned and reassigned according to consumer demand.
4. **Rapid Elasticity:** Capabilities can be elastically provisioned and released, in some cases automatically, to scale rapidly outward and inward commensurate with demand. To the consumer, the capabilities available for provisioning often appear to be unlimited.
5. **Measured Service:** Cloud systems automatically control and optimize resource use by leveraging a metering capability at some level of abstraction appropriate to the type of service (e.g., storage, processing, bandwidth, and active user accounts).

---

## 7.2. Cloud Service Models & Virtualization

```
+-------------------------------------------------------------+
|                     CLOUD SERVICE MODELS                    |
|                                                             |
|   +-----------------------------------------------------+   |
|   |  SaaS (Software as a Service)                       |   |
|   |  e.g., Microsoft 365, Cloud-based HR Portals        |   |
|   +-----------------------------------------------------+   |
|   |  PaaS (Platform as a Service)                       |   |
|   |  e.g., AWS Elastic Beanstalk, Heroku                |   |
|   +-----------------------------------------------------+   |
|   |  IaaS (Infrastructure as a Service)                 |   |
|   |  e.g., AWS EC2, Google Compute Engine               |   |
|   +-----------------------------------------------------+   |
+-------------------------------------------------------------+

```

### 1. Infrastructure as a Service (IaaS)

Provides fundamental computing infrastructure—physical or virtual machines, raw storage, firewalls, and networks—on demand. The consumer does not manage the underlying physical cloud infrastructure but has control over operating systems, storage, and deployed applications.

### 2. Platform as a Service (PaaS)

Provides a cloud environment where customers can build, test, run, and manage applications without the complexity of buying and maintaining the underlying hardware and OS layers. The provider supplies runtime environments, databases, and development tools.

### 3. Software as a Service (SaaS)

Delivers fully functional, end-user applications over the internet, managed entirely by the third-party service vendor. Applications are typically accessed via web browsers, eliminating local installations and maintenance.

### Virtualization: The Enabler of Cloud Computing

**Virtualization** is the fundamental technology that underpins cloud computing. It abstracts physical hardware resources (CPUs, memory, network interfaces, and storage) using a software abstraction layer called a **Hypervisor** (or Virtual Machine Monitor - VMM).

* **Type 1 (Bare-Metal) Hypervisors:** Run directly on the host's physical hardware (e.g., VMware ESXi, Microsoft Hyper-V). Highly secure and performant, making them standard for banking data centers.
* **Type 2 (Hosted) Hypervisors:** Run on top of an existing host operating system (e.g., VirtualBox, VMware Workstation).
* *Banking Value:* Virtualization allows banks to run multiple isolated Virtual Machines (VMs) on a single physical server, dramatically increasing hardware utilization, optimizing energy footprints, and creating snapshot-based disaster recovery environments.

---

## 7.3. Emerging Technologies in Digital Banking

### 1. Application Program Interface (API)

An API is a structured set of definitions and protocols that allows different software applications to communicate and exchange data with one another seamlessly.

* **Open Banking Paradigm:** APIs allow licensed fintech apps to securely fetch account balances or initiate payments directly from the core bank database with the customer's explicit consent, breaking down traditional data silos.

### 2. Internet of Things (IoT)

IoT refers to the network of physical physical objects ("things") embedded with sensors, software, and network connectivity that enables them to collect, transmit, and exchange data.

* *Banking Impact:* Real-time supply chain asset monitoring for commercial inventory financing, and smart biometric tracking networks deployed inside remote ATM booths.

### 3. Big Data

Big Data describes datasets whose scale, distribution, and velocity exceed the processing capabilities of traditional relational database systems. It is characterized by the **5 Vs**: **Volume** (scale of data), **Velocity** (speed of generation), **Variety** (structural forms), **Veracity** (trustworthiness), and **Value** (business utility).

* *Banking Use Case:* Processing multi-million transaction logs daily alongside customer clickstreams to build automated credit risk assessment and real-time behavioral analytics engines.

### 4. Distributed Ledger Technology (DLT)

DLT refers to a decentralized, cryptographically secure digital infrastructure where a common database ledger is replicated, shared, and synchronized across a network of independent nodes without a central controlling authority. **Blockchain** is a specific type of sequential, cryptographic DLT.

* *Banking Impact:* Simplifying cross-border interbank settlements, verifying trade finance letters of credit, and issuing Central Bank Digital Currencies (CBDCs).

### 5. Artificial Intelligence (AI) and Machine Learning (ML)

* **Artificial Intelligence:** The broader scientific discipline of engineering computer systems capable of performing tasks that traditionally require human intelligence.
* **Machine Learning:** A subfield of AI focused on building algorithms that systematically learn patterns from historical training data to make predictions on new data without being explicitly programmed.
* *Banking Impact:* Automated Anti-Money Laundering (AML) transaction parsing, credit-scoring neural networks, and conversational customer-service chatbots.

---

## 7.4. Cyber Security Architectures & Threat Management

### Authentication vs. Authorization

* **Authentication:** The procedural mechanism of verifying **who** an entity is (e.g., confirming identity via passwords, MFA tokens, or fingerprint biometrics).
* **Authorization:** The mechanism of determining **what** resources an authenticated user is legally permitted to access or modify (e.g., checking system roles before allowing a teller to authorize a loan disbursement).

### Password Management Architecture

Enforces password hygiene across the corporate directory using policies such as mandatory length constraints, complexity metrics (uppercase, special characters), cryptographic hashing using salts (e.g., bcrypt/SHA-256), account lockout thresholds after consecutive failures, and multi-factor authentication integration.

### Access Control Mechanisms

1. **Discretionary Access Control (DAC):** The resource owner controls access permissions.
2. **Mandatory Access Control (MAC):** The system enforces strict access rules based on security labels and clearance levels, commonly used in high-security environments.
3. **Role-Based Access Control (RBAC):** Permissions are assigned to specific operational roles, simplifying user permission management in large organizations.
4. **Attribute-Based Access Control (ABAC):** Dynamically grants access by evaluating attributes related to the subject, resource, and current context (e.g., allowing access only during business hours from an authorized IP address).

### Threats and Vulnerabilities

* **Phishing:** Deceptive communication attacks designed to trick bank staff or clients into surrendering sensitive credentials or encryption keys.
* **Ransomware:** Cryptographic malware designed to encrypt local file structures and demand financial ransoms in exchange for the decryption key.
* **Inside Threats:** Malicious or negligent acts by authorized bank employees who misuse their access privileges to compromise sensitive data.
* **Zero-Day Vulnerabilities:** Software flaws that are exploited by attackers before the vendor has developed or released a security patch.

---

## 7.5. Review & Practice Questions

### Section A: Multiple-Choice Questions (MCQs) — Specially Designed for Paper I (Section B)



Choose the single best option. Write your answers in uppercase (A, B, C, D) as per exam rules.

#### Q1. Which cloud computing characteristic describes the ability to rapidly provision and release infrastructure components to match changing system demands?

A. Resource Pooling

B. Broad Network Access

C. Rapid Elasticity

D. Measured Service

#### Q2. A bank implements a cloud deployment model where they rent computing runtimes and database tools from a provider to develop their own applications, without managing the underlying operating system. This service model is called:

A. Infrastructure as a Service (IaaS)

B. Platform as a Service (PaaS)

C. Software as a Service (SaaS)

D. Network as a Service (NaaS)

#### Q3. What software layer is responsible for abstracting physical hardware resources to run multiple isolated Virtual Machines on a single physical host?

A. Kernel

B. Operating System

C. Hypervisor

D. API Gateway

#### Q4. Which of the following is an example of compile-time or runtime software protocols that enable distinct application architectures to securely share data and communicate?

A. Blockchain

B. Application Program Interface (API)

C. Hypervisor

D. Virtual Private Network (VPN)

#### Q5. In Big Data architectures, which "V" describes the structural diversity of the incoming data streams (e.g., structured SQL data, semi-structured XML, and unstructured logs)?

A. Volume

B. Velocity

C. Variety

D. Veracity

#### Q6. What core benefit does Distributed Ledger Technology (DLT) offer to interbank cross-border payment settlements?

A. Centralizes all authorization logs under a single regulatory node

B. Eliminates the need for cryptographic hash functions

C. De-escalates network access requirements by disabling encryption

D. Provides a decentralized, immutable ledger that reduces the reliance on central intermediaries

#### Q7. The distinct process of validating the identity claim of an application user trying to log into a system is known as:

A. Authorization

B. Authentication

C. Auditing

D. Accounting

#### Q8. Which access control mechanism grants resource access based on security levels and classification labels defined by a central administrator?

A. Discretionary Access Control (DAC)

B. Role-Based Access Control (RBAC)

C. Mandatory Access Control (MAC)

D. Attribute-Based Access Control (ABAC)

#### Q9. What security threat involves exploiting an unpatched software vulnerability that is completely unknown to the vendor or the public?

A. Phishing

B. Man-in-the-Middle Attack

C. Zero-Day Vulnerability

D. Brute-Force Exploit

#### Q10. A machine learning system that maps input data to known labels using historical, pre-labeled training datasets is categorized as:

A. Unsupervised Learning

B. Supervised Learning

C. Reinforcement Learning

D. Semi-Supervised Clustering

---

### Answers and Explanations for Section A (MCQs)

| Question No. | Correct Answer | Technical Explanation |
| --- | --- | --- |
| **Q1** | **C** | Rapid elasticity allows cloud resources to scale up or down dynamically to match workload changes. |
| **Q2** | **B** | PaaS provides a managed application platform and database tools, allowing developers to focus on building software without managing the underlying operating system. |
| **Q3** | **C** | A hypervisor creates a virtualization layer that abstracts physical hardware, enabling multiple independent virtual machines to run on a single physical host. |
| **Q4** | **B** | APIs provide a standardized set of protocols that allow different software applications to communicate and exchange data securely. |
| **Q5** | **C** | Variety refers to the different formats of incoming data, which can include structured database tables, semi-structured files, and unstructured log files. |
| **Q6** | **D** | DLT provides a shared, immutable ledger across network nodes, which can streamline cross-border settlements by reducing the need for traditional intermediaries. |
| **Q7** | **B** | Authentication verifies a user's identity claim, whereas authorization defines what resources that user is permitted to access. |
| **Q8** | **C** | Mandatory Access Control (MAC) enforces access restrictions based on centrally defined security labels and clearance levels. |
| **Q9** | **C** | A zero-day vulnerability is a security flaw that is exploited before the software vendor has discovered it or released a patch. |
| ** | Q10** | **B** |

---

### Section B: Subjective Questions (Practice Material for Paper II — Section B)



#### Q1. Distinguish between Authentication and Authorization. Explain the role of Multi-Factor Authentication (MFA) in protecting online banking systems against credential theft. (Short Answer - 5 Marks)



##### Model Answer:

**Authentication** and **Authorization** are complementary security processes used to manage system access, but they serve different purposes:

* **Authentication:** Verifies the identity of a user or system component attempting to log into a network. It answers the question, *"Who are you?"* The identity claim is validated using factors like passwords, biometrics, or security tokens.
* **Authorization:** Occurs after successful authentication and determines what resources, directories, or actions the validated user is permitted to access. It answers the question, *"What are you allowed to do?"* This is typically managed via role-based permissions (RBAC) or access control lists.

##### Role of Multi-Factor Authentication (MFA) in Online Banking:

MFA significantly enhances online banking security by requiring users to provide two or more independent authentication factors before access is granted. These factors are drawn from three primary categories:

1. **Knowledge Factor:** Something the user knows (e.g., a password or PIN).
2. **Possession Factor:** Something the user has (e.g., a hardware token or a one-time password sent to a registered mobile device).
3. **Inherence Factor:** Something the user is (e.g., biometric data like a fingerprint or facial recognition scan).

```
   [ Password Input ]   +   [ Mobile OTP / Token ]   =   [ Access Granted ]
   (Knowledge Factor)        (Possession Factor)
            │                         │
            ▼                         ▼
  Compromised by Phishing!     Remains Secure! ---------> Attack Blocked

```

If an attacker steals a user's password through a phishing campaign or data breach, they still cannot access the account without the second factor (such as a time-sensitive OTP or biometric scan). This reduces the risk of credential theft and unauthorized access.

---

#### Q2. What is Cloud Computing? Discuss the functional differences between Infrastructure as a Service (IaaS), Platform as a Service (PaaS), and Software as a Service (SaaS). Evaluate the primary challenges a state-owned bank faces when migrating legacy systems to a public cloud environment. (Long Answer - 10 Marks)



##### Model Answer:

**Cloud Computing** is a model that enables convenient, on-demand network access to a shared pool of configurable computing resources (such as networks, servers, storage, applications, and services) that can be rapidly provisioned and managed with minimal service provider interaction. It shifts the IT management model from upfront Capital Expenditures (CapEx) to scalable Operational Expenditures (OpEx).

---

### Service Model Responsibility Matrix

The operational differences between cloud service models depend on which responsibilities are handled by the cloud vendor versus the customer:

| Functional Layer | On-Premises Legacy | IaaS Framework | PaaS Framework | SaaS Framework |
| --- | --- | --- | --- | --- |
| **End-User Applications** | Bank Manages | Bank Manages | Bank Manages | Vendor Manages |
| **Data & Databases** | Bank Manages | Bank Manages | Bank Manages | Vendor Manages |
| **Runtime Environments** | Bank Manages | Bank Manages | Vendor Manages | Vendor Manages |
| **Operating Systems** | Bank Manages | Bank Manages | Vendor Manages | Vendor Manages |
| **Virtualization & Hypervisor** | Bank Manages | Vendor Manages | Vendor Manages | Vendor Manages |
| **Physical Servers & Network** | Bank Manages | Vendor Manages | Vendor Manages | Vendor Manages |
| **Data Center Infrastructure** | Bank Manages | Vendor Manages | Vendor Manages | Vendor Manages |

* **Infrastructure as a Service (IaaS):** The vendor provides core computing infrastructure, such as virtual servers, network loops, and raw storage blocks. The bank retains full administrative control over the operating system, middleware, databases, and application deployments.
* **Platform as a Service (PaaS):** The vendor manages the hardware infrastructure, operating system, and runtime environments. The bank is responsible for writing and deploying its own application code and managing data configurations.
* **Software as a Service (SaaS):** The vendor manages the entire stack, delivering fully functional software applications directly to users over a web browser, eliminating the need for local infrastructure management.

---

### Challenges Faced by State-Owned Banks in Public Cloud Migration

Migrating legacy banking architectures to a public cloud environment presents several significant operational, regulatory, and security challenges:

```
                  ┌──────────────────────────────────────────────┐
                  │ PUBLIC CLOUD MIGRATION CHALLENGES FOR BANKS  │
                  └──────────────────────┬───────────────────────┘
            ┌────────────────────────────┼────────────────────────────┐
            ▼                            ▼                            ▼
┌──────────────────────┐     ┌──────────────────────┐     ┌──────────────────────┐
│ Regulatory & Data    │     │ Legacy Architectural │     │ Vendor Lock-in       │
│ Sovereignty Risks    │     │ Monoliths            │     │ & Hidden Costs       │
└──────────────────────┘     └──────────────────────┘     └──────────────────────┘

```

#### 1. Regulatory Compliance and Data Sovereignty

Central bank guidelines (such as those from Nepal Rastra Bank) often place strict conditions on where financial records and customer data can be physically stored. If a public cloud provider does not maintain an authorized local data center region within the country, migrating sensitive banking data abroad can violate data sovereignty laws and compliance regulations.

#### 2. Legacy Monolithic Architectures

Many state-owned banking networks rely on older core banking software systems that were designed to run on dedicated, on-premises mainframes. These legacy structures are often deeply interdependent, making them difficult to decouple and migrate without extensive code refactoring or complete systems replacement.

#### 3. Vendor Lock-In Risks

Migrating data and workloads to a specific public cloud provider's proprietary databases or APIs can make it difficult and expensive to switch providers or pull workloads back on-premises in the future.

#### 4. Cybersecurity Boundaries and Shared Responsibility

Moving to the cloud shifts the bank from an isolated, physical network perimeter to a shared responsibility model. Misconfigurations in cloud storage access controls, API keys, or identity management systems can expose critical financial systems to public internet threats if not properly managed by internal security teams.
