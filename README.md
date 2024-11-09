# RPI5 PCIe 3.0 Hub
An opensource PCIe 3.0 Hub design based on ASM2806 with Raspberry Pi 5's PCIe FPC connector design

![](/img/DSC7148.jpg)

## Important note

ASM2806A != ASM2806, this design is using ASM2806 specifically.

## Introduction

So basically this is the follow-up on [this](https://www.willwhang.dev/Solar-eclipse-2024/#pcie-30-hub-development), that I finally found a schematic containing how to use [ASM2806](https://www.asmedia.com.tw/product/012Yq70sX2zI9xN5/7c5YQ79xz8urEGr1), which is a PCIe 3.0 hub with x2 upstream and four x1 downstream.  

The board is designed to give that schematic a try, so it is not really designed like those PCIe 2.0 NVMe hub RPi hat designs you might have seen before. Its design is to make my life easier when debugging/soldering it. (i.e single sided components + 5cm x 5cm board size).

Also, because I'm going to build a few boards for RPi5 that I want to have a PCIe 3.0 hub with, all the PCIe connectors on board are the same as RPi’s PCIe FPC "[standard](https://datasheets.raspberrypi.com/pcie/pcie-connector-standard.pdf)", and the testing is also done on RPi5.

![](/img/pcb.jpg)

Is it working? Yeap!

![](/img/cmdline.jpg)


## Detailed notes

So if you compare the PCB files here and the one in the photo, you can see that there is a jump wire in the photo. At the same time, if you look at the PCB files, there are two jumper JP1 and JP2. This is because RPI5 FPC has a power enable signal, where the device should use this signal to power down and up. However, this makes the debugging (checking rail voltage) process slightly difficult, so the jumper here is for forcing ASM2806/downstream device power on. JP1 is for ASM2806 power enable signal, either connect it directly to 5V or using the RPI enable signal. JP2 is the downstream PCIe FPC power enable signal, and it is either 3.3V or RPI enable signal. The one shown in the photo has the downstream PCIe FPC power enable signal bodge wired to the RPI enable signal, and that has been fixed in the PCB file here on GitHub.

For ASM2806, it is configured as four downstream ports with each getting x1 PCIe 3.0. Both upstream clock and downstream clock are based on 100Mhz PCIe clock. No SPI flash is required to function (thankfully). It requires three voltages: 3.3V/2.5V/1.05V. Most of the DC-DC converter here is just grabbing the design/parts I have on my hand and are overspec a bit so you can change it for sure. For the rail naming, there is also 3.3V suspend, but I connected it to 3.3V directly, and the 1.05V I'm following the schematic to add some filtering.

Surprisingly, it doesn't take that much power. As you can see in the power consumption table in the schematic, I can stress test without a heatsink for hours, but the chip does get hot to touch. If that worries you, then you might have to consider adding a small heatsink, especially if the PCB copper is not large enough to dissipate the heat.

There are some signals that I'm interested in, specifically the HOTPLUG signal, and it is attached to a test pad, but I'm not able to see it function by pulling this pin up or down.

Finally, the thing that caught me during the test is how the schematic connects the crystal. In the schematic, there is a 10pf cap on the XO/XI pins, which I'm still quite puzzled about (Normally, I expect to see like a 1M ohm resistor across for the Crystal driver), but it is necessary to have this chip work.


## Performance

Yes, each PCIe downstream device independently can consume up to PCIe 3.0 x1 speed, but once you have more than one going, I’m surprised the bandwidth doesn't seem to be equally shared. I'm not quite sure if that is caused by the tool I use to stress test (four NVMe drives + FIO), or is it caused by the NVMe drive itself (maximum supported MPS). Following Jeff Geerling's [guide](https://www.jeffgeerling.com/blog/2021/getting-faster-10-gbps-ethernet-on-raspberry-pi) I've also enabled pci=pcie_bus_perf and the hub can support up to 512 byte MPS also:

```
[    0.752521] pci 0000:00:00.0: Max Payload Size set to  512/ 512 (was  128), Max Read Rq  512
[    0.752532] pci 0000:01:00.0: Max Payload Size set to  512/ 512 (was  128), Max Read Rq  512
[    0.752543] pci 0000:02:00.0: Max Payload Size set to  512/ 512 (was  128), Max Read Rq  512
[    0.752558] pci 0000:03:00.0: Max Payload Size set to  512/ 512 (was  128), Max Read Rq  512
[    0.752568] pci 0000:02:02.0: Max Payload Size set to  512/ 512 (was  128), Max Read Rq  512
[    0.752588] pci 0000:04:00.0: Max Payload Size set to  128/ 128 (was  128), Max Read Rq  128
[    0.752598] pci 0000:02:06.0: Max Payload Size set to  512/ 512 (was  128), Max Read Rq  512
[    0.752616] pci 0000:05:00.0: Max Payload Size set to  256/ 256 (was  128), Max Read Rq  256
[    0.752627] pci 0000:02:0e.0: Max Payload Size set to  512/ 512 (was  128), Max Read Rq  512
[    0.752644] pci 0000:06:00.0: Max Payload Size set to  256/ 256 (was  128), Max Read Rq  256
```

## Footnotes
* [Interactive BOM](https://htmlpreview.github.io/?https://github.com/will127534/PCIe3_Hub/blob/main/bom/ibom.html) [(provided by InteractiveHtmlBom)
](https://github.com/openscopeproject/InteractiveHtmlBom)
* designed with Kicad v8.0
* Please refrain from inquiring about the source of the chip. Additionally, please note that this chip is quite expensive (approximately 15-20 USD), which is one of the reasons why you may not have encountered any PCIe 3.0 hubs for RPi5 other than this one.
