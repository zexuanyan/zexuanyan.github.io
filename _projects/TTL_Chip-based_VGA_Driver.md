---
layout: page
title: TTL Chip-Based VGA Driver
description: A Breadboard-based VGA Driver that supports live-editing of the image displayed on the monitor.
img: assets/img/TTL_VGA_Project/VGA_Driver_Cover.jpg
importance: 1
category: Technical
---

For my ECE 110/120 Honors Lab project, my team made a VGA driver out of breadboards and TTL chips. If you are interested in the technical details, you can find the full lab report [here](../../assets/pdf/SP23_Honors_Lab_Final_Reports.pdf). In this post, I will give you a quick rundown of what this project is about!

<div class="row">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.html path="assets/img/TTL_VGA_Project/VGA_Driver_Cover.jpg" title="VGA Driver Cover" class="img-fluid rounded z-depth-1" %}
    </div>
</div>
<div class="caption">
    VGA Driver Displaying Uninitialized Data in the SRAM
</div>


# Introduction
I have been wondering how computers display stuff for a long time. Upon some research, I found the VGA signal format and [Ben eater](https://www.youtube.com/@BenEater)'s VGA driver using many ttl chips that can display static images. To have a solid understanding of the VGA single format and build upon Ben Eater's project, I first tried to recreate this project by redesigning the circuit myself and then added user inputs to it so that the user can control the content displayed on the monitor with a keypad and a mouse. A block diagram is shown below.

<div class="row">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.html path="assets/img/TTL_VGA_Project/Flow_Chart.png" title="flow chart" class="img-fluid rounded z-depth-1" %}
    </div>
</div>
<div class="caption">
    Project Flow Chart
</div>
