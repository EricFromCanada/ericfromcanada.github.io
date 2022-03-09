---
title: Creating macOS installer DMG or ISO disk images for virtualization
---

Anyone who's ever sought to install a macOS / Mac OS X from the past decade within VMware ESXi has discovered the need to first convert the installer application to a bootable disk image file. VMware Fusion will happily create one for you if given an installer application when creating a new VM, but ESXi is not similarly equipped.

A search for how to convert macOS installers to disk images reveals that there are plenty of blog posts, forum threads, and bash scripts dedicated to solving the issue. However, many of these either aren't up to date, aren't easy to understand, or simply don't work.

So, I set out to assemble the collected wisdom on the topic into a relatively comprehensible bash script that, given the path to any macOS installer application from OS X Lion to macOS 12 and beyond, generates a bootable ISO in the current directory. This post explains what it's doing and why.

## Running the script

Download the script from here:

> [Mac-Installer-Imageizer.sh](https://github.com/EricFromCanada/byte-bucket/blob/master/bash/Mac-Installer-Imageizer.sh)

Run it with:

    bash Mac-Installer-Imageizer.sh path-to-installer.app

Note that it'll ask for your password when processing installers for macOS 11 and up.

### Commentary

#### disk image output format

One consistent thread among all the how-tos on this topic is the final step of converting the disk image to ISO. But what if I told you that wasn't necessary? Turns out that both VMware Fusion _and_ ESXi can read a number of .dmg formats without issue.

If you aren't aware, not all .dmg disk images are created equal; there are many ways that the data in such disk images can be stored, as listed in the `hdiutil` man page. In my testing, ESXi 6.0, ESXi 6.7, and VMware Fusion 11 were able to read .dmg images in UDRW (read-write), UDRO (read-only), UDCO (ADC-compressed), and UDZO (zlib-compressed) format. Fusion 11 was also able to read UDBZ (bzip2-compressed) images, but not ESXi.

Unless this value is set to "iso", the script saves in UDZO format, which can shave a gig or two off the final disk image size.

#### check input

The script requires a path to a macOS / OS X installer application. Originally I'd wanted it to prompt for the path interactively, but it wasn't dealing with the escaped spaces that Terminal automatically inserts when you drag a file into the window from the Finder without needless additional complexity.

After saving the value of the passed installer path (which uses [substring removal](https://wiki.bash-hackers.org/syntax/pe#substring_removal) to trim a possible trailing slash), it proceeds only if it can locate the InstallESD.dmg or SharedSupport.dmg file, which is in the same location on all macOS installer versions. The presence of SharedSupport.dmg indicates a macOS 11 or greater installer, which necessitates the use of `createinstallmedia` and `sudo`, so it asks for the user's password in advance.

#### grab OS name from the installer app filename

Since the installer path contains the OS name, this trims the ".app" off the end and passes it through `sed` to drop the text leading up to it. Naturally, this assumes the installer package names haven't been modified.

A tip for any Mac users trying to follow `sed` examples online: Mac/BSD `sed` requires the `-E` flag to get the regex behaviour that Linux's version has by default. Also, I use colons as the regular expression delimiter when processing Mac pathnames; it's the character least likely to show up since the Finder actively prevents naming things with it.

#### create blank destination disk image with single partition

This creates the initial blank disk image in `/tmp`. Some scripts convert BaseSystem.dmg to a writable image and then expand it, but starting fresh is simpler and works just as well. Its maximum size is 16G, but since it's a sparseimage it'll only use as much disk space as it needs. The final argument omits the file extension because `hdiutil` will add one on its own.

Although BaseSystem.dmg has been using the GPT layout since 10.11, a generated disk image using GPT isn't seen as bootable by VMware Fusion or ESXi, in my testing. Haven't looked into why, but it shouldn't be a problem as long as APM continues to work.

#### mount destination / source disk image

Rather than passing `-attach` to the previous command, mounting the newly-created image is done with a separate command to allow including some more option flags:

- `-noverify` skips verification when mounting the image — not required since it's blank and doesn't actually have a checksum set
- `-nobrowse` prevents the mounted volume from showing up in the Finder, which isn't required but can help avoid mishaps
- `-mountpoint` lets the script specify a path at which to mount the image, allowing for hardcoded paths later on — note that this doesn't change the volume name that the Finder would display (if not for the previous flag)

#### for macOS 11+, run `createinstallmedia`

The installer for macOS Big Sur has a drastically different internal structure than those that came before it. Unless someone else does the dirty work of reverse engineering it, this script defers to its built-in `createinstallmedia` command.

Note that if you have `USE_CREATEINSTALLMEDIA=1` already set in your shell environment, it'll attempt to use `createinstallmedia` for other macOS versions as well. It'll only work for 10.13 and later since it omits the `--applicationpath` argument, which was simpler than attempting to deal with a potentially [bugged 10.12 Sierra installer]({% post_url 2020-02-04-multi-macos-install-drive-diskmaker %}).

#### restore boot disk image to destination disk image

Installers for macOS 10.x are processed according to their three varieties:

- 10.7–10.8: InstallESD.dmg is all that's required
- 10.9–10.11: BaseSystem.dmg, found within InstallESD.dmg, is the starting point
- 10.12–10.15: BaseSystem.dmg is now in the same directory as InstallESD.dmg

In each case, `asr` does a block copy of the given disk image to the destination mount point. The colon at the end of the last case avoids a potential error noted in [prepare-iso.sh](https://github.com/geerlingguy/macos-virtualbox-vm/blob/master/prepare-iso.sh#L78). When done, `asr` always remounts the destination disk image, so `sleep 2` gives the system a moment to notice the new volume before proceeding.

#### remount restored disk image with original mount point

Because `asr` does a block copy, the destination disk image's volume assumes the same name as its source. Here the script unmounts the most recently mounted volume, done by `asr` in the previous step, and remounts it at its preferred static path.

#### grab full OS version

Since it's helpful to know precisely which version of macOS will be installed by the resulting disk image, the script fetches it from SystemVersion.plist for inclusion in the final filename.

#### replace Packages link with actual files

For 10.9–10.12, BaseSystem.dmg contains a "Packages" symlink that needs to be replaced with a proper folder for the installer to work when booted. Since `ditto` can't replace a symlink on its own, `rm` removes it and `rsync` copies the data, skipping some firmware updates that won't be used when creating VMs. (Later versions embed firmware updates in a .pkg and can't be so easily omitted.)

Strangely, 10.13–10.15 require the linked folder's contents to instead be copied to the same directory as the symlink, which can be done by `ditto`.

#### copy installer dependencies

For some unknown reason, only 10.10–10.12 require BaseSystem.dmg be copied into the same volume that it was cloned to earlier.

#### detach source and destination disk images

Unmounts the source and destination disk images without fanfare.

The remaining steps in the `finalizeDiskImage` function are applied to all installer versions.

#### resize and compact destination disk image to minimize empty space

These two commands trim the maximum size of the destination sparseimage.

- `hdiutil resize -size min` resizes it to the minimum size as reported by `hdiutil resize -limits`. Note that this only trims space not already claimed by the filesystem that was cloned by `asr`.
- `hdiutil compact` does some further janitorial work to reclaim space, which doesn't find much, but doesn't take long to do so either.

#### convert destination disk image to DMG or ISO

This generates the final disk image in the current directory using the preferred format. By default it specifies the UDZO format to create a zlib-compressed read-only ".dmg" image, which is still a readable format for VMware Fusion / ESXi, VirtualBox, and the like.

Specifying "iso" uses the UDTO format, a.k.a. "DVD/CD master" according to the `hdiutil` man page, which is effectively an ISO with a ".cdr" extension. Since the disk image's filesystem remains solely HFS+, it's still writable if mounted, and the data is stored uncompressed. The subsequent `mv` moves the final image to the current directory and changes its extension to ".iso".

Another way of generating an ISO is to use `hdiutil makehybrid` which, instead of preserving the original HFS+ filesystem, creates a read-only hybrid HFS+/Joilet/UDF filesystem (according to the flags specified) without any unused space. However, unless run with `sudo`, this errors out when encountering files and folders on the source that lack read permissions for the current user.

#### remove original destination disk image

Provided the final disk image exists and has data, the script removes the intermediary sparseimage from `/tmp` and prints a message.
