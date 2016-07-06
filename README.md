A working overlay for OpenWrt DD (r49293 / e737bc7).

Based on github.com/H-LK/MLWG3

Also it contains ralink package files from github.com/rssnsj/openwrt-hc5x61
and some patches against github.com/i80s/mtk-sources. Most tricky bug
was caused by the 5G wifi driver (mt7610e) which tried to read a
configuration file from a kernel space and failed. Doing this stuff
from a kernel module is bad (we have ioctls, configfs, proc and
firmware requests) but I still implemented a dirty fix by avoiding
direct `f_op->read` access and using `kernel_read` helper instead.
This driver should be refactored someday to get rid of a hacky OS
abstraction layer and be integrated into mac80211 driver
infrastructure. Until then we still have to use a userspace utility
(uci2dat) which converts OpenWrt wireless configuration file into
another format suitable for the driver. This file is loaded when the
wireless interface is brought up.

5GHz wireless chip works now.

For some reason the first time I flashed the image using the stock
firmware's mtd tool (into both KernelA and KernelB), it did not boot
until I hooked up a USB-UART to the board, initiated the TFTP kernel
upload procedure and then aborted it. It asked for a confirmation,
showed some "erasing" message and then rebooted into a working OpenWrt
installation. Subsequent sysupgrades worked just fine.

openwrt-hc5x61 had a missing eeprom image so I got it from the
original firmware. I don't know if it is really needed but the driver
reads it. If the read fails it fallbacks to some default values.

# TODO

Original firmware has a `power_check` utility waiting to be reverse
engineered. It accesses some GPIO pin to read out the battery status.

SD/MMC does not work. Adding status = "okay"; to the corresponding
section of the device tree and loading a kernel module breaks the
network driver, possibly due to wrong pinctrl configuration.
