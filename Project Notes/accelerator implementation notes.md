## [Implementation of a RISC-V Processor with Hardware Accelerator](https://odr.chalmers.se/server/api/core/bitstreams/8a3fa1e8-9dd1-4e4c-a598-6d4967d381bd/content)

- ZedBoard
  
  - Zynq-700
    - Xilinx Artix-7 FPGA
    - ARM Cortex-A9 MPCore processor

- PULPino
  
  - 32-bit RISC-V 
    - 5 MHz
  - Written in SystemVerilog

- Accelerator connected to APB bus on PULPino
  
  - AXI could be used but more complicated
  - Write to virtual memory address

## [PULPino GitHub](https://github.com/pulp-platform/pulpino/tree/master/fpga)

- Supports ZedBoard and ZYBO
- Tested with Vivado 2015.1
- Could use this: [GitHub - RISCV model for Verilator](https://github.com/aignacio/riscv_verilator_model) to support Arty A7-100T
