# Install Guide

## Install Vivado

- To prevent issues use an Ubuntu 20.04.6 LTS system
  
  - [Download link](https://ubuntu.com/download/alternative-downloads)

- Install vitis core dev suit (includes vivado)
  
  - [Download link](https://www.xilinx.com/support/download/index.html/content/xilinx/en/downloadNav/vitis.html)
  
  - During install deselecting some board options can significantly reduce install size
  
  - After install setup drivers by running:
    
    ```bash
    cd /<Xilinx Install>/Vivado/2023.1/data/xicom/cable_drivers/lin64/install_script/install_drivers/
    sudo ./install_drivers
    ```

- Install board files if required
  
  - My bord files were found here: [GitHub - Digilent/vivado-boards](https://github.com/Digilent/vivado-boards)
  
  - Bord files are installed at:
    
    ```
    <Vivado Install>\<Vivado_version>\data\xhub\boards\XilinxBoardStore\boards\Xilinx\
    ```

- Ensure you board is set up correctly. Mine (Nexys A7 100t) required jumpers to be set

- Check Vivado board shows up when creating a new project

- Check board can be connected to under hardware manager in Vivado

- Add `<Xilinx Install>/Vivado/2023.1/bin` to PATH

## Install vivado-risk-v

- Clone the repo:
  
  ```bash
  git clone https://github.com/eugene-tarassov/vivado-risc-v.git
  ```

- Setup repo:
  
  ```bash
  make apt-install
  make update-submodules
  ```

- Vivado updated to version 2023.1.1 so I had to change the name of `riscv-2023.1.tcl` under `vivado-risk-v/board/<Board Name>` to `2023.1.1.tcl`

- Build the bitsteam:
  
  ```bash
  make CONFIG=rocket64b1 BOARD=nexys-a7-100t bitstream
  ```

- Set up the sd card:
  
  - Delete all partitions and ensure it is not mounted befor running
  
  ```bash
  ./mk-sd-card
  ```

- Flash bitstram to the board:
  
  - In different termial run:
    
    ```bash
    hw_server
    ```
  
  - Then in main terminal run:
    
    ```bash
    make CONFIG=rocket64b1 BOARD=nexys-a7-100t flash
    ```
  
  - This did not work for me so I ran the following and then manualy programed the device through the hardware manager GUI
    
    ```bash
    make CONFIG=rocket64b1 BOARD=nexys-a7-100t bitstream vivado-gui
    ```
  
  - You can flash to memory through GUI
    
    [Using Vivado GUI to program the FPGA flash memory](https://github.com/eugene-tarassov/vivado-risc-v/blob/master/docs/vivado-flash.md)

## Run FPGA

- Install and run putty with sudo
  
  ```bash
  sudo apt install putty putty-tools
  sudo putty
  ```

- Connect to the FPGAs serial port at a baud rate of 115200
  
  My board was on port /dev/ttyUSB1 but yours may be different

- You should see Linux booting it will take quite a long time 
  
  [Slow boot on arty-a7-100t · Issue #189 · eugene-tarassov/vivado-risc-v · GitHub](https://github.com/eugene-tarassov/vivado-risc-v/issues/189)

- You can log in with Username, Password of debian, debian or root, root

## Using linux

- I had issues installing programs with apt. Adding the following to `/etc/apt/sources.list` fixed it
  
  ```
  deb [arch=riskv64] http://ftp.ports.debian.org/debian-ports unstable main
  ```
  
  [Can not install pip · Issue #44 · eugene-tarassov/vivado-risc-v · GitHub](https://github.com/eugene-tarassov/vivado-risc-v/issues/44)
