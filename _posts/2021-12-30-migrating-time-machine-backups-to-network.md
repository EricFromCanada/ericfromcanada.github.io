---
title: Migrating Time Machine backups from local disk to network
---

In wanting to move my Time Machine backups to a network drive without losing any history, I set to looking up what methods others had used and what quirks to watch out for.

Backups made by Time Machine to a local disk are simply stored in a `Backups.backupdb` folder on the disk, while backups to a network drive are stored in said folder within a sparsebundle disk image, since Time Machine relies on specific features of HFS+ or APFS. The main problem is that there's no official way to preserve your existing backups when making [this type of destination switch](https://web.archive.org/web/20210226200806/https://support.apple.com/en-ca/HT202380), so we need to improvise — and it quickly became apparent that many of the [suggestions](https://apple.stackexchange.com/q/35149) on [how to do so](https://apple.stackexchange.com/q/104277) are either outdated or don't fully understand the situation.

## HFS+ backups (macOS 10.x)

A common suggestion for creating the disk image is to add the network drive as a new backup destination, stop the backup once the disk image has been created, then manually mount the image and replace its `Backups.backupdb` folder with the one from your local drive. If you actually try this though, you're likely to run into either or both of these issues:

1. The system protects Time Machine backups with the `TMSafetyNet.kext` mechanism, so clearing off the disk image requires using either `tmutil delete Backups.backupdb` or running `rm -rf Backups.backupdb` through its [`bypass`](https://superuser.com/a/387464) command, located in `/System/Library/Extensions/TMSafetyNet.kext/Contents/Helpers/bypass`.

1. Because the disk image's mounted volume (named "Time Machine Backups") is formatted as case-sensitive HFS+ ("HFSX"), attempting to copy `Backups.backupdb` with the Finder will throw this error: ["The volume has the wrong case sensitivity for a backup"](https://apple.stackexchange.com/q/221996). Presumably Time Machine used HFSX to ensure compatibility for the narrow set of users who use that on their internal drives, but since Time Machine has no problem copying from HFS+ to HFSX, why the Finder won't let us also do so in this case is a mystery.

Can we just use the command line to copy the data, you might ask? Not so fast: Time Machine on HFS+ relies on hard-linked directories, a concept so volatile that they can only be created with a system call, and therefore so rare that command line utilities don't copy them correctly.

The simplest indicator of a directory being hard-linked is different paths having the same inode number, as shown by `ls -li`.  Note how on the source disk, most of the top-level inode numbers stay the same between backups:

    % ls -li /Volumes/1TB\ External/Backups.backupdb/MacBook\ Pro/2021-12-23-110648/Macintosh\ HD\ -\ Data
    total 0
    124839 drwxrwxr-x@  7 root  admin   238 23 Dec 09:59 Applications
    131322 drwxr-xr-x@ 62 root  wheel  2108  1 Sep 17:25 Library
    114119 drwxr-xr-x@  3 root  wheel   102  6 Jun  2020 System
    132805 drwxr-xr-x@  5 root  admin   170 30 Oct  2020 Users
    131195 drwxr-xr-x@  2 root  wheel    68 23 Dec 10:05 Volumes
    131196 drwxr-xr-x@  2 root  wheel    68  5 Jun  2020 cores
    117892 drwxr-xr-x@  2 root  wheel    68  5 Jun  2020 mnt
    131194 drwxr-xr-x@  2 root  wheel    68  5 Jun  2020 opt
    131595 drwxr-xr-x@  6 root  wheel   204 26 Jul 17:58 private
       207 drwxr-xr-x@  2 root  wheel    68  5 Jun  2020 sw
       208 drwxr-xr-x@  5 root  wheel   170 30 Oct  2020 usr
    % ls -li /Volumes/1TB\ External/Backups.backupdb/MacBook\ Pro/2021-12-23-112359/Macintosh\ HD\ -\ Data
    total 0
    124839 drwxrwxr-x@  7 root  admin   238 23 Dec 09:59 Applications
    157335 drwxr-xr-x@ 62 root  wheel  2108  1 Sep 17:25 Library
    114119 drwxr-xr-x@  3 root  wheel   102  6 Jun  2020 System
    158225 drwxr-xr-x@  5 root  admin   170 30 Oct  2020 Users
    131195 drwxr-xr-x@  2 root  wheel    68 23 Dec 10:05 Volumes
    131196 drwxr-xr-x@  2 root  wheel    68  5 Jun  2020 cores
    117892 drwxr-xr-x@  2 root  wheel    68  5 Jun  2020 mnt
    131194 drwxr-xr-x@  2 root  wheel    68  5 Jun  2020 opt
    157462 drwxr-xr-x@  6 root  wheel   204 26 Jul 17:58 private
       207 drwxr-xr-x@  2 root  wheel    68  5 Jun  2020 sw
    133736 drwxr-xr-x@  5 root  wheel   170 30 Oct  2020 usr

But when copied incorrectly, they're all given distinct numbers:

    % ls -li /Volumes/Time\ Machine\ Backups/Backups.backupdb/MacBook\ Pro/2021-12-23-110648/Macintosh\ HD\ -\ Data
    total 0
    123693 drwxrwxr-x@  5 root  admin   238 23 Dec 09:59 Applications
    129635 drwxr-xr-x@ 61 root  wheel  2108  1 Sep 17:25 Library
    232164 drwxr-xr-x@  3 root  wheel   102  6 Jun  2020 System
    235777 drwxr-xr-x@  4 root  admin   170 30 Oct  2020 Users
    239757 drwxr-xr-x@  2 root  wheel    68 23 Dec 10:05 Volumes
    239758 drwxr-xr-x@  2 root  wheel    68  5 Jun  2020 cores
    239759 drwxr-xr-x@  2 root  wheel    68  5 Jun  2020 mnt
    239760 drwxr-xr-x@  2 root  wheel    68  5 Jun  2020 opt
    239761 drwxr-xr-x@  6 root  wheel   204 26 Jul 17:58 private
    242940 drwxr-xr-x@  2 root  wheel    68  5 Jun  2020 sw
    242941 drwxr-xr-x@  5 root  wheel   170 30 Oct  2020 usr
    % ls -li /Volumes/Time\ Machine\ Backups/Backups.backupdb/MacBook\ Pro/2021-12-23-112359/Macintosh\ HD\ -\ Data
    total 0
    251408 drwxrwxr-x@  5 root  admin   238 23 Dec 09:59 Applications
    257836 drwxr-xr-x@ 61 root  wheel  2108  1 Sep 17:25 Library
    360339 drwxr-xr-x@  3 root  wheel   102  6 Jun  2020 System
    364157 drwxr-xr-x@  4 root  admin   170 30 Oct  2020 Users
    368161 drwxr-xr-x@  2 root  wheel    68 23 Dec 10:05 Volumes
    368162 drwxr-xr-x@  2 root  wheel    68  5 Jun  2020 cores
    368163 drwxr-xr-x@  2 root  wheel    68  5 Jun  2020 mnt
    368164 drwxr-xr-x@  2 root  wheel    68  5 Jun  2020 opt
    368165 drwxr-xr-x@  6 root  wheel   204 26 Jul 17:58 private
    371674 drwxr-xr-x@  2 root  wheel    68  5 Jun  2020 sw
    371676 drwxr-xr-x@  5 root  wheel   170 30 Oct  2020 usr

These are confirmed not to work:

- `cp`: man page: "Note that cp copies hard-linked files as separate files."
- `ditto`: man page: "ditto preserves file hard links (but not directory hard links)…"
- `rsync` preserves hard links when using `-H`, but this only applies to files. To be sure, I copied a small backup set with two snapshots from one disk to another using the flags `-aHAXENxv --fileflags --inplace` to preserve as much as possible, and as predicted, the copied version both took up twice the space, and the inodes of the subfolders of each set did not match as they had in the original, as shown above.
- `asr` is [reportedly unreliable](https://www.cafe-encounter.net/p3034/copying-a-timemachine-backup-to-a-network-drive).
- Although `hdiutil` can create a disk image file directly from a folder or device, it won't let you turn it directly into a sparsebundle, as attempting to do so returns "hdiutil: create: -type cannot be specified with -srcfolder or -srcdevice", which the man page does not mention.
- Disk Utility's **Restore** function can [clone from one local backup drive to another](https://discussions.apple.com/thread/250270273) as long as the Mac is [booted into recovery mode](https://apple.stackexchange.com/q/340432/20773), but that won't work for restoring to a disk image on a network drive since there's no way to mount it.

In fact, [only the Finder](https://support.apple.com/en-ca/guide/mac-help/mh15137/10.15/mac) ([since macOS 10.13.4](https://apple.stackexchange.com/a/323691)) has been [endowed with the ability](https://apple.stackexchange.com/a/368519) to correctly copy hard-linked directories.

The workaround for both issues is to use Disk Utility to reformat the disk image's volume as regular HFS+ ("Mac OS Extended (Journaled)"), then use Finder to copy the data. This may take hours (or even days) to complete, and [may end abruptly with strange errors](https://dancarrphotography.com/blog/2020/03/05/the-best-way-to-copy-time-machine-backups/); user reports vary depending on the macOS version used.

An alternative—and potentially more reliable—route is to use [SuperDuper!](https://www.shirt-pocket.com/SuperDuper/) to move the wholesale contents of the disk to the disk image. It reformats the disk image volume as HFS+ and, by some dark wizardry, preserves hard-linked directories.

There is room for improvement, though. Sparsebundle disk images store their data in a series of equal-sized "band" files which by default are 8MB each. Because Time Machine didn't bother using a larger size until macOS 10.15 (briefly [256MB](https://eclecticlight.co/2019/11/11/time-machine-and-backing-up-in-catalina/#comment-46416) before settling on 64MB), backups to disk images created on earlier macOS versions are much slower than they need to be. Everyone seems to [agree](https://edoardofederici.com/improve-time-machine-performance/) that a band size of [128MB](https://gist.github.com/SebastianJ/b3e9af8a641df1dc73c0) is [ideal](https://arzur.net/blog/2010/08/31/time-machine-on-a-network-drive-you-will-need-to-increase-the-band-size/) for _unencrypted_ backups; anyone encrypting their backups may want to experiment first or [stick with the defaults](https://www.artembutusov.com/time-machine-backup-disk-migration-to-network-drive/).

To create a disk image with a 128MB band size and all the necessary metadata for your Mac, use my [_Time-Machine-NASifier.command_](https://github.com/EricFromCanada/byte-bucket/blob/master/bash/Time-Machine-NASifier.command) script:

1. Copy it to the network volume you'll be backing up to and run it (right-click and select _Open_ to get past Gatekeeper). It'll create a new disk image in the same directory.
1. Run `hdiutil resize -size <num>g /path/to/diskimage` to set the new disk image's max size to be enough to hold your backups. (Or just edit the script before running.)
    - Time Machine will later resize the disk image's max size to up to twice that of the source volume.
1. Use SuperDuper!'s **Backup - all files** option to clone from the old disk to the new disk image.
1. In _System Preferences > Time Machine_, remove the original backup destination and start backing up to the network drive instead.
    - This must be done last, because doing so places the disk image volume under `TMSafetyNet.kext` protection, which prevents SuperDuper! from showing it as a destination.

This is what I ended up doing, and although the copy took about 54 hours to copy almost a terabyte, it completed successfully.

## APFS backups (macOS 11+)

The bad news is that because new Time Machine backup sets since macOS Big Sur rely on [APFS snapshots](https://eclecticlight.co/2020/06/29/apfs-changes-in-big-sur-how-time-machine-backs-up-to-apfs-and-more/), it's [not currently possible](https://eclecticlight.co/2021/03/24/the-trouble-with-snapshots-how-can-you-copy-them/) to clone drives containing them. Using the Finder to drag a Time Machine snapshot from one drive to another gives the "disallow" cursor, `asr` and other utilities make no mention of cloning snapshot data, and attempting a restore with Disk Utility fails with "Error finding volume with appropriate role in container". As for SuperDuper! v3.5, it will clone a Time Machine drive to a disk image, but the resulting volume will still appear empty, even though it takes up equivalent space.

The good news is that [_Time-Machine-NASifier.command_](https://github.com/EricFromCanada/byte-bucket/blob/master/bash/Time-Machine-NASifier.command) still works for creating a disk image with a larger band size than the [default 64MB](https://eclecticlight.co/2021/04/16/time-machine-to-apfs-using-a-network-share/). When run on macOS 11 or later, it'll automatically create a case-sensitive APFS disk image instead.
