# Baremetal code

## Example hello_world

Downloaded compiled the example hello world code that prints to the UART. This worked with no issues.

## GPIO Control

### AXI GPIO

- The AXI GPIO block can be added to the block design, set interface 1 to LEDs.
  
  - `S_AXI` should be connected to a new `MXX_AXI` connection in the `io_axi_s` block. 
    
    - The new connection can be created by re-custermising IP of the `io_axi_s` block and incrementing `Number of Master Interfaces`.
  
  - `s_axi_aclk` and `s_axi_aresetn` connects to the same lines as the `aclk` and `aresetn` inputs to `io_axi_s`.
  
  - Use the connection automation to connect the GPIO to the boards LEDs

- The drivers provided by Xilinx are big and complicated but the actual process of controling the output is not. I trawled through the drivers to get the important part to make the following baremetal code.
  
  - The code needs to be able to set 32bit blocks of memory at a 64bit address
    - `uint32_t` and `uint64_t` are used for this
  - The drivers provided offsets from the base address for the data and direction registers for the two channels. These offsets are applied to the base address and then the 32bit value is written there.
  - The each bit in the Data register represents one gpio output bit. so `0x01` sets the first bit if writen.

### Code

- Clone the hello_world example.

- Replace main.c with:
  
  ```c
  #include <stdint.h>
  
  #include "common.h"
  #include "kprintf.h"
  
  /************************** Constant Definitions *****************************/
  
  #define XGPIO_DATA_OFFSET    0x0   /**< Data register for 1st channel */
  #define XGPIO_TRI_OFFSET    0x4   /**< I/O direction reg for 1st channel */
  #define XGPIO_DATA2_OFFSET    0x8   /**< Data register for 2nd channel */
  #define XGPIO_TRI2_OFFSET    0xC   /**< I/O direction reg for 2nd channel */
  
  #define XGPIO_GIE_OFFSET    0x11C /**< Glogal interrupt enable register */
  #define XGPIO_ISR_OFFSET    0x120 /**< Interrupt status register */
  #define XGPIO_IER_OFFSET    0x128 /**< Interrupt enable register */
  
  #define XGPIO_CHAN_OFFSET  8    /**< Channel offeset */
  
  #define LED 0x01   /* Assumes bit 0 of GPIO is connected to an LED  */
  #define XGPIO_AXI_BASEADDRESS 0x60040000
  #define GPIO_CHANNEL 1
  #define DELAY 10000000
  
  /****************************************************************************/
  
  static void set_32(uint64_t Addr, uint32_t Value)
  {
      volatile uint32_t *LocalAddr = (volatile uint32_t *)Addr;
      *LocalAddr = Value;
  }
  
  #define Gpio_WriteReg(BaseAddress, RegOffset, Data) \
      set_32((BaseAddress) + (RegOffset), (uint32_t)(Data))
  
  /****************************************************************************/
  
  int main(void)
  {
      volatile int Delay;
  
      kprintf("\n");
  
      uint64_t DataOffset = ((GPIO_CHANNEL - 1) * XGPIO_CHAN_OFFSET) + XGPIO_DATA_OFFSET;
  
      /* Loop forever blinking the LED */
  
      while (1) {
          /* Set the LED to High */
          Gpio_WriteReg(XGPIO_AXI_BASEADDRESS, DataOffset, LED);
          kprintf("High\n");
  
          /* Wait a small amount of time so the LED is visible */
          for (Delay = 0; Delay < DELAY; Delay++);
  
          /* Clear the LED bit */
          Gpio_WriteReg(XGPIO_AXI_BASEADDRESS, DataOffset, 0x00);
          kprintf("Low\n");
  
          /* Wait a small amount of time so the LED is visible */
          for (Delay = 0; Delay < DELAY; Delay++);
      }
  
  }
  ```

- This code is bits from the driver glued together it can be made much cleaner.

- Compile with `make clean all` and copy `boot.elf` to the SD card.

- When running the Far right led should slowly blink and the UART should print when it is set high and low.
