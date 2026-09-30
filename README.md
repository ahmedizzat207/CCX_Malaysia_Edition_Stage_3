# ChampionCHIP eXperience - Stage 3

This repository collects the Stage 3 GPIO/UART test firmware, RV32I firmware builder, and RVBL emulator documentation. It is based on the [official CCX Malaysia Stage 3 repository](https://github.com/championchip-experience-community/CCX_Malaysia_Edition_Stage_3) and documents how these resources relate to the [RVBL-2 multicycle RISC-V hardware project](https://github.com/ahmedizzat207/riscv32_multicycle).

## Hardware project

The RVBL-2 repository contains the processor RTL, GPIO and UART peripherals, and their interface definitions. The firmware in this repository exercises those memory-mapped peripherals, while the emulator documentation describes the GPIO/UART protocol used to communicate with a processor integrated into the AWS FPGA environment. Check the hardware repository for its current memory map and implementation details when adapting the firmware.

AWS FPGA access is not required for the current local development stage. The official challenge instructions determine when FPGA access is needed.

## Resources

### GPIO and UART test firmware

Folder: `RVBL-GPIO-UART-Test-Firmware`

Contains the official test firmware and instructions for validating GPIO and UART behavior in a local processor simulation. See its [README](RVBL-GPIO-UART-Test-Firmware/README.md).

### RVBL firmware builder

Folder: `RVBL-Firmware-Builder`

Contains a Makefile and scripts for assembling RV32I source and generating instruction-memory firmware. See its [README](RVBL-Firmware-Builder/README.md).

### RVBL emulator

Folder: `RVBL-Emulator`

Documents emulator options, GPIO/UART data flow, and the FPGA register interface. See its [README](RVBL-Emulator/README.md).

## Repository structure

```text
.
├── RVBL-Emulator/
├── RVBL-Firmware-Builder/
├── RVBL-GPIO-UART-Test-Firmware/
└── README.md
```

## Acknowledgements

The Stage 3 challenge resources are maintained by the [ChampionCHIP eXperience community](https://github.com/championchip-experience-community). The RVBL-2 hardware project acknowledges **Equipe 15**:

- Ahmed Izzat Sidahmed Ali Tahir
- Ng Kah Lok
- Gan Shao Hng
- Jolin Tan
- Fatin Nuralya Binti Mohamad

Thanks also to the ChampionCHIP eXperience organizing committee, the project mentors, and the ChipInventor platform team for the challenge framework, tools, and technical guidance.