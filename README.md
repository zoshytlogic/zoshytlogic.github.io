**3o|||sheet**: A comprehensive development suite for industrial controllers (virtual or physical). It consists of three independent parts: 1) Development Environment, 2) Compiler, 3) Runtime Environment on hardware.

**3o|||sheet IDE**: A lightweight, cross-platform development environment that can even run on single-board computers. Its purpose is to translate programming languages (any) into a set of text instructions for the 3o|||sheet compiler. 
It also installs a virtual RTOS (proprietary) if the user parallelizes the program into independent tasks. Currently supports LD, FBD, and a text editor for languages like ST. 
Features: debugging, online monitoring of values within blocks, and live code updates for individual tasks without physically stopping the PLC or halting other tasks and production processes.

**3o|||sheet Compiler**: A proprietary, self-developed compiler: an independent, cross-platform application. The most intelligent part. You can write programs without the 3o|||sheet IDE, using any text editor like Visual Studio Code. 
This development competes with major brands in Industrial Automation and Control Systems (IACS), aiming to surpass Codesys. The compiler's "native" code has its own unique style, a 
hybrid of assembly syntax (instructions) and C language (data), designed for the simplest possible translation of other high-level languages to the platform. Regardless of the development environment (LD, FBD, ST, proprietary or third-party),
it must translate into our `.3osheet` code. Although the system's data/variables are virtual, their physical size is the same as in C or Assembly. 
There is no redundancy for variables. Unlike Lua, Java, or MicroPython, where a `bool` variable can take over 20 bytes, here everything is optimized for extremely limited hardware. All responsibility for typing and array handling lies with the compiler.

**3o|||sheet Runtime**: A register-based virtual machine running on the hardware. Written in C. It does not use low-level code for specific architectures and compiles quickly for any hardware (ARM, RISC-V, x86); 
the difference lies only in peripheral configuration, which can also be initialized from the virtual level. 

Minimum system requirements:
4 KB RAM, 64 KB Flash (full version), 
32 KB Flash (limited version). 

These requirements are for the virtual machine only. User programs require additional program memory (to RAM and Flash), with an approximate calculation of 8-12 bytes per basic instruction for LD language, and 60-100 bytes per timer/counter. For custom LD/FBD, each atomic instruction within it (ADD, SUB, various increments, branch jumps) is 4 bytes. 3o|||sheet Runtime is a full-fledged machine. It contains all necessary instruction types, including those for working with the virtual stack/task context. This allows execution of any programs (deep function calls, recursion, multithreading/multi-core). It was specifically developed for IACS and strict task execution timeframes.

**Advantages over Global Brands**
This is the first virtualization system for hardware with critically low resources. Global manufacturers have PLCs with preemptive multitasking (but not on hardware with 8KB RAM). Ours is the 
first feasible PLC that can operate in several modes simultaneously (unlike brands which typically offer only one thing). It is fundamentally new. Based on a microcontroller with 16 KB RAM, 
multiple runtime instances can be launched, partitioned by process criticality. Partitioning the program on the production floor is based on criticality. A critical error in code or process will not stop the PLC; other runtime instances will continue working.

Imagine you are an IACS engineer who least wants to stop a process due to downtime. You can create several isolated runtime instances, partitioned by criticality:
*   **VM0**: Common tasks + RTOS. Although a frozen task won't affect others as the peripheral timer will switch context, a critical code error could crash the system/machine.
*   A conventional PLC would crash entirely from a critical error. Here, only one runtime instance crashes. Here we can install HMI, non-critical communication, and other code with potential risks.
*   **VM1, VM2**: Critical code, safety (maximum simplicity, avoiding stack or deep calls) – executed by separate, independent runtime instances.

At the physical level, in the CPU, each runtime instance has its own memory area (its own byte array). Whatever happens there, only one will crash; the others continue working. 
The physical CPU is also completely safe; for it, each machine is just a byte array where data and program are not its own, and whatever happens – even complete deletion – is the problem of whoever uses that array, not the physical CPU's problem.

Using this architecture, a mechanism for creating PLCs for a potential SIL3/SIL4 safety standard is currently under development, providing mechanisms not found even in global brands:
*   Multiple program executors.
*   Code can be executed in a safe mode. For example, one executor without access to physical processes runs the code, a second one checks if the machine crashed after that,
*   and most importantly – checks the execution hash for "silent errors" which might go unnoticed by the compiler and even the runtime (no exception will be thrown, but data could be erroneous, leading to false positives and subsequent physical output).
*   A third executor (a full copy of the first, but with access to physical processes), after confirming the code and data are safe, executes and affects the actual physical production processes.

Furthermore, this approach allows creating fully identical processes within one PLC. For instance, connecting different duplicate physical sensors to them for redundancy in case one fails physically.
*   Sensor duplication for redundancy against physical failure or malfunction.
*   Executor duplication for redundancy against software failure.

**Strict Isolation**: A failure in VM1 (e.g., due to faulty data from Sensor 1) does not affect VM2 execution at all. In a classic PLC with shared memory, a faulty program could corrupt shared data and crash the entire controller.
*   **Arbiter**: Compares data, restarts crashed executors, etc.

Such mechanisms were previously only available in Linux-based systems, not on microcontrollers with 8-16 KB RAM in a PLC format. But Linux is not a real-time system and is generally a dark forest for the safety of critical systems.

**Virtualization performance** is 2-4 times slower compared to native code. Creating native platforms for PLCs is an easier path, as one's own LD/ST programs can be translated to C and compiled, for example, with a C compiler, without worrying about implementation details oneself (as done by open-source projects like Beremiz and various domestic PLC developers). However, virtualization provides extensive capabilities for safety and control, changing programs on the fly without stopping production processes, while fully complying with strict time intervals.
