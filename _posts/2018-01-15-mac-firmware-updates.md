---
title: Adventures with installing old Mac firmware updates
---

At work, we have a quad-core 2012 Mac mini Server (Macmini6,2) that's been running VMware ESXi from day one. For this reason it's never had its firmware updated, so today I set about to fix that.

To start, I booted the mini off of an unused laptop in Target Disk Mode via a Thunderbolt cable, and in System Information could see that its Boot ROM Version was `MM61.0106.B00`. According to Apple's list of [current firmware versions](https://support.apple.com/en-ca/HT201518), it should be `MM61.0106.B0A`, so I had work to do.

I started by checking Software Update. Then I waited… and waited… and waited.

> Didn't get a response from the Apple Software Update server.

Strange, because Software Update was working on other machines. Nevertheless, a search for ["Mac mini 2012 firmware update"](https://support.apple.com/downloads/Mac%2520mini%25202012%2520firmware%2520update) on Apple's support site turned up "Mac mini EFI Firmware Update 1.7" as the first update applicable to my model. I downloaded it manually, ran the installer, and rebooted.

It didn't take. It seems that a firmware update won't run on a Mac when booted over Thunderbolt.

Clearly this wasn't going to be as simple as I'd hoped. Fortunately I had a 2.5" drive with a functioning El Capitan system on hand, so after some [surgery](https://www.ifixit.com/Guide/Mac+Mini+Late+2012+Hard+Drive+Replacement/11716) I had my ESXi SSD set aside and the OS X drive installed and booted. With this setup, the above firmware update installed fine. Upon reboot, with the Boot ROM now at `MM61.0106.B03`, the App Store's Software Update was working again, which recommended "Thunderbolt Firmware Update v1.2", which also installed fine. (At this point the Boot ROM may have been `MM61.0106.B04`; I didn't check.) But after that, no further firmware updates were suggested. Something was clearly awry: my search earlier had also shown a v1.8 firmware update for this model, but when I ran it manually, it said:

> Your machine does not need this update.

More digging on Apple's support site found these five firmware updates that are applicable to the 2012 mini, ordered here by post date:

- [Mac mini EFI Firmware Update 1.7](https://support.apple.com/kb/DL1616) (already installed)
- [Thunderbolt Firmware Update v1.2](https://support.apple.com/kb/DL1653) (already installed)
- [Mac EFI Security Update 2015-001](https://support.apple.com/kb/DL1823)
- [Mac mini EFI Firmware Update v1.8](https://support.apple.com/kb/DL1828)
- [Mac EFI Security Update 2015-002](https://support.apple.com/kb/DL1848)

So it seems that firmware updates need to be installed in the order of their release to work. Now, the problem was that manually downloading and running "Mac EFI Security Update 2015-001" failed with:

> This update requires OS X version 10.8.5 or 10.9.5.

Not wanting to sit through an OS reinstallation just for a firmware update, I tried running the installer on the command line. Turns out doing so skips the OS version check, so the installer ran without issue. After mounting the firmware disk image, this is what I ran in Terminal:

    sudo installer -pkg /Volumes/EFI\ Update/Mac2015EFIUpdate.pkg -target /

One reboot later the firmware was installed, and the Boot ROM was now at `MM61.0106.B08`. Now, finally, did "Mac mini EFI Firmware Update v1.8" show up in Software Update. Installing this bumped the Boot ROM to `MM61.0106.B09`, and installing "Mac EFI Security Update 2015-002" (which says it requires OS X 10.9.5) using the same Terminal technique described above bumped it to `MM61.0106.B0A`. Done, finally.

…or not. Apple now [bundles firmware updates](http://maclabs.jazzace.ca/2017/08/05/macos-and-security-updates.html) with macOS patch updates and security updates, so I'd need to install macOS 10.12.6 to get the mini up to `MM61.0106.B1F`. It is entirely possible that the above saga could have been skipped had I done this first.
