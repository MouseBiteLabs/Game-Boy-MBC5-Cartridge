# Technical Design Document

This write-up will serve as an attempt of explaining some of the features on this cartridge. I won't go into heavy detail about the functionality of the MBC5, but I will attempt to explain some higher level functions and the reasoning behind the circuit.

Please correct me on any information I may have gotten wrong.

## Schematic

![image](https://github.com/MouseBiteLabs/Game-Boy-MBC5-Cartridge/assets/97127539/06a5fa1e-97e7-4bd4-8fff-14995a18a9aa)

## Cart Edge Pins

Very briefly, I will categorize the different pins on the 32-pin cartridge edge connector.

- Pin 1 and pin 32 are VCC and GND, respectively.
- Pin 2 is the CLK or PHI pin. THis is not used on the MBC5 cartridge.
- Pin 3 is the /WR pin, pin 4 is the /RD pin, pin 5 is the /CS pin. These signals partially determine when data is read from the ROM or when data is read/written to the RAM.
- Pins 6 through 21 are the address pins A0 through A15.
- Pins 22 through 29 are the data pins D0 through D7.
- Pin 30 is the /RST (or reset) pin. When this pin is low, the Game Boy will cease operation.
- Pin 31 is a generally unused pin in Game Boy cartridges, but here it is connected to the EEPROM's /WE pin for compatibility with the GBxCART RW.

## MBC5 - Memory Bank Controller

The MBC5 is one of the most popular mappers for Game Boy Color cartridges, and *greatly* expands the accesible space of a Game Boy cartridge. I won't go into detail how it operates, because you can find sufficient explanations on memory mappers elsewhere, and it doesn't have any extraneous features like MBC3's real-time clock. It's a pretty straightforward device by just looking at the pinout. A high level explanation is that it is used to expand the addressable memory of a Game Boy cartridge. The RA14-RA22 and AA13-AA16 outputs are used to access higher memory banks on the ROM and RAM chips.

Further sections will provide more context for some of the other MBC5 pins and the logic behind how they are connected.

## ROM and RAM

The connections to the address and data pins here are mostly self-explanatory. However, the upper address pins are controlled by the MBC5.
- A14 through A21 on the ROM chip are controlled by the RA14 to RA21 outputs from the MBC5. On my cart, the RA22 output doesn't go anywhere. This would be used for expanding the ROM space from 4 MB (32 Mbit) to the max size of 8 MB (64 Mbit), but barely any games use this much space. The only board I could find that actually supports it is <a href="https://gbhwdb.gekkio.fi/cartridges/CGB-B82J-0/gekkio-1.html">DMG-A15-10</a>.
- A13 to A16 on the SRAM chip are controlled by the AA13 to AA16 outputs from the MBC5. This means you can address up to a whole 1 Mbit of RAM! The max SRAM size wasn't used on too many games, but it does include Game Boy Wars 3 (Advance Wars series), so it felt worthy of inclusion.

Other pins include: 

- The ROM's /CE pin is controlled by the A15 address pin and the /OE pin is controlled by the /RD pin on the cart edge (pin 4).
- The ROM's /WE pin, which was not on original cartridges, is connected to pin 31 on the cart edge for programming via the GBxCART RW as previously mentioned.
- The RAM's /OE pin is connected to the /RD pin on the cart edge (pin 4).
- The RAM's /WE pin is connected to /WR on the cart edge (pin 3).
- The RAM's /CE pin requires a bit of explanation, which will be covered in a later section.

## Battery Management and Data Retention Considerations

Ensuring current coming out of the battery is as minimal as possible while the Game Boy is turned off is crucial for long-lasting save data. The higher the current, the lower the battery life! But we need to do this without breaking compatibility with normal operation. So here are the things we need to keep in mind:

1) SRAM must have continuous power to retain save data.

2) Data on the SRAM must be accessible during normal gameplay.

3) The number of battery-powered current-consuming components should be minimized, and those that are powered must use as little current as possible.

4) Operation while the voltage is ramping down after turning off the power switch must be restricted to prevent junk data from being written to cartridge RAM.

Original MBC5 carts manage all of this with U4, which was (as far as I can tell) exclusively the MM1134 or BA6375. But with this board, you can also use battery management ICs MM1026 and BA6129, which were used on MBC1 and MBC3 carts. I'll go over the functional requirements here, and for convenience, I'll be using the MM numbers when talking about them.

### SRAM Operational Background

For most Game Boy games, there were two main sizes of SRAM chips used - 64 Kbit and 256 Kbit (further refered to as just 64K or 256K). The pinouts of these two types of chips are nearly identical: 

![image](https://github.com/MouseBiteLabs/Game-Boy-MBC1-Cartridge/assets/97127539/3d5b20d1-d4e6-4598-b218-566d6fefaa61)

The big differences are the number of address pins and the number of chip enable pins. The 64K chip has 13 address pins and two chip enables (/CE and CE2), and the 256K chip has 15 address pins and one chip enable (just /CE). The additional address pins on the 256K SRAM gives it more memory, but one of the chip enable pins had to be sacrificed to accommodate this. This is kind of important, because the chip enable pins influence two main things - they tell the chip when data is being accessed, *and* how much current the chip is drawing during standby. For 64K SRAM chips, that's pretty easy - use one chip enable pin for data access, and the other for data retention mode. But for 256K SRAM, you have to do both of those functions at once. Look at the requirements for the low "data retention current" on the AS6C6264 datasheet and the AS6C62256 datasheet:

![image](https://github.com/MouseBiteLabs/Game-Boy-MBC1-Cartridge/assets/97127539/40d23786-5e9e-480f-b8f8-28d90928e621)

For 64K, /CE needs to be at VCC, *or* CE2 needs to be at GND for the current to be minimized. For 256K, /CE needs to be at VCC, full stop (in this case, VCC is whatever voltage appears on the SRAM's VCC pin, so when power is off that's the battery voltage). That's not super convenient if the power is off - something needs to make the /CE pin go to VCC, which means whatever that something is better not be pulling a lot of current, lest we lower our battery life.

*If you use a 256 Kbit SRAM chip, but solder the jumpers on my board for 64 Kbit, the CE2 pin on the 64 Kbit RAM chip will be continuously enabled by setting it high. And for 1 Mbit SRAM, there's an extra four pins to expand the memory space, and it also restores the CE2 pin that the 64 Kbit SRAM has. For easy implementation, my boards still only use the /CE pin for data access and retention, and keeps the new CE2 pin continuously enabled.*

Games that have 64K SRAM can use the MBC5 for controlling the /CE pin for data access, and U4 (the MM chips) for keeping CE2 at GND when power was off. But for 256K SRAM, that CE2 pin isn't available anymore. As an example - for MBC1 games, if a cartridge had an MM1026 chip for battery management, the MBC1 was powered via of the battery as well, and had internal logic to keep /CE logic high when disabled. The MM1026 has a /CE output as well that will pull the output up to the battery voltage, however it *does not* have a way to allow the /CE pin to be accessed during normal gameplay. So the MBC1 needs to take care of both functions, but since it's powered from the battery when power is off, that means the battery life would be impacted. 

Luckily, the /CE pin on 256K SRAM chips can be managed completely by the MM1134. And all MBC5 games used this chip, if it had battery-backed RAM. The MM1026 wasn't used at all. Using the MM1134 removed the requirement of keeping the MBC5 powered while the cartridge was off, if using larger than 64K RAM. Handy! But on my custom carts, I add in an extra bit of circuitry to allow you to do this with just an MM1026, or completely new off-the-shelf components without the need for a donor - just in case you have an MBC5 donor that doesn't have RAM on it. But before covering that, let's talk about the other important part of the battery backup system - how to properly and safely manage shifting between console power to battery power.

### Reset IC - MM1026

When you turn the power switch off on a DMG, the voltage on the VCC supply will ramp down. However, the DMG CPU requires 5 V to operate properly. A sufficiently lower voltage supplying the CPU may cause erroneous operation on the I/O pins of the CPU, even for a brief period of time. This can corrupt data stored on the SRAM of the game cartridge, say if the CPU accidentally writes junk data to part of the memory. This is bad! 

In order to prevent this, cartridges that have battery-backed SRAM on them use the battery management chip U4 to shut down operation of the CPU on pin 30 of the cart edge - the /RST (or /RESET) line. Asserting this to GND will stop all operation on the I/O pins, preventing corruption from occuring. According to the MM1026 datasheet, the /RESET output and CS output pins will be GND whenever voltage on the VCC pin is below 4.2V.

*Please note that the MM1026 has a /RESET output pin, but this is not the same as the Game Boy CPU /RESET input, the MBC1 /RESET input, or the cart edge /RST pin 30! I will call the MM1026's pin 2 the "Reset output pin" from now on for clarity.*

![image](https://github.com/MouseBiteLabs/Game-Boy-MBC1-Cartridge/assets/97127539/c3d14c7a-4695-4fc1-92df-16f425d90c79)

It is also used to pull the memory mapper's /RESET input to GND as well. This is more important on boards that powered their MBC chips off of the battery (particularly MBC3) as it greatly reduces power consumption, but it also further prevents any junk data from being written to the SRAM during power-down.

### Reset IC - MM1134

The MM1134 is nearly identical to the MM1026, with one distinct difference that's crucial for OEM MBC5 games. There is one extra input that was previously an unconnected pin on the MM1026 - the /Y input. This input is intended to be used with a RAM /CE signal. It controls the /CS output (pin 5) on the MM1134. See the schematic and timing diagram below:

![image](https://github.com/MouseBiteLabs/Game-Boy-MBC1-Cartridge/assets/97127539/8d95eb57-2370-415c-8ff0-402ed947fdcd)

Essentially, the /CS output will follow the /Y input, *unless* VCC is below the 4.2V power-off threshold. Then, /CS will be pulled to VOUT (the combination of VCC and VBAT). Hey, this is great, because it means we can control that pesky 256K SRAM /CE pin *and* put it in a low power state when the power shuts off!

*Note: If you ground the /Y input, then the MM1134 will act exactly like an MM1026.*

### Open Collector Requirements on CPU /RESET Input

One minor, but important, note that drives a few design choices - there are actually two outputs on the MM1026/MM134 that go low when power is below 4.2 V. One of them is open collector (pin 2, the "Reset output"), and one is instead driven high by an internal transistor or pulled low by an internal pull-down (pin 3, the "CS output"). This is important because it means *you cannot safely use the CS output pin to control the Game Boy's /RESET line.*

![image](https://github.com/MouseBiteLabs/Game-Boy-MBC1-Cartridge/assets/97127539/5fb78360-aba5-40cd-b881-bf815daa9ee1)

On the DMG, the CPU's /RESET input is also connected to half of the power switch. When the switch is fully turned off, the /RESET input is pulled to GND directly.

![image](https://github.com/MouseBiteLabs/Game-Boy-MBC1-Cartridge/assets/97127539/225ab129-f5d6-421e-90c0-88530c2a48d6)

*[Image adapted from <a href="https://github.com/Gekkio/gb-schematics/blob/main/DMG-CPU-06/schematic/DMG-CPU-06.pdf">gekkio's DMG CPU board schematic</a>]*

This means that to ensure no damage is done to the battery management chip, any output connected to the Game Boy's /RESET input must be **open collector**. Open collector outputs can either float their output, or (in this case) pull it to GND - reference the "Pin 2 Reset Output" schematic above. That means pulling it to GND with the power switch won't damage anything internal to the chip, it will just safely bypass the pin's function.

But, in the case of pin 3 on the MM chips, the "CS output" pin, imagine a scenario where the power switch was turned off, but the capacitors on the board have not yet discharged down to 0V. This means there'd still be some residual voltage on the VCC net. But, the CS output pin is connected to VCC via the transistor internal to the chip, and if VCC hasn't dropped far enough, then this would still be pulled high to VCC. So if you had connected pin 3 to the /RESET input on the Game Boy, in this scenario, there would be a path from VCC on the MM's supply pin, *through* the transistor on pin 3, through the power switch on the DMG, to GND - short circuiting the capacitors on the board. This means that for a moment, the internal transistor driving pin 3 would experience a surge of current. Since the output is not designed to do that, it could cause damage to the part. So let's avoid that situation!

In conclusion, any output connected to the /RESET net on the Game Boy *must* be open collector.

*On the GBC, they added a chip that pulls the /RESET line to GND through an open collector output whenever the voltage on VCC drops below 3.5 V, instead of using the power switch to ground the input. If the DMG had this, maybe Nintendo wouldn't have needed these MM chips but could use something a bit simpler. But we must design cartridges with DMG compatibility in mind!* 

## Battery Management Implementation Options

Ok, with the background taken care of, I'll talk about how my MBC5 board manages all of this, and the different compatibility options. If you reference the schematic above, you will see three options for games that use SRAM:
1) MM1134 ("Group A" components)
2) MM1026 ("Group A" and "Group C" components)
3) No donor MM chip ("Group B" components)

### Using an MM Chip

For reference again, here is a pinout diagram of the MM1026 and MM1134.

![image](https://github.com/MouseBiteLabs/Game-Boy-MBC1-Cartridge/assets/97127539/fd0f15fb-96d5-4d6b-a1fa-3b0cd7466f58)

And here is the section of the schematic for using one of these chips ("Group A" components).

![image](https://github.com/MouseBiteLabs/Game-Boy-MBC5-Cartridge/assets/97127539/02d25858-d04c-4bd3-897d-0e26f476c40c)

#### Creating the Battery-Backed Power Supply

The VOUT pin is a combination of the VCC and VBAT pins. VCC is connected to the Game Boy's 5V supply, and VBAT is connected to the battery (through a current-limiting resistor). When VCC drops below 3.3V during power-down, the MM chip switches VOUT to be supplied by VBAT instead of VCC. Conversely, when it passes above 3.3V during power-on, the MM chip switches VOUT to be supplied by VCC instead. This guarantees constant power to the SRAM to keep save data retained at all times, assuming the battery is not dead.

#### Reset Pin Management

Because anything connected to the /RESET input pin on the cart edge must be open collector, the open collector output of the MM chip, pin 2, is connected to the /RESET cart edge pin (pin 30). The MBC's /RESET input, pin 13, is connected to the driven CS output of the MM chip, pin 3.

#### Controlling the SRAM /CE Pin (MM1134)

The RAM_/CS output of the MBC5 is connected to the /Y input (pin 7) of the MM1134, and if SJ3 is bridged to connect it to RAM_/CS_G (as it should be when using an MM1134), the /CS output (pin 5) will be connected to the SRAM /CE input. As explained earlier, the state of the /CS output of the MM1134 is the same as the /Y input, unless VCC is below 4.2V, in which case the /CS output is pulled up to the VOUT voltage to satisfy low-current data retention requirements of the SRAM.

#### Controlling the SRAM /CE Pin (MM1026)

Because the MM1026 does not have the gated /CS output functionality, and the MBC5 is not powered by the battery on my cart design, we need to add the function back with some external components, which I have labelled as "Group C" components in the schematic.

![image](https://github.com/MouseBiteLabs/Game-Boy-MBC5-Cartridge/assets/97127539/191c1c0b-0448-41c6-85b2-a94e8411c637)

As a reminder: RAM_/CS connects to the MBC5's RAM_/CS output pin, RAM_/CS_G connects to the SRAM's /CE input, and VCC_SRAM connects to the MM1026 VOUT pin.

- When the CS output is driven low (when VCC is below 4.2V), conduction between the drain and source of Q2 will be off; the RAM_/CS_G net will be pulled up to the battery-backed voltage via R7, no matter what RAM_/CS is doing.
- When the CS output is driven high (when VCC is above 4.2V), conduction between the drain and source of Q1 is allowed; RAM_/CS_G will follow the RAM_/CS output, which will allow the MBC5 to control the SRAM /CE input.

Thus, this circuit essentially adds the MM1134's gated /CS output functionality back into the circuit.

### Using Brand New Components

If you do not have one of these MM chips, as you can only typically get them from donor Game Boy cartridges and not all games use them, you can achieve the same results using commercially-available components. In order to do so, you need to populate "Group B" components which features the TPS3613 - a Texas Instruments chip that is almost identical to the MM1134, with a few minor differences.

![image](https://github.com/MouseBiteLabs/Game-Boy-MBC5-Cartridge/assets/97127539/44fd110a-8cf6-4293-ba6f-ae321d27b7c0)

Here is the timing diagram from the TPS3613 datasheet, showing the relations between the inputs and outputs of the chip.

![image](https://github.com/MouseBiteLabs/Game-Boy-MBC1-Cartridge/assets/97127539/6c6149f9-d40f-4ec4-b151-b59f4dcc069b)

#### Creating the Battery-Backed Power Supply

Like the MM chips, VOUT is created by VCC or VBAT. But instead of a hard internal threshold, the output of this pin is determined by the voltage on the SENSE pin. If the voltage on SENSE is above an internal set voltage of 1.15 V (shown as VIT in the diagram), VOUT is equivalent to VCC. When the SENSE voltage drops below VIT, and when VCC drops below VBAT, VOUT switches over to being powered by VBAT instead.

The voltage on SENSE is determined by the voltage divider made with R9 and R10. The MM chips have an internal threshold of 4.2 V for determining that the power is going off and the /RESET outputs should be asserted to protect the RAM from spurious writes. So for this design, I'll stick to that threshold.

#### Reset Pin Management

The /RESET and RESET outputs of the TPS3613 are push-pull. This is slightly inconvenient, because it means you cannot directly connect them to the /RST pin on the cart edge (pin 30), because as discussed earlier, any connection to this pin must be open collector. So, we just have to make it open collector (or in this case, open drain). Adding Q1, an N-channel MOSFET, achieves this. 

- When voltage on the SENSE pin is above the internal 1.15V threshold, the /RESET output pin is asserted high allowing the MBC5 to function, and the RESET output pin is asserted low. This will keep Q1 off and allow /RST, pin 30 on the cart edge, to float.
- When voltage on the SENSE pin drops below the internal 1.15V threshold, the /RESET output pin is asserted low turning off the MBC5, and the RESET output pin is asserted high. This will turn Q1 on, and pull pin 30 on the cart edge to GND. Note that the positive RESET output follows the voltage on the VDD pin, so once the 5V supply on the Game Boy depletes, Q1 will be off again, but since there will be no power to the MBC5, this doesn't make a difference.

#### Controlling the SRAM /CE Pin

The /CEIN input and /CEOUT output pins of the TPS3613 act exactly like the /Y input and /CS output pins of the MM1134 - /CEOUT will follow /CEIN as long as the voltage on the SENSE pin is above 1.15V. So the RAM_/CS output is wired similarly to the MM1134 implementation.

## Estimating Battery Life

You can get a very rough estimate of battery life by measuring the voltage in millivolts across the battery-series resistor R1. Just use Ohm's Law to find the current: I = V / R where V is the voltage in millivolts you measure and R is the resistance of R1 in ohms. If you use these units, the current will be in milliamps. 

*Fun fact: you can measure this voltage using TP2 and TP3 on the back of the board as your multimeter probe points.*

Then, find the milliamp-hour rating of your selected battery (preferrably from a datasheet). For example, a Renata CR2032 battery is rated for 225 mAh. Take this number and divide by the milliamps calculated above, to get a rough estimate of the number of hours the battery can supply when the console is turned off (when it is on, the Game Boy powers the cart so the battery is unused). Divide by 24 and then 365 (or just divide by 8760) and you will get a number of years.

So for an overall equation to determine the very approximate number of years the battery will survive:

Years = Resistance of R1 (ohms) * Battery capacity (mAh) / Voltage (mV) / 8760 

For an example: an MBC5 cartridge where R1 is 10 kΩ (or 10000 Ω), using a CR2025 rated for 165 mAh, and a voltage of 10 mV measured across the terminals of R1, yields 10000 * 165 / 10 / 8760 = 18.84 years of battery survival. This number can be different due to changing environmental conditions, self-discharge of the battery, and also actual capacity of the battery - discharging a battery at very small currents will increase its apparent capacity, and as the voltage decreases, the current required for data retention decreases as well.

Just backup your save data within a decade of making the cart and you'll be fine.

## A Few Extra Capacitors

I added spots for a few extra capacitors, in the event some compatibility issues arise. By default, you should ignore these components, as they were not present in most cartridges. If I find instances where they are needed, I will note them in this repo. For now, assume they are unnecessary.

![image](https://github.com/MouseBiteLabs/Game-Boy-MBC5-Cartridge/assets/97127539/6b0be647-af29-420b-b825-94c0fd3309e6)

1) C9 is a capacitor connected nearby the MBC5's /WR pin to GND. These were populated on some MBC1 cartridges, and have a value of approximately 1 nF (0.001 uF).
2) C10 is a capacitor connected nearby the MBC5's /RESET pin to GND. A spot for this capacitor was included on some MBC1 carts, but I cannot locate an instance of it actually being used when looking through the gbhwdb. I don't actually know what value this capacitor would need to be, so I am guessing 1 nF.
3) C11 is a capacitor that I have added that was not used on any original Game Boy board. It connects to the SRAM's /CE pin, bypassing to GND. I have added this in the event incompatibilities arise with newer SRAM chips that have much faster speeds than older SRAM chips. This was a problem I encountered on some of my NES cartridges, so this is here just in case.

## Resources

- <a href="http://www.devrs.com/gb/files/hardware.html">Jeff Frohwein's GameBoy Tech Page</a>
- <a href="https://gbhwdb.gekkio.fi/">Game Boy Hardware Database</a>
- <a href="https://catskull.net/gb-rom-database/">Nintendo Gameboy Game List</a>
- <a href="https://www.gbxcart.com/">insideGadgets discord server for GBxCart RW compatibility requirements</a>
- <a href="https://www.alldatasheet.com/datasheet-pdf/pdf/99104/MITSUBISHI/MM1026.html">System Reset IC Datasheet</a>
- <a href="https://www.ti.com/lit/ds/symlink/tps3613-01.pdf?HQS=dis-mous-null-mousermode-dsf-pf-null-wwe&ts=1698238885366&ref_url=https%253A%252F%252Feu.mouser.com%252F">TPS3613 Datasheet</a>
- <a href="https://www.alliancememory.com/wp-content/uploads/pdf/Alliance%20Memory_64K_AS6C6264v2.0July2017.pdf">AS6C6264 Datasheet</a>
- <a href="https://www.alliancememory.com/wp-content/uploads/pdf/AS6C62256.pdf">AS6C62256 Datasheet</a>
- <a href="https://github.com/Gekkio/gb-schematics/blob/main/DMG-CPU-06/schematic/DMG-CPU-06.pdf">Gekkio's Game Boy Schematic Resources</a>

## License
<a rel="license" href="http://creativecommons.org/licenses/by-sa/4.0/"><img alt="Creative Commons License" style="border-width:0" src="https://i.creativecommons.org/l/by-sa/4.0/80x15.png" /></a><br />This work is licensed under a <a rel="license" href="http://creativecommons.org/licenses/by-sa/4.0/">Creative Commons Attribution-ShareAlike 4.0 International License</a>. You are able to copy and redistribute the material in any medium or format, as well as remix, transform, or build upon the material for any purpose (even commercial) - but you **must** give appropriate credit, provide a link to the license, and indicate if any changes were made.

©MouseBiteLabs 2023
