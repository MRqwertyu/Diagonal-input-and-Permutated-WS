# DiP Systolic Array Accelerator

This repository contains the RTL design, testbenches, and software drivers for a custom Diagonal-input and Permutated (DiP) Systolic Array accelerator. Designed as a complete System-on-Chip (SoC), this project compares a standard isomorphic hardware architecture against a highly optimized, clock-gated architecture designed to reduce dynamic power during sparse matrix multiplication (such as post-ReLU data).

The primary target board is the Xilinx Zybo Z7 (Zynq-7000), utilizing the fused ARM Cortex-A9 processor to feed data and read results via a Dual-Port BRAM bridge.

## Features

* **Dual Architectural Implementations:** Includes both a standard Isomorphic baseline (no clock gating) and an Optimized architecture featuring Operand Isolation (clock gating) to heavily reduce dynamic power consumption.


* **SoC Integration:** Features a True Dual-Port Block RAM (BRAM) bridge to safely interface the 125 MHz accelerator fabric (Programmable Logic) with the 667 MHz ARM processor (Processing System) via the AXI4 protocol.


* **Continuous Streaming Controller:** A soft-coded, dynamic control module that streams multi-tile matrix multiplications back-to-back without halting.


* **Power Profiling:** Built-in Verilog testbench configurations to generate SAIF (Switching Activity Interchange Format) files for accurate, cycle-by-cycle dynamic power estimation in Vivado.


* **Automated Verification:** Includes MATLAB and Python scripts to generate sparse hex memory files (`.mem`) and verify hardware matrix outputs against software-calculated baselines.



## Repository Structure

* `src/hw/`: Contains all synthesizable Verilog/SystemVerilog modules.
* `dip_accelerator_top.v`: The top-level SoC module containing the BRAM bridge and routing.


* `dip_controller.v`: The continuous streaming state machine.


* `dip_array.v`: The core Systolic Array mathematical engine.


* `pe_unit.v`: The Processing Element, available in standard and clock-gated (Optimized) variations.


* `matrix_memory.v` & `weight_memory.v`: Combinational matrix loading memories.




* `src/sw/`: Contains the Xilinx Vitis C-code drivers (`main.c`) to wake the ARM processor, trigger the accelerator, and print results over UART.


* `sim/`: Contains the Verilog testbench (`tb_top.v`) and sample `.mem` files (e.g., `matrix_a.mem`, `weights_natural.mem`).


* `scripts/`: Python and MATLAB verification scripts.


* **Systolic Array**: Core documentation and architectural block diagrams referencing the datapath skewing and permutation logic.

## Hardware Architecture

The accelerator uses a 2D grid of Processing Elements (PEs) written using SystemVerilog unpacked arrays to handle matrix dot-products.

* **Input Wavefronts:** Data matrices are skewed diagonally to ensure MAC units multiply the correct staggered values per clock cycle.


* **Data Sink:** To prevent Vivado's synthesis engine from trimming the DSP slices, the array's output is directly anchored into an inferred hardware BRAM block, acting as a physical data sink for the ARM processor.



## Getting Started

### 1. Hardware Synthesis (Vivado)

1. Create a new Vivado project targeting the Zybo Z7 board.


2. Import all Verilog files from the `src/hw/` directory.


3. Create a Block Design (`soc_design`) and instantiate the ZYNQ7 Processing System.


4. Run Block Automation to apply the Zybo DDR3/UART presets, and enable the `M AXI GP0` interface.


5. Instantiate `dip_accelerator_top`, make external connections for controls (`start`, `rst_n`, LEDs), and connect the AXI BRAM Controller.


6. Generate the HDL Wrapper and run **Generate Bitstream**.


7. Export the hardware (`.xsa` file) making sure to check **Include Bitstream**.



### 2. Software Deployment (Vitis)

1. Launch Xilinx Vitis and create a new **Platform Project** using the exported `.xsa` file.


2. Build the platform to generate the Board Support Package (BSP) drivers.


3. Create a new **Application Project** targeting the platform using the "Hello World" template.


4. Replace the template code with the provided C driver from `src/sw/main.c`.


5. Connect the Zybo Z7 via USB, open a Serial Terminal (115200 baud), and click **Launch on Hardware**.



## Power Analysis

To compare the Isomorphic and Optimized architectures:

1. Run the Behavioral Simulation in Vivado using the 30-tile sparse matrix `.mem` files.


2. Use the Tcl console to record toggling activity: `open_saif dip_power.saif`, `log_saif [get_objects -r *]`, run the simulation, and `close_saif`.


3. Run an Out-Of-Context (OOC) Implementation in Vivado.


4. Open the Power Report and supply the generated `.saif` file to calculate the dynamic power drop achieved by the clock-gating isolation.
