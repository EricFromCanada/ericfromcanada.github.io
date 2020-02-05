---
title: Details of Disk Utility’s partitioning decisions
---

I decided it was time I became familiar with the nuts and bolts of how disks on Macs are partitioned by Disk Utility, and why.

After reading up on [MBR](https://en.wikipedia.org/wiki/Master_boot_record), [GPT](https://en.wikipedia.org/wiki/GUID_Partition_Table), and [how they compare](http://www.rodsbooks.com/gdisk/whatsgpt.html), I assembled these tools:

- Disk Utility: the "new" version, as shipped with each version of OS X/macOS
- Disk Utility: the "original" (or "good") version, [patched](https://justus.berlin/2015/10/restore-old-disk-utility-in-os-x-el-capitan/) so it'll still run on OS X 10.11
- `diskutil`: Apple's official disk management tool
- `gpt`: the FreeBSD tool for GPT disks, shipped with macOS
- `fdisk`: a much older tool for MBR disks, also shipped with macOS
- `gdisk`: a newer fdisk-based [tool for GPT disks](http://rodsbooks.com/gdisk/)
    - installable via [Homebrew](https://brew.sh) with `brew install gptfdisk`

## Partition tables, partition tables everywhere

For reference, my internal HFS+ drive according to `diskutil`:
```
$ diskutil list
/dev/disk0 (internal, physical):
   #:                       TYPE NAME                    SIZE       IDENTIFIER
   0:      GUID_partition_scheme                        *1.0 TB     disk0
   1:                        EFI EFI                     209.7 MB   disk0s1
   2:          Apple_CoreStorage Macintosh HD            999.7 GB   disk0s2
   3:                 Apple_Boot Recovery HD             650.0 MB   disk0s3
/dev/disk1 (internal, virtual):
   #:                       TYPE NAME                    SIZE       IDENTIFIER
   0:                  Apple_HFS Macintosh SSD          +999.3 GB   disk1
                                 Logical Volume on disk0s2
                                 FB27242A-A745-4666-AB33-9A934785F0CB
                                 Unencrypted
```
Here we clearly see the internal drive is GPT with a 200MB EFI system partition (ESP), a Core Storage partition containing the boot volume, and a recovery partition. (Run `diskutil cs list` for a better view of how Core Storage volumes are structured.)

The same drive according to `gpt`:
```
$ sudo gpt show /dev/disk0
       start        size  index  contents
           0           1         PMBR
           1           1         Pri GPT header
           2          32         Pri GPT table
          34           6
          40      409600      1  GPT part - C12A7328-F81F-11D2-BA4B-00A0C93EC93B
      409640  1952530904      2  GPT part - 53746F72-6167-11AA-AA11-00306543ECAC
  1952940544     1269536      3  GPT part - 426F6F74-0000-11AA-AA11-00306543ECAC
  1954210080           7
  1954210087          32         Sec GPT table
  1954210119           1         Sec GPT header
```
The `gpt` tool shows everything, and counts according to 512-byte sectors. The [protective MBR](https://en.wikipedia.org/wiki/GUID_Partition_Table#MBR_variants) occupies one 512-byte sector, the GPT header occupies the next sector, and the partition table entries follow. Then at sector 34 there's a 3KB gap so that the first partition starts on a 4KB boundary (i.e. 20KB from the start of the disk). The ESP is _409600 sectors * 512 bytes per sector / 1024 bytes per KB / 1024 KB per MB_ = 200MB. Then our remaining partitions, another boundary alignment gap, and the secondary GPT table and header.

This follows the behaviour described in [Secrets of the GPT](https://developer.apple.com/library/archive/technotes/tn2166/_index.html#//apple_ref/doc/uid/DTS10003927). If you've read that and are wondering why there are no 128MB gaps after each non-ESP partition, note the article doesn't mention that those gaps are only added after HFS+ partitions when defined directly within the partition table, and not when encapsulated within a [Core Storage](https://en.wikipedia.org/wiki/Core_Storage) logical volume group.

For example, an 8GB USB drive partitioned with original Disk Utility to use GPT for two HFS+ partitions looks like:
```
$ sudo gpt show /dev/disk2
     start      size  index  contents
         0         1         PMBR
         1         1         Pri GPT header
         2        32         Pri GPT table
        34         6
        40    409600      1  GPT part - C12A7328-F81F-11D2-BA4B-00A0C93EC93B
    409640   5234376      2  GPT part - 48465300-0000-11AA-AA11-00306543ECAC
   5644016    262144
   5906160   9490664      3  GPT part - 48465300-0000-11AA-AA11-00306543ECAC
  15396824    262151
  15658975        32         Sec GPT table
  15659007         1         Sec GPT header
```
Note the 128MB gap after the first partition, and the 128MB + 7 sector gap after the second. This gap is slightly larger because making the end of the second HFS+ partition align to the next 4KB boundary (8 sectors later) would have made the gap one sector short of 128MB.

But if we repartition so the first is FAT:
```
$ diskutil list /dev/disk2
/dev/disk2 (external, physical):
   #:                       TYPE NAME                    SIZE       IDENTIFIER
   0:      GUID_partition_scheme                        *8.0 GB     disk2
   1:                        EFI EFI                     209.7 MB   disk2s1
   2:       Microsoft Basic Data UNTITLED 1              4.0 GB     disk2s2
   3:                  Apple_HFS Untitled 2              3.7 GB     disk2s3
```
we get:
```
$ sudo gpt show /dev/disk2
gpt show: /dev/disk2: Suspicious MBR at sector 0
     start      size  index  contents
         0         1         MBR
         1         1         Pri GPT header
         2        32         Pri GPT table
        34         6
        40    409600      1  GPT part - C12A7328-F81F-11D2-BA4B-00A0C93EC93B
    409640      2008
    411648   7829504      2  GPT part - EBD0A0A2-B9E5-4433-87C0-68B6B72699C7
   8241152   7155672      3  GPT part - 48465300-0000-11AA-AA11-00306543ECAC
  15396824    262151
  15658975        32         Sec GPT table
  15659007         1         Sec GPT header
```
...a just-under 1MB gap _before_ the FAT volume, but not after. This is OS X following Windows' practice of aligning partitions it creates on [1MB boundaries](https://www.thomas-krenn.com/en/wiki/Partition_Alignment#Windows) for disks over 4GB, which translates to the location of the partition's first sector (411648) being divisible by 2048. (Note that OS X does this for all disk sizes rather than on 64KB/128 sector boundaries for smaller disks, as Windows does.)

As for that "suspicious MBR", we can use `fdisk` to read its master boot record partition table:
```
$ sudo fdisk /dev/disk2
Disk: /dev/disk2	geometry: 974/255/63 [15659008 sectors]
Signature: 0xAA55
         Starting       Ending
 #: id  cyl  hd sec -  cyl  hd sec [     start -       size]
------------------------------------------------------------------------
 1: EE 1023 254  63 - 1023 254  63 [         1 -     409639] <Unknown ID>
 2: 0B 1023 254  63 - 1023 254  63 [    411648 -    7829504] Win95 FAT-32
 3: AF 1023 254  63 - 1023 254  63 [   8241152 -    7155672] HFS+
 4: 00    0   0   0 -    0   0   0 [         0 -          0] unused
```
Aha: the presence of a FAT partition made Disk Utility create a [hybrid MBR](https://www.rodsbooks.com/gdisk/hybrid.html). This would in theory allow the disk to mount on older systems that don't support GPT.

- #1 covers everything from the start of the disk to the end of the ESP and is marked as part of the protective MBR (EE)
- #2 is the FAT (0B) partition
- #3 is the HFS+ (AF) partition

As the linked article explains, hybrid MBRs can be a dangerous hack, and Windows XP can't actually mount this disk as-is unless the hybrid MBR table is reordered with `gdisk` to list any mountable partitions _before_ the EE partition. (Otherwise you'll be told "this partition is not enabled" when assigning a drive letter in Disk Management, and reformatting what shows in My Computer will wipe out the primary GPT structures and the ESP in favour of a 200MB FAT volume!)

The protective MBR that Disk Utility usually creates on disks (meant to actively deter non-GPT-aware disk utilities from thinking the disk is unformatted) looks like this:
```
$ sudo fdisk /dev/disk0
Disk: /dev/disk0	geometry: 121643/255/63 [1954210120 sectors]
Signature: 0xAA55
         Starting       Ending
 #: id  cyl  hd sec -  cyl  hd sec [     start -       size]
------------------------------------------------------------------------
 1: EE 1023 254  63 - 1023 254  63 [         1 - 1954210119] <Unknown ID>
 2: 00    0   0   0 -    0   0   0 [         0 -          0] unused
 3: 00    0   0   0 -    0   0   0 [         0 -          0] unused
 4: 00    0   0   0 -    0   0   0 [         0 -          0] unused
```
where the sole EE partition covers the entire disk.

And if we reformat our 8GB USB drive as GPT with HFS+ and then FAT:
```
$ sudo gpt show /dev/disk2
gpt show: /dev/disk2: Suspicious MBR at sector 0
     start      size  index  contents
         0         1         MBR
         1         1         Pri GPT header
         2        32         Pri GPT table
        34         6
        40    409600      1  GPT part - C12A7328-F81F-11D2-BA4B-00A0C93EC93B
    409640   7829504      2  GPT part - 48465300-0000-11AA-AA11-00306543ECAC
   8239144    264152
   8503296   7153664      3  GPT part - EBD0A0A2-B9E5-4433-87C0-68B6B72699C7
  15656960      2015
  15658975        32         Sec GPT table
  15659007         1         Sec GPT header
```
The gap between #2 and #3 is 262144 sectors trailing the HFS+ partition (the 128MB gap mentioned above) plus 2008 sectors leading the FAT partition so it starts on a megabyte boundary. Also note the 2015 sector trailing gap, which wasn't present when FAT came first. This gives the FAT partition a size that also ends on a megabyte boundary.

And if we use HFS+ followed instead by ExFAT, do we get the same gap and hybrid MBR?
```
$ sudo gpt show /dev/disk2
gpt show: /dev/disk2: Suspicious MBR at sector 0
     start      size  index  contents
         0         1         MBR
         1         1         Pri GPT header
         2        32         Pri GPT table
        34         6
        40    409600      1  GPT part - C12A7328-F81F-11D2-BA4B-00A0C93EC93B
    409640   7826976      2  GPT part - 48465300-0000-11AA-AA11-00306543ECAC
   8236616    262584
   8499200   7157760      3  GPT part - EBD0A0A2-B9E5-4433-87C0-68B6B72699C7
  15656960      2015
  15658975        32         Sec GPT table
  15659007         1         Sec GPT header
```
Here, the gap between #2 and #3 is 262144 sectors (128MB) trailing the HFS+ partition plus 440 sectors leading the ExFAT partition (a 1MB boundary gap, as before), and 2015 sectors trailing it. And "Suspicious MBR" indicates the presence of a hybrid MBR.

## Further down the MBR rabbit hole

Having gone this far, we may as well keep digging. Using the original Disk Utility, I'll format the same 8GB drive as MBR with a variety of partitions.
```
$ diskutil list /dev/disk2
/dev/disk2 (external, physical):
   #:                       TYPE NAME                    SIZE       IDENTIFIER
   0:     FDisk_partition_scheme                        *8.0 GB     disk2
   1:                  Apple_HFS HFS                     2.1 GB     disk2s1
   2:                  Apple_HFS HFSCS                   2.1 GB     disk2s2
   3:                 DOS_FAT_32 FAT                     2.0 GB     disk2s3
   4:               Windows_NTFS EXFAT                   1.7 GB     disk2s5
```
Note that:

- `diskutil` refers to MBR as `FDisk_partition_scheme`
- both standard and case-sensitive HFS+ are `Apple_HFS`
- FAT is marked as `DOS_FAT_32` (so presumably it's not FAT12 or FAT16)
- ExFAT is marked as `Windows_NTFS` (what???)
- all four partitions were supposed to be 2GB, but the last got shorted by a few hundred MB because of the 128MB trailing gap for each HFS+ partition, which `diskutil` includes in their total sizes

Where is `diskutil` getting the partition type info? Presumably from the MBR [partition type IDs](https://en.wikipedia.org/wiki/Partition_type). Can we see those directly?
```
$ sudo gpt show /dev/disk2
     start      size  index  contents
         0         1         MBR
         1         1
         2   4176896      1  MBR part 175
   4176898         1
   4176899   4176896      2  MBR part 175
   8353795         1
   8353796   3914752      3  MBR part 11
  12268548   3390460      4  MBR part 5
```
Neat, `gpt` can read MBR tables. And it does show partition IDs, but in decimal for some reason.

- first sector is the MBR
- second sector is the (unlabelled) [EBR](https://en.wikipedia.org/wiki/Extended_boot_record) which points to the final partition (see below)
- then our HFS+ and case-sensitive HFS+ partitions with just a 1-sector boundary after each, because 128MB gaps are only added on GPT disks (175 == 0xAF)
- then our FAT/ExFAT partitions (11 == 0x0B)

To see both the MBR and EBR contents, use `fdisk`:
```
$ sudo fdisk /dev/disk2
Disk: /dev/disk2	geometry: 974/255/63 [15659008 sectors]
Signature: 0xAA55
         Starting       Ending
 #: id  cyl  hd sec -  cyl  hd sec [     start -       size]
------------------------------------------------------------------------
 1: AF 1023 254  63 - 1023 254  63 [         2 -    4176896] HFS+
 2: AF 1023 254  63 - 1023 254  63 [   4176899 -    4176896] HFS+
 3: 0B 1023 254  63 - 1023 254  63 [   8353796 -    3914752] Win95 FAT-32
 4: 05 1023 254  63 - 1023 254  63 [  12268548 -    3390460] Extended DOS
Signature: 0xAA55
         Starting       Ending
 #: id  cyl  hd sec -  cyl  hd sec [     start -       size]
------------------------------------------------------------------------
 1: 07 1023 254  63 - 1023 254  63 [  12268549 -    3390459] HPFS/QNX/AUX
 2: 00    0   0   0 -    0   0   0 [         0 -          0] unused
 3: 00    0   0   0 -    0   0   0 [         0 -          0] unused
 4: 00    0   0   0 -    0   0   0 [         0 -          0] unused
```
This lets us see the actual hex value [partition type IDs](https://en.wikipedia.org/wiki/Partition_type#List_of_partition_IDs) in the MBR that Disk Utility chose to assign.

- AF is HFS or HFS+
- 0B is FAT32 with CHS addressing (instead of the newer 0C for LBA addressing; I'm guessing this is for max backwards compatibility)
- 05 is the EBR, covering everything past the third partition

And then in the extended partition table:

- 07 is ambiguous — it could be IFS, HPFS, NTFS, ExFAT, or old QNX. And now we know why `diskutil` just shows the last partition as `Windows_NTFS`, because it can't tell which it actually is without reading the partition's data itself.

What does `gdisk` say?
```
$ sudo gdisk /dev/disk2
GPT fdisk (gdisk) version 1.0.4

Warning: Devices opened with shared lock will not have their
partition table automatically reloaded!
Partition table scan:
  MBR: MBR only
  BSD: not present
  APM: not present
  GPT: not present


***************************************************************
Found invalid GPT and valid MBR; converting MBR to GPT format
in memory. THIS OPERATION IS POTENTIALLY DESTRUCTIVE! Exit by
typing 'q' if you don't want to convert your MBR partitions
to GPT format!
***************************************************************

Warning! Main partition table overlaps the first partition by 32 blocks!
You will need to delete this partition or resize it in another utility.

Warning! Secondary partition table overlaps the last partition by
33 blocks!
You will need to delete this partition or resize it in another utility.

Command (? for help): p
Disk /dev/disk2: 15659008 sectors, 7.5 GiB
Sector size (logical): 512 bytes
Disk identifier (GUID): 082F14ED-F53A-4922-8382-3BF9908F4433
Partition table holds up to 128 entries
Main partition table begins at sector 2 and ends at sector 33
First usable sector is 34, last usable sector is 15658974
Partitions will be aligned on 1-sector boundaries
Total free space is 3 sectors (1.5 KiB)

Number  Start (sector)    End (sector)  Size       Code  Name
   1               2         4176897   2.0 GiB     AF00  Apple HFS/HFS+
   2         4176899         8353794   2.0 GiB     AF00  Apple HFS/HFS+
   3         8353796        12268547   1.9 GiB     0700  Microsoft basic data
   5        12268549        15659007   1.6 GiB     0700  Microsoft basic data
```
What a weird message. Is it detecting an old secondary GPT table at the end of the disk, making it think it should still be GPT? This would cause it to think there's partition overlap even though the disk isn't GPT at all. And the last two partitions are labelled the same because `gdisk` is translating the MBR type IDs to their [GPT equivalents](https://en.wikipedia.org/wiki/Microsoft_basic_data_partition), which are the same for FAT and ExFAT.

## Once more time, with GPT

And now we'll repartition with the same volume types, but with GPT instead.
```
$ diskutil list /dev/disk2
/dev/disk2 (external, physical):
   #:                       TYPE NAME                    SIZE       IDENTIFIER
   0:      GUID_partition_scheme                        *8.0 GB     disk2
   1:                        EFI EFI                     209.7 MB   disk2s1
   2:                  Apple_HFS HFS                     2.0 GB     disk2s2
   3:                  Apple_HFS HFSCS                   2.0 GB     disk2s3
   4:       Microsoft Basic Data FAT                     2.0 GB     disk2s4
   5:       Microsoft Basic Data EXFAT                   1.5 GB     disk2s5
```
Note:

- addition of the standard 200MB ESP
- both the standard and case-sensitive HFS+ partitions have the same label
- FAT and ExFAT partitions are both labelled "Microsoft Basic Data"

```
$ sudo gpt show /dev/disk2
gpt show: /dev/disk2: Suspicious MBR at sector 0
     start      size  index  contents
         0         1         MBR
         1         1         Pri GPT header
         2        32         Pri GPT table
        34         6
        40    409600      1  GPT part - C12A7328-F81F-11D2-BA4B-00A0C93EC93B
    409640   3923256      2  GPT part - 48465300-0000-11AA-AA11-00306543ECAC
   4332896    262144
   4595040   3914752      3  GPT part - 48465300-0000-11AA-AA11-00306543ECAC
   8509792    263840
   8773632   3913728      4  GPT part - EBD0A0A2-B9E5-4433-87C0-68B6B72699C7
  12687360   2969600      5  GPT part - EBD0A0A2-B9E5-4433-87C0-68B6B72699C7
  15656960      2015
  15658975        32         Sec GPT table
  15659007         1         Sec GPT header
```
Note:

- we still have a hybrid MBR
- only the HFS+ partitions have a trailing 128MB gap
- the second HFS+ partition's trailing gap is 1696 sectors larger than it needs to be so that the FAT partition that follows starts on a 1MB (2048-sector) boundary
- the full [GPT GUIDs](https://en.wikipedia.org/wiki/GUID_Partition_Table#Partition_type_GUIDs) are:
    - `C12A7328-F81F-11D2-BA4B-00A0C93EC93B` - ESP
    - `48465300-0000-11AA-AA11-00306543ECAC` - HFS+
    - `EBD0A0A2-B9E5-4433-87C0-68B6B72699C7` - Microsoft Basic Data, which could equal MBR type IDs 06, 07, 0B, 01, 04, 0C, 0E

```
$ sudo fdisk /dev/disk2
Disk: /dev/disk2	geometry: 974/255/63 [15659008 sectors]
Signature: 0xAA55
         Starting       Ending
 #: id  cyl  hd sec -  cyl  hd sec [     start -       size]
------------------------------------------------------------------------
 1: EE 1023 254  63 - 1023 254  63 [         1 -     409639] <Unknown ID>
 2: AF 1023 254  63 - 1023 254  63 [    409640 -    3923256] HFS+
 3: AF 1023 254  63 - 1023 254  63 [   4595040 -    3914752] HFS+
 4: 0B 1023 254  63 - 1023 254  63 [   8773632 -    3913728] Win95 FAT-32
```
This hybrid MBR only lists the first four partitions, leaving out the ExFAT one entirely.
```
$ sudo gdisk /dev/disk2
GPT fdisk (gdisk) version 1.0.4

Warning: Devices opened with shared lock will not have their
partition table automatically reloaded!
Partition table scan:
  MBR: hybrid
  BSD: not present
  APM: not present
  GPT: present

Found valid GPT with hybrid MBR; using GPT.

Command (? for help): p
Disk /dev/disk2: 15659008 sectors, 7.5 GiB
Sector size (logical): 512 bytes
Disk identifier (GUID): 53C15232-A63B-437C-A5E3-8110AC399A91
Partition table holds up to 128 entries
Main partition table begins at sector 2 and ends at sector 33
First usable sector is 34, last usable sector is 15658974
Partitions will be aligned on 8-sector boundaries
Total free space is 528005 sectors (257.8 MiB)

Number  Start (sector)    End (sector)  Size       Code  Name
   1              40          409639   200.0 MiB   EF00  EFI System Partition
   2          409640         4332895   1.9 GiB     AF00  HFS
   3         4595040         8509791   1.9 GiB     AF00  HFSCS
   4         8773632        12687359   1.9 GiB     0700  FAT
   5        12687360        15656959   1.4 GiB     0700  EXFAT
```
Nothing special here, except to note that this Disk Utility has set each partition's GPT label, which is distinct from its name that shows in Finder and is not set by the new Disk Utility (except for "EFI System Partition").

By the way, I also ran through the above exercises with a 250GB spinning disk without seeing any changes in behaviour.

So, with original Disk Utility:

- 128MB gaps are only created after HFS+ partitions, and not for EF (ESP), AF05 (Apple Core Storage), or AB00 (Recovery HD) partitions
- the presence of a FAT or ExFAT partition on a GPT disk has it create a hybrid MBR for the first four partitions, including the ESP
- FAT and ExFAT partitions get leading gaps so they start on a 1MB boundary, plus a trailing gap if it's the last partition on the disk

## Trying out the new Disk Utility because we have to

The rewritten Disk Utility that shipped with OS X 10.11 is buggy and underpowered, i.e. changing the partition map only lets you create one volume, and because "free space" is not an option, creating custom-sized and -formatted disk partitions is only possible if you format it as HFS+ first, then click [Partition] to add and change partitions. Even then it won't let you reduce the first to less than 5.4GB, and disks smaller than that can't be repartitioned at all.

### macOS 10.12

Maybe some of its bugs were fixed with the release of macOS 10.12 Sierra, but I still found it not respecting requested partition sizes, not seeing newly inserted disks without a relaunch, not being able to erase a disk on the first attempt, and so on.

Anyway, let's see how it behaves with formatting a USB drive as GPT with HFS+, FAT, and ExFAT.
```
$ diskutil list /dev/disk2
/dev/disk2 (external, physical):
   #:                       TYPE NAME                    SIZE       IDENTIFIER
   0:      GUID_partition_scheme                        *7.7 GB     disk2
   1:                        EFI EFI                     209.7 MB   disk2s1
   2:                  Apple_HFS HFS                     5.4 GB     disk2s2
   3:       Microsoft Basic Data FAT                     993.8 MB   disk2s3
   4:       Microsoft Basic Data ExFAT                   993.8 MB   disk2s4
```
Note the uneven partition sizes. I had to reformat as GPT+HFS, then split the last third of the disk twice because of the arbitrary limitation on shrinking the first partition.
```
$ sudo gpt show /dev/disk2
gpt show: /dev/disk2: Suspicious MBR at sector 0
     start      size  index  contents
         0         1         MBR
         1         1         Pri GPT header
         2        32         Pri GPT table
        34         6
        40    409600      1  GPT part - C12A7328-F81F-11D2-BA4B-00A0C93EC93B
    409640  10577728      2  GPT part - 48465300-0000-11AA-AA11-00306543ECAC
  10987368    262144
  11249512   1941040      3  GPT part - EBD0A0A2-B9E5-4433-87C0-68B6B72699C7
  13190552   1941048      4  GPT part - EBD0A0A2-B9E5-4433-87C0-68B6B72699C7
  15131600         3
  15131603        32         Sec GPT table
  15131635         1         Sec GPT header
```
If you check Disk Utility's output as it does its repartitioning, you'll see it shrinks the initial HFS+ partition, creates a new 2GB HFS+ partition in the empty space, then shrinks that partition to 1GB and creates a third HFS+ partition; only then does it reformat the latter two volumes as FAT and ExFAT. This behaviour may be a factor in why I was seeing inconsistent results. On this drive only the HFS+ partition has its 128MB trailing gap, but on other drives that same gap would still be there after the FAT or ExFAT partitions. Also note that it didn't align those two partitions on 1MB boundaries, as the original Disk Utility would.
```
$ sudo fdisk /dev/disk2
Disk: /dev/disk2	geometry: 941/255/63 [15131636 sectors]
Signature: 0xAA55
         Starting       Ending
 #: id  cyl  hd sec -  cyl  hd sec [     start -       size]
------------------------------------------------------------------------
 1: EE 1023 254  63 - 1023 254  63 [         1 -     409639] <Unknown ID>
 2: AF 1023 254  63 - 1023 254  63 [    409640 -   10577728] HFS+
 3: 0B 1023 254  63 - 1023 254  63 [  11249512 -    1941040] Win95 FAT-32
 4: 07 1023 254  63 - 1023 254  63 [  13190552 -    1941048] HPFS/QNX/AUX
```
But, it's still creating a hybrid MBR.

### macOS 10.13

Can macOS High Sierra do any better? I found that Disk Utility's worst interface quirks had been resolved at this point.

For example, in 10.12 while erasing a drive as GPT+FAT, the dialog failed to convert the default disk name "Untitled" to uppercase, resulting in the volume just being named "U". The 10.13 version now auto-uppercases text in the Erase dialog, although it still doesn't in the Partition dialog, which can lead to failure if anything other than uppercase letters are in your FAT volume's name. (This can happen even if you set a name with lowercase, tab out of the field, and then uppercase the name, because Disk Utility will inexplicably use the _first_ name you typed when it does the partitioning, and then update the volume name at the end of the process.)

It also now allows repartitioning 4GB and smaller drives. Just remember to select _View > Show All Devices_ to see more than just formatted volumes.
```
$ diskutil list /dev/disk2
/dev/disk2 (external, physical):
   #:                       TYPE NAME                    SIZE       IDENTIFIER
   0:      GUID_partition_scheme                        *7.7 GB     disk2
   1:                        EFI EFI                     209.7 MB   disk2s1
   2:                  Apple_HFS HFS                     3.7 GB     disk2s2
   3:       Microsoft Basic Data FAT                     1.8 GB     disk2s3
   4:       Microsoft Basic Data ExFAT                   1.6 GB     disk2s4

$ sudo gpt show /dev/disk2
     start      size  index  contents
         0         1         PMBR
         1         1         Pri GPT header
         2        32         Pri GPT table
        34         6
        40    409600      1  GPT part - C12A7328-F81F-11D2-BA4B-00A0C93EC93B
    409640   7229904      2  GPT part - 48465300-0000-11AA-AA11-00306543ECAC
   7639544    263688
   7903232   3612672      3  GPT part - EBD0A0A2-B9E5-4433-87C0-68B6B72699C7
  11515904    264192
  11780096   3088384      4  GPT part - EBD0A0A2-B9E5-4433-87C0-68B6B72699C7
  14868480    263123
  15131603        32         Sec GPT table
  15131635         1         Sec GPT header

$ sudo fdisk /dev/disk2
Disk: /dev/disk2	geometry: 941/255/63 [15131636 sectors]
Signature: 0xAA55
         Starting       Ending
 #: id  cyl  hd sec -  cyl  hd sec [     start -       size]
------------------------------------------------------------------------
 1: EE 1023 254  63 - 1023 254  63 [         1 -   15131635] <Unknown ID>
 2: 00    0   0   0 -    0   0   0 [         0 -          0] unused
 3: 00    0   0   0 -    0   0   0 [         0 -          0] unused
 4: 00    0   0   0 -    0   0   0 [         0 -          0] unused
```
You can see that Disk Utility isn't bothering to reclaim the 128MB gap left over from when each partition was formatted as HFS+. It is at least adjusting the start and end sectors for FAT and ExFAT partitions to align on 1MB boundaries again. Also note that it's no longer creating a hybrid MBR.

A similar quirk comes up if you erase a disk as GPT+(Ex)FAT, it'll add a leading 1MB boundary gap (and won't format as HFS+ first!):
```
$ sudo gpt show /dev/disk2
     start      size  index  contents
         0         1         PMBR
         1         1         Pri GPT header
         2        32         Pri GPT table
        34         6
        40    409600      1  GPT part - C12A7328-F81F-11D2-BA4B-00A0C93EC93B
    409640      2008
    411648  14718976      2  GPT part - EBD0A0A2-B9E5-4433-87C0-68B6B72699C7
  15130624       979
  15131603        32         Sec GPT table
  15131635         1         Sec GPT header
```
If you then use Partition to change it to HFS+, or select just the volume and use Erase (and so not rewrite the partition table):
```
$ sudo gpt show /dev/disk2
     start      size  index  contents
         0         1         PMBR
         1         1         Pri GPT header
         2        32         Pri GPT table
        34         6
        40    409600      1  GPT part - C12A7328-F81F-11D2-BA4B-00A0C93EC93B
    409640      2008
    411648  14456832      2  GPT part - 48465300-0000-11AA-AA11-00306543ECAC
  14868480    263123
  15131603        32         Sec GPT table
  15131635         1         Sec GPT header
```
it'll add a trailing 128MB gap, but won't reclaim that leading gap.

And of course, as of 10.13 we can create APFS volumes, which always require GPT and are actually containers for sub-partitions that all draw from the same storage pool. This is how a 4GB USB drive with a single APFS container shows up. (Another bug: after formatting, Disk Utility displayed the drive as two separate devices, but after an unplug/replug it displayed properly as _SanDisk > Container disk3 > apfs_.)
```
$ diskutil list
...
/dev/disk2 (external, physical):
   #:                       TYPE NAME                    SIZE       IDENTIFIER
   0:      GUID_partition_scheme                        *4.0 GB     disk2
   1:                        EFI EFI                     209.7 MB   disk2s1
   2:                 Apple_APFS Container disk3         3.8 GB     disk2s2

/dev/disk3 (synthesized):
   #:                       TYPE NAME                    SIZE       IDENTIFIER
   0:      APFS Container Scheme -                      +3.8 GB     disk3
                                 Physical Store disk2s2
   1:                APFS Volume apfs                    905.2 KB   disk3s1
```
Note how an APFS disk takes up two `/dev` entries.
```
$ sudo gpt show /dev/disk2
    start     size  index  contents
        0        1         PMBR
        1        1         Pri GPT header
        2       32         Pri GPT table
       34        6
       40   409600      1  GPT part - C12A7328-F81F-11D2-BA4B-00A0C93EC93B
   409640  7403440      2  GPT part - 7C3457EF-0000-11AA-AA11-00306543ECAC
  7813080        7
  7813087       32         Sec GPT table
  7813119        1         Sec GPT header
```
No trailing gap for APFS containers, just enough to align on 4KB blocks. And still no hybrid MBR, obviously.

My testing on macOS 10.14 didn't find any changes in how its Disk Utility formats disks. I did dig up some more bugs in its interface though, which I've filed with Apple. If any bugfixes or other significant changes surface in Disk Utility with macOS 10.15 I'll update this post.
