
# Neutrogena Mask Activator Hack
[Original post](https://deathandthepenguinblog.wordpress.com/2018/01/03/hacking-the-neutrogena-visible-light-therapy-system-to-get-99-lives/) by [edrosten](https://deathandthepenguinblog.wordpress.com/author/edrosten/). Absolutely amazing read!

[Translated to raspberry pi](https://github.com/msandres13/neuprog) by [msandres13](https://github.com/msandres13) for 3.3v, no need to translate from 5v as with Arduino.
##

This hack resets the [ATMLH640]([http://ww1.microchip.com/downloads/en/DeviceDoc/atmel-5193-seeprom-at93c46d-datasheet.pdf](http://ww1.microchip.com/downloads/en/DeviceDoc/atmel-5193-seeprom-at93c46d-datasheet.pdf)) EEPROM in the mask activator with a bit bang, replacing the current count with "30".

### How do I do it?
1. Wire the mask's pins 1, 2, 3, 4 (left side) to a breadboard or adaptor. See [datasheet](http://ww1.microchip.com/downloads/en/DeviceDoc/atmel-5193-seeprom-at93c46d-datasheet.pdf) for pinout. Wire positive and negative terminals to +/- of capacitor C4.
![Wiring](https://github.com/brianmgr/NeutrogenaMask/blob/master/PCB.jpg)
![Adaptor](https://github.com/brianmgr/NeutrogenaMask/blob/master/BreadboardAdaptor.jpg)
2. Connect these pins as follows to the pi:
	1. CS = 5
	2. CK = 21
	3. DI/MOSI = 20
	4. DO/MISO = 19
	5. C4+ = 3v3
	6. C4- = GND
3. On your pi, install pigpio: `sudo apt-get install pigpio`
4. Compile the .cpp file: `sudo gcc -Wall -pthread -lpigpio neuprog.cpp -o neuprog`
5. Run the compiled binary: `sudo ./neuprog`

### How do I know if it worked?
After the bit bang has been completed, successful output should look *something* like this:

```
sudo ./neuprog
Simple bitbang SPI EEPROM tool for AT93C64D
Setup done...

Pre-write EEPROM dump:
0x00: aa aa aa aa aa aa aa aa aa aa ff ff ff ff ff ff
0x10: aa aa aa aa aa aa aa aa aa aa ff ff ff ff ff ff
0x20: aa aa aa aa aa aa aa aa aa aa ff ff ff ff ff ff
0x30: aa ff aa aa aa aa aa aa aa aa ff ff ff ff ff ff
0x40: aa aa aa aa aa aa aa aa aa aa ff ff ff ff ff ff
0x50: 31 ff ff ff ff 31 ff ff ff ff ff ff ff ff ff ff
0x60: 95 ff ff ff 34 ff ff ff ff ff ff ff ff ff ff ff
0x70: ff ff ff ff ff ff ff ff ff ff ff ff ff ff ff ff
 >>> 0x55,0x50 == 0x31 0x31

Writing 30 uses (0x31,0x31) to EEPROM:
 >>> Wrote 0x55,0x50 => 0x31 0x31

Post-write EEPROM dump:
0x00: aa aa aa aa aa aa aa aa aa aa ff ff ff ff ff ff
0x10: aa aa aa aa aa aa aa aa aa aa ff ff ff ff ff ff
0x20: aa aa aa aa aa aa aa aa aa aa ff ff ff ff ff ff
0x30: aa ff aa aa aa aa aa aa aa aa ff ff ff ff ff ff
0x40: aa aa aa aa aa aa aa aa aa aa ff ff ff ff ff ff
0x50: 31 ff ff ff ff 31 ff ff ff ff ff ff ff ff ff ff
0x60: 95 ff ff ff 34 ff ff ff ff ff ff ff ff ff ff ff
0x70: ff ff ff ff ff ff ff ff ff ff ff ff ff ff ff ff
 >>> 0x55,0x50 == 0x31 0x31

Finished
```
    
### Finishing up
After you've gotten the successful write confirmation, unplug the data wires, then the power wires.

**Important:** Wait a bit for the capacitors to drain on their own before continuing. This could take 20 seconds or more.

*Why, you ask?*
Right before totally drained, a new count is written to the EEPROM. If you discharge the capacitor early, the write will not happen. If you plug 3v3 power back in too early, the old value may still be shown.

After you've gotten a cup of coffee, connect the power again. You should see '- -' toward the top of the LCD. After a couple seconds, that should disappear and a '30' will show up.


### What problems might I run into?
If output is all zeros, it's likely that one of the following happened:
- A wire isn't connected
- The wiring is incorrect
- There's a solder bridge on the IC
- The IC was damaged by the soldering iron
- If you're unable to run the compiled binary, make sure you have the correct user permissions or run with sudo.
