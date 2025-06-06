
# RISC-V Week 1: Bare-Metal Toolchain, Programming, and Debugging

## 1. 🛠️ Install & Sanity-Check the Toolchain

```bash
tar -xzf riscv-toolchain-rv32imac-x86_64-ubuntu.tar.gz

# Add to PATH
nano ~/.bashrc
# Add this line:
export PATH=/home/arpita_yaligeti/riscv/bin:$PATH
source ~/.bashrc

# Sanity check
riscv32-unknown-elf-gcc --version
riscv32-unknown-elf-objdump --version
riscv32-unknown-elf-gdb --version
```

---

## 2. 🧾 Minimal C Hello World

```c
#include <stdio.h>
int main() {
    printf("Hello, RISC-V!\n");
    return 0;
}
```

```bash
riscv32-unknown-elf-gcc -o hello.elf hello.c
file hello.elf
```

---

## 3. 🔍 From C to Assembly

```bash
riscv32-unknown-elf-gcc -S -O0 hello.c
```

Prologue: Setup stack, save ra/s0.  
Epilogue: Restore stack, use `ret` to return.

---

## 4. 🔄 ELF to HEX + Disassembly

```bash
riscv32-unknown-elf-objdump -d hello.elf > hello.dump
riscv32-unknown-elf-objcopy -O ihex hello.elf hello.hex
```

Disassembly format:
```
Address: Opcode Mnemonic Operands
10074:   1141   addi sp, sp, -16
```

---

## 5. 📋 RV32 Register Table

See the full table of x0–x31, ABI names like ra, sp, a0–a7, s0–s11, t0–t6.

---

## 6. 🐞 GDB: Breakpoint & Register Inspect

```bash
riscv32-unknown-elf-gdb hello.elf

(gdb) target sim
(gdb) load
(gdb) break main
(gdb) run
```

Use a minimal C program with no `printf`.

---

## 7. 🖥️ UART Output with QEMU

Write a UART program using:
```c
#define UART0 0x10000000
void uart_putc(char c) { *(volatile char *)UART0 = c; }
void uart_puts(const char *s) { while (*s) uart_putc(*s++); }
```

```bash
riscv32-unknown-elf-gcc -nostdlib -T linker.ld -o hello2.elf hello2.c startup.s
qemu-system-riscv32 -nographic -machine virt -bios none -kernel hello2.elf
```

---

## 8. ⚙️ Optimization: -O0 vs -O2

```bash
riscv32-unknown-elf-gcc -S -O0 hello.c -o hello_O0.s
riscv32-unknown-elf-gcc -S -O2 hello.c -o hello_O2.s
```

- `-O0`: debug friendly
- `-O2`: inlines, dead-code removed

---

## 9. ⏱️ Read CSR Cycle Counter

```c
static inline uint32_t rdcycle(void) {
    uint32_t cycles;
    __asm__ volatile ("rdcycle %0" : "=r"(cycles));
    return cycles;
}
```

Compile with linker/startup and run on QEMU.

---

## 10. 🔁 Toggle GPIO (0x10012000)

```c
volatile uint32_t *gpio = (uint32_t *)0x10012000;
*gpio = 0x1;
```

Use `volatile` to prevent optimization.

---

## 11. 📜 Linker Script (Minimal)

```ld
SECTIONS {
    .text 0x00000000 : { *(.text*) }
    .data 0x10000000 : { *(.data*) }
}
```

---

## 12. 🧬 crt0.S Overview

Responsibilities:
- Set up stack
- Zero .bss
- Call `main()`
- Loop forever

Example inside `.S` file and `.ld` file to define sections.

---

## 13. ⏲️ Machine-Timer Interrupt (MTIP)

- Set `mtimecmp`, `mie`, `mstatus`, `mtvec`
- Write handler in C:
```c
void __attribute__((interrupt)) mtimer_handler(void) {
    *gpio ^= 0x1;
    *MTIMECMP = *MTIME + 1000000;
}
```

---

## 14. 🔒 RV32 ‘A’ Extension (Atomic)

Instructions like `lr.w`, `sc.w`, `amoadd.w`, `amoswap.w`, etc. used for safe concurrent memory access.

---

## 15. 🧵 Two-Thread Mutex with `lr.w` / `sc.w`

```c
void lock_mutex(volatile int *lock) {
    int tmp;
    do {
        asm volatile (
            "lr.w %[val], (%[addr])\n"
            "bnez %[val], 1f\n"
            "li %[val], 1\n"
            "sc.w %[val], %[val], (%[addr])\n"
            "1:" : [val] "=&r"(tmp) : [addr] "r"(lock) : "memory"
        );
    } while (tmp != 0);
}
```

---

## 16. 🖨️ Retarget `printf` using `_write()`

```c
int _write(int file, const char *ptr, int len) {
    for (int i = 0; i < len; i++) {
        while (!(*uart_status_reg & UART_TX_READY_BIT));
        *uart_tx_reg = ptr[i];
    }
    return len;
}
```

Compile with:
```bash
riscv32-unknown-elf-gcc -nostartfiles -T linker.ld crt0.S main.c syscalls.c -o prog.elf -lc -lgcc
```

---

## 17. 🧠 Verify Endianness

```c
union {
    uint32_t value;
    uint8_t bytes[4];
} test;

test.value = 0x01020304;

printf("Byte order: %02x %02x %02x %02x\n",
       test.bytes[0], test.bytes[1], test.bytes[2], test.bytes[3]);
```

Compile:
```bash
riscv32-unknown-elf-gcc -march=rv32imac -mabi=ilp32 endian.c -o endian.elf
qemu-riscv32 ./endian.elf
```

---

Let me know if you want this as a GitHub repo with source files.
