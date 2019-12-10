---
title: macOS 10.15 Catalina on VMware ESXi 6.0
---

I have a [Mac mini]({% post_url 2018-01-15-mac-firmware-updates %}) that hosts VMs of every OS X/macOS version from Leopard to Mojave using VMware ESXi 6.0. Because I'm already an old Dutch guy who hates change, I wanted to see if it was possible to get a Catalina VM running without having to upgrade to ESXi 6.5 or 6.7, which would require abandoning its .NET client that I still prefer.

**TL;DR** You need to first create a new installation using VMware Fusion and provide an updated EFI.

To get a new version of macOS running in a VM, one could:

1. upgrade an existing VM from an earlier version,
2. create an installation from scratch by booting a new VM off of installation media, or
3. migrate a virtual disk containing an installation created by another program.

I have not tried the first option. If you have a Mojave VM already running, it might actually work to clone and modify it as described below before running the upgrade.

## First attempt using an ISO

Installing macOS to a freshly-created VM within ESXi requires converting the macOS installer application to a bootable ISO file. This procedure is already [well-documented](https://www.geekrar.com/create-macos-catalina-iso-file/) [elsewhere](https://techsviewer.com/how-to-install-macos-10-15-catalina-on-vmware-on-windows-pc/) since Apple [provides](https://support.apple.com/HT201372) a `createinstallmedia` tool within the installer app itself.

I had no problems creating an ISO as described, but it refused to finish the boot process when attached to a new VM in ESXi—it would always throw the "unbootable volume" symbol at around the 60% mark—despite working in VMware Fusion. I tested ISOs created from the macOS 10.15.1 (15.1.03) installer under 10.11 and 10.13, but both behaved the same way; the workarounds [described in this post](https://www.jomebrew.com/2019/10/macos-catalina-beta-on-vmwwre-esxi-67-u2.html) seem not to work for ESXi 6.0.

## Second attempt using Fusion

The latest version of VMware Fusion (11.5) supports macOS Catalina, so you can use it to create a new VM that boots directly from the installer app. Older versions will need to use the workarounds [described](https://planetvm.net/blog/?p=64552) [here](https://robservatory.com/install-macos-10-15-catalina-in-a-fusion-virtual-machine/). I'm still on Fusion 8.5.10, but following the steps in the linked posts got me a working Catalina VM. (One customization I did make was to boost the disk size to 48GB before running the installer, mostly because Xcode will want there to be a *lot* of free space available before it'll install.)

Next, create a new VM within VMware ESXi using the .NET client. (I haven't fully tested whether using the HTML5 client makes a difference, but this was what worked for me.) These were the modifications I made during setup:

- Configuration: Custom
- Name: 10.15 Catalina
- Virtual Machine Version: 9 (ESXi 5.1 and later)
  - This is the minimum version required for 10.13 and later, in my experience.
- Guest Operating System: Other > Apple Mac OS X 10.8 (64-bit)
- CPUs: 2 cores
- Disk: Do not create a disk

This gives you a VM in need of a disk and a bit of modification.

### Migrate the disk

To get the virtual disk from Fusion into ESXi, use `scp` to copy the VMDK files from your Mac to the server. (Be sure to enable SSH on your ESXi server, and shut down the source VM first!)

    # from Mac to server
    scp -C /path/to/virtual/machine.vmwarevm/Virtual\ Disk.vmdk root@server-ip-address:/tmp

Then SSH into the server and use `vmkfstools` to convert the VMDK to an ESXi-friendly format and a matching filename, located within the VM's directory:

    # while logged into server
    cd /vmfs/volumes/datastore-name/10.15\ Catalina
    vmkfstools -i /tmp/Virtual\ Disk.vmdk 10.15\ Catalina.vmdk -d thin

You should now have a tiny "10.15 Catalina.vmdk" description file and giant "10.15 Catalina-flat.vmdk" data file in the current directory. Use the ESXi client's VM editor to add the disk.

### Replace the firmware

If you try to boot the VM now, it may freeze during the boot process, or fail to boot at all. This is because the EFI firmware provided by ESXi doesn't understand the APFS disk format. This we know thanks to this [fantastic post](https://licson.net/post/vmware-apfs/) that describes how to modify and insert a firmware file into an existing VM, and even provides the modified firmware to save us all the trouble. That version works for 10.13 and 10.14 VMs, but did not for 10.15. Fortunately, all one needs to do is create a new firmware from more up-to-date sources, listed here:

- The latest version of [VMware Workstation Pro](https://www.vmware.com/ca/products/workstation-pro/workstation-pro-evaluation.html) - I used v15.5.1
- [The APFS UEFI Driver extract](https://github.com/darkhandz/XPS15-9550-Mojave/blob/master/CLOVER/drivers64UEFI/ApfsDriverLoader-64.efi?raw=true) - derived from macOS Mojave
- [UEFITool, a tool for editing UEFI BIOS](https://github.com/LongSoft/UEFITool/releases) - download the latest non-*_NE_* release, which retains the ability to make firmware modifications
- [FFS to convert the APFS driver to UEFI module](https://github.com/pbatard/ffs/releases) - unchanged

On Windows 7 or later, grab the `EFI64.ROM` file from an installation of VMware Workstation and modify it according to the above post. Or, just download the version I made: [**efi64_apfs-2019.rom**]({% link /assets/static/efi64_apfs-2019.rom %})

Upload the new `efi64_apfs-2019.rom` file to your VM directory on the server, then SSH into the server again and use a text editor to open the VMX file. Below the `firmware = "efi"` line, add this:

    efi64.filename = "efi64_apfs-2019.rom"

Save and close the file and start up the VM. If all went well, you should have a macOS Catalina VM running under VMware ESXi 6.0!
