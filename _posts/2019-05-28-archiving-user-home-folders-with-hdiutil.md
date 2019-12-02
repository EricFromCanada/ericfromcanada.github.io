---
title: Archiving User Home Folders with hdiutil
---

Today while attempting to use System Preferences to delete a user from a macOS 10.13 system and archive their home folder to a disk image, I ran into this error:

> **Unable to delete the user account [name]**

> An error occurred while backing up this user's home folder.

And in the Console, I found this under the _writeconfig_ process:

>     DIHLDiskImageCreate: internal error - missing kDIHLCreateImageComponentsKey in return dictionary

>     DIHLDiskImageCreate failed: 999 (options:{
        "create-content-spec" =     {
            "iff-spec" =         {
                "any-owners" = 1;
                "copy-uid" = 504;
                "no-cross-dev-nodes" = 1;
                scrub = 1;
                url = "file:///Users/username/";
            };
        };
        "create-target-spec" =     {
            "image-type" = UDIF;
            overwrite = 1;
            url = "file:///Users/Deleted%20Users/username.dmg";
        };
        "suppress-uiagent" = 1;
    })

If you're also seeing this, you already know that the few search results for this particular error have no solutions. But while messing around on my own Mac with `hdiutil create` I saw the same error when using its `-copyuid` flag, even though the command was run with `sudo`:

    $ sudo hdiutil create -verbose -ov -copyuid username -srcfolder /Users/username /Users/Shared/username.dmg
    …
    uid 504 does not have ownership of /Users/username/Sites/.localized - setting needAuth to YES
    Scanning…
    Error 80 (Authentication error).
    /Users/username/Sites/.localized: Authentication error
    …
    hdiutil[32231:28746982] DIHLDiskImageCreate: internal error - missing kDIHLCreateImageComponentsKey in return dictionary
    DIHLDiskImageCreate() returned 999
    need-authentication: true
    hdiutil: create: returning 999
    hdiutil: create failed - internal error

The path in question was owned by _root_, and after changing its ownership to _username_ (`sudo chown username /Users/username/Sites/.localized`), the same command worked. It also worked by dropping the `-copyuid` flag entirely, but since (according to the error message above) this is a flag used by System Preferences when archiving accounts, you'll need to search in the to-be-deleted home directory for any files or folders that are either owned by _root_ or unreadable by the owner, and either fix their permissions or delete them outright.

    NAME=username
    sudo find /Users/$NAME -user root
    sudo find /Users/$NAME ! -perm -u=r

These commands will find those files or folders, after changing _username_ to suit.

## Option 1: Solve the 999 error by repairing permissions

To fix the files with wrong ownership, you can use an [under-documented](https://eclecticlight.co/2017/06/15/something-odd-you-cant-fix-sierra-re-introduces-repairing-permissions/)  `diskutil` command to repair permissions within a specified user's home folder. Since that still won't fix files that have been set to be unreadable by their owner, the next command finds within the user's home folder any files whose owner read bit is unset, and pipes their paths to `xargs` for correction with `chmod`.

    NAME=username
    diskutil resetUserPermissions / `id -u $NAME`
    sudo find /Users/$NAME ! -perm -u=r -print0 | xargs -0 sudo chmod u+r

There's always a chance this could break something, but since you're deleting the user anyway, it shouldn't matter.

At this point, deleting and archiving the user should succeed… or at least fail with a [different error](https://medium.com/@ambroselittle/cant-delete-original-admin-user-on-macos-high-sierra-1d79fb438246).

## Option 2: If you want something done right…

Instead of relying on macOS to create the disk image for you, you can always just do it yourself—and add some optimizations that macOS doesn't.

    su username
    cd ~
    chflags nohidden Library
    rm -rf Library/Caches/*
    mkdir -p .Trash
    mv Downloads/*.{dmg,iso} \
        Library/Application\ Support/iCloud/* \
        Library/Application\ Support/MobileSync/Backup/* \
        Library/Application\ Support/SyncServices/* \
        Library/iTunes/i*\ Software\ Updates/* \
        Library/Logs/* \
        .Trash
    exit

Make the Library folder visible, clear its Caches folder, and shove some common disk space hogs into the user's Trash beforehand. This is both reversible and faster than deleting them outright. (For other things you could omit, check the paths that Time Machine always excludes from backups in `/System/Library/CoreServices/backupd.bundle/Contents/Resources/StdExclusions.plist`.)

Once back in another user account's shell, set a bash variable to the username we want to archive, and create a disk image of the home folder using these parameters:

```sh
NAME=username
sudo hdiutil create \
-verbose \
-ov \
-format ULFO \
-fs HFS+ \
-anyowners \
-nocrossdev \
-scrub \
-spotlight \
-volname $NAME \
-srcfolder /Users/$NAME \
/Users/Shared/$NAME.dmg
```

Let me save you a trip to `man hdiutil` with some explanations.

- `-verbose` - show what it's doing in detail.
- `-ov` - overwrite an existing disk image file if necessary. Helps in case you run the command multiple times.
- `-format ULFO` - compress the disk image with lzfse, which has better compression and speed than zlib, although only OS X 10.11 and later will be able to open it. (If that's an issue, use UDZO instead.)
- `-fs HFS+` - manually specify HFS+ rather than APFS (for backwards compatibility) and without a journal, which isn't needed on read-only disk images.
- `-anyowners` - don't fail on errors with setting file ownership.
- `-nocrossdev` - ensure we don't try to grab files that actually reside on another volume.
- `-scrub` - omit the user's Trash and other unwanted paths when creating the disk image.
- `-spotlight` - mentioned only in `hdiutil create -help` and not in the man page, this will create a Spotlight index. Helpful if you want to search the disk image by content later on.
- `-volname $NAME` - set the name of the disk image's mounted volume.
- `-srcfolder /Users/$NAME` - the source for the disk image.
- `/Users/Shared/$NAME.dmg` - where to put the final disk image.

In my test, a 22GB home folder became a 19.8GB disk image when using the method above, and a 26.8GB disk image when archived via System Preferences. This is because the latter method creates a UDRW (UDIF read/write image) with no compression. Strangely, when double-clicked this image is mounted as read-only, and can only be mounted as read-write with `hdiutil attach username.dmg -readwrite`. I don't know how to set or unset this behaviour.
