# pico_usb_troubleshooting
How to use open source tools to help debug Raspberry Pi Pico USB Host and USB Device Issues

## What can go wrong?
There can be a lot of different reasons that an embedded USB Host doesn't work with a USB device.
The problems will either be hardware related or software related. Most of the hardware
issues 

Commercial embedded USB packet sniffers are very good, but most cost US $500 or more. Thanks
to the Raspberry Pi Pico or other similar RP2040 boards and the great work from the open
source community, you can build your own USB sniffer for much less money.



## What does a USB packet sniffer do?
A USB packet sniffer displays the USB traffic between a USB host all downstream USB
hubs and devices.

