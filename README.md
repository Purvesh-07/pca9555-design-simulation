# PCA9555 Design and Simulation

16-bit I2C GPIO expander (slave only), written in Verilog-2001 and verified with a
self-checking testbench. Simulated in Icarus Verilog (72/72 checks pass); the
`run.do` script is for ModelSim Intel FPGA Edition 18.1.

## Folder layout
| File | Purpose |
|---|---|
| `rtl/pca9555.v` | Top level (ports match the datasheet pins) |
| `rtl/i2c_front_end.v` | 2-FF synchronizers, START/STOP and SCL edge detection |
| `rtl/i2c_slave.v` | Protocol FSM: address match, command byte, read/write, ACK, port toggle |
| `rtl/reg_file.v` | Input/Output/Polarity/Configuration registers, read mux |
| `rtl/io_ports.v` | Tri-state pin drivers, pin synchronizers |
| `rtl/int_logic.v` | Snapshot compare, INT# set/clear |
| `tb/tb_pca9555.v` | I2C master model, scoreboard, 12 tests (2 devices on the bus) |
| `sim/run.do` | ModelSim compile/simulate/wave script |
| `sim/simulation_log.txt` | Log of a passing run |

## Run in ModelSim
```
cd pca9555/sim
vsim -do run.do          (or: do run.do from the ModelSim console)
```
Look for `ALL TESTS PASSED` in the transcript.

## Design notes / assumptions
* Slave address `0100 A2 A1 A0`; command byte bits [2:0] select the register; bit 0 toggles after each data byte.
* `clk` oversamples SCL/SDA (testbench: 100 MHz clk, 400 kHz SCL). Keep clk >= ~8x SCL.
* Write data takes effect at the 9th (ACK) clock; INT# is cleared at the ACK clock of an Input Port read.
* Interrupt: input-configured pin differs from the snapshot taken at the last read of its port. Output pins never interrupt. Comparison uses raw pin values (polarity inversion does not affect it).
* Pin and INT# timing is zero-delay RTL (no tiv/tir/tps parameters). No SCL/SDA glitch filter beyond the 2-FF synchronizer.
* Command bytes above 0x07 are ACKed and use only bits [2:0] (reserved codes are not modelled).
