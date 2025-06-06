# RISC-V Week 1: Bare-Metal Programming & Toolchain Setup

This repository provides a comprehensive walkthrough for setting up and experimenting with a RISC-V RV32IMAC toolchain. It includes steps from toolchain installation to debugging, simulation, and basic bare-metal programming.

---

## 🔧 Toolchain Setup

1. **Unpack the Toolchain**
   ```bash
   tar -xzf riscv-toolchain-rv32imac-x86_64-ubuntu.tar.gz
   ```

2. **Update PATH**
   ```bash
   nano ~/.bashrc
   # Add:
   export PATH=/home/arpita_yaligeti/riscv/bin:$PATH
   source ~/.bashrc
   ```

3. **Verify Installation**
   ```bash
   riscv32-unknown-elf-gcc --version
   riscv32-unknown-elf-objdump --version
   riscv32-unknown-elf-gdb --version
   ```

---

## 📝 Hello World in C

Create `hello.c`:
```c
#include <stdio.h>
int main() {
    printf("Hello, RISC-V!\n");
    return 0;
}
```

Compile:
```bash
riscv32-unknown-elf-gcc -o hello.elf hello.c
file hello.elf
```

Generate assembly:
```bash
riscv32-unknown-elf-gcc -S -O0 hello.c
```

---

## 🔍 ELF to HEX & Disassembly

Disassemble:
```bash
riscv32-unknown-elf-objdump -d hello.elf > hello.dump
```

Convert to Intel HEX:
```bash
riscv32-unknown-elf-objcopy -O ihex hello.elf hello.hex
```

---

## 📜 Registers and ABI

RV32 Integer Registers:
- `x0–x31` with ABI names like `zero, ra, sp, gp, tp, t0–t6, s0–s11, a0–a7`

---

## 🐞 Debugging with GDB

Run:
```bash
riscv32-unknown-elf-gdb hello.elf
```

GDB commands:
```gdb
target sim
load
break main
run
```

---

## 🚀 Running Bare-Metal ELF on QEMU

Run:
```bash
qemu-system-riscv32 -nographic -machine virt -bios none -kernel hello2.elf
```

---

## 🔄 Optimization Comparison

```bash
riscv32-unknown-elf-gcc -S -O0 hello.c -o hello_O0.s
riscv32-unknown-elf-gcc -S -O2 hello.c -o hello_O2.s
```

---

## 🕒 Inline Assembly: Cycle Counter

```c
static inline uint32_t rdcycle(void) {
    uint32_t cycles;
    __asm__ volatile ("rdcycle %0" : "=r"(cycles));
    return cycles;
}
```

---

## 🧱 Bare-Metal GPIO Access

```c
volatile uint32_t *gpio = (uint32_t *)0x10012000;
*gpio = 0x1;
```

---

## 📍 Linker Script Example

```ld
SECTIONS {
  .text 0x00000000 : { *(.text*) }
  .data 0x10000000 : { *(.data*) }
}
```

---

## ⚙️ Interrupts: MTIP Example

Enable machine-timer interrupt using:
- `mie`, `mstatus`, `mtvec`
- `MTIMECMP` and `MTIME` registers

---

## 🔒 Atomic Instructions

Use `lr.w` / `sc.w` for atomic access.

---

## 📡 Retarget `printf()` to UART

```c
int _write(int file, const char *ptr, int len) {
    while (!(*uart_status_reg & UART_TX_READY_BIT));
    *uart_tx_reg = ptr[i];
}
```

---

## 🔁 Endianness Check

```c
union {
    uint32_t value;
    uint8_t bytes[4];
} test = {0x01020304};
```

---

## 📂 Files Overview

- `hello.c`, `hello2.c`: Sample programs
- `startup.s`: Assembly startup
- `linker.ld`, `rdcycle.ld`: Linker scripts
- `trap.S`: Interrupt entry
- `crt0.S`: Runtime initialization

---

## 🧠 Summary

This project demonstrates:
- Toolchain installation
- ELF/HEX handling
- GDB and QEMU usage
- Assembly inspection
- Interrupts & atomic ops
- Inline CSR access
- Minimal embedded C development
