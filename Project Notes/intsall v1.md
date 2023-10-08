Use an Ubuntu 20.04.6 LTS working environment

[Alternative downloads | Ubuntu](https://ubuntu.com/download/alternative-downloads)

---

Install vitis core dev suite (includes vivado)

[Downloads](https://www.xilinx.com/support/download/index.html/content/xilinx/en/downloadNav/vitis.html)

Deselecting some board options in the intall list can reduce install size.

---

I needed to have en_US.UTF-8 generated in /etc/locale.gen and run `locale-gen`

When installing vitis is had issues with it getting stuck on 'Generating installed device list'

It requires libncurses.so.5 if you have version 6 installed you can create a symlink following this post: [linux - error while loading shared libraries: libncurses.so.5: - Stack Overflow](https://stackoverflow.com/a/42686791)

Or try installing these packages. They may not all be necessary.

`libncurses5-dev, libncursesw5-dev, libncurses5, libtinfo5, libtinfo-dev, ncurses-compat-libs` were mentioned.

references:

[Reddit - Dive into anything](https://www.reddit.com/r/Xilinx/comments/s7lcgq/help_vivado_ml_20212_stuck_on_generating/?onetap_auto=true)

[Xilinx Customer Community](https://support.xilinx.com/s/question/0D52E00006hpRxQSAU/vivado-20202-installation-stuck-at-generating-installed-device-list-on-ubuntu-2004lts?language=en_US)

[Xilinx Customer Community](https://support.xilinx.com/s/question/0D52E00006iHjbcSAC/vivado-20211-installation-hangs-at-generating-installed-device-list?language=en_US)

---

bord files are installed at

```
<Vivado Install>\<Vivado_version>\data\xhub\boards\XilinxBoardStore\boards\Xilinx\
```

reference: [Xilinx Customer Community](https://support.xilinx.com/s/article/The-board-file-location-with-the-latest-Vivado-tools?language=en_US)

My bord files were found here: [GitHub - Digilent/vivado-boards](https://github.com/Digilent/vivado-boards)

---

once installed it did not detect the board to fix this i manualy installed drivers

```bash
cd /<Xilinx Install>/Vivado/2023.1/data/xicom/cable_drivers/lin64/install_script/install_drivers/
sudo ./install_drivers
```

reference: [Reddit - Dive into anything](https://www.reddit.com/r/FPGA/comments/l5in82/vivado_hardware_manager_on_linux_and_cable_drivers/)

---

install vivado-risk-v and make sd card and bitstream

I had to replace `python` with `python-is-python3` and remove `libtinfo5` under apt-install in the Makefile.

```bash
sudo apt install git make
git clone https://github.com/eugene-tarassov/vivado-risc-v.git
cd vivado-risc-v
make apt-install
make update-submodules
```

I had to add `<Xilinx Install>/Vivado/2023.1/bin` to PATH

```bash
make CONFIG=rocket64b1 BOARD=nexys-a7-100t bitstream
```

```bash
./mk-sd-card
```

```bash
make CONFIG=rocket64b1 BOARD=nexys-a7-100t flash
```

output:

```
env HW_SERVER_URL=tcp:localhost:3121 \
 xsdb -quiet board/jtag-freq.tcl
rlwrap: warning: your $TERM is 'xterm-256color' but rlwrap couldn't find it in the terminfo database. Expect some problems.

env HW_SERVER_ADDR=localhost:3121 \
env CFG_PART=s25fl128sxxxxxx0-spi-x1_x2_x4 \
env mcs_file=workspace/rocket64b1/nexys-a7-100t-riscv.mcs \
env prm_file=workspace/rocket64b1/nexys-a7-100t-riscv.prm \
 env XILINX_LOCAL_USER_DATA=no vivado -mode batch -nojournal -nolog -notrace -quiet -source board/program-flash.tcl
INFO: [Common 17-1239] XILINX_LOCAL_USER_DATA is set to 'no'.
INFO: [Labtools 27-2285] Connecting to hw_server url TCP:localhost:3121
INFO: [Labtools 27-3415] Connecting to cs_server url TCP:localhost:0
INFO: [Labtools 27-3417] Launching cs_server...
INFO: [Labtools 27-2221] Launch Output:


******** Xilinx cs_server v2023.1.0
  ****** Build date   : Apr 10 2023-16:59:24
    **** Build number : 2023.1.1681142364
      ** Copyright 2017-2022 Xilinx, Inc. All Rights Reserved.
      ** Copyright 2022-2023 Advanced Micro Devices, Inc. All Rights Reserved.



INFO: [Labtoolstcl 44-466] Opening hw_target localhost:3121/xilinx_tcf/Digilent/210292ABF187A
INFO: [Labtools 27-1435] Device xc7a100t (JTAG device index = 0) is not programmed (DONE status = 0).
INFO: [Labtools 27-3164] End of startup status: HIGH
INFO: [Labtools 27-2302] Device xc7a100t (JTAG device index = 0) is programmed with a design that has 1 SPI core(s).
Mfg ID : 1   Memory Type : 20   Memory Capacity : 18   Device ID 1 : 0   Device ID 2 : 0
Performing Erase Operation...
Erase Operation successful.
Performing Blank Check Operation...
Blank Check Operation successful. The part is blank.
Performing Program and Verify Operations...
Program/Verify Operation successful.
INFO: [Labtoolstcl 44-377] Flash programming completed successfully
program_hw_cfgmem: Time (s): cpu = 00:00:00.12 ; elapsed = 00:01:11 . Memory (MB): peak = 1393.711 ; gain = 7.000 ; free physical = 4728 ; free virtual = 57068
```

This did not work tried gui programming

add memory device,ok to program it now, select `workspace/rocket64b1/nexys-a7-100t-riscv.mcs` and ok.

Right click on the FPGA device - Boot from Configuration Memory Device

This failed to boot from memory so i just programmed it directly.

open uart console:

pipx install pyserial

`sudo pyserial-miniterm /dev/ttyUSB1 115200`

you can chek the device by looking at `/dev/`

after directly programming it o got this eero over uart

```
RISC-V 64, Boot ROM V3.6
Cannot mount SD: Not a valid FAT volume
```

solved by reformatting sd card to FAT32 and then re running `./mk-sd-card`

[GitHub - eugene-tarassov/vivado-risc-v: Xilinx Vivado block designs for FPGA RISC-V SoC running Debian Linux distro](https://github.com/eugene-tarassov/vivado-risc-v)
