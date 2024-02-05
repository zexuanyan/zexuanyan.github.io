---
layout: page
title: TTL Chip-Based VGA Driver
description: A Breadboard-based VGA Driver that supports live editing of the image displayed on the monitor.
img: assets/img/TTL_VGA_Project/VGA_Driver_Cover.jpg
importance: 1
category: Technical
---

In my freshman year, my team and I made a VGA driver out of breadboards and TTL chips for our class project. In this post, I will give you a quick rundown of what this project is about! 

Please note that this page works as a summary of the project and doesn't include all the information about this project. If you are interested in the technical details, please refer to the [report](../../assets/pdf/SP23_Honors_Lab_Final_Reports.pdf) for a detailed explanation. Credit for this project also goes to my teammates Phillip Chen (pxchen2@illinois.edu), Tony Liu (zikunl2@illinois.edu), and Hanwen Zhang (hanwenz7@illinois.edu). If there are any conflicts between this summary and the report, please refer to the report as the correct version.

<img class="center-fit" src="../../assets/img/TTL_VGA_Project/VGA_Driver_Cover.jpg"/>
<div class="caption">
    Figure 1: VGA Driver Displaying Uninitialized Data in the SRAM
</div>


# 1. Introduction
I have been wondering how computers work since I was a kid. After taking ECE 120 (Introduction to Computing), many of my questions were answered regarding generally what's in a computer. However, how a computer displays stuff remained a mystery to me. To unveil this mystery, I decided to look up a very classic way of displaying graphics: VGA. Upon some research, I found the VGA signal format and [Ben eater](https://www.youtube.com/@BenEater)'s VGA driver using many ttl chips that can display static images, which eventually led to this project here. 

This circuit takes two user inputs (a keyboard and a mouse) and displays the input on the monitor. The mouse will draw lines on the screen while the keypad will generate shapes around the position of the curser. A block diagram is shown below.

<img class="center-fit" src="../../assets/img/TTL_VGA_Project/Flow_Chart.png"/>
<div class="caption">
    Figure 2: Project Flow Chart
</div>

# 2. Some background information
## 2.1 Keypad
We considered using a whole keyboard and displaying characters on the screen at the beginning. Such an approach would require a very large font ROM to store the drawing of each character, so we abandoned that idea and switched to a simpler approach. 

We used a 4x4 keypad, and each key on the keypad has a different function. For example, the "*" key will fill the whole screen with a solid color. We can also select what color to use through the A and C keys. To keep our project simple, we used 2-bit encoding for each color (R, G, and B) channel, so there are, in total, 64 different possible color outputs.

The circuit inside of the keypad is a 4x4 scanning matrix that, when a button is pressed, connects an R channel and a C channel shown in Fig. 3, telling the microcontroller it is connected to that a key is pressed. Again, a detailed explanation can be found in the lab report. 

<img class="center-fit" src="../../assets/img/TTL_VGA_Project/4x4-Keypad-Internal-Structure.png"/>
<div class="caption">
    Figure 3: Keypad Schematics
</div>

## 2.2 Mouse
We used a standard USB mouse that supports PS/2 for this project. While moving this mouse, it would move a cursor on the monitor and while holding the left key, it would draw colored pixels through the path it moved through.

<img class="center-fit" src="../../assets/img/TTL_VGA_Project/usb.png"/>
<div class="caption">
    Figure 4: USB Connector Pinout
</div>    

There are four pins on the connector as shown in Figure 4. We decided to communicate with the mouse through [PS/2 Protocol](https://www.burtonsys.com/ps2_chapweske.htm) because it is much simpler than USB. In this case, the D- pin will carry its data signal while the D+ pin carries its clock signal. Like the keypad, we used a microcontroller to decode the signals from the mouse. 


## 2.3 Microcontroller

The microcontroller will be responsible for taking the input from both the mouse and the keypad, processing them, and loading drawing data into the memory chip (a 512Kx 8-bit SRAM). We applied an Arduino Mega 2560 Rev3 shown in Figure 5 as the microcontroller for our project mostly due to its large number of I/O pins.

<img class="center-fit" src="../../assets/img/TTL_VGA_Project/Arduino.png"/>
<div class="caption">
    Figure 5: Microcontroller Utilized, source from Arduino Official Website
</div>    
For example, when the clear button is pressed, the Arduino will take that input from the keypad, go through all the memory locations we used in the memory chip, and set all of them to zero.

## 2.4 VGA Protocol
The idea behind VGA protocol is inspired heavily by the big old CRT monitors. In them, there is an electron gun that shoots electrons to every pixel, determining what color they are. The electron gun starts at the upper left corner of the screen, goes to the right pixel by pixel, and switches to the next line when it finishes a whole line. The original term "VGA" refers to the 640 x 480 @ 60Hz resolution on a monitor, and here, we are using this word to refer to all signals that use a VGA port. 

Note that other than the visible areas we see on the screen, many "hidden" sections should be considered when generating the VGA signal. A breakdown of those areas is shown in Figure 6. The sync pulses (Vertical Sync and Horizontal Sync) are especially important in this application.

<img class="center-fit" src="../../assets/img/TTL_VGA_Project/vga_theory.png"/>
<div class="caption">
    Figure 6: VGA Signal Area
</div>    

The size of each section is determined by the signal resolution. For example, for an 800 x 600 @ 60Hz signal, the required horizontal and vertical timing information is shown in Figure 7.

<img class="center-fit" src="../../assets/img/TTL_VGA_Project/vga_timing.png"/>
<div class="caption">
    Figure 7: VGA Timing Specification
</div>    

To simplify our project, we generated an 800 x 600 @ 60Hz signal but cut the width to 200, leading to the horizontal and vertical timing shown in Figures 8 and 9, respectively:

<img class="center-fit" src="../../assets/img/TTL_VGA_Project/modified_vga_horizontal_timing.png"/>
<div class="caption">
    Figure 8: Modified VGA Horizontal Timing
</div> 

<img class="center-fit" src="../../assets/img/TTL_VGA_Project/modified_vga_vertical_timing.png"/>
<div class="caption">
    Figure 9: Modified VGA Vertical Timing [18]
</div> 

As long as the horizontal and vertical sync pulses match the timing specification shown in Figure 7, the monitor will recognize it as a valid VGA signal.

## 2.5 Video Memory
The video memory, as the name suggests, stores the content to display on the monitor. Sadly, we weren't able to find any available dual-port memory chip that would easily fit on a breadboard, so we went for single-port SRAMs as our video memory unit. 

One thing worth mentioning is that we tried to do double buffering. 
As demonstrated by Figure 10, the main idea behind double buffering is that the monitor will display video memory in Buffer A that is fully generated by the graphics card while the graphics card is writing to Buffer B. After the monitor finishes reading from Buffer A, it starts reading from Buffer B while the graphics card writes the new frame to Buffer A. 
<img class="center-fit" src="../../assets/img/TTL_VGA_Project/double_buffering.gif"/>
<div class="caption">
    Figure 10: Idea Behind Double Buffering
</div> 

This approach could largely prevent problems like screen tearing and have a smoother display. However, due to time constraints, we did not fully finish this part in software although it is supported by hardware.

# 3. Implementation
# 3.1 VGA Signal Generation
A simple way to generate a VGA signal that matches the timing specification mentioned above is with counters. A binary counter (We specifically used SN74LS161A) simply counts up in binary representation (0000, 0001, 0010, and so on) when given a clock signal. We used NAND gates to detect if we should toggle the sync pulses at the correct pixel and applied two flip-flops to toggle the sync pulses.

For example, to match the horizontal timing requirements, we would have four 8-input NAND gates to detect if the current pixel is in the Visible Area (pixel 0 - 199), Front Porch (pixel 200 - 209), Sync Pulse (pixel 210 - 241), or Back Porch (pixel 242 - 263). At pixel 210, a flip-flop will be toggled and toggled again at pixel 241 to generate the horizontal sync pulse. The counters will reset after it reaches 264 and start counting from 0 up again as it finishes a whole line. The detailed schematics can be found in Figure 7 and Figure 8 in the lab report.

## 3.2 From Input to Video Memory
As mentioned before, the input from both the keypad and the mouse is processed by the Arduino. While it is relatively easy to write the driver for the keypad, processing the input from the mouse is rather difficult and was not the main focus of this project. Therefore, we used an open-source library written by [getis](https://github.com/getis) to read correct inputs from the mouse. 

When reading from the video memory, the address pins of the SRAM are connected to the counters from the VGA signal generation unit (explained in Section 3), and the output is connected to the VGA connector. When writing into it, we used a few MUXes to make the address pins connected to the Arduino only and drive the input pins also by the Arduino. Then, we used a nested for loop in software to load all video data into the SRAM.

## 3.3 From Video Memory to Monitor
VGA connectors usually have 15 pins, but there are only 5 pins that are not ground and important to us: the RGB signal, Horizontal Sync, and Vertical Sync. As mentioned in Section 2.4, as long as the sync pulses have the correct length and timing, the monitor will recognize the signal as a valid VGA signal. To that end, we can simply connect the sync pulses from the flip-flop (mentioned in Section 3.1) to the HSync and VSync signals directly. 

Displaying colored pixels is slightly more complicated. While testing, we simply connected the R, G, and B signals of the VGA connector to the first bit of the corresponding signal for the video memory (we have 2-bit storage for each color). The approach to display multiple colors is described in the lab report in detail.

# 4. Progress
## 4.1 VGA Generation Circuit
All of the work to this point was done in my first semester (Fall 2022). At the end of this semester, we were able to generate a valid VGA signal to be recognized by the monitor. By connecting the R, G, and B signals to certain output pins of the counter directly, we can easily display color strips on the monitor as shown in Figure 11. 

<img class="center-fit" src="../../assets/img/TTL_VGA_Project/Display_color_strips.jpg"/>
<div class="caption">
    Figure 11: Displaying Multiple Color Strips
</div> 

## 4.2 Video Memory with Input and Output

# 5. Conclusions

# 6. Works Cited

1 B. eater, ”Let’s build a video card!”, Eater.net, 2022. [Online].  Available: https://eater.net/vga.[Accessed:25- Sep- 2022].

2 B. Eater, The world’s worst video card?. 2019.

3 B. Eater, the World’s worst video card? The exciting conclusion. 2019.

4 jdh, I built my own graphics card. 2021.

5 W. Green, Project F. projectf.io, 2022 [Online]. Available: https://projectf.io/posts/video-timings-vga-720p-1080p/

6 ABRACON, LLC ”HALF SIZE DIP LOW VOLTAGE 5.0V CRYSTAL CLOCK OSCILLATOR,” ACH-10.000MHZ-EK datasheet [revised Nov. 2016]

7 Texas Instruments, ”Synchronous 4-Bit Counters,” SN74LS161AN datasheet, Oct. 1976 [revised Mar.1988]

8 Texas Instruments, ”SNx400, SNx4LS00, and SNx4S00 Quadruple 2-Input Positive-NAND Gates,” SN74LS00 datasheet, Dec. 1983 [revised May 2017]

9 Texas Instruments, ”8-Input Positive-NAND Gates,” SN74LS30, Dec. 1983 [revised Mar. 1988]

10 Texas Instruments, ”Hex Inverters,” SN74LS04, Dec. 1983 [revised Jan. 2004]

11 Desai, S. (2022, June 6). VGA pinout. Hardware connector pinouts and cables circuits wirings. Retrieved December 6, 2022, from ”https://pinoutguide.com/Video/VGA15-pinout.shtml ”

12 SECONS Ltd. (2008). SVGA signal 800 x 600 @ 60 hz timing. Retrieved December 6, 2022, from http://tinyvga.com/vga-timing/800x600@60Hz

13 Beneater (2018) Beneater/EEPROM-programmer: Arduino EEPROM programmer, GitHub. MIT. Available at: https://github.com/beneater/eeprom-programmer (Accessed: December 6, 2022).

14 Components101, “4x4 keypad module,” Components101, 2021. [Online].
Available: https://components101.com/misc/4x4-keypad-module-pinout-configuration-features-datasheet. [Accessed: 03-Feb-2023].

15 Components101, ”USB-A-Jack-Pinout” Components101, 2021. [Online].
Available: https://components101.com/sites/default/files/component-pin/USB-A-Jack-Pinout.png [Accessed: 04-Feb-2023].

16 Getis, “Getis/arduino-PS2-mouse-handler: PS2 mouse handler library for arduinos,” GitHub, 25-Oct-2021. [Online]. Available: https://github.com/getis/Arduino-PS2-Mouse-Handler. [Accessed: 01-Apr-2023].

17 PixArt Imaging Inc., “PAW3515DB DS Simple 1.0 - epsglobal,” PAW3515DB SERIES USB OPTICAL MOUSE SINGLE CHIP, Apr-2013. [Online].Available: https://www.epsglobal.com/Media-Library/EPSGlobal/Products/files/pixart/PAW3515DB.pdf
?ext=.pdf. [Accessed: 04-Feb-2023].

18 Oracle, "Double Buffering and Page Flipping," Java Documentation, "https://docs.oracle.com/javase/tutorial/extra/fullscreen/doublebuf.html". [Accessed: 05-Feb-2024].

<style>
    .center-fit {
        display: block;
        max-width: 100%;
        max-height: 100%;
        margin-left: auto;
        margin-right: auto;
    },

    .center-fit-small {
        display: block;
        max-width: 100%;
        max-height: 100%;
        margin-left: auto;
        margin-right: auto;
    }
</style>