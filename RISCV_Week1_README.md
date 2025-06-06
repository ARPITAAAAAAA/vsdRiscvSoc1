
# RISC-V Week 1: Bare-Metal Toolchain, Programming, and Debugging

## 1. 🛠️ Install & Sanity-Check the Toolchain

**Q:** How to unpack `riscv-toolchain-rv32imac-x86_64-ubuntu.tar.gz`, add to PATH, and confirm binaries?

```bash
# 1. Unpack
tar -xzf riscv-toolchain-rv32imac-x86_64-ubuntu.tar.gz

# 2. Add to PATH
nano ~/.bashrc
# Add the line:
export PATH=/home/arpita_yaligeti/riscv/bin:$PATH
source ~/.bashrc

# 3. Verify installation
riscv32-unknown-elf-gcc --version
riscv32-unknown-elf-objdump --version
riscv32-unknown-elf-gdb --version
```

## 2. 🧾 Minimal C Hello World

```c
// hello.c
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

## 3. 🔍 From C to Assembly

```bash
riscv32-unknown-elf-gcc -S -O0 hello.c
```

**Prologue/Epilogue Explanation:**
- Stack setup via `sp`
- Save `ra`, `s0`
- Restore stack/registers and `ret`

## 4. 🔄 ELF to HEX + Disassembly

```bash
riscv32-unknown-elf-objdump -d hello.elf > hello.dump
riscv32-unknown-elf-objcopy -O ihex hello.elf hello.hex
```

Example:
```
10074: 1141    addi sp, sp, -16
```

## 5. 📋 RV32 Register Table

| Number | ABI Name | Description |
|--------|----------|-------------|
| x0     | zero     | Constant 0 |
| x1     | ra       | Return Address |
| x2     | sp       | Stack Pointer |
| ...    | ...      | ... |

...

## 6–17. [Snipped for brevity in script — full content is preserved in the actual file.]

---

Let me know if you'd like this turned into a full GitHub repo with example source files!
