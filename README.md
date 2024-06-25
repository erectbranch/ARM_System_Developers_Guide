<div width="100%" height="100%" align="center">
  
<h1 align="center">
  <p align="center">ARM System Developer's Guide</p>
  <a href="https://www.sciencedirect.com/book/9781558608740/arm-system-developers-guide">
    <img width="50%" src="cover.png" />
  </a>
</h1>
  
<b>Andrew N. Sloss, Dominic Symes, Chris Wright 저</b></br>
Morgan Kaufmann · 2004년 04월 08일 출간</b>

</div>

## :bulb: 목표

- **ARM 코어의 동작 원리를 이해한다.**

  > ARM 코어의 동작 원리를 이해하고, C와 어셈블리 예제를 통해 ARM 프로세서를 제어하는 방법을 익힌다.

</br>

## 🚩 정리한 문서 목록

> ▶(folded), ▼(unfolded)

<details markdown="1">
<summary><h3>📖 ARM Processor Fundamentals</h3></summary>

 - [ARM Embedded System](https://github.com/erectbranch/ARM_System_Developers_Guide/tree/master/ch01)

   > CISC vs RISC, RISC Design Philosophy(instructions, pipelines, registers, load-store architecture), ARM Design Philosophy
   
   > Embedded System Hardware(ARM processor, controller, peripheral, bus), ARM Bus Technology(master-slave, AMBA Bus Protocol), Memory(Hierarchies, Memory Width)

   > Embedded System Software(Boot loader, Operating System, Application Software)

 - [Dataflow and Registers](https://github.com/erectbranch/ARM_System_Developers_Guide/tree/master/ch02/summary01)

   > ARM Core Dataflow(Instruction Decoder, Sign Extend, Register File, Barrel Shifter, MAC, ALU, Address Register, Incrementer)

   > General Purpose Registers: 16 Data Registers(Stack Pointer, Link Register, Program Counter), 2 Process Status Registers(cpsr, spsr)
   
   > 20 Banked Registers(Fast Interrupt Request, Interrupt Request, Supervisor, Abort, Undefined)
   
   > Current Program Status Register(Condition flags, Interrupt Masks, Thumb state, Processor mode)

 - [Pipeline, Interrupt, Extensions](https://github.com/erectbranch/ARM_System_Developers_Guide/tree/master/ch02/summary02)

   > Pipeline, Pipeline Executing Characteristics

   > Exceptions, Interrupts, Vector Table

   > Core Extensions: Cache, Tightly Coupled Memory, Memory Management Hardware(MPU, MMU), Coprocessor
</details>

<details markdown="1">
<summary><h3>🗂 ARM Instruction Set</h3></summary>

 - [Data Processing Instructions](https://github.com/erectbranch/ARM_System_Developers_Guide/tree/master/ch03/summary01)

   > Suffix s(Update Condition Flags), Move Instructions(MOV, MVN), Barrel Shifter(LSL, LSR, ASR, ROR, RRX) 

   > Arithmetic Instructions(ADC, ADD, RSB, RSC, SBC, SUB), Logical Instructions(AND, ORR, EOR, BIC)

   > Multiply Instructions(MLA, MUL), Long Multiply Instructions(SMLAL, SMULL, UMLAL, UMULL)

 - [Branch, Load-Store Instructions](https://github.com/erectbranch/ARM_System_Developers_Guide/tree/master/ch03/summary02)

   > Branch Instructions(B, BL, BX, BLX)

   > Single-Register Transfer(LDR, STR, LDRB, STRB, LDRH, STRH, LDRSB, LDRSH), Addressing Mode(Preindex with writeback, Preindex, Postindex), Load and Store Doubleword(LDRD, STRD)

   > Multiple-Register Transfer(LDM, STM), Addressing Mode(IA, IB, DA, DB), Stack Operations(FA, FD, EA, ED)
  
 - [SWI, Program Status Register  Instructions, ARMv5E Extensions](https://github.com/erectbranch/ARM_System_Developers_Guide/tree/master/ch03/summary03)

   > Software Interrupt Instruction(SWI), Software Interrupt Handler

   > Program Status Register Instructions(MRS, MSR), Coprocessor Instructions(CDP, MRC, MCR, LDC, STC)

   > Loading Constants(LDR, ADR), Conditional Execution

   > ARMv5E Extensions: Counting Leading Zeros(CLZ), Saturated Arithmetic(QADD, QDADD, QSUB, QDSUB), Signed Multiply Instructions
(SMLAxy, SMLALxy, SMLAWy, SMULxy, SMULWy)
</details>

<details markdown="1">
<summary><h3>📈 Efficient C Programming</h3></summary>

 - [C Compiler, Data Types, Looping Structures](https://github.com/erectbranch/ARM_System_Developers_Guide/tree/master/ch05/summary01)

   > Efficient C Data Types(Local Variable, Function Argument, Division)

   > Looping Constructs(Fixed Number of Iterations, Variable Number of Iterations, Loop Unrolling)
  
 - [Register Allocation, Function Calls, Aliasing](https://github.com/erectbranch/ARM_System_Developers_Guide/tree/master/ch05/summary02)

   > Spilled Variable, Argument Registers, General Variable Registers

   > Optimizing Function Calls, Pointer Aliasing

 - [Alignment, Endian, Division, Inline Assembly](https://github.com/erectbranch/ARM_System_Developers_Guide/tree/master/ch05/summary03)

   > Structure Alignment(Reordering, Padding), Endianness(Big-endian, Little-endian)

   > Division(cycles, avoiding division)

   > Inline Functions, Inline Assembly
</details>

<details markdown="1">
<summary><h3>🔨 Optimizing ARM Assembly Code</h3></summary>

 - [Assembly Function, Instruction Scheduling](https://github.com/erectbranch/ARM_System_Developers_Guide/tree/master/ch06/summary01)

   > Assembly Function, Call Subroutine

   > Instruction Scheduling(instruction cycles, ARM9TDMI pipeline), Pipeline Hazards, Loop Scheduling(Preloading, Unrolling)

 - [Register Allocation](https://github.com/erectbranch/ARM_System_Developers_Guide/tree/master/ch06/summary02)

   > Procedure Call(ATPCS, AAPCS)
   
   > Allocating Variables, Code Implementation to Reduce Register Usage

 - [Conditional Execution, Looping Constructs](https://github.com/erectbranch/ARM_System_Developers_Guide/tree/master/ch06/summary03)

   > Conditional Execution(Unsigned Comparison, Signed Comparison Cascading Conditions)

   > Looping Constructs(Loop Unrolling, Loop Counter Packing, Decrement Loop Counter, Negative Indexing, Log Indexing)

 - [Bit Manipulation, Switch, Unaligned Data](https://github.com/erectbranch/ARM_System_Developers_Guide/tree/master/ch06/summary04)

   > Bit Manipulation(Fixed-width Packing/Unpacking, Variable-width Packing/Unpacking), Efficient Switches(switch-absolute, switch-relative, Using Hash Function)

   > Handling Unaligned Data
</details>

<details markdown="1">
<summary><h3>⏰ Exception and Interrupt Handling</h3></summary>

 - [Exception Handling](https://github.com/erectbranch/ARM_System_Developers_Guide/tree/master/ch09/summary01)

   > Exception, ARM Processor Modes(FIQ, IRQ, SVC, Abort, Undefined), Vector Table

   > Exception Priority, Link Register Offsets

 - [Interrupt, Interrupt Stack](https://github.com/erectbranch/ARM_System_Developers_Guide/tree/master/ch09/summary02)

   > Interrupts(External, Internal), Minimize Interrupt Latency(Nested Interrupt Handler, Prioritization), IRQ and FIQ Exceptions

   > Interrupt Stack, Stack Overflow, Stack Design
</details>


</br>

## :mag: 목차

### 1. ARM Embedded Systems	

    1.1 The RISC Design Philosophy	
    1.2 The ARM Design Philosophy	
    1.3 Embedded System Hardware	
    1.4 Embedded System Software	
	
### 2. ARM Processor Fundamentals	

    2.1 Registers	
    2.2 Current Program Status Register	
    2.3 Pipeline	
    2.4 Exceptions, Interrupts, and the Vector Table	
    2.5 Core Extensions	
    2.6 Architecture Revisions	
    2.7 ARM Processor Families	
	
### 3. Introduction to the ARM Instruction Set	

    3.1 Data Processing Instructions	
    3.2 Branch Instructions	
    3.3 Load-Store Instructions	
    3.4 Software Interrupt Instruction	
    3.5 Program Status Register Instructions	
    3.6 Loading Constants	
    3.7 ARMv5E Extensions	
    3.8 Conditional Execution	
	
### 4. Introduction to the Thumb Instruction Set	

    4.1 Thumb Register Usage	
    4.2 ARM-Thumb Interworking	
    4.3 Other Branch Instructions	
    4.4 Data Processing Instructions	
    4.5 Single-Register Load-Store Instructions	
    4.6 Multiple-Register Load-Store Instructions	
    4.7 Stack Instructions	
    4.8 Software Interrupt Instruction	
	
### 5. Efficient C Programming	

    5.1 Overview of C Compilers and Optimization	
    5.2 Basic C Data Types	
    5.3 C Looping Structures	
    5.4 Register Allocation	
    5.5 Function Calls	
    5.6 Pointer Aliasing	
    5.7 Structure Arrangement	
    5.8 Bit-fields	
    5.9 Unaligned Data and Endianness	
    5.10 Division	
    5.11 Floating Point	
    5.12 Inline Functions and Inline Assembly	
    5.13 Portability Issues	
	
### 6. Writing and Optimizing ARM Assembly Code	

    6.1 Writing Assembly Code	
    6.2 Profiling and Cycle Counting	
    6.3 Instruction Scheduling	
    6.4 Register Allocation	
    6.5 Conditional Execution	
    6.6 Looping Constructs	
    6.7 Bit Manipulation	
    6.8 Efficient Switches	
    6.9 Handling Unaligned Data	
        
### 7. Optimized Primitives	

    7.1 Double-Precision Integer Multiplication	
    7.2 Integer Normalization and Count Leading Zeros	
    7.3 Division	
    7.4 Square Roots	
    7.5 Transcendental Functions: log, exp, sin, cos	
    7.6 Endian Reversal and Bit Operations	
    7.7 Saturated and Rounded Arithmetic	
    7.8 Random Number Generation	
	
### 8. Digital Signal Processing	

    8.1 Representing a Digital Signal	
    8.2 Introduction to DSP on the ARM	
    8.3 FIR filters	
    8.4 IIR Filters	
    8.5 The Discrete Fourier Transform	
	
### 9. Exception and Interrupt Handling	

    9.1 Exception Handling	
    9.2 Interrupts	
    9.3 Interrupt Handling Schemes	
	
### 10. Firmware	

    10.1 Firmware and Bootloader	
    10.2 Example: Sandstone	
	
### 11. Embedded Operating Systems	

    11.1 Fundamental Components	
    11.2 Example: Simple Little Operating System	
	
### 12. Caches	

    12.1 The Memory Hierarchy and Cache Memory	
    12.2 Cache Architecture	
    12.3 Cache Policy	
    12.4 Coprocessor 15 and Caches	
    12.5 Flushing and Cleaning Cache Memory	
    12.6 Cache Lockdown	
    12.7 Caches and Software Performance	
	
### 13. Memory Protection Units	

    13.1 Protected Regions	
    13.2 Initializing the MPU, Caches, and Write Buffer	
    13.3 Demonstration of an MPU system	
        
### 14. Memory Management Units	

    14.1 Moving from an MPU to an MMU	
    14.2 How Virtual Memory Works	
    14.3 Details of the ARM MMU	
    14.4 Page Tables	
    14.5 The Translation Lookaside Buffer	
    14.6 Domains and Memory Access Permission	
    14.7 The Caches and Write Buffer	
    14.8 Coprocessor 15 and MMU Configuration	
    14.9 The Fast Context Switch Extension	
    14.10 Demonstration: A Small Virtual Memory System	
    14.11 The Demonstration as mmuSLOS