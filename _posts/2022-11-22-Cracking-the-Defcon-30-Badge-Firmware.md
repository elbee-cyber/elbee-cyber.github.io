---
layout: post
title: Cracking the DEFCON 30 Badge Firmware
---

**2022-11-22**
Physical firmware extraction and reverse engineering of the DEFCON 30 badge.

# Table of contents
1. [Backstory](#backstory)
2. [Firmware Extraction](#extraction)
3. [Analysis and Finding Check](#analysis)


## Cracking the DEFCON 30 Badge Firmware

> **_NOTE:_**  This article represents an area or subject that I am activley learning and is meant for documentational/educational/entertainment purposes. It should not be heeded as professional advice. Please notify me of errors using any of my socials below.

Before beginning this post there are some important things to note. At the time of this post, I am very inexperienced in the world of hardware, IoT, firmware extraction and firmware analysis. Because of this, the majority of my process, including the use of the physical equipment used to interact with the flash chip, was done via trial and error. This is a very dirty article by someone who barley knows what they're doing, but I think it is still important to post this to demonstrate the learning process.

<a name="backstory"></a>
### Backstory and Objective 
Once upon a time I was very excited to fly to Las Vegas and attend <a href="https://defcon.org/html/defcon-30/dc-30-index.html">DEFCON 30</a> the annual go-to security conference. This was going to be my first conference and I had purchased pre-registration tickets, which was a new system that DEFCON was using to allow attendees to guaranttee a badge and semi-skip LINECON. However, because of a personal emergency that appeared last minute, I was unable to attend and I gifted my pre-registration to my good friend and mentor <a href="https://ctftime.org/user/3509">playoff-rondo</a>. Later on when I was catching up with him he gave me his badge. DEFCON badges usually have some sort of challenge on them and this year's badge was some sort of piano keyboard. <a href="https://github.com/Kybr-git/DC30-Badge-Challenge-Writeup/blob/main/README.md">This is the writeup made by the attendee that solved the badge challenge and won a black badge</a>, as you can see it is meant to be solved in part by interacting with other attendees who have a different variant of the badge. This is obviously not possible for me, so my goal was to reverse engineer the badge and find what key combo needs to be pressed in order to win the first part of the challenge.

<a name="extraction"></a>
# Part 1: Extracting Firmware from SPI Flash
I began by using a magnifying glass, pen and paper to identify all the visible chips on the PCB. 
<br>
<p align="center">
  <img src="/assets/2022-11-22/Screenshot_1.png" />
</p>
<br>
The two important chips to note are the <a href="https://www.winbond.com/hq/product/code-storage-flash-memory/serial-nor-flash/?__locale=en&partNo=W25Q16JV">Winbond W25Q16JV</a> (flash chip) and the <a href="https://thepihut.com/products/raspberry-pi-rp2040-microcontroller">RP2 B2</a> (microcontroller). The flash memory chip will be used to grab the firmware and will be the target chip to physically extract. The microcontroller will be used to determine the architecture, conventions and other information that will be useful when analyzing the firmware.

There are tools available to extract firmware from a surface-mounted chip without having to actually remove the component, however I will be using a <a href="https://www.aliexpress.us/item/2251832631316605.html?spm=a2g0o.ppclist.product.2.42fduQgFuQgFn0&pdp_npi=2%40dis%21USD%21US%20%248.25%21%248.00%21%21%21%21%21%402101c84a16691420847026788ea9a4%2112000018677635870%21btf&_t=pvid%3A91be51b8-068b-46f5-ad65-2474e20aa1d4&afTraceInfo=32817631357__pc__pcBridgePPC__xxxxxx__1669142084&gatewayAdapt=glo2usa&_randl_shipto=US">TL866II+ universal programmer</a>, so I will have to remove the chip from the PCB, feed the chip to the device using the appropiate adapter and connect the programmer to my VM for extraction. A heat gun would be ideal for removing a small surface mounted chip like this without damaging the PCB, however I do not have access to such equipment, so I use a soldering iron and tweezers. I heat up each joint of the chip with the iron and lift the leg up with tweezers and a magnifying glass. After I have detatched the chip from the board, I go around with the iron again and a desoldering pump in an attempt to clean up as much excess solder as possible from each leg. Then I lock the flash chip in the corresponding adapter and insert it into the TL866II+.
<br>
<p align="center">
  <img src="/assets/2022-11-22/Screenshot_2.png" />
</p>
<br>
Unfortunatelly, the whole ordeal was pretty messy and resulted in a bit of copper from the PCB being destroyed.
<br>
<p align="center">
  <img src="/assets/2022-11-22/Screenshot_3.png" />
</p>
<br>
In my ubuntu machine I use <a href="https://gitlab.com/DavidGriffith/minipro/">the minipro program</a> to interact with the chip programmer. The following command writes the firmware to a file. 
```
minipro -p "W25Q16JV@SOIC8" -r flash.bin
```

I can verify that this is the firmware by running `strings` and examining the output.
<br>
<p align="center">
  <img src="/assets/2022-11-22/Screenshot_4.png" />
</p>
<br>

<a name="analysis"></a>
# Part 2: Analyzing Firmware and Finding the Comparison Function
The MCU is a rpi2040 which makes use of the <a href="https://en.wikipedia.org/wiki/Execute_in_place">"execute in place"</a> feature. This is a feature that improves performance by enabling execution directly in flash storage rather than execution in the MCU's limited memory area. It is important that we keep this in mind before starting to analyze the firmware. Firmware has no entry point and instead has a defined address at which program execution begins at. We can find this base address by viewing the <a href="https://datasheets.raspberrypi.com/rp2040/rp2040-datasheet.pdf">rpi2040 datasheet</a> and navigating to the "Address Map" page from the table of contents. The base address will be the XIP address (execute in place). We need to specify this address as the base address in whatever we use to analyze the firmware to load the correct segment.
<br>
<p align="center">
  <img src="/assets/2022-11-22/Screenshot_5.png" />
</p>
<br>

Using Binaryninja I can specify the base address when creating a new project.
<br>
<p align="center">
  <img src="/assets/2022-11-22/Screenshot_6.png" />
</p>
<br>

I start to try and identify useful symbols and organize the code by searching through strings and going to their code references.
<p align="center">
  <img src="/assets/2022-11-22/Screenshot_7.png" />
</p>
<br>

Some areas have been incorrectly loaded as symbols so I undefine them.
<p align="center">
  <img src="/assets/2022-11-22/Screenshot_8.png" />
</p>
<br>
<p align="center">
  <img src="/assets/2022-11-22/Screenshot_9.png" />
</p>
<br>





### Sources
- https://research.kudelskisecurity.com/2018/09/25/analyzing-arm-cortex-based-mcu-firmwares-using-binary-ninja/
- https://www.winbond.com/hq/support/documentation/levelOne.jsp?__locale=en&DocNo=DA00-W25Q16JV.1
- https://datasheets.raspberrypi.com/rp2040/rp2040-datasheet.pdf
