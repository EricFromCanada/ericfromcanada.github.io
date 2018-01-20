---
title: Installing Java 6 on Tiger and Leopard
---

For another project, I found myself needing to install Java 6 (a.k.a. 1.6) on Mac OS X Tiger and Leopard, both Intel and PowerPC.

If you were using Macs back then, you may recall that Apple's Java 6 was only officially released for Leopard on 64-bit Intel, and didn't become the default JVM until Snow Leopard (at which point a 32-bit Intel version was also made available). You may _not_ recall that several developer previews for Tiger were released, the last of which was [JavaSE6dp9.dmg](https://hell.meiert.org/core/dmg/java-6-dp.dmg), which supplied a PowerPC and 32-bit Intel JVM. With some hackery, it can still be installed on Tiger or Leopard.

### Leopard on 64-bit Intel

As long as your CPU is a Core 2 Duo or later, all that's required is a quick preference change. Since Java 1.5 is still the default on Leopard, open "Java Preferences" in `/Applications/Utilities` and reorder the list of JVMs so that "Java SE 6" is first in the list. The change takes effect immediately, and can be confirmed on the command line with `java -version`.

### Leopard on PowerPC or 32-bit Intel

As the developer preview installer won't work properly on Leopard, instead we'll extract the JVM and install it by hand. First, move aside the Java 6 for 64-bit Intel that's already installed, but non-functional.

    sudo mv /System/Library/Frameworks/JavaVM.framework/Versions/1.6.0{,.old}
    sudo mv /System/Library/Frameworks/JavaVM.framework/Versions/Current{,.old}

(Ah, remember back when all you needed was an admin password to muck about in the system files with wild abandon? Good times.)

Next, use [Pacifist](http://www.charlessoft.com/pacifist_olderversions.html) to extract from the installer package the path `/System/Library/Frameworks/JavaVM.framework/Versions/1.6.0` and copy it to the same location on disk. (You can also burrow into the package using right-click and "Show Package Contents", then double-click to extract "Archive.pax.gz" and find said path in the newly-created "Archive" folder.)

    sudo cp -pR ~/Desktop/JavaSE6Release1.pkg/Contents/Archive/System/Library/Frameworks/JavaVM.framework/Versions/1.6.0 \
        /System/Library/Frameworks/JavaVM.framework/Versions/

You'll then create a new "Current" symlink pointing at the folder you just copied.

    cd /System/Library/Frameworks/JavaVM.framework/Versions/
    sudo ln -s 1.6.0 Current

Now, `java -version` will show that Java 6 is active.

    $ java -version
    java version "1.6.0-dp"
    Java(TM) SE Runtime Environment (build 1.6.0-dp-b88-34)
    Java HotSpot(TM) Core VM (build 1.6.0-b88-17-release, interpreted mode, sharing)

### Tiger on PowerPC or Intel

If you run the installer on a fully updated Tiger system, it'll inform you that "This volume contains a newer version of Java." So, first move aside the files it's checking.

    sudo mv /System/Library/Frameworks/JavaVM.framework/Resources/Info-macos.plist{,.old}
    sudo mv /System/Library/Frameworks/JavaVM.framework/Resources/version.plist{,.old}

After running the installer, open the _new_ "Java Preferences" in `/Applications/Utilities/Java/Java SE 6`. Set "Use version" to "Java SE 6" and reorder the list in "Java Application Runtime Settings" so "Java SE 6" is first. After you click "Save" and quit, confirm that Java 6 is active by running `java -version`.
