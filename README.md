# 3o|||sheet

A comprehensive development suite for industrial controllers (virtual or physical).

It consists of three independent parts:
1.  **Development Environment (IDE)**
2.  **Compiler**
3.  **Runtime Environment**

---

## 3o|||sheet IDE

A lightweight, cross-platform development environment that can even run on single-board computers.

*   **Purpose:** To translate programming languages (any) into a set of text instructions for the 3o|||sheet compiler.
*   **RTOS:** Installs a proprietary virtual RTOS if the user parallelizes the program into independent tasks.
*   **Supported Languages:** Currently supports LD, FBD, and a text editor for languages like ST.
*   **Features:**
    *   Debugging
    *   Online monitoring of values within blocks
    *   **Live code updates** for individual tasks without physically stopping the PLC or halting other tasks and production processes.

## 3o|||sheet Compiler

A proprietary, self-developed, independent, and cross-platform application. The most intelligent part of the system.

*   **Flexibility:** Programs can be written without the IDE using any text editor (e.g., Visual Studio Code).
*   **Native Code Style:** A hybrid of assembly syntax (instructions) and C language (data), designed for simple translation of other high-level languages to the platform.
*   **Universal Target:** Any development environment (LD, FBD, ST) must translate into our `.3osheet` code.
*   **Optimized for Weak Hardware:**
    *   Virtual variables have the same physical size as in C or Assembly (no redundancy).
    *   Unlike Lua, Java, or MicroPython (where a `bool` can take >20 bytes), everything is optimized for extremely limited hardware.
    *   All responsibility for typing and array handling lies with the compiler.

## 3o|||sheet Runtime

A register-based virtual machine running on the hardware, written in C.

*   **Portability:** Does not use low-level architecture-specific code. Compiles quickly for ARM, RISC-V, x86. Peripheral configuration can be initialized from the virtual level.
*   **Minimal Requirements:**
    *   **4 KB RAM**
    *   **64 KB Flash** (full version)
    *   **32 KB Flash** (limited version)
    *   *Note: These are for the VM only. User programs require additional memory.*
*   **Memory Footprint (Approx.):**
    *   LD: 8-12 bytes per basic instruction.
    *   Timers/Counters: 60-100 bytes each.
    *   Custom LD/FBD atomic instructions (ADD, SUB, jumps, etc.): 4 bytes each.
*   **Capabilities:** A full-fledged machine with instructions for virtual stack/task context, supporting deep function calls, recursion, and multithreading/multi-core execution.
*   **Designed for IACS:** Developed specifically for Industrial Control Systems and strict task execution timeframes.

---

## Advantages over Global Brands

### First Virtualization for Critically Low Resources
This is the first virtualization system for hardware with critically low resources. While global brands have PLCs with preemptive multitasking, they don't run on hardware with just **8KB RAM**. Ours is the first feasible PLC that can operate in several modes simultaneously.

### Fault Isolation by Criticality
Based on a microcontroller with **16 KB RAM**, you can launch multiple runtime instances, partitioned by process criticality. **A critical error in one instance will not stop the PLC; others continue working.**

**Imagine:** You can create isolated executors divided by criticality:

*   **VM0:** General tasks + RTOS (HMI, non-critical communication, risky code). A crash here only affects this instance.
*   **VM1, VM2:** Critical code, safety (simple processes, no stack/deep calls). Executed by separate, independent instances.

At the physical CPU level, each instance has its own memory array. Whatever happens in one instance (even total data deletion) is contained and does not affect the physical CPU or other instances.

### Pioneering Safety Mechanisms (SIL3/SIL4 Target)
Leveraging this architecture to develop mechanisms for a potential SIL3/SIL4 safety standard, providing features not found in global brands:

*   **Multiple Executors & Safe Mode:**
    1.  **Executor 1:** Runs code without access to physical processes.
    2.  **Executor 2:** Verifies the first didn't crash and checks execution hashes for "silent errors" (unnotected data corruption).
    3.  **Executor 3:** A copy of the first, but *with* physical access, executes only after code and data are verified safe.
*   **Redundancy:**
    *   Sensor duplication for redundancy against physical failure.
    *   Executor duplication for redundancy against software failure.
*   **Strict Isolation:** A failure in VM1 (e.g., from a faulty sensor) does not affect VM2. In a classic PLC, this could crash the entire controller.
*   **Arbiter:** Compares data, restarts crashed executors, etc.

Such mechanisms were previously only available in Linux-based systems, which are not real-time and are unsuitable for critical safety. Our system brings them to microcontrollers with **8-16 KB RAM**.

---

## Performance & Philosophy

*   **Virtualization performance** is 2-4 times slower than native code.
*   Creating native platforms (e.g., translating LD/ST to C) is an easier path, as taken by projects like Beremiz.
*   However, **virtualization provides extensive capabilities** for safety, control, and live updates without stopping production, while still meeting strict timing deadlines.
