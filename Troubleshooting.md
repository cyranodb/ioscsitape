# Introduction #

As mentioned elsewhere Mac OS X must first be able to see the tape drive connected to it before IOSCSITape can provide the `/dev/st0` entries for use. This page will describe some of techniques to verify that Mac OS X does indeed see the drive.

# 1. Make sure Mac OS X can see you bus(es) #



# 2. Make sure Mac OS X sees your tape drive #

Unlike in some previous versions of Mac OS X in Snow Leopard tape drives don't seem to show up in System Profiler - in neither the GUI application or the CLI `system_profiler` application. However the IO Registry will show the "device nubs" for tape drives. To look for the tape drive SCSI device nub, issue this command:

```
ioreg -r -c IOSCSIPeripheralDeviceNub
```

This should give a number of listings. For a MacBook Pro these results are common:

```
+-o IOSCSIPeripheralDeviceNub  <class IOSCSIPeripheralDeviceNub, id 0x100001ad4$
  | {
  |   "IOClass" = "IOSCSIPeripheralDeviceNub"
  |   "CFBundleIdentifier" = "com.apple.iokit.IOSCSIArchitectureModelFamily"
  |   "IOProviderClass" = "IOSCSIProtocolServices"
  |   "IOCFPlugInTypes" = {"7D66678E-08A2-11D5-A1B8-0030657D052A"="IOSCSIArchit$
  |   "SCSITaskDeviceCategory" = "SCSITaskUserClientDevice"
  |   "IOUserClientClass" = "SCSITaskUserClient"
  |   "IOProbeScore" = 0
  |   "Peripheral Device Type" = 1
  |   "IOMatchCategory" = "SCSITaskUserClientIniter"
  |   "Vendor Identification" = "SONY"
  |   "Protocol Characteristics" = {"Physical Interconnect"="USB","Read Time Ou$
  |   "Product Revision Level" = "0103"
  |   "SCSITaskUserClient GUID" = <00af5107c5a638a9675d0000>
  |   "Product Identification" = "SDX-420C"
  | }
  | 
  +-o SCSITaskUserClientIniter  <class SCSITaskUserClientIniter, id 0x100001ad5$

+-o IOSCSIPeripheralDeviceNub  <class IOSCSIPeripheralDeviceNub, id 0x10000027e$
  | {
  |   "IOProbeScore" = 0
  |   "CFBundleIdentifier" = "com.apple.iokit.IOSCSIArchitectureModelFamily"
  |   "IOProviderClass" = "IOSCSIProtocolServices"
  |   "IOClass" = "IOSCSIPeripheralDeviceNub"
  |   "IOMatchCategory" = "SCSITaskUserClientIniter"
  |   "SCSI Device Characteristics" = {"IOMaximumBlockCountRead"=8192,"IOMaximu$
  |   "Peripheral Device Type" = 0
  |   "Vendor Identification" = "APPLE"
  |   "Protocol Characteristics" = {"Physical Interconnect"="USB","Read Time Ou$
  |   "Product Identification" = "SD Card Reader"
  |   "Product Revision Level" = "1.00"
  | }
  | 
  +-o com_apple_driver_AppleUSBCardReaderSBC  <class com_apple_driver_AppleUSBC$
    +-o IOBlockStorageServices  <class IOBlockStorageServices, id 0x10000028e, $
      +-o IOBlockStorageDriver  <class IOBlockStorageDriver, id 0x10000028f, re$

+-o IOSCSIPeripheralDeviceNub  <class IOSCSIPeripheralDeviceNub, id 0x10000025a$
  | {
  |   "IOProbeScore" = 0
  |   "CFBundleIdentifier" = "com.apple.iokit.IOSCSIArchitectureModelFamily"
  |   "IOProviderClass" = "IOSCSIProtocolServices"
  |   "IOClass" = "IOSCSIPeripheralDeviceNub"
  |   "IOMatchCategory" = "SCSITaskUserClientIniter"
  |   "Peripheral Device Type" = 5
  |   "Vendor Identification" = "MATSHITA"
  |   "Product Identification" = "DVD-R   UJ-868"
  |   "Protocol Characteristics" = {"Write Time Out Duration"=15000,"AHCI Port $
  |   "Product Revision Level" = "KB19"
  | }
  | 
  +-o IOSCSIPeripheralDeviceType05  <class IOSCSIPeripheralDeviceType05, id 0x1$
    +-o IODVDServices  <class IODVDServices, id 0x100000280, registered, matche$
      +-o IODVDBlockStorageDriver  <class IODVDBlockStorageDriver, id 0x1000002$
      +-o SCSITaskUserClientIniter  <class SCSITaskUserClientIniter, id 0x10000$
```


In this listing there are three devices with the `IOSCSIPeripheralDeviceNub` type. The first is a SONY AIT-1 (SDX-420C) tape drive. The second is an SD card reader and the third is the DVD-R drive for the computer. **The key indicator for a tape drive** is the _"Peripheral Device Type" = 1_ attribute of the nub. This indicates a SCSI sequential (tape) drive. This **must** be present for IOSCSITape to find the drive (IOSCSITape actually looks in the IO Registry for this exact attribute to find the drive at all). Note also in the second and third devices that they have drivers attached to them. For the DVD drive there is the `IODVDBlockStorageDriver`, and for the SD card reader there is `IOBlockStorageDriver`. The tape drive, because no driver/KEXT is associated with it, has nothing below. Until, of course, IOSCSITape gets installed and is loaded.

# 3. Make sure IOSCSITape is loaded and is attached to and sees the tape drive #

After installing IOSCSITape ensure that the KEXT is loaded by looking through the `kextstat` output:

```
% kextstat | grep IOSCSITape
  153    0 0x5762e000 0x5000     0x4000     com.googlecode.ioscsitape.IOSCSITape (1.0.0d1) <46 5 4 3 1>
```

If IOSCSITape is loaded and successfully attached to the tape drive the `ioreg` command from above should look like this:

  * Note the IOSCSITape entry at the bottom of the listing

```
+-o IOSCSIPeripheralDeviceNub  <class IOSCSIPeripheralDeviceNub, id 0x100001ad4$
  | {
  |   "IOClass" = "IOSCSIPeripheralDeviceNub"
  |   "CFBundleIdentifier" = "com.apple.iokit.IOSCSIArchitectureModelFamily"
  |   "IOProviderClass" = "IOSCSIProtocolServices"
  |   "IOCFPlugInTypes" = {"7D66678E-08A2-11D5-A1B8-0030657D052A"="IOSCSIArchit$
  |   "SCSITaskDeviceCategory" = "SCSITaskUserClientDevice"
  |   "IOUserClientClass" = "SCSITaskUserClient"
  |   "IOProbeScore" = 0
  |   "Peripheral Device Type" = 1
  |   "IOMatchCategory" = "SCSITaskUserClientIniter"
  |   "Vendor Identification" = "SONY"
  |   "Protocol Characteristics" = {"Physical Interconnect"="USB","Read Time Ou$
  |   "Product Revision Level" = "0103"
  |   "SCSITaskUserClient GUID" = <00af5107c5a638a9675d0000>
  |   "Product Identification" = "SDX-420C"
  | }
  | 
  +-o SCSITaskUserClientIniter  <class SCSITaskUserClientIniter, id 0x100001ad5$
  +-o IOSCSITape  <class IOSCSITape, id 0x100001ad9, !registered, !matched, act$
```

This means IOSCSITape has attached to the tape drive and can send SCSI commands to it. If this is the case, then in the system log (`/var/log/system.log` on Mac OS X 10.5 or `/var/log/kernel.log` on Mac OS X 10.6) IOSCSITape will print messages like these:

```
Mar 26 19:11:58 localhost kernel[0]: rst0: <SONY, SDX-420C, 0103> tape
Mar 26 19:11:58 localhost kernel[0]: rst0: density code: 48, 512-byte blocks, write-enabled, buffered
Mar 26 19:11:58 localhost kernel[0]: rst0: min/max block size: 2/16777215
```

At that point the `/dev/rst0` entry should be in `/dev/`:

```
% ls -l /dev/rst0
crw-rw-r--  1 jesse  operator   21,   0 Mar 26 19:11 /dev/rst0
```

If one reaches this point then congratulations: the tape drive should be operating correctly with the driver. From here one can finish the GettingStarted guide or otherwise use the tape drive as they wish.