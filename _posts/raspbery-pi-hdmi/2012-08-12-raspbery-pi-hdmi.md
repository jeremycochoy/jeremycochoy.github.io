---
layout: post
title: Raspbery PI with HDMI
description: A simple config file required to boot your raspbery with HDMI.
author: Jérémy Cochoy
date: 2012-08-12 +0100
categories: raspbery embeded hdmi adminsys
lang: en
...

Today, I was testing my [RPi][raspbery-wikipedia], just to check that "_everything is alright_". I bought a cheap 1A/5V micro usb power supply from china (from a well known auction website) and writed the official raspbian image. I plugged my new VGA/HDMI flat screen, and booted the RPi. Then.... nothing.

Actualy, to make it works, I had to modify the `config.txt` file, as said by _Aptoitos_ on [the website rs-online][rs-online] :
```
hdmi_drive=2
config_hdmi_boost=4
hdmi_group=1
hdmi_mode=16
hdmi_force_hotplug=1
disable_overscan=0
```

After adding and uncommenting those lines in the config.txt file, it worked fine (although I still have to push the auto button of my screen to tell him something great is coming from HDMI input).

I can't tell exactly what was the problem (signal too low to be recognized, bad display mode, or something else) but then it works fine, and now I'll have to wait for a USB keyboard :)

For more informations about which setting change what : <http://www.rs-online.com/designspark/electronics/knowledge-item/what-you-need-to-get-your-raspberry-pi-up-and-running>

[raspbery-wikipedia]: https://en.wikipedia.org/wiki/Raspberry_Pi
[rs-online]: https://www.rs-online.com/designspark/what-you-need-to-get-your-raspberry-pi-up-and-running