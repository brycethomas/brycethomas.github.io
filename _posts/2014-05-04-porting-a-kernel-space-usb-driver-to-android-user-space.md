---
layout: post_page
title: Porting a Kernel Space Linux USB driver to Android User Space
---

This post is about how to port a Kernel Space Linux USB driver to
Android User Space.  USB OTG is an intriguing feature of Android that
perhaps hasn't gotten as much interest as it deserves.  I suspect this
may be in part due to developers not knowing where to start.  When I
[partially ported a USB Wi-Fi
driver](https://github.com/brycethomas/liber80211) back in 2012 there
wasn't much in the way of guides online&mdash;I pieced together the
puzzle one disparate piece of information at a time.  My hope is that
this guide will bring all the relevant information together and ignite
some more interest on USB OTG on Android.

First, some background.  Android 3.1 introduced support for [USB
On-The-Go](http://en.wikipedia.org/wiki/USB_On-The-Go) (OTG).  This
allows an Android device to act as as the USB host (much like a
desktop computer) rather than as a peripheral device (e.g. a thumb
drive).  Essentially, it makes it possible to plug *other* USB
peripherals into an Android device.  This is interesting because it
means that all manner of existing USB peripherals can be made to
interface with Android devices.  Some peripherals work out of the box.
For example, many USB mice can be connected directly to an Android
device and a cursor will appear on the device's screen.  A lot of
other peripherals however do not have native driver support on
Android.  

The way to get peripherals to work with Android when there is no
native driver support is to use Android's [USB
Host](http://developer.android.com/guide/topics/connectivity/usb/host.html)
API to reimplement the peripheral's driver in user space.  The USB
host API let's applications register to be notified when a specific
peripheral is connected to the device and perform arbitrary byte-level
IO with the peripheral.  Because the USB Host API is an official
Android API that's available from user space, there's no need to root
the Android device.  As an example,
[liber80211](https://github.com/brycethomas/liber80211) (which [can be
found on the Google Play
Store](https://play.google.com/store/apps/details?id=com.brycestrosoft.liber80211))
let's one do nifty packet capture on an Android device using an
external USB Wi-Fi card and does *not* require root.

Before we get too far into things, a bit of a warning&mdash;USB OTG is
stilly a bit flaky in the Android ecosystem.  Some devices
(e.g. Galaxy Nexus, Nexus 7) support USB OTG beautifully.  Some other
devices mostly support it but will require an external power source
with power hungry peripherals.  And some other handsets just don't
properly support it at all.  For handsets that don't support it at
all, this is sometimes a hardware issue and sometimes a software
issue.  Some people have had luck with rooting certain handsets and
patching the operating system drivers to add USB OTG support, but this
isn't ideal.  Other devices flat out won't work at the hardware level.
So, even though Android >= 3.1 is the formal prerequisite for USB OTG,
it can still be a bit of a mine field.  Hopefully USB OTG matures on
Android over time and by increasing awareness of the issues, perhaps
this will help expedite that process.

One more thing&mdash;Linux drivers are typically written in C while
the Android USB Host API is written in Java.  You'll need to be at
least familiar with both languages.

### Finding a Peripheral's Existing Driver Source Code

So you have a USB peripheral that works on regular Linux and you'd
like to port the driver to Android.  To find the Linux source code
that's handling the peripheral you'll first need to gather some of the
peripheral's metadata.  Per the USB protocol every USB peripheral
reports a few key pieces of information to the Linux host upon
connection.  The host then uses to delegate responsibility for the
peripheral to a specific driver.  The first two key pieces of
information you want are:

1. **Vendor ID**: a 16-bit string which uniquely identifies the
peripheral's vendor.
2. **Product ID**: a 16-bit string which uniquely identifiers the
specific peripheral.

On Linux, you can retrieve the Vendor ID and Product ID with the
`lsusb` command.  Here's an example of the output on my machine:

```
~$ lsusb
Bus 002 Device 011: ID 0bda:8187 Realtek Semiconductor Corp. RTL8187 Wireless Adapter
Bus 001 Device 001: ID 1d6b:0002 Linux Foundation 2.0 root hub
Bus 002 Device 001: ID 1d6b:0002 Linux Foundation 2.0 root hub
Bus 001 Device 002: ID 8087:8008 Intel Corp. 
Bus 002 Device 002: ID 8087:8000 Intel Corp. 
Bus 002 Device 003: ID 174c:2074 ASMedia Technology Inc. 
Bus 002 Device 004: ID 0b05:17d0 ASUSTek Computer, Inc. 
Bus 002 Device 006: ID 046d:0826 Logitech, Inc. 
Bus 002 Device 007: ID 045e:071d Microsoft Corp. 
Bus 002 Device 008: ID 046d:c531 Logitech, Inc. 
```

In the output above the Vendor ID and Product ID are the two
colon-separated 4 character hex strings.  So for example, the USB
wireless card I had connected to my computer has Vendor ID `0bda` and
Product ID `8187`.  If you can't identify the specific peripheral
you're interested in porting, unplug it and run `lsusb` again to see
which item is now omitted in the output.  Keep a note of the Vendor ID
and Product ID.  For the time being they'll help us to find the
peripheral's Linux driver source code.  You'll also need them later on
though as a way of registering your Android application as capable of
handling connected peripherals with that Vendor ID and Product ID.

TODO: write rest of section here once I get more answers on driver
matching from
http://stackoverflow.com/questions/11661679/where-in-the-linux-source-does-recognition-of-specific-usb-devices-happen.

### Navigating the Linux Driver Source Code 

At this point it's assumed you've been able to find the relevant
peripheral driver source code file using the method outlined above.
What would be nice to know now is where in this file the "entrypoint"
is, i.e. once Linux has handed responsibility for the peripheral
driver, what function is it that executes.  What you should be looking
for is a call to
[`module_usb_driver()`](https://www.kernel.org/doc/htmldocs/usb/API-module-usb-driver.html).
`module_usb_drver()` accepts a struct of type `usb_driver`, so you
should see in the call that something is being passed in.  This struct
has a number of fields, but the one of immediate interest is
`.probe`.  `.probe` is a function pointer to the method that initially
gets called.  So, `.probe` is essentially the "main" method as far as
control of a USB device is concerned, so this is where one would start
looking to begin to understand how the Linux peripheral driver works.
To give an example, at time of writing, here is what can be found in
`/drivers/net/wireless/rtl818x/rtl8187/dev.c`:

```
static struct usb_driver rtl8187_driver = {
	.name		= KBUILD_MODNAME,
	.id_table	= rtl8187_table,
	.probe		= rtl8187_probe,
	.disconnect	= rtl8187_disconnect,
	.disable_hub_initiated_lpm = 1,
};

module_usb_driver(rtl8187_driver);
```

Sure enough, there's a function named `rtl8187_probe` in the same file
and this is the main entry point for this specific driver.  At this
point you can begin to peruse the Linux driver code and get a feel for
how your specific peripheral works.  

One tool I found particularly useful when analysing the source code
for my USB wireless LAN card was
[CTAGS](http://ctags.sourceforge.net/).  Not to drift too far off
topic, but essentially the way CTAGS work is that you run the program
over a source code directory and it generates a standardized index
file which describes, for example, the file and line number of calls
to each function.  This index file can then be used in combination
with a text editor (e.g. Emacs, vi) and keyboard shortcuts to navigate
the codebase.  The two indispensible features of CTAGS for navigating
driver code are recursively jumping into function definitions from
function calls and then recursively jumping back out.  This is very
helpful for digging into functions to see where the actual byte-level
transfers are taking place between the host and the peripheral.
Ultimately it is these byte-level messages that the user space Android
drive must speak in terms of and this is a good way to get a handle on
it.  

