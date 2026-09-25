# RVBL Emulator

The RVBL Emulator is the host application used to interact with the RV32I processor after it is integrated into the AWS FPGA environment. It sends application inputs to the processor and reads the results produced through GPIO and UART.

The emulator will be executed only during the AWS FPGA stage. However, its interface must be considered while defining the application firmware so that the expected input and output protocol is clear before FPGA integration.

## Communication flow

```text
Emulator
   |
   | PCIe / AXI-Lite
   v
AWS FPGA top
   |
   +---- GPIO ---- Application firmware
   |
   +---- UART ---- Application firmware
```

The application firmware does not call the emulator directly. It accesses the memory-mapped GPIO and UART peripherals normally. The FPGA top connects those peripherals to registers that the emulator can access through PCIe and AXI-Lite.

## Defining the application protocol

Each application must define:

- which GPIO pins are inputs and outputs;
- whether a GPIO input is used as a trigger;
- how many bytes are sent to the processor through UART RX;
- the order and meaning of those bytes;
- how many bytes the firmware returns through UART TX;
- the meaning of the GPIO and UART results.

The emulator is generic: the meaning of each value is defined by the application firmware. For example, a 32-bit value may represent an altitude, price, sensor measurement, command, or any other application input.

## Repository purpose

This repository is provided as a reference for understanding the emulator interface and how values are exchanged with the processor. The emulator is not built or executed during the current local development stage.

During the AWS FPGA stage, the emulator will already be available in the environment and will be executed there as `./emu`.

## Command format

```shell
./emu [options]
```

Numeric values may be written in decimal or hexadecimal notation:

```text
120
0x78
```

## Options

| Option | Description | Data direction |
|---|---|---|
| `-h` | Shows the available options | — |
| `-d DIR` | Configures the direction of the external GPIO interface | Emulator → FPGA top |
| `-i DATA` | Applies an 8-bit value to the GPIO input pins | Emulator → processor |
| `-o DATA` | Reads the GPIO output pins and compares them with `DATA` | Processor → emulator |
| `-t1 DATA` | Sends one byte to the processor UART RX | Emulator → processor |
| `-t2 DATA` | Sends two bytes to the processor UART RX | Emulator → processor |
| `-t4 DATA` | Sends four bytes to the processor UART RX | Emulator → processor |
| `-r1 DATA` | Reads one byte from the processor UART TX and compares it with `DATA` | Processor → emulator |
| `-r2 DATA` | Reads two bytes from the processor UART TX and compares it with `DATA` | Processor → emulator |
| `-r4 DATA` | Reads four bytes from the processor UART TX and compares it with `DATA` | Processor → emulator |

### GPIO direction

Each bit supplied to `-d` controls one external GPIO line:

```text
0: the emulator drives an input value toward the processor
1: the emulator releases the line and observes the processor output
```

For the GPIO test firmware, `P3-P0` are processor inputs and `P7-P4` are processor outputs. Therefore, the corresponding emulator direction is:

```shell
-d 0xF0
```

### GPIO input and output

`-i` applies a value to the processor GPIO inputs. `-o` reads the processor GPIO outputs and checks whether the result matches the expected value.

Example:

```shell
./emu -d 0xF0 -i 0x02 -o 0x20
```

This sequence:

1. configures `P3-P0` as values driven by the emulator and `P7-P4` as values observed by the emulator;
2. applies `0x02` to the GPIO input;
3. expects `0x20` on the GPIO output.

### UART transmit options

The `-t` options describe data sent by the emulator to the processor. From the firmware perspective, these bytes arrive through UART RX.

```shell
./emu -t1 0x30
./emu -t2 0x1234
./emu -t4 0x12345678
```

Multi-byte values are sent least-significant byte first. For example:

```text
-t4 0x12345678

UART byte order: 0x78, 0x56, 0x34, 0x12
```

The application firmware must read and interpret the bytes in the same order.

### UART receive options

The `-r` options describe data returned by the processor. The emulator reads the bytes transmitted through UART TX and compares them with the expected value.

```shell
./emu -r1 0x30
./emu -r2 0x1234
./emu -r4 0x12345678
```

The first byte transmitted by the firmware occupies the least-significant byte of the expected value.

The current FPGA interface stores up to eight bytes returned by the processor in each execution. The application response must be designed within this limit.

## Complete GPIO and UART test

The GPIO and UART test firmware can be checked with a single execution:

```shell
./emu -d 0xF0 -i 0x02 -o 0x20 -t1 0x30 -r1 0x30
```

Expected behavior:

```text
GPIO input:   0x02
GPIO output:  0x20

UART RX:      0x30
UART TX:      0x30
```

## Application example

Consider an application that receives three 32-bit altitude measurements, checks whether any value exceeds a limit, and returns a one-byte result.

```shell
./emu \
  -d 0xF0 \
  -i 0x01 \
  -t4 90 \
  -t4 105 \
  -t4 80 \
  -r1 1 \
  -o 0x10
```

One possible protocol is:

```text
GPIO input 0x01  -> start trigger
UART values      -> altitude measurements: 90, 105 and 80
UART result 0x01 -> limit exceeded
GPIO output 0x10 -> processing completed
```

This is only an illustrative protocol. Each application defines its own data meaning, processing behavior, and expected result.

## Important execution rule

All operations belonging to one application test must be passed in the same emulator command. Each new execution initializes the FPGA connection and resets the processor before processing the options.

The options are processed as a sequence. Their order must follow the protocol expected by the application firmware. For example, if the firmware waits for a GPIO trigger before reading UART data, the GPIO input option must appear before the UART transmit options.

## Emulator register interface

The emulator communicates with the FPGA top through the following AXI-Lite addresses:

| Address | Interface | Purpose |
|---:|---|---|
| `0x00` | Reset | Controls the processor reset |
| `0x04` | GPIO input/direction | Bits 7–0 contain input data; bits 15–8 contain direction |
| `0x08` | GPIO output | Reads the processor GPIO output |
| `0x0C` | UART input | Sends a byte to the processor UART RX |
| `0x10` | UART output 0 | First four bytes returned by the processor |
| `0x14` | UART output 1 | Next four bytes returned by the processor |
| `0x18` | UART position | Indicates the current position in the receive buffer |
| `0x1C` | Program counter | Exposes the processor program counter |

These addresses belong to the FPGA top interface. They are different from the GPIO and UART addresses accessed by the application firmware inside the processor.
