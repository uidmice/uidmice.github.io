---
layout: post
title: SGTL-5000 footprint & implementation
tag: [Hardware, PCB]
---

SGTL5000 is a low-power stereo codec with headphone amplifier from NXP. Its ultra-low-power with very high performance makes it suitable for portable products as well as low-power IoT devices. Surprisingly, I failed to find existing footprint for it online.

I am currently working on a project called CommonSense, which aims to provide a common hardware platform able to host different types of sensors. Sensors will be plugged in and unplugged as daughter cards to the main base board. I am designing the sensor daughter board and trying to pin down an audio dec that is able to record microphone input and play back headphone output on 3.5 mm jack. I originally used TLV320DAC26 from TI, but it's an old chip and the power performance is not as good as newer chips. So I want to replace it with SGTL5000.  

## SGTL5000

Digital input & output: one I2S port supporting
* I2S
* Left Justified
* Right Justified
* PCM

Here is a library for SGTL5000. [link](https://os.mbed.com/users/aidan1971/code/SGTL5000/)

## SGTL5000 20-QFN
The datasheet gives an example application schematics.

![sgtl5000]({{ site.baseurl }}/images/sgtl5000.png)

20-QFN typical application schematic with cap-coupled headphone design

I switched to capless headphone design with 3.5 mm jack for microphone input and headphone output as shown below:
![my-sgtl5000]({{ site.baseurl }}/images/stgl5000-my.png)

Here is a footprint I created for the part with thermal vias on the center pad.
![sgtl5000-footprint]({{ site.baseurl }}/images/sgtl5000-footprint.png)
