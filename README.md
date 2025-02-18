# dumb-processor
Circuit simulated 4-bit microprocessor design

# running
Open the PROCESSOR.txt file in falstad circuit simulator at https://www.falstad.com/circuit/

# Features
* 3 general purposer 4-bit registers
* 1 constant writing register
* Static program memory
* Writable program counter with 7-segment display
* 4-bit adder, with carry flag
* 4-bit xor
* Program ROM memory, 8-bit instruction size, 8-bit address size
* RAM memory, 4-bit word size, 4-bit address size
* 7-segment output

# Register map
| name | source | destination |
|------|--------|-------------|
| R1 | 0x1 | 0x1 |
| R2 | 0x2 | 0x2 |
| R3 | 0x3 | 0x3 |
| R1 inverted | 0x4 | - |
| Save | 0x5 | 0x0 |
| Add | 0xa | - |
| Add carry bit | 0xb | - |
| Xor | 0xd | - |
| RAM | 0xc | 0xc |
| 7-seg display | - | 0xa |
| Program counter jump | - | 0xf |

# Working principle and programming
The processor works solely by moving data between registers. The only special operation is loading constant values, but this is
done in a similar manner to moving from register to register. The data is moved trough data bus, to which all the registers are
connected to, and the register behaviour is controlled bu the instruction bus.

The fixed size 8-bit instructions consist of two parts, 4-bit source address and 4-bit destination address. An instruction is a combination of
these. This allows for example to move the value in register 1 to register 2. In the 8-bit instruction, the 4 most significant
bits represent the source register, and the 4 least significant bits represent the destination register to where the data stored
in source register will be moved to.

Loading constant values to a register is done by setting the destination register to `save register`, and source value to which
ever value is to be saved. In practise, this register does not read from the data bus to its destination register, but from the 4
most significant bits of the instruction bus.

Another special register is the program counter. Performing a jump operation is done by setting the destination register to
program counter destination register address. In this case, the source register is ignored.

## Functional registers
In addition to basic read/write registers, the processor has some functional registers too:
  * addition
  * Xor
  * Readable/writable RAM memory
  * Writeable program counter to do jumps

To save in register count, there segister may use the general purpose registers as input values for their operations.

### Addition
Addition adds numbers in registers R1 and R2 together. When set as an source register, the value can be stored to any general
purpose register, 7-segment display, or RAM.

#### Addition carry
in case of a overflow (sum > 15), the carry bit is set and can be used as a source register to write to any general
purpose register, 7-segment display, or RAM.

### Xor
Performs xor operation on value in register R1 When set as an source register, the value can be stored to any general
purpose register, 7-segment display, or RAM.

### RAM
Ram memory has 4-byte data line and 4-byte address line. Ram address is read directly from register R3. Ram can be set bot as
source and destination register.

### Program counter
This processor has 8-bit writeable program counter, that can perform conditional jumps. loading a value to the program counter
involves all general purpose registers R1, R2, and R3. The 4 most significant bits to load to program counter are read from R2,
and the 4 least significant bits are read from R1. Jump condition depends on the R3. If R3 is not zero, jump is performed. If R3
is zero, program countter carries to next instruction in ROM.

NOTE! Due to how program counter incrementation is built, the jump address should be pointed to ROM address before the wanted
instruction. In more detail on this in clock signal explanation.

## 7-segment display
The 7-segment display register enables data to be shown for the user. Writing to it will display the data hex value.

## Clock signal
On the rising edge of the clock signal, the program counter is increased and a new instruction gets loaded from ROM to the
instruction bus. On the Falling edge of the clock signal, the register read/write operations are triggered, and data is moved
between registers.

Because of this, when data is loaded to the program counter register, the next rising edge of the clock signal will increment it
to the next instruction. That is why instruction in address `0` will never be executed , and during a jump operation, the jump
address should point to ROM address before the desired instruction.
