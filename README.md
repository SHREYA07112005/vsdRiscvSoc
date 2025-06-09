# RISC-V Toolchain and Programming Tasks

This README provides detailed instructions, code snippets, command-line commands, and expected outputs for the 17 tasks outlined in the RISC-V programming document. Each task guides you through setting up the RISC-V toolchain, writing and compiling code for RV32IMC, and exploring advanced concepts like inline assembly, interrupts, and memory-mapped I/O.

## Prerequisites
- A Linux system (Ubuntu recommended).
- Basic knowledge of C programming and command-line operations.
- Optional: Spike or QEMU for emulation if no RISC-V hardware is available.

## Task 1: Install & Sanity-Check the Toolchain
**Objective**: Unpack the RISC-V toolchain, add it to PATH, and verify `gcc`, `objdump`, and `gdb`.

**Command-Line**:
```bash
# Unpack the toolchain
tar -xzf riscv-toolchain-rv32imac-x86_64-ubuntu.tar.gz -C $HOME

# Add to PATH in ~/.bashrc
echo 'export PATH=$HOME/riscv/bin:$PATH' >> ~/.bashrc
source ~/.bashrc

# Verify binaries
riscv32-unknown-elf-gcc --version
riscv32-unknown-elf-objdump --version
riscv32-unknown-elf-gdb --version
```

**Expected Output**:
```
riscv32-unknown-elf-gcc (GCC) 10.2.0
...
objdump (GNU Binutils) 2.35
...
GNU gdb (GDB) 9.2
...
```

## Task 2: Compile "Hello, RISC-V"
**Objective**: Write and cross-compile a minimal "Hello World" C program for RV32IMC.

**Code Snippet** (`hello.c`):
```c
#include <stdio.h>
int main() {
    printf("Hello, RISC-V!\n");
    return 0;
}
```

**Command-Line**:
```bash
riscv32-unknown-elf-gcc -march=rv32imc -mabi=ilp32 -o hello.elf hello.c
file hello.elf
```

**Expected Output**:
```
hello.elf: ELF 32-bit LSB executable, RISC-V, version 1 (SYSV), statically linked, not stripped
```

## Task 3: From C to Assembly
**Objective**: Generate the assembly file for `hello.c` and explain the prologue/epilogue.

**Command-Line**:
```bash
riscv32-unknown-elf-gcc -S -O0 -march=rv32imc -mabi=ilp32 hello.c -o hello.s
cat hello.s
```

**Expected Output** (partial `hello.s`):
```
main:
    addi    sp,sp,-16     # Prologue: Allocate stack space
    sw      ra,12(sp)     # Save return address
    sw      s0,8(sp)      # Save frame pointer
    ...
    lw      ra,12(sp)     # Epilogue: Restore return address
    lw      s0,8(sp)      # Restore frame pointer
    addi    sp,sp,16      # Deallocate stack space
    jr      ra            # Return
```
**Explanation**: The prologue (`addi sp,sp,-16`, `sw ra,12(sp)`) sets up the stack frame and saves the return address. The epilogue (`lw ra,12(sp)`, `addi sp,sp,16`, `jr ra`) restores the state and returns.

## Task 4: Hex Dump & Disassembly
**Objective**: Generate a hex file and disassemble the ELF, explaining the output fields.

**Command-Line**:
```bash
riscv32-unknown-elf-objdump -d hello.elf > hello.dump
riscv32-unknown-elf-objcopy -O ihex hello.elf hello.hex
cat hello.dump
```

**Expected Output** (partial `hello.dump`):
```
00010074 <main>:
   10074:   fe010113                addi    sp,sp,-16
   10078:   00112623                sw      ra,12(sp)
   1007c:   00812423                sw      s0,8(sp)
```
**Explanation**: Each line shows:
- **Address** (e.g., `10074`): Memory address of the instruction.
- **Opcode** (e.g., `fe010113`): Machine code in hex.
- **Mnemonic** (e.g., `addi`): Instruction name.
- **Operands** (e.g., `sp,sp,-16`): Instruction arguments.

**Hex File** (`hello.hex`, partial):
```
:1010074000FE101130011262300812423...
```

## Task 5: ABI & Register Cheat-Sheet
**Objective**: List RV32 registers and their ABI roles.

**Output**:
| Register | ABI Name | Role |
|----------|----------|------|
| x0       | zero     | Always zero |
| x1       | ra       | Return address (caller-saved) |
| x2       | sp       | Stack pointer |
| x3       | gp       | Global pointer |
| x4       | tp       | Thread pointer |
| x5-x7, x28-x31 | t0-t6 | Temporaries (caller-saved) |
| x8-x9, x18-x27 | s0-s11 | Saved registers (callee-saved) |
| x10-x17  | a0-a7    | Arguments/return values (caller-saved) |

**Calling Convention**:
- `a0-a7`: Pass arguments, return values.
- `s0-s11`: Callee must save/restore if used.
- `t0-t6`: Caller must save if needed.

## Task 6: Stepping with GDB
**Objective**: Debug `hello.elf` using GDB, setting a breakpoint at `main`.

**Command-Line**:
```bash
riscv32-unknown-elf-gdb hello.elf
(gdb) target sim
(gdb) break main
(gdb) run
(gdb) step
(gdb) info reg a0
(gdb) disassemble
(gdb) quit
```

**Expected Output**:
```
Breakpoint 1 at 0x10074: file hello.c, line 3.
Breakpoint 1, main () at hello.c:3
3           printf("Hello, RISC-V!\n");
a0             0x0      0
Dump of assembler code for function main:
   0x00010074 <+0>:     addi    sp,sp,-16
=> 0x00010078 <+4>:     sw      ra,12(sp)
...
```

## Task 7: Running Under an Emulator
**Objective**: Run `hello.elf` on Spike or QEMU.

**Command-Line**:
```bash
spike --isa=rv32imc pk hello.elf
# OR
qemu-system-riscv32 -nographic -kernel hello.elf
```

**Expected Output**:
```
Hello, RISC-V!
```

## Task 8: Exploring GCC Optimization
**Objective**: Compare assembly output with `-O0` vs `-O2`.

**Command-Line**:
```bash
riscv32-unknown-elf-gcc -S -O0 -march=rv32imc -mabi=ilp32 hello.c -o hello_O0.s
riscv32-unknown-elf-gcc -S -O2 -march=rv32imc -mabi=ilp32 hello.c -o hello_O2.s
diff hello_O0.s hello_O2.s
```

**Expected Output** (simplified diff):
- `-O0`: Verbose, with redundant stack operations.
- `-O2`: Optimized, with inlining, dead-code elimination, and fewer instructions.
**Explanation**: `-O2` removes unnecessary stores, uses registers efficiently, and may inline `printf`.

## Task 9: Inline Assembly Basics
**Objective**: Read the cycle counter using inline assembly.

**Code Snippet** (`rdcycle.c`):
```c
static inline uint32_t rdcycle(void) {
    uint32_t c;
    asm volatile ("csrr %0, cycle" : "=r"(c));
    return c;
}
int main() {
    printf("Cycle count: %u\n", rdcycle());
    return 0;
}
```

**Explanation**:
- `csrr %0, cycle`: Reads the `cycle` CSR into output operand `%0`.
- `"=r"(c)`: Output constraint, stores result in register `c`.
- `volatile`: Prevents compiler from optimizing away the instruction.

**Command-Line**:
```bash
riscv32-unknown-elf-gcc -march=rv32imc -mabi=ilp32 -o rdcycle.elf rdcycle.c
```

## Task 10: Memory-Mapped I/O Demo
**Objective**: Toggle a GPIO register at `0x10012000`.

**Code Snippet** (`gpio.c`):
```c
int main() {
    volatile uint32_t *gpio = (uint32_t *)0x10012000;
    *gpio = 0x1;
    return 0;
}
```

**Command-Line**:
```bash
riscv32-unknown-elf-gcc -march=rv32imc -mabi=ilp32 -o gpio.elf gpio.c
```

**Explanation**: `volatile` ensures the compiler does not optimize away the store to the memory-mapped address.

## Task 11: Linker Script 101
**Objective**: Create a linker script for specific section placement.

**Code Snippet** (`rv32imc.ld`):
```
SECTIONS {
    .text 0x00000000 : { *(.text*) }
    .data 0x10000000 : { *(.data*) }
}
```

**Command-Line**:
```bash
riscv32-unknown-elf-gcc -march=rv32imc -mabi=ilp32 -T rv32imc.ld -o hello.elf hello.c
```

**Explanation**: Places `.text` in Flash (0x0) and `.data` in SRAM (0x10000000).

## Task 12: Start-up Code & crt0
**Objective**: Understand `crt0.S` functionality.

**Code Snippet** (`crt0.S`, simplified):
```
    .section .text
    .global _start
_start:
    la sp, _stack_top    # Set stack pointer
    mv fp, zero          # Zero frame pointer
    la a0, _bss_start
    la a1, _bss_end
clear_bss:
    beq a0, a1, call_main
    sw zero, 0(a0)
    addi a0, a0, 4
    j clear_bss
call_main:
    call main
loop:
    j loop
```

**Explanation**: Initializes stack, zeroes `.bss`, calls `main`, and loops infinitely.

**Source**: Use `newlib` or device-specific `crt0.S` from RISC-V repositories.

## Task 13: Interrupt Primer
**Objective**: Enable and handle a machine-timer interrupt.

**Code Snippet** (`timer.c`):
```c
#include <stdint.h>
void __attribute__((interrupt)) timer_isr(void) {
    volatile uint32_t *mtimecmp = (uint32_t *)0x02004000;
    *mtimecmp += 1000; // Reset timer
}
int main() {
    volatile uint32_t *mtimecmp = (uint32_t *)0x02004000;
    volatile uint32_t *mie = (uint32_t *)0xd001001c;
    *mtimecmp = 1000; // Set compare value
    *mie |= 0x80;     // Enable MTIP
    asm volatile ("csrsi mstatus, 0x8"); // Enable interrupts
    while (1);
}
```

**Command-Line**:
```bash
riscv32-unknown-elf-gcc -march=rv32imc -mabi=ilp32 -o timer.elf timer.c
```

## Task 14: rv32imac vs rv32imc - What's the "A"?
**Objective**: Explain the atomic extension.

**Answer**:
- **Atomic Instructions**: `lr.w` (load-reserved), `sc.w` (store-conditional), `amoadd.w`, etc.
- **Use**: Enables lock-free data structures and thread synchronization in OS kernels.

## Task 15: Atomic Test Program
**Objective**: Implement a spin-lock using `lr.w`/`sc.w`.

**Code Snippet** (`spinlock.c`):
```c
#include <stdint.h>
void spin_lock(volatile uint32_t *lock) {
    uint32_t tmp;
    asm volatile (
        "1: lr.w %0, (%1)\n"
        "   bnez %0, 1b\n"
        "   li %0, 1\n"
        "   sc.w %0, (%1), %0\n"
        "   bnez %0, 1b\n"
        : "=&r"(tmp)
        : "r"(lock)
    );
}
int main() {
    volatile uint32_t lock = 0;
    spin_lock(&lock);
    return 0;
}
```

**Command-Line**:
```bash
riscv32-unknown-elf-gcc -march=rv32imac -mabi=ilp32 -o spinlock.elf spinlock.c
```

## Task 16: Using Newlib printf Without an OS
**Objective**: Retarget `_write` for UART output.

**Code Snippet** (`uart.c`):
```c
#include <stdio.h>
#include <unistd.h>
int _write(int fd, char *buf, int len) {
    volatile uint32_t *uart_tx = (uint32_t *)0x10000000;
    for (int i = 0; i < len; i++) {
        *uart_tx = buf[i];
    }
    return len;
}
int main() {
    printf("Hello via UART!\n");
    return 0;
}
```

**Command-Line**:
```bash
riscv32-unknown-elf-gcc -march=rv32imc -mabi=ilp32 -nostartfiles -o uart.elf uart.c
```

**Expected Output** (on emulator):
```
Hello via UART!
```

## Task 17: Endianness & Struct Packing
**Objective**: Verify RV32’s little-endian default.

**Code Snippet** (`endian.c`):
```c
#include <stdio.h>
#include <stdint.h>
int main() {
    union {
        uint32_t val;
        uint8_t bytes[4];
    } u = { .val = 0x01020304 };
    for (int i = 0; i < 4; i++) {
        printf("Byte %d: 0x%02x\n", i, u.bytes[i]);
    }
    return 0;
}
```

**Command-Line**:
```bash
riscv32-unknown-elf-gcc -march=rv32imc -mabi=ilp32 -o endian.elf endian.c
qemu-system-riscv32 -nographic -kernel endian.elf
```

**Expected Output**:
```
Byte 0: 0x04
Byte 1: 0x03
Byte 2: 0x02
Byte 3: 0x01
```
**Explanation**: RV32 is little-endian, so `0x01020304` is stored as `04 03 02 01`.

## Notes
- Run commands in a Linux environment compatible with the toolchain.
- Use Spike or QEMU for tasks requiring emulation (e.g., Task 7, 16, 17).
- Ensure `volatile` is used for memory-mapped I/O to prevent optimization issues.
- For linker scripts and startup code, adapt addresses to your hardware or emulator.

## Resources
- **RISC-V Toolchain**: https://github.com/riscv/riscv-gnu-toolchain
- **Spike**: https://github.com/riscv/riscv-isa-sim
- **QEMU**: https://www.qemu.org
- **Newlib**: https://sourceware.org/newlib/
