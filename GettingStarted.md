# Introduction #

IOSCSITape is an IOKit driver KEXT (kernel extension). This is a package that lives in `/System/Library/Extensions/` on a Mac OS X system and provides the BSD-file `/dev/stN` (where N represents your tape drive(s) start from 0) for manipulating the tape drive including reading and writing data to it.

# Supported Devices #

IOSCSITape can use any device that is identified by Mac OS X as being a SCSI sequential (tape) device _regardless of the physical interconnect bus_. This includes tapes drives connected via USB, Firewire, Parallel SCSI, ATAPI, Fibre Channel and should include USB-to-SCSI (and similar) adapters. The only pre-requisite is that Mac OS X itself must "see" the device before IOSCSITape can discover it. See the [Troubleshooting](Troubleshooting.md) page for more information.

# Steps #

  1. Install the package installer from the [downloads](http://code.google.com/p/ioscsitape/downloads/list) section. It will install these files & packages:
    * `/System/Library/Extensions/IOSCSITape.kext`
    * `/usr/local/bin/mt`
    * `/usr/local/share/man/man1/mt.1`
    * `/usr/local/include/mtio.h` (for Snow Leopard)
  1. Check the `/var/log/system.log` (`/var/log/kernel.log` on Snow Leopard) to see the the model and some information about your tape drive(s).
  1. Use the `/dev/rst0` (where N represents your tape drives starting from 0) device and the `/usr/local/bin/mt` command to manipulate the tape drive (See below for some examples of use)
  1. If things aren't working, check out the [Troubleshooting](Troubleshooting.md) page
  1. Beware of known issues (see below)
  1. Provide feedback! The more feedback the better this project can become. Talk on the [ioscsitape-discuss](http://groups.google.com/group/ioscsitape-discuss) list or submit issues via the [issue tracker](http://code.google.com/p/ioscsitape/issues/list).

# Specific Tool Uses #

## Copy zeros or random data to a tape with `dd` ##

```
dd if=/dev/zero of=/dev/rst0 count=10
```

Replace `/dev/zero` with `/dev/urandom` for random (but very slow) data.

## Read data from a tape with `dd` ##

```
dd if=/dev/rst0 of=/dev/null
```

## Copy your Desktop to and from tape with `tar` ##

```
mt -f /dev/rst0 rewind
tar cf /dev/rst0 ~/Desktop
```

Optionally add the `v` switch to tar (`tar cvf`) to see the files as they are copied. Now extract the files:

```
cd /tmp
mt -f /dev/rst0 rewind
tar xf /dev/rst0
```

## Get status of driver and read position of tape ##

```
mt -f /dev/rst0 status
```

**Note:** For all of the above `mt` command one can set the `TAPE` environment variable to forgo the `-f` switch for each invocation. In a future release `mt` will use the same device name convention that the IOSCSITape driver does. Also the `mt` manual page might be of some use.

# Beware of known issues #

  * The current releases have the IOKitDebug property set which means the device may be more verbose and slower performing (though the project itself is compiled for Release rather than Debug).
  * Check the [Issue Tracker](http://code.google.com/p/ioscsitape/issues/list) for any current or known issues.
  * As above your tape drive must be seen by Mac OS X before IOSCSITape can see it. See the [Troubleshooting](Troubleshooting.md) page for more info.