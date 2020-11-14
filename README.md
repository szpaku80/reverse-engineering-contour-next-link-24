#  Reversing Engineering the Contour Next Link 2.4

[![Gitter chat](https://badges.gitter.im/medtronic-flash/Lobby.png)](https://gitter.im/medtronic-flash/Lobby)

This repository is part of a reverse engineering project to understand the communication process between Medtronic 630G / 640G / 670G pumps and Contour Next Link 2.4 blood glucose meter and Medtronic compatible equipment. It includes my analysis of CNL connections and test point leads. This knowledge, combined with Pazaan's past achievements, should drive the project forward.

TOC
===
  * [Current Project Status](#current-project-status)
  * [What We Currently Know](#what-we-currently-know)
  * [Hardware Overview, what I have discovered](#hardware-overview-what-i-have-discovered)
    * [Front](#front)
    * [Back](#back)
  * [Reverse engineering](#Reverse-engineering)
    * [Attempting to Dump the Firmware](#attempting-to-dump-the-firmware)

Current Project Status
======================
Steps currently taken in the reversing project:
 * Dissasemble CNL24 and identify chipsets onboard
 * Attempt to read the Firmware of the Ti cc2430 ZigBee radio
   *  Since then, no significant success has been achieved

This repository will guide you through the next steps:
  * Attempt to read flash memory from MX25L1606-8006E_DS_EN
  * Microcontroller reading via dabug connector
    * Pending hardware delivery
	

What We Currently Know
======================
The OTA protocol layer is 802.15.4 spec, and pretty sure its ZigBee. A good portion of the protocol is already discovered by Pazaan on his repo at https://github.com/pazaan/decoding-contour-next-link. The current issue at hand, is that the CNL24's USB layer appears to block writing actions, and only supports reading. So with a CNL24 alone, we can read Medtronic CGM and pump settings over USB, but if we want to loop with the pumps, we have to get direct over the air access.

We need to figure out how the AES key is generated by the firmware to figure out how to connect our own custom radios to the pump. Unfortunately, the possibility of remotely changing the base level has not been discovered. It probably doesn't exist. As a result, Medtronic 630G / 640G pumps may not be suitable for OpenAPS solutions

Hardware Overview, what I have discovered
=================

## Front

Nothing super interesting up front. Mostly just the BG Meter bits.


The main integrated circuit is the Renesas PD70F3769 microcontroller. Package: 100-pin plastic LQFP package.
 The discussed microcontroller has :
 * 512 KB flash memory
 * 40 KB RAM
 * 64 MB Logical space 
 * 13 MB External memory area
 * Max. frequency: 20 MHz
 
 https://www.renesas.com/us/en/doc/products/mpumcu/doc/v850/r01uh0001ej0400_v850esjx3l.pdf
 
 


**Unknown circuits**

On the Top side, there are two more unknown ICs
 * 1?
 * 2?
Their function is yet unknown.
! ZDJĘCIE

**meter measuring system**

This is probably the system used to measure the glucose from the sample on the test strip. We can omit it because it is not involved in connecting the CNL with the pump.

TOSHIBA
T5DBO0
1624 HUL
181961



![Front](https://github.com/applehat/contour-next-link-24-teardown/raw/master/front.jpg)

![Board Top/Front](https://github.com/applehat/contour-next-link-24-teardown/raw/master/board_top.jpg)



## Back side

**Ti SoC CC2430 ZigBee Radio**

Na tej stronie znajduje się uład radiowy CC2430-F128. 

Jest to układ RF firmy texas instrument, RF transceiver with an industry-standard enhanced 8051 MCU, 128 KB flash memory, 8 KB RAM. 
Z doświadczenia wiem, że 8051 nieobsługuje wszytkich standardowych funkcji, dlatego dobrze się wczytać dokumentację i erate.
Chip jest ustawiany i programowany przy każdym uruchomieniu. Nie przechowuje w sobie kodu programu. Dlatego musimy się skupić na pamieci MX25L i mikrokontrolerze PD70F3769.



http://www.ti.com/lit/ds/symlink/cc2430.pdf

CC2430-F128
5CW01HG
1549


**Flash Memory**
 
 Mikrokontroler PD70F3769  współpracuje z pamięcią flash MX25L1606-8006E_DS_EN
 This is 16M-BIT CMOS SERIAL FLASH
http://www.zlgmcu.com/mxic/pdf/NOR_Flash_c/MX25L1606-8006E_DS_EN.pdf

!ZDJĘCIE


**Układ NCP372**
Positive and Negative overvoltage protection controller.  


!zdjęcie




![Back](https://github.com/applehat/contour-next-link-24-teardown/raw/master/back.jpg)

![Back Naked](https://github.com/applehat/contour-next-link-24-teardown/raw/master/back-without-lipo-or-shielding.jpg)

![Board Bottom/Back](https://github.com/applehat/contour-next-link-24-teardown/raw/master/board_bottom.jpg)

![Debug Pins](https://github.com/applehat/contour-next-link-24-teardown/raw/master/debug-pins.jpg)

Reverse engineering
=========
Having access to specialized equipment, I desoldered the PD70F3769 and CC2430 chips. I analyzed the connections of the systems on the motherboard. I was able to create a CNL schematic diagram.
On the motherboard are placed test points. I was able to tag most of them. We can distinguish pins: flash memory bus, connections between the processor and the radio. The most important is the debug port.
With it we can connect to the PD70F3769 control unit.

!zdjęcie

I include the diagram and the TinyCAD library files.

**what to do next**

I hope that this documentation will be used further to develop a replacement device connecting the pump with the phone. This creates a wireless pump status data connector that can send information to Nightscout without a CNL connection via the USB port.






## Attempting to Dump the Firmware



