# Game Boy MBC5 Cartridge

This is my design of a flashable MBC5-based cartridge for the Game Boy. The MBC5 mapper greatly augments the memory access of the Game Boy for larger games. Most of the games that came out in the last years of the Game Boy and Game Boy Color used the MBC5 mapper.

This circuit board should cover most, if not all, MBC5 games. The features are as follows:

- Able to make games up to 32 Mb in size, that use up to 1024 Kb of RAM
- The option to add battery backup to the cartridge *without* the need of the original battery management ICs - perfect for MBC5 donors that didn't have batteries in them
- Fully compatible with the <a href="https://www.gbxcart.com/">GBxCart RW</a> so you can transfer games and save files to and from the board
- An option for making a multicart with completely separate areas of RAM between games that is triggered by resetting the console or power cycling

[image of board scan]
[image of assembled board]

All gerbers and source files can be found in this repo, as this project is fully open source. Technical documentation of the board can be found in the Technical folder.

## Disclaimer

I am not responsible for any damage you do to your self or your property. I do not guarantee design compatibility. You may encounter issues with certain games! Attempt this project at your own risk.

## Board Characteristics and Order Information

The zipped folder contains all the gerber files for this board. The following options **must** be chosen when ordering boards for yourself.

- Thickness: 0.8mm
- Surface Finish: ENIG
- Gold Fingers: Yes, 30° chamfer

**I sell this board on Etsy, so you don't have to buy multiples from board fabricators:**

[link]

You can alternatively use the zipped folder at any board fabricator you like. You may also buy the board from PCBWay using this link (disclosure: I receive 10% of the sale value to go twoards future PCB orders of my own):

[link]

The board is also listed on OshPark as well. **Be sure to get them in 0.8mm thickness if you order from here.**

[link]

## Board Configurations

The board comes with five sets of jumper pads for solder bridges. SJ1, SJ2, SJ3, and SJ4 require you to solder bridge the middle pad either to the left or right pads. SJ7 is configured by either leaving the pads unsoldered or bridging them with solder. Here are the situations where you need to add solder bridges.

### SRAM Size Selection (SJ1 and SJ2)

These jumpers are located underneath the MBC5 chip, and labeled "64K" and "256K/1M". Solder them to configure the amount of RAM your cart uses. You must configure these pads for every game you make, unless you do not need RAM. <a href="https://catskull.net/gb-rom-database/">You can find a list of games here with their respective RAM sizes.</a>

- If the game you're making uses 256Kb or more of SRAM, then solder the middle pads of the two jumper sets to the right pads.
- If the game you're making uses 64Kb of SRAM, then solder the middle pads of the two jumper sets to the left pads.
- If you are making a multicart, solder the pads according to the larger size of the SRAM between the two games (if one game uses 256Kb and the other uses 64Kb, set the jumpers to 256Kb). The max size of SRAM in multicart mode is 512Kb per game.
- SJ1 and SJ2 must be soldered in the same direction.

Note that you can make games that only require 64Kb of RAM and still use a 256Kb or 1Mb SRAM chip. You still need to configure the jumpers to the 64Kb setting, though.

**I recommend using 64Kb or 256Kb RAM chips, unless you need 1Mb for the game you are making or are making a multicart (you need to use 1Mb RAM for multicarts).**

### Multicart Jumpers (SJ3 and SJ4)

Solder the middle pads of SJ3 and SJ4 to the left (towards "SINGLE") for single games, and solder them to the right (towards "MULTI") for multicarts. You must have these pads soldered in one direction or the other for every game you make!

### Multicart Programming Jumper (SJ7)

When you program a multicart, you must append the second game ROM file to the first game ROM file, and *then* program with the GBxCart. It must be programmed in one step. When you are program your multicart, SJ7 must be **UNSOLDERED** to program correctly. After your multicart has been programmed, bridge SJ7 with solder to enable the multicart functionality.

## Bill of Materials (BOM)

Your parts list will vary depending on the game you are trying to make, and what chips you have for the battery management (if any). Note that C8 - C10 footprints are only included for edge cases that may require them; you can ignore them unless you run into issues.

Please carefully review the parts you need for the board you are trying to make. Do not add any parts to your build that don't appear in the column for the game you are making. This means you *cannot* populate every component on the board at the same time.

| Reference Designators | Value/Part Number             | Package          | Description        | No save carts | Save carts with U4 | Save carts without U4 | Source                                           |
| --------------------- | ----------------------------- | ---------------- | ------------------ | ------------- | ------------------ | --------------------- | ------------------------------------------------ |
| B1                    | CR2025                        | CR2025           | Backup Battery     |               | X                  | X                     | [https://mou.sr/3PLccol](https://mou.sr/3PLccol) |
| C1                    | 0.1uF                         | 0603             | Capacitor (MLCC)   | X             | X                  | X                     | [https://mou.sr/3ENc15O](https://mou.sr/3ENc15O) |
| C2                    | 0.1uF                         | 0603             | Capacitor (MLCC)   | X             | X                  | X                     | [https://mou.sr/3ENc15O](https://mou.sr/3ENc15O) |
| C3                    | 10uF                          | 0603             | Capacitor (MLCC)   |               | X                  | X                     | [https://mou.sr/3mZtSkF](https://mou.sr/3mZtSkF) |
| C4                    | 0.1uF                         | 0603             | Capacitor (MLCC)   |               | X                  | X                     | [https://mou.sr/3ENc15O](https://mou.sr/3ENc15O) |
| C6                    | 0.1uF                         | 0603             | Capacitor (MLCC)   |               | X                  | X                     | [https://mou.sr/3ENc15O](https://mou.sr/3ENc15O) |
| C7                    | 0.1uF                         | 0603             | Capacitor (MLCC)   |               |                    | X                     | [https://mou.sr/3ENc15O](https://mou.sr/3ENc15O) |
| C8                    | 0.001uF                       | 0603             | Capacitor (MLCC)   |               |                    |                       | [https://mou.sr/3PpAXoM](https://mou.sr/3PpAXoM) |
| C9                    | 0.001uF                       | 0603             | Capacitor (MLCC)   |               |                    |                       | [https://mou.sr/3PpAXoM](https://mou.sr/3PpAXoM) |
| C10                   | 0.001uF                       | 0603             | Capacitor (MLCC)   |               |                    |                       | [https://mou.sr/3PpAXoM](https://mou.sr/3PpAXoM) |
| Q1                    | Si2301CDS                     | SOT-23           | P-Channel FET      |               |                    | X                     | [https://mou.sr/3LvBdSb](https://mou.sr/3LvBdSb) |
| Q2                    | MMBT3904                      | SOT-23           | NPN BJT            |               |                    | X                     | [https://mou.sr/3Rv7yfA](https://mou.sr/3Rv7yfA) |
| R1                    | 10k                           | 0603             | Resistor           |               | X                  | X                     | [https://mou.sr/3riR7IH](https://mou.sr/3riR7IH) |
| R2                    | 10k                           | 0603             | Resistor           | X             | X                  | X                     | [https://mou.sr/3riR7IH](https://mou.sr/3riR7IH) |
| R3                    | 10k                           | 0603             | Resistor           |               |                    | X                     | [https://mou.sr/3riR7IH](https://mou.sr/3riR7IH) |
| R4                    | 10k                           | 0603             | Resistor           |               |                    | X                     | [https://mou.sr/3riR7IH](https://mou.sr/3riR7IH) |
| U1                    | 29F016, 29F032, 29F033        | TSOP-48, TSOP-40 | Flash EEPROM       | X             | X                  | X                     | AliExpress or eBay                               |
| U2                    | MBC5                          | QFP-32           | MBC5 Mapper        | X             | X                  | X                     | Donor MBC5 Game Boy cartridge                    |
| U3                    | AS6C6264, AS6C62256, AS6C1008 | SOP-28, SOP-32   | SRAM               |               | X                  | X                     | [https://mou.sr/3ZxG6jd](https://mou.sr/3ZxG6jd) |
| U4                    | MM1134, BA6735                | SOIC-8           | Battery Management |               | X                  |                       | Donor Game Boy cartridge                         |
| U5                    | TPS3840DL42                   | SOT-23-5         | Supervisory IC     |               |                    | X                     | [https://mou.sr/46lxKxA](https://mou.sr/46lxKxA) |
| U6                    | LM66100DCKR                   | SC70-6           | Ideal Diode        |               |                    | X                     | [https://mou.sr/450kfSE](https://mou.sr/450kfSE) |
| U7                    | 74AHC1G79                     | SC-74A-5         | Flip Flop          |               | If multicart       | If multicart          | https://mou.sr/3PPoGve                           |

## Things to Remember

- The footprint for the EEPROM is specifically for 29F016 - it has 48 pins. However, 29F032 and 29F033 are only 40 pin devices. They still work fine on the board though - place them in the center of the footprint, and leave the outer two pins on each corner empty.
- The 29F016, 29F032, and 29F033 have been known to occasionally be defective upon arrival. They're usually only available from AliExpress.
- For battery management, use either U4 *or* U5 and U6 and supporting components. **Do not** use U4, U5, and U6 all on one board. They will interfere with each other.
- There is not compatibility with MM1026 or BA6129 battery management ICs on this cartridge. You cannot use these for U4. This is not a huge deal, because as far as I can tell from the <a href="https://gbhwdb.gekkio.fi/cartridges/mbc5.html">gbhwdb</a>, no MBC5 games ever used these chips in the first place, so any MBC5 donors with batteries should have the MM1134/BA6735 as it is.
- Kb is kilo**bits** and Mb is mega**bits**. Sometimes you will find game ROM and RAM sizes defined in terms of KB or kilo**bytes** and MB or mega**bytes**. You can convert Kb and Mb to KB and MB by dividing Kb or Mb by 8. For example, 256 Kb = 32 KB.
- You only need to provide ROM and RAM chips that have at least *or greater* the size of the game you are trying to make. That means you can use a 256Kb SRAM chip for a game that only requires 64Kb!
- If you are using a 64Kb or 256Kb RAM chip, it will be 28 pins. The footprint for the SRAM is 32 pins to accomodate the 1Mb RAM chips. If you are using a 28-pin device, then populate them so that the package sits at the bottom of the footprint and leave the top four pins (two on either side) empty.

## Revision History

### v1.0
- Release revision

## Resources and Acknowledgements

- <a href="http://www.devrs.com/gb/files/hardware.html">Jeff Frohwein's GameBoy Tech Page</a>
- <a href="https://gbhwdb.gekkio.fi/">Game Boy Hardware Database</a>
- <a href="https://catskull.net/gb-rom-database/">Nintendo Gameboy Game List</a>
- <a href="https://www.gbxcart.com/">insideGadgets discord server for GBxCart RW compatibility requirements</a>
- <a href="https://www.ti.com/lit/ds/symlink/lm66100.pdf?HQS=dis-dk-null-digikeymode-dsf-pf-null-wwe&ts=1694502124931&ref_url=https%253A%252F%252Fwww.ti.com%252Fgeneral%252Fdocs%252Fsuppproductinfo.tsp%253FdistId%253D10%2526gotoUrl%253Dhttps%253A%252F%252Fwww.ti.com%252Flit%252Fgpn%252Flm66100">LM66100 Datasheet</a>
- <a href="https://www.alldatasheet.com/datasheet-pdf/pdf/99104/MITSUBISHI/MM1026.html">System Reset IC Datasheet</a>
- Board outline from <a href="https://tinkerer.us/projects/homebrew-gameboy-cartridge.html">Dillon Nichols's Homebrew Gameboy Cartridge project</a>
- Thank you to <a href="https://github.com/Gekkio">gekkio</a> for their deep Game Boy knowledge resources, and for collaboration in demystifying some of the design choices on Game Boy cartridges
- Thanks to the awesome members of the <a href="https://moddedgameboy.club/">Modded Gameboy Club</a> for their feedback and support during the entire project development

## License

<a rel="license" href="http://creativecommons.org/licenses/by-sa/4.0/"><img alt="Creative Commons License" style="border-width:0" src="https://i.creativecommons.org/l/by-sa/4.0/80x15.png" /></a><br />This work is licensed under a <a rel="license" href="http://creativecommons.org/licenses/by-sa/4.0/">Creative Commons Attribution-ShareAlike 4.0 International License</a>. You are able to copy and redistribute the material in any medium or format, as well as remix, transform, or build upon the material for any purpose (even commercial) - but you **must** give appropriate credit, provide a link to the license, and indicate if any changes were made.

©MouseBiteLabs 2023
