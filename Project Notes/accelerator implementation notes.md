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


## Adding GPIO AXI IO block
- In IO block add AXI GPIO IP block
  - Connect S_AXI to M04_AXI in io_axi_s
  - Connect s_axi_aclk to axi_clk (some other ip are connected to different clocks check this)
  - Connect s_axi_aresetn to axi_reset
  - Use auto connect to connect GPIO to relevent board interface (LED)

- Under address editor
  - Right click the IO entry and select assign 
  - Set the base address asignment

- Add to device tree in `vivado-risc-v/board/nexys-a7-100t/bootrom.dts`
  - Check that the address matches what is set in vivado
  - [device tree explanation](https://www.kernel.org/doc/Documentation/devicetree/bindings/gpio/gpio-xilinx.txt)

```
gpio: gpio@60040000 {
            #gpio-cells = <2>;
            compatible = "xlnx,xps-gpio-1.00.a";
            gpio-controller ;
            interrupt-parent = <&L2>;
            interrupts = <4>;
            reg = < 0x60040000 0x10000 >;
            xlnx,all-inputs = <0x0>;
            xlnx,dout-default = <0x0>;
            xlnx,gpio-width = <0x10>;
            xlnx,interrupt-present = <0x1>;
            xlnx,is-dual = <0>;
            xlnx,tri-default = <0xffffffff>;
        };
```

- Synthasise in vivado
- Build bitstream with make command
- This did not work
