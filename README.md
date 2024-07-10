# pico_usb_host_troubleshooting
How to use open source tools to help debug Raspberry Pi Pico USB Host Issues

## What can go wrong?
There can be a lot of different reasons that an embedded USB Host doesn't work with a USB device.
The problems will either be hardware related or software related.

### Host Wiring and Power
Before you connect any USB device to your embedded USB host project, be to check is that your
embedded USB host VBus power supply is a solid 5V. Make sure sure that the VBus pin and the ground
pin on your board's USB A Host connector are correctly wired. The way I check this is with this board:
![image](https://github.com/rppicomidi/pico_usb_host_troubleshooting/assets/94197396/ea6d0757-4154-4a0d-8247-8d8c0935ec27)

That is a fairly costly board, between $7 and $10 US. You can certainly get by with lower cost breakout
boards or just hacked old cables or connectors, but I already owned this board and I find it convenient
to use. All I do is plug the male USB A connector of the board to the USB A female host connector on
my project, and I have full access to VBus, Ground, D+ and D-.

At this point, you should make sure the D+ pin and D- pin are correctly wired. Your breakout board and
your ohm meter should be enough to test here.

### Cables
USB cables vary widely in quality and performance. If you are lucky, you get what your pay for.
For testing your new USB host project, use as short a cable as possible. That way, you are less
likely to encounter signal problems on the data lines and voltage drop on the VBus line. If your
USB device does not work with your embedded USB Host project, try a different cable if you can.

### Power Problems
Put a probe on the VBus pin when you plug in the USB device. Does 5V stay 5V? Does it droop low
and then recover? Does it get loaded down to below 5V and stay there? Any of these issues indicate
that your project's VBus supply is not strong enough. You may be able to fix this by inserting a
powered USB hub between your host's USB A port and the downstream device you are testing.

Also, do not rule out a problem with the cable. The resistance of the VBus wire in the cable
might be too high for the amount of current the attached device draws. Use a breakout board
to measure the cable voltage drop at the USB Device end of the cable. If there is too
much voltage drop, replace the cable.

### Data Issues with an Electrical Cause
Detecting electrical issues that cause data loss can be tricky. Equipment to properly measure
USB eye patterns can be fairly expensive, Also, verifying USB clock frequency issues can be
a challenge without expensive equipment. Fortunately, low speed and full speed USB are pretty
forgiving here, usually. The best solution is keep your USB Host project's wiring to the host
connector short and to use the appropriate matching resistors (22-27 ohms) between the RP2040
chip's I/O pins and the USB Host D+ and D- pins. Also make sure your device works with the
cable you are using and some other USB host, like a computer.

That is not very helpful, I know. Sometimes, hope is plan.

## Software Issues
This section assumes you have already solved basic software issues like program compile issues,
memory leaks, basic software logic issues, and so on. The focus is on what can go wrong
between a USB Host and a USB Device that can cause the Host and Device to not communicate
properly.

### Device USB Descriptors
The first thing a USB Host does when you connect a USB Device to it is reset the USB Device
and read the USB Device Descriptors from the Device. Device Descriptors are data structures
stored within each USB Device. The descriptors describe what the device capabilities and
requirements are. The host reads the descriptors to determine if the host hardware and
software support the requirements spelled out in the descriptors, and to determine what
to expect the device is able to do based on its capabilities. This process is called
enumeration.

Unlike a PC, an embedded USB host usually only supports a limited number of device types.
For example, some basic MIDI Host projects only support USB Hubs and USB MIDI 1.0 devices.
If you plug a device that your Host project does not support, the Host will not be able
to plug in the device.

### Dumping the USB Descriptors of a USB Device
Reading USB descriptors is a something every computer with a USB port can do. Just connect
your device to your computer, let it enumerate, and then have a look. The next section
describes common issues with the descriptor. This section describes how to look at the
descriptor on Linux, Mac and Windows systems.

Linux computers come with a command
```
lsusb
```
that will list every connected USB device along with the device VID and PID ID numbers.
To get a dump of the complete USB descriptor of a particular device, use this command:
```
lsusb -d "your device's VID:PID" -v
```
For example:
```
$ lsusb -d 2e8a:000a -v

Bus 001 Device 037: ID 2e8a:000a Raspberry Pi Pico
Couldn't open device, some information will be missing
Device Descriptor:
  bLength                18
  bDescriptorType         1
  bcdUSB               2.00
  bDeviceClass          239 Miscellaneous Device
  bDeviceSubClass         2 
  bDeviceProtocol         1 Interface Association
  bMaxPacketSize0        64
  idVendor           0x2e8a 
  idProduct          0x000a 
  bcdDevice            1.00
  iManufacturer           1 Raspberry Pi
  iProduct                2 Pico
  iSerial                 3 E66118604B49B426
  bNumConfigurations      1
  Configuration Descriptor:
    bLength                 9
    bDescriptorType         2
    wTotalLength       0x0054
    bNumInterfaces          3
    bConfigurationValue     1
    iConfiguration          0 
    bmAttributes         0x80
      (Bus Powered)
    MaxPower              250mA
    Interface Association:
      bLength                 8
      bDescriptorType        11
      bFirstInterface         0
      bInterfaceCount         2
      bFunctionClass          2 Communications
      bFunctionSubClass       2 Abstract (modem)
      bFunctionProtocol       0 
      iFunction               0 
    Interface Descriptor:
      bLength                 9
      bDescriptorType         4
      bInterfaceNumber        0
      bAlternateSetting       0
      bNumEndpoints           1
      bInterfaceClass         2 Communications
      bInterfaceSubClass      2 Abstract (modem)
      bInterfaceProtocol      0 
      iInterface              4 
      CDC Header:
        bcdCDC               1.20
      CDC Call Management:
        bmCapabilities       0x00
        bDataInterface          1
      CDC ACM:
        bmCapabilities       0x06
          sends break
          line coding and serial state
      CDC Union:
        bMasterInterface        0
        bSlaveInterface         1 
      Endpoint Descriptor:
        bLength                 7
        bDescriptorType         5
        bEndpointAddress     0x81  EP 1 IN
        bmAttributes            3
          Transfer Type            Interrupt
          Synch Type               None
          Usage Type               Data
        wMaxPacketSize     0x0008  1x 8 bytes
        bInterval              16
    Interface Descriptor:
      bLength                 9
      bDescriptorType         4
      bInterfaceNumber        1
      bAlternateSetting       0
      bNumEndpoints           2
      bInterfaceClass        10 CDC Data
      bInterfaceSubClass      0 
      bInterfaceProtocol      0 
      iInterface              0 
      Endpoint Descriptor:
        bLength                 7
        bDescriptorType         5
        bEndpointAddress     0x02  EP 2 OUT
        bmAttributes            2
          Transfer Type            Bulk
          Synch Type               None
          Usage Type               Data
        wMaxPacketSize     0x0040  1x 64 bytes
        bInterval               0
      Endpoint Descriptor:
        bLength                 7
        bDescriptorType         5
        bEndpointAddress     0x82  EP 2 IN
        bmAttributes            2
          Transfer Type            Bulk
          Synch Type               None
          Usage Type               Data
        wMaxPacketSize     0x0040  1x 64 bytes
        bInterval               0
    Interface Descriptor:
      bLength                 9
      bDescriptorType         4
      bInterfaceNumber        2
      bAlternateSetting       0
      bNumEndpoints           0
      bInterfaceClass       255 Vendor Specific Class
      bInterfaceSubClass      0 
      bInterfaceProtocol      1 
      iInterface              5
```

MacOS Homebrew also supports the `lsusb` command. However, you will need to install the
"good" version of the command using brew install:
```
brew install lsusb
```
The version conflicts with the default version in Homebrew usbutils, but I like this
version better.

Windows PC users can install the [Thesycon USB Descriptor Dumper](https://www.thesycon.de/eng/usb_descriptordumper.shtml).
The UI for this tool is not bad.

### Common Descriptor Issues a Descriptor Dump Can Help You to Solve
#### Descriptor too big
Sometimes the device's configuration descriptor is too large to fit in the buffer your
embedded project has allocated for enumeration. Look for the length in the Configuration
Descriptor header. For example:
```
  Configuration Descriptor:
    bLength                 9
    bDescriptorType         2
    wTotalLength       0x0054 <== 84 bytes
```
The solution is to make the descriptor buffer larger if you can, or accept that device
won't work if you cannot.

#### Descriptor contains length error
Sometimes the device has an issue where the length fields within the descriptor headers
are wrong. This is a common problem with MIDI devices, for example. The best solution
to this is to make sure your host driver is robust to incorrect descriptor header length
fields.

#### Endpoint type not supported
The USB Class Specification for each type of USB device will specify the type of each
endpoint required to make the device work. Sometimes, device manufacturers make
mistakes here. For example, The USB MIDI Class Specification 1.0 requires the data
endpoints to be of the Bulk type. I have encountered devices that use Interrupt
Type endpoints. I was able to modify the Host driver to support both Bulk and Interrupt
type endpoints.

### USB Packet Issues
USB commuicates via message packets. The Host requests data and the device responds with either
the data requested or some handshake that says it is not ready or cannot provide the data.
Sometimes finding issues requires you to look at the USB data the host is reading from
the device or that the host is sending to the device.

## USB packet sniffers
A USB packet sniffer is a passive device that captures the USB packets that are passed
between the host and device. It has some means to store or display the captured packets, either
in real-time or after some other limit such as number of packets captured, elapsed
time, or using a UI to stop the capture. A USB packet sniffer if probably the most
convenient way to debug USB traffic.

If the USB host is in a computer running Linux, macOS or Windows, then there are software-based
solutions that allow you display the USB packets between the computer and an attached USB
device. However, a typical embedded USB host does not have the resources to support that.
You need to connect an external hardware-based USB packet sniffer between your embedded
host and your device.

Commercial embedded USB packet sniffers are very good, but most cost about $500 US or more.
A less costly solution is to use hardware/software logic analyzers to capture the USB packets,
but the good ones are still in the $100 US neighborhood. This cost is hard to justify for
a tool that you probably won't use very often.

Thanks to the Raspberry Pi Pico or other similar RP2040 boards and the great work from the open
source community, you can build your own USB sniffer for much less money. It doesn't even
have to be a dedicated tool. My packet sniffer is just my USB breakout board, 3 jumper
wires, and a Raspberry Pi Pico. When I do not need the packet sniffer, I can use all the
pieces for something else. Here is a photo of my packet sniffer hardware:
![usb-packet-sniffer](https://github.com/rppicomidi/pico_usb_host_troubleshooting/assets/94197396/aa77cb6c-54bd-4c75-ab03-2c05a797ecdc)



