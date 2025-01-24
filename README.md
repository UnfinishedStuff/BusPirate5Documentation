# BusPirate5Documentation
Documentation to remind me how to use the Bus Pirate 5 with Ubuntu.

## Setting the device up

* Plug in the device
* Check the serial port it is connected to: `ls -l /dev/serial/by-id` in a terminal.  This will list two ports for the BP5, the first one (normally `/dev/ttyACM0`) is fine.
* Connect using TIO with a baudrate of 115,200 bits per second: `tio /dev/ttyACM0 -b 115200`
* There should be a "connected" message.  Hit Enter, and it will ask if you want to use VT100-compatible mode.  Say yes.

## Helpful shortcuts
* Typing `= INT` where `INT` is any number in any format will print that value in hexadecimal, decimal, and binary to help with conversions. Note that you need a space between `=` and the number.

## Using I2C

* The BP5 has VOUT at the top (red wire), then SCA (orange), then SCL (yellow).  Ground is in black at the bottom.
* Once you've connected to the BP5, type `m` and hit Enter to put it into mode-selection mode.  Select I2C (option `5` in the current firmware).  It will ask you about the bus speed: select as required for your breakout.
* Type `W` and hit enter to configure the power.  Select a value appropriate for your board.
* Type `P` to enable pull-up resistors.
* Finally, type `scan` and hit Enter.  This will scan for all possible devices, and return the (W)rite and (R)ead addresses for it.
* The BP5 has specific shorthand for different operations:
  * `[` and `]` start and stop bus operations, respectively
  * Type numbers in decimal (`255`), hex (`0xFF`), or binary (`0b11111111`) format to set up a read/write address or select registers.  Values should be separated by a space.
  * `r` is the command to read one byte.  Either repeat it for reading multiple bytes, or define the number after a colon, e.g. `r:8` reads eight bytes.

For example, the MCP9808 breakout from Adafruit has Write address of 0x30, and a read address of 0x31.  Register `0x06` contains a 16-bit device ID, with a default value of `0x0054`.

Read this register using:
* `[ 0xA4 0x06 ][ 0xA5 r:2]`
* This command Starts (`[`) the bus, writes to the Write address (`0xA4`) of the device, and sets the register pointer to the ID register (`0x06`) before stopping the bus (`]`).
* Then it Starts the bus (`[`), writes to the Read address (`0xA5`), and reads two bytes (`r:2`) before stopping the bus (`]`).

The output looks like the following:
```
I2C START
TX: 0x30 ACK 0x06 ACK 
I2C STOP
I2C START
TX: 0x31 ACK 
RX: 0x00 ACK 0x54 NACK 
I2C STOP
```

It shows you all of the I2C transactions, and then it shows the two read bytes in the line starting with `RX`.  As you can see, `0x00` and `0x54` match the ID register value in the datasheet of `0x0054`.
