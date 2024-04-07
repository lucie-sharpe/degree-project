# Custom IP

- In vivado `tools->Create and Package New IP`
  
  - Select: `Create new AXI4 Pereferal`
  
  - Set name and location
  
  - Keep default AXI interface settings
  
  - Select Edit IP and finish

- IP Editor
  
  - Ignore FREQ_HZ warning
  
  - In the `xxx_AXI_inst.v` file:
    
    - You can use the values of `slv_reg0-3` to get values sent by the proccessor 
    
    - To return values modify the block labled `Address decoding for reading registers` around line 375 as shown below.
      
      ```verilog
      always @(*)
      begin
        // Address decoding for reading registers
        case ( axi_araddr[ADDR_LSB+OPT_MEM_ADDR_BITS:ADDR_LSB] )
          2'h0 : reg_data_out <= slv_reg0;
          2'h1 : reg_data_out <= reg0;
          2'h2 : reg_data_out <= 32'd9999;
          2'h3 : reg_data_out <= reg1;
          default : reg_data_out <= 0;
        endcase
      end
      ```
      
      This sets a constant value when the processor reads the 3rd 32bit block, the value of the `slv_reg0` that can be written to when reading the 1st block and custom values when reading the other two blocks.
    
    - The custom values are set from the value writen to `slv_reg0` using the following.
      
      ```verilog
      always @( slv_reg0 )
      begin
        reg0 <= slv_reg0;
        reg1 <= reg0 + 32'd300;
      end
      ```
  
  - Under `Package IP->File Groups` click `Merge changes from IP File Group Wizard`
  
  - Under `Package IP->Review and Package` select `Re-Package IP`

- Connecting the IP block
  
  - Add the IP block to the block design inside the IO block by searching the name you set in the IP list.
  
  - S_AXI`should be connected to a new`MXX_AXI`connection in the`io_axi_s` block.
    
    - The new connection can be created by re-custermising IP of the `io_axi_s` block and incrementing `Number of Master Interfaces`.
  
  - `s_axi_aclk` and `s_axi_aresetn` connects to the same lines as the `aclk` and `aresetn` inputs to `io_axi_s`.
  
  - Under the `Address Editor` right click the new ip block and select assign then set the `master base address`

- Baremetal code
  
  - The following is the baremetal code i used with this custom IP:
    
    ```c
    
    ```

- 
