## AMD RaidXpert drivers "updated" for linux 6.x

There are just a few subtle modifications here, but all that is required for the
driver to work should be available.

This is just a temporary solution, hopefully AMD will release updates soon.
It's not very likely that these drivers will be included as part of the kernel
in their current form any time soon, because of the:

### Licensing fun

There are no clear licenses anywhere, other than an explicit mention that there
are two distinct licenses, one governing the rights to the blob, and other - to
the source files.

Part of the license mentions that:

```
(iii) this source code is the confidential information of DHS and may not be
disclosed to any third party;
```

whatever this means. The sources are openly accessible from AMDs download
page; in fact - this is how these have been acquired.

Lastly, there are no mentions that the open-source part cannot be modified, other
than:

```
(iv) you may not make any modification or take any action that would cause this
software (...) to fall under any GPL license or any other open source license.
```

No such modifications have been made, warranting that the source continues to
remain licensed the same way it was when it was released by AMD.

## Building

Follow regular linux development process, or go through the official building
details included by AMD (I've left them here).

Once you get everything installed, just call `make -C src`,

## Loading

To make this work, you have to either

### System boot.

This requires you to blacklist both `ahci` and `nvme` modules and placing the
`rcraid.ko` alongside the modules. This part is documented by AMD already.

Note that each time the kernel is updated, you will have to redo this process
again, or your system will just be inaccessible.

Simply build and install the package:

```
dpkg-buildpackage -b -uc -us
sudo dpkg -i ../rcraid-dkms-*.deb
```

NOTE: This is experimental. This may not work well. Be prepared that this may
render your system non-bootable. Have a way to revert. Normally this means
booting unmodified kernel, or perhaps the previously installed kernel.

### On demand.

Requires you to blacklist just the `ahci` module and manually unbind NVMe
drives that are part of your RAID setup.

To do the latter - this script will match PCI devices and their `/dev/nvme*`
nodes:

```
cd /sys/bus/pci/drivers/nvme
for f in 0000:*; do echo $f is $(ls $f/nvme); done
```

e.g.:

```
0000:4c:00.0 is nvme3
0000:21:00.0 is nvme0
0000:43:00.0 is nvme1
0000:44:00.0 is nvme2
```

Next, for each of these nodes that you know that are part of the RAIDXpert
setup, unbind them:

```
cd /sys/bus/pci/drivers/nvme
echo 0000:44:00.0 | sudo tee unbind
echo 0000:4c:00.0 | sudo tee unbind
```

Note: `/dev/nvme2*` and `/dev/nvme3*` will become instantly unavailable.

Finally, load the module:

```
sudo insmod rcraid.ko
```

A simple script can go a long way here.

Good luck.
