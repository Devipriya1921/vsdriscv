# Pipelined RISC-V CPU Core
The following repository contains all the information and codes that's needed to a build a 4-stage Pipelined RISC-V Core which is designed during the [RISC-V MYTH Workshop](https://github.com/stevehoover/RISC-V_MYTH_Workshop). 
The core supports the RV32I Base Integer Instruction Set. It is developed and implemented in [TL-Verilog](http://tl-x.org/) on [Makerchip platform](https://makerchip.com/).

# Table of Contents
* [Introduction to RISC-V ISA](#introduction-to-risc-v-isa)
* [GNU Compiler Toolchain](#gnu-compiler-toolchain)
* [Introduction to ABI](#introduction-to-abi)
* [Digital Logic with TL-Verilog and Makerchip](#digital-logic-with-tl-verilog-and-makerchip)
  - [Combinational Logic](#combinational-logic)
    - [Inverter](#inverter)
    - [Binary Addition](#binary-addition)
    - [Multiplexer](#multiplexer)
    - [Combinational Calculator](#combinational-calculator)
  - [Sequential Logic](#sequential-logic)
    - [Fibonacci Sequence](#fibonacci-sequence)
    - [Counter](#counter)
    - [Sequential Calculator](#sequential-calculator)
  - [Pipelined Logic](#pipelined-logic)
    - [Snapshot representing Pipelined Logic](#Snapshot-representing-Pipelined-Logic)
    - [2-cycle calculator](#2-cycle-calculator)
  - [Validity](#validity)
    - [Valid When Condition](#Valid-When-Condition)
    - [To Compute Total Distance](#To-Compute-Total-Distance)
    - [2-cycle calculator with validity](#2-cycle-calculator-with-validity)
* [Basic RISC-V CPU Micro-architecture](#basic-risc-v-cpu-micro-architecture)
  - [Instruction Fetch Logic](#Instruction-Fetch-Logic)
  - [Instruction Decode Logic](#Instruction-Decode-Logic)
  - [Register File Read and Write](#register-file-read-and-write)
  - [Execute ALU Operations](#Execute-ALU-Operations)
  - [Control Branch Instruction Logic](#Control-Branch-Instruction-Logic)
* [Pipelined RISC-V CPU Micro-architrcture](#pipelined-risc-v-cpu-micro-architecture)
  - [Pipelinig the CPU](#pipelining-the-cpu)
  - [Load and Store Instructions and Memory](#load-and-store-instructions-and-memory)
  - [Completing the RISC-V Core](#completing-the-risc-v-core)
* [Acknowledgements](#acknowledgements)
* [Author](#author)


# Introduction to RISC-V ISA 

* A RISC-V hardware platform can contain one or more RISC-V-compatible processing cores together with other non-RISC-V-compatible cores, fixed-function accelerators, various physical memory structures, I/O devices, and an interconnect structure to allow the components to communicate.
  A component is termed a core if it contains an independent instruction fetch unit. A RISC-V compatible core might support multiple RISC-V-compatible hardware threads, or harts, through
  multithreading.
* A RISC-V hart has a single byte-addressable address space of 2XLEN bytes for all memory accesses. A word of memory is defined as 32 bits (4 bytes). Correspondingly, a halfword is 16 bits (2 bytes), a
  doubleword is 64 bits (8 bytes), and a quadword is 128 bits (16 bytes).
* The base RISC-V ISA has fixed-length 32-bit instructions that must be naturally aligned on 32-bit boundaries. However, the standard RISC-V encoding scheme is designed to support ISA extensions
  with variable-length instructions, where each instruction can be any number of 16-bit instruction parcels in length and parcels are naturally aligned on 16-bit boundaries.
* In the base RV32I ISA, there are four core instruction formats (R/I/S/U). All are a fixed 32 bits in length and must be aligned on a four-byte boundary in memory.

For more details on RISC-V ISA you may visit [here](https://github.com/riscv/riscv-isa-manual/releases/download/draft-20200727-8088ba4/riscv-spec.pdf).

# GNU Compiler Toolchain

The GNU toolchain is a broad collection of programming tools produced by the GNU Project. These tools form a toolchain (a suite of tools used in a serial manner) 
used for developing software applications and operating systems. Toolchain is simply a set of tools used to compile a piece of code to produce an executable program (TL Verilog in our case). 
Similar to other ISAs, RISC-V also has its own toolchain. 

  * Preprocessor - Processes source code before compilation. 
  * Compiler - Takes the input provided by preprocessor and converts to assembly code.
  * Assembler - Takes the input provided by compiler and converts to relocatable machine code.
  * Linker - Takes the input provided by Assembler and converts to Absolute machine code.

Mentioned below are steps to use RISC-V toolchain,

  * Command to use the risc-v gcc compiler:

    `riscv64-unknown-elf-gcc -Ofast -mabi=lp64 -march=rv64i -o <object filename> <C filename>`

    Different available options for the command:

    `riscv64-unknown-elf-gcc <compiler option -O1 ; Ofast> <ABI specifier -lp64; -lp32; -ilp32> <architecture specifier -RV64 ; RV32> -o <object filename> <C      filename>`

    For more details you may visit [here](https://www.sifive.com/blog/all-aboard-part-1-compiler-args)

  * Command to view the assembly code:
    
    `riscv64-unknown-elf-objdump -d <object filename>`
    
  * Command to use SPIKE simualtor to run risc-v obj file:
  
    `spike pk <object filename>`
    
    Command to use SPIKE as debugger:
    
    `spike -d pk <object Filename>` with degub command as `until pc 0 <pc of your choice>`

    You can refer the below links to install the complete risc-v toolchain locally on linux machine,
      1. [RISC-V GNU Toolchain](http://hdlexpress.com/RisKy1/How2/toolchain/toolchain.html)
      2. [RISC-V ISA SImulator - Spike](https://github.com/kunalg123/riscv_workshop_collaterals)
    
    Once done with the installation as mentioned in the above links, add the PATH to .bashrc file for further use.

Test Case for the above commands (Summation of 1 to 5):

![3 sum1ton file generation](https://user-images.githubusercontent.com/83152452/170860019-e2ebe0db-cabd-4a7b-b153-b8e61621ab27.png)

  * Below image shows the disassembled file `sum1ton.o` with `main` function highlighted.

    ![10  calculations - 2 - 11 instructions](https://user-images.githubusercontent.com/83152452/170859984-9605839c-9306-486d-82f8-fa79ae766829.png)
    
  * To view the registers we can use command as `reg <core> <register name>`. 

    Below image shows how to debug the disassembled file using Spike simulator where a1,a2,sp registers are checked and verified before and after the instructions got executed.

   ![19  execution of commands and verification in calci](https://user-images.githubusercontent.com/83152452/170859997-5893dea9-2d68-46ac-a264-21c421f32cd9.png)

* Lab for Unsigned number checking:

![7  lab for unsigned checking](https://user-images.githubusercontent.com/83152452/170860316-e2dc6bc0-5980-47bf-b059-5b98311fff58.png)


# Introduction to ABI

* Application Binary Interface (ABI) defines a system interface for compiled application programs. It establishes a standard binary interface for application 
programs on systems that implement the interfaces.
* The Application Binary Interface is the sum total of what the application programmer needs to understand in order to write programs.
* The programmer does not have to understand or know what is going on within the Application Execution Environment.
* An Application Binary Interface would combine the processor ISA along with the OS system-call interface. 
* The below table gives the list of registers, their short description and ABI name of every register in RISC-V ISA:

![ABI interface](https://user-images.githubusercontent.com/83152452/170860880-d6bf41f9-de2f-492d-a428-0257fad4f891.png)

Test Case for ABI Call (Summation of 1 to 9):

  ![2  execution of code](https://user-images.githubusercontent.com/83152452/170861552-8af8e221-0218-4710-912e-713349cd6d77.png)


# Digital Logic with TL-Verilog and Makerchip

* [Makerchip](https://makerchip.com/) is a free online environment for developing high-quality integrated circuits. You can code, compile, simulate, and debug Verilog 
designs, all from your browser. 
* Your code, block diagrams, and waveforms are tightly integrated. Makerchip introduces ground-breaking capabilities for advanced Verilog
design, it also makes circuit design easy and fun! Tutorials [here](https://makerchip.com/sandbox/) will get you going.
* Following are some unique features of TL-Verilog:
- Easy Pipelining
- Organized Waveforms
- Organized Diagrams
- Linked Design and Debug
- Tool Integrations

* All the examples shown below are done on Makerchip IDE using TL-verilog.


## Combinational Logic
Below are snapshots of some of the example codes using Combinational Logic:

### Inverter
* Starting with basic example in combinational logic is an inverter. To write the logic of inverter using TL-verilog is $out = ! $in;. 
* There is no need to declare $out and $in unlike Verilog. There is also no need to assign $in. 
* Random stimulus is provided, and a warning is produced.

![8  Inverter exp and results](https://user-images.githubusercontent.com/83152452/170862141-19bf5049-884d-426c-994c-645300611177.png)

### Binary Addition

![9  Binary addition - vector example](https://user-images.githubusercontent.com/83152452/170862158-e861ac82-02c6-460c-8af4-c3a4d534eeac.png)

### Multiplexer

![10  MUX example](https://user-images.githubusercontent.com/83152452/170862165-3eb7eb3a-bf34-4f37-b98f-dd023feee754.png)

### Combinational Calculator
* A simple implementation of a single stage basic calculator is done in TL-Verilog. 
* The calculator will have two 32-bit input data and one 3-bit opcode. 
* Depending upon the opcode value, calculator operation is selected.

![11  Combinational calculator](https://user-images.githubusercontent.com/83152452/170862169-4916e34c-8ace-4246-ae64-6c3d85a47c86.png)


## Sequential Logic
* Basic example in Sequential Logic is Fibonacci Series with reset. 

### Fibonacci Sequence
* To write the logic of Series using TL-Verilog is `$num[31:0] = $reset ? 1 : (>>1$num + >>2$num)`. 
* This operator `>>?` is ahead of operator which will provide the value of that signal 1 cycle before and 2 cycle before respectively.

![1  Fibonacci sequence](https://user-images.githubusercontent.com/83152452/170862297-261663fb-cee5-467d-9869-9620e2e901f1.png)

### Counter

![2  counter](https://user-images.githubusercontent.com/83152452/170862300-deed9c47-9036-4dee-a682-efb43fef3774.png)

### Sequential Calculator 
It remembers the last result, and uses it for the next calculation.

![3  Sequential calculator](https://user-images.githubusercontent.com/83152452/170862303-6a283515-4061-41f4-9c32-8705044c322a.png)


## Pipelined Logic

### Snapshot representing Pipelined Logic
* Timing abstract powerful feature of TL-Verilog which converts a code into pipeline stages easily. 
* Whole code under `|pipe` scope with stages defined as `@?`

![3  Pipelined Logic](https://user-images.githubusercontent.com/83152452/170862508-2565333c-a6ee-40c8-ad2c-fc7d61acbbfd.png)

### 2-cycle calculator 
* It clears the output alternatively and output of given inputs are observed at the next cycle.

![4  Cycle calculator lab](https://user-images.githubusercontent.com/83152452/170862518-c30c1484-b873-4367-b66d-47084488c8b9.png)


## Validity
* TL-Verilog supports a very unique feature called validity. Using validity, we can define tha condition when a specific signal will hold a valid content. 
* The validity condition is written using ?$valid_variable_name.
* Validity in TL-verilog means signal indicates validity of transaction and described as "when" scope else it will work as don't care. 
* Denoted as `?$valid`. Validity provides easier debug, cleaner design, better error checking, automated clock gating.
* Some example codes snapshots on Validity are shown below:

### Valid When Condition

![1  Lab on validity and valid when condition](https://user-images.githubusercontent.com/83152452/170863042-c9533bd0-af7c-4db2-81e2-ebd0fb04b640.png)

### To Compute Total Distance

![2  Lab to compute total distance](https://user-images.githubusercontent.com/83152452/170863045-cdfaf1fa-e399-468f-babb-f704ff61b1fc.png)

### 2-cycle calculator with validity 
* The calculator operation will only be carried out when there is no reset and it is a valid cycle.

![3  Lab on 2-cycle calculator with validity](https://user-images.githubusercontent.com/83152452/170863050-8a9d1b6b-9d51-4ac7-8ec9-6469bb568886.png)


# Basic RISC-V CPU Micro-architecture
* This section will cover the implementation of a simple 3-stage RISC-V Core / CPU. 
* The 3-stages broadly are: Fetch, Decode and Execute. 
* The figure shown below is the basic block diagram of the CPU core:

![1  Example RISC-V block diag-1](https://user-images.githubusercontent.com/83152452/170863619-537dd216-f0bb-4524-b28c-76b8bd77cbb9.png)


## Instruction Fetch Logic
* Program Counter, also called as Instruction Pointer is a block which contains the address of the next instruction to be executed. 
* It is feed to the instruction memory, which in turn gives out the instruction to be executed. 
* The program counter is incremented by 4, every valid iteration. 
* The output of the program counter is used for fetching an instruction from the instruction memory. 
* The instruction memory gives out a 32-bit instruction depending upon the input address. 
* During Fetch Stage, processor fetches the instruction from the IM pointed by address given by PC.
* The below snippet shows the Instruction Fetch Implementation in Makerchip:

![3  Fetch lab](https://user-images.githubusercontent.com/83152452/170863743-6bd9c830-b98b-4911-bad7-256fd6f973b5.png)


## Instruction Decode Logic
* The 32-bit fetched instruction has to be decoded first to determine the operation to be performed and the source / destination address.
* There are 6 types of Instructions namely,
  - R-type - Register 
  - I-type - Immediate
  - S-type - Store
  - B-type - Branch (Conditional Jump)
  - U-type - Upper Immediate
  - J-type - Jump (Unconditional Jump)
* Instruction Format includes Opcode, immediate value, source address, destination address. 
* During Decode Stage, processor decodes the instruction based on instruction format and type of instruction.
* Generally, RISC-V ISA provides 32 Register each of width = XLEN (for example, XLEN = 32 for RV32).
* Here, the register file used allows 2 - reads and 1 - write simultaneously.
* Below is snapshot from Makerchip IDE after performing the Decode Stage:

![4  decode lab](https://user-images.githubusercontent.com/83152452/170863752-3e8eb1d5-d3ef-41ed-80de-0c41a590ffd5.png)


## Register File Read and Write

If the Register file is 2 read, 1 write means, 2 read and 1 write operation can happen simultanously.

Inputs:
  * Read_Enable   - Enable signal to perform read operation
  * Read_Address1 - Address1 from where data has to be read 
  * Read_Address2 - Address2 from where data has to be read 
  * Write_Enable  - Enable signal to perform write operation
  * Write_Address - Address where data has to be written
  * Write_Data    - Data to be written at Write_Address

Outputs:
  * Read_Data1    - Data from Read_Address1
  * Read_Data2    - Data from Read_Address2

Below is snapshot from Makerchip IDE after performing the Register File Read followed by Register File Write.

![5  Register file read](https://user-images.githubusercontent.com/83152452/170863766-0f106a7f-f086-4fa5-be3b-c9a93a0b0da6.png)

![6  Register file write](https://user-images.githubusercontent.com/83152452/170863772-49072d3f-5f76-4aba-9d22-9842cc7c35db.png)



## Execute ALU Operations
* Depending upon the decoded operation, the instruction is executed. Arithmetic and Logical Unit (ALU) used if required. 
* If the instruction is a branching instruction the target branch address is computed separately, the instruction is executed.
* The result is stored back to the Register File, depending upon the destination register index.
* During the Execute Stage, both the operands perform the operation based on Opcode.
* Below is snapshot from Makerchip IDE after performing the Execute Stage:

![7  ALU lab - Execute](https://user-images.githubusercontent.com/83152452/170863777-8432bef2-c682-4907-8c4d-7cfbed210258.png)


## Control Branch Instruction Logic
* During Decode Stage, branch target address is calculated and fed into PC mux. 
* Before Execute Stage, once the operands are ready branch condition is checked.
* Below is snapshot from Makerchip IDE after including the control logic for branch instructions.

![8  Branches - Control logic](https://user-images.githubusercontent.com/83152452/170863781-55eef78d-143b-4bb2-81d9-b561b17cfdda.png)


# Pipelined RISC-V CPU Micro-architrcture
* Pipelining processes increases the overall performance of the system. Thus, the previously designed cores can be pipelined. 
* The "Timing Abstraction" feature of TL-Verilog makes it easy in converting non-piepleined CPU to pipelined CPU using timing abstract feature of TL-Verilog. 
* This allows easy retiming without any risk of funcational bugs. 
* More details reagrding Timing Abstract in TL-Verilog can be found in the IEEE Paper ["Timing-Abstract Circuit Design in Transaction-Level Verilog" by Steven Hoover.](https://ieeexplore.ieee.org/document/8119264)

## Pipelining the CPU
* Pipelining the CPU with branches involves 3 cycle delay and the other instructions are pipelined which can be done in the following way:
```
|<pipe-name>
    @<pipe stage>
       Instructions present in this stage
       
    @<pipe_stage>
       Instructions present in this stage
       
```

* Below is snapshot of pipelined CPU with a test case of assembly program which does summation from 1 to 9 then stores to r10 of register file. In snapshot `r10 = 45`. Test case:
```
*passed = |cpu/xreg[10]>>5$value == (1+2+3+4+5+6+7+8+9);

```

![1  Pipelining the CPU - 1](https://user-images.githubusercontent.com/83152452/170864516-2eb8c329-60ee-4bbf-84b2-35b3c004d54b.png)

![1  Pipelining the CPU - 2](https://user-images.githubusercontent.com/83152452/170864522-7c6e7501-f66c-4e31-bbf6-9ada76e06cf2.png)


* There are various hazards to be taken into consideration while implementing a pipelined design. Some of them are:
- Undefined and improper updating of Program Counter (PC).
- Read-before-Write Hazard issues.


## Load and Store Instructions and Memory
* A Data memory can be added to the Core. The Load-Store operations will add up a new stage to the core. 
* Thus, making it now a 4-Stage Core / CPU.
* Similar to branch, load will also have 3 cycle delay. So, added a Data Memory 1 write/read memory.

Inputs:
  * Read_Enable - Enable signal to perform read operation
  * Write_Enable - Enable signal to perform write operation
  * Address - Address specified whether to read/write from
  * Write_Data - Data to be written on Address (Store Instruction)

Output: 
  * Read_Data - Data to be read from Address (Load Instruction)

* Added test case to check fucntionality of load/store. Stored the summation of 1 to 9 on address 4 of Data Memory and loaded that value from Data Memory to r17.
```
*passed = |cpu/xreg[17]>>5$value == (1+2+3+4+5+6+7+8+9);
```
* Below is snapshot from Makerchip IDE after including load/store instructions:

![2  Load and store instructions   memory - 1](https://user-images.githubusercontent.com/83152452/170864641-0d4bbb26-8581-4400-98b2-fa276b112c83.png)

![2  Load and store instructions   memory - 2](https://user-images.githubusercontent.com/83152452/170864651-e2e30993-a3db-4a23-abb2-5244e8b9ac3b.png)


## Completing the RISC-V Core
* After pipelining is proved in simulations, the operations for Jump Instructions are added. 
* Instruction Decode and ALU Implementation for RV32I Base Integer Instruction Set are also added.
* Below is final Snapshot of Complete Pipelined RISC-V CPU:

![3  Completing RISC-V CPU - 1](https://user-images.githubusercontent.com/83152452/170864665-4ed1dc25-4df1-436d-b19e-7e09b93ce875.png)

![3  Completing RISC-V CPU - 2](https://user-images.githubusercontent.com/83152452/170864669-aefa443e-9d80-4cfd-a923-d26ad801441f.png)

![3  Completing RISC-V CPU - 3](https://user-images.githubusercontent.com/83152452/170864676-340c5982-a492-4bcc-b5f1-08d142b9a990.png)


# Acknowledgements
- [Kunal Ghosh](https://github.com/kunalg123), Co-founder, VSD Corp. Pvt. Ltd.
- [Steve Hoover](https://github.com/stevehoover), Founder, Redwood EDA
- [Shivani Shah](https://github.com/shivanishah269), TA
- [Shivam Potdar](https://github.com/shivampotdar), TA
- [Vineet Jain](https://github.com/vineetjain07), TA


## Author
- [A Devipriya](https://github.com/Devipriya1921), B.E (Electronics and Communication Engineering), Bangalore - adevipriya1900@gmail.com
