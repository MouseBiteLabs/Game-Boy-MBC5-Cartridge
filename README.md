# Game Boy MBC5 Cartridge

This is my design of a flashable MBC5-based cartridge for the Game Boy. The MBC5 mapper greatly augments the memory access of the Game Boy for larger games. Most of the games that came out in the last years of the Game Boy and Game Boy Color used the MBC5 mapper.

This circuit board should cover most, if not all, MBC5 games. The features are as follows:

- Able to make games up to 32 Mb in size, that use up to 1024 Kb of RAM
- Compatibility with all four of the major Game Boy battery management ICs - MM1026, MM1134, BA6129, and BA6735
- The option to add battery backup to the cartridge *without* the need of the original battery management ICs - perfect for MBC5 donors that didn't have batteries in them
- Fully compatible with the <a href="https://www.gbxcart.com/">GBxCart RW</a> so you can transfer games and save files to and from the board

[image of board scan]
[image of assembled board]

All gerbers and source files can be found in this repo, as this project is fully open source. Technical documentation of the board can be found in the Technical folder.

## Disclaimer

I am not responsible for any damage you do to your self or your property. I do not guarantee design compatibility. You may encounter issues with certain games! Attempt this project at your own risk.

If you are using this board to make games other than for personal use, **you must have permission from the originator to use and distribute any ROM images or other related material.** You are responsible for making sure you adhere to any license requirements. DO NOT use my circuit boards for profiting from stolen work - this especially includes homebrew content, ROM hacks, and using fan-made labels without permission from the originator.

**Please note that version 1.3 is technically untested, however, the only consequential change is an additional 0.25 mm on the bottom of the board edge for better fitment, so I don't expect issues.**

## Board Characteristics and Order Information

The zipped folder contains all the gerber files for this board. The following options **must** be chosen when ordering boards for yourself.

- Thickness: 0.8mm
- Surface Finish: ENIG
- Gold Fingers: Yes, 30° chamfer

**Currently not selling on Etsy, but will in the future. Stay tuned.**

<a href="https://www.etsy.com/shop/MouseBiteLabs"><img src="https://github-production-user-asset-6210df.s3.amazonaws.com/97127539/239718536-5c9aefe3-0628-4434-b8d8-55ff80ac3bbc.png" alt="PCB from Etsy" /></a> 

You can use the zipped folder at any board fabricator you like. You may also buy the board from PCBWay using this link (disclosure: I receive 10% of the sale value to go towards future PCB orders of my own):

[link]

<a href="https://oshpark.com/shared_projects/">The board is also listed on OSH Park as well.</a> **Be sure to get them in 0.8mm thickness if you order from here.**

## Required Equipment

The EEPROMs on the board needs to be programmed somehow. I recommend using the GBxCart, as mentioned. These boards are fully compatible with it, and it makes reflashing games extremely easy using <a href="https://github.com/lesserkuma/FlashGBX">lesserkuma's FlashGBX software</a>.

Alternatively, you can buy an EEPROM programmer with a TSOP adapter. The downside to this method is that you have to desolder the chip every time you want to program it. The <a href="https://flashcatusb.com/">FlashcatUSB</a> is one popular option in retro spaces.

## Battery Safety

When assembling a board, I recommend populating all the parts *except* the battery, and getting it to run initially without it. This is to make it easier to fix any solder connections that might need fixing, without having to worry about getting the battery hot. And if you need to rework anything near the battery after you've put it on the board, be safe and remove it before putting a hot soldering iron next to it.

And this should go without saying, but if you're assembling these boards with a hot plate or hot air, *do not* solder the battery on this way. You should use an iron, and keep heat off of the battery as much as possible. 

(Also check polarity!)

## Board Configurations

The board comes with five sets of jumper pads for solder bridges. SJ1, SJ2, SJ3, and SJ4 require you to solder bridge the middle pad either to the left or right pads. SJ7 is configured by either leaving the pads unsoldered or bridging them with solder. Here are the situations where you need to add solder bridges.

### SRAM Size Selection (SJ1 and SJ2, or SW1)

These jumpers are located underneath the MBC5 chip, and labeled "64K" and "256K/1M". Solder them to configure the amount of RAM your cart uses. You must configure these pads for every game you make, unless you do not need RAM. <a href="https://catskull.net/gb-rom-database/">You can find a list of games here with their respective RAM sizes.</a>

- If the game you're making uses 256Kb or more of SRAM, then solder the middle pads of the two jumper sets to the right pads.
- If the game you're making uses 64Kb of SRAM, then solder the middle pads of the two jumper sets to the left pads.
- SJ1 and SJ2 must be soldered in the same direction.
- The footprint of these selection pads should allow for a DPDT switch, part number CAS-220A1, to be placed on these pads instead of having to bridge the pads with solder.

[image of solder jumpers]

Note that you can make games that only require 64Kb of RAM and still use a 256Kb or 1Mb SRAM chip. You still need to configure the jumpers to the 64Kb setting, though.

**I recommend using 64Kb or 256Kb RAM chips, unless you absolutely need 1Mb for the game you are making.**

### Using an MM1134 or BA6735 for U4 (SJ3)

Bridge the jumper SJ3 if you have either an MM1134 or BA6735 for U4, specifically. Any other battery management IC must leave SJ3 unsoldered.

## Test Points and Final Checkout

On the back of the board are five test points. Here's where they are connected:

- TP1: SRAM supply voltage
- TP2: Battery voltage (after R1)
- TP3: Battery voltage (positive terminal of battery)
- TP4: Ground
- TP5: VCC input voltage

After you assemble your game, you should measure the current out of the battery. But first, you should program it with the GBxCart, or if you programmed the EEPROM separately, put it into a Game Boy and cycle power once. Then, flip the PCB upside down on a non-conductive surface (not your leg), and set your multimeter in DC millivolts (or volts). Put the positive probe on TP3 and the negative probe on TP2. If you used a 10kΩ for R1, as indicated in the BOM, you should read a voltage in the single of millivolts. If you have something much higher, especially voltages above 10mV, then you likely have an issue or short circuit on the board somewhere.

**Note: If using the replacement battery management IC in U5, you need to power up the game at least once before battery currents will make sense.**

## Bill of Materials (BOM) - UPDATE

Your parts list will vary depending on the game you are trying to make, and what chips you have for the battery management (if any). Note that C9 - C11 footprints are only included for edge cases that may require them; you can ignore them unless you run into issues.

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

- The footprint for the EEPROM is specifically for 29F016 - it has 48 pins. However, 29F032 and 29F033 are only 40 pin devices. They still work fine on the board though - place them in the center of the footprint, and leave the outer two pins on each corner empty
- The 29F016, 29F032, and 29F033 have been known to occasionally be defective upon arrival. They're either used, or new old stock, and usually only available from AliExpress.
- The footprint for the battery can fit a CR2032, CR2025, or CR2016 with solder tabs. The only difference is the mAh capacity (larger number = longer life). If you get Panasonic tabbed batteries, you may have to trim the battery tabs to make them fit on the footprint.
  - For untabbed coin cells, you can find battery retainer adapters online, <a href="https://retrogamerepairshop.com/products/hdr-game-boy-game-battery-retainer?variant=40511013290156">like this one.</a>
- For battery management, use either U4 *or* U5 and supporting components. **Do not** use U4 and U5 simultaneously on one board. They will interfere with each other.
- Kb is kilo**bits** and Mb is mega**bits**. Sometimes you will find game ROM and RAM sizes defined in terms of KB or kilo**bytes** and MB or mega**bytes**. You can convert Kb and Mb to KB and MB by dividing Kb or Mb by 8. For example, 256 Kb = 32 KB.
- You only need to provide ROM and RAM chips that have at least *or greater* the size of the game you are trying to make. That means you can use a 256Kb SRAM chip for a game that only requires 64Kb!

## Revision History

### v1.3
- Extended cart edge down by 0.25 mm for better fitment
- Added OSHW logo and "SUPPORT ORIGINAL CREATORS!"

### v1.2
- Replaced non-donor battery management circuitry with a TPS3613-based circuit for smaller BOM and easier routing
  
### v1.1
- Moved all parts on the top down to allow for compatibility with DMG-style shells
- Rotated battery for more space
- Widen SRAM footprint for easier soldering
- Renamed some reference designators for consistency between designs
- Changed silkscreen for clarity
- Removed multicart option

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
