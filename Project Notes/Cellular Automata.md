# Game of life

**Cell:** 2 states (alive/dead)

**Board:** 2D

**Rules:**

1. If the cell is alive, then it stays alive if it has either 2 or 3 live neighbors otherwise it dies.

2. If the cell is dead and has 3 live neighbors, set state to alive.

## FPGA Implementation

```verilog
module board
    #(
    parameter BOARD_WIDTH = 0,
    parameter BOARD_HEIGHT = 0,
    localparam BOARD_SIZE = BOARD_WIDTH * BOARD_HEIGHT
    )(
    input clk, rst,
    input set_state, generate_state,
    input [(BOARD_SIZE-1):0] new_board_state,
    output [(BOARD_SIZE-1):0] board_state
    );

    // Board Register

    localparam DEFAULT_EDGE_VALUE = 1'b0; // Value to be used if neighbour is off edge of board

    wire [BOARD_SIZE:0] cell_state_register;
    assign cell_state_register[BOARD_SIZE] = DEFAULT_EDGE_VALUE; // Final bit if register holds default edge value

    assign board_state = cell_state_register[(BOARD_SIZE-1):0]; // Output current state

    // Board Cells

    genvar x;
    genvar y;
    generate
        for (y=0; y<BOARD_HEIGHT; y=y+1) begin
            for (x=0; x<BOARD_WIDTH; x=x+1) begin
                board_cell #(x,y,BOARD_WIDTH,BOARD_HEIGHT) cell_module (clk, rst, set_state, generate_state, new_board_state[(y * BOARD_WIDTH) + x], cell_state_register, cell_state_register[(y * BOARD_WIDTH) + x]);
            end
        end
    endgenerate


endmodule

module board_cell
    #(
    parameter X = 0,
    parameter Y = 0,
    parameter BOARD_WIDTH = 0,
    parameter BOARD_HEIGHT = 0
    )(
    input clk, rst,
    input set_state, generate_state,
    input new_state,
    input [(BOARD_WIDTH*BOARD_HEIGHT):0] board_state,
    output reg state
    );

    // Number of elements in board
    localparam BOARD_SIZE = BOARD_WIDTH*BOARD_HEIGHT;

    // -------------------
    // Get neigbour values
    // -------------------

    // Check if neighbours are valid at compile time
    localparam N_VALID = (Y - 1) >= 0;
    localparam E_VALID = (X + 1) < BOARD_WIDTH;
    localparam S_VALID = (Y + 1) < BOARD_HEIGHT;
    localparam W_VALID = (X - 1) >= 0;

    // Generate neigbours and cell indexs at compile time
    // BOARD_SIZE index holds the value used when cell is of the edge of board
    localparam C_IDX  = (Y * BOARD_WIDTH) + X;
    localparam N_IDX  = N_VALID            ? C_IDX - BOARD_WIDTH : BOARD_SIZE;
    localparam E_IDX  = E_VALID            ? C_IDX + 1           : BOARD_SIZE;
    localparam S_IDX  = S_VALID            ? C_IDX + BOARD_WIDTH : BOARD_SIZE;
    localparam W_IDX  = W_VALID            ? C_IDX - 1           : BOARD_SIZE;
    localparam NE_IDX = N_VALID && E_VALID ? N_IDX + 1           : BOARD_SIZE;
    localparam NW_IDX = N_VALID && W_VALID ? N_IDX - 1           : BOARD_SIZE;
    localparam SE_IDX = S_VALID && E_VALID ? S_IDX + 1           : BOARD_SIZE;
    localparam SW_IDX = S_VALID && W_VALID ? S_IDX - 1           : BOARD_SIZE;

    // Get state of neighbours
    assign n = board_state[N_IDX];
    assign e = board_state[E_IDX];
    assign s = board_state[S_IDX];
    assign w = board_state[W_IDX];
    assign ne = board_state[NE_IDX];
    assign nw = board_state[NW_IDX];
    assign se = board_state[SE_IDX];
    assign sw = board_state[SW_IDX];

    // Get number of alive neighbours and current state of cell
    wire [3:0] alive_neighbours;
    assign alive_neighbours = n + e + s + w + ne + nw + se + sw;
    assign current_state = board_state[C_IDX];

    // --------------
    // Set next state
    // --------------

    always @ (posedge clk) begin
        // reset cell
        if (rst)
            state <= 0;

        // Set a new inital state
        else if (set_state) 
            state <= new_state;

        // Set new state based on neighbours
        else if (generate_state) begin
            // If alive, make dead when less than 2 or greater than 3 neighbours
            if (current_state) begin
                if (alive_neighbours < 2 || alive_neighbours > 3) 
                    state <= 1'b0;
            end

            // If dead, make alive if 3 neighbours
            else if (alive_neighbours == 3)
                state <= 1'b1;
            else
                state <= current_state;
        end
    end

endmodule
```

### Resource Utilization

**Single Cell:**

![](C:\Users\callu\AppData\Roaming\marktext\images\2024-02-26-14-17-59-image.png)

**32x32:**

![](C:\Users\callu\AppData\Roaming\marktext\images\2024-02-27-15-25-06-image.png)

64x64:

![](C:\Users\callu\AppData\Roaming\marktext\images\2024-02-27-17-11-19-image.png)

# Wolfram Rules

### Resource Utilization

**1024:**

![](C:\Users\callu\AppData\Roaming\marktext\images\2024-02-27-16-03-39-image.png)

get area usage stats

get timing compared to software

overview of fpga

how risc5 is implemented
