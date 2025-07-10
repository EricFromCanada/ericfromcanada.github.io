---
title: Running non-Metal iMovie on macOS Monterey
---

With the release of macOS 10.14 Mojave, Apple dropped support for all Mac models with GPUs that were incapable of running Metal, their new graphics API. At the same time, updated versions of Apple's own apps started assuming that Metal would be available. Workarounds eventually emerged that allowed Mojave and later versions to be installed on non-Metal Macs, but they'd still need to run older versions of [Apple's apps that did not require Metal](https://archive.org/download/apple-apps-for-non-metal-macs).

Although this worked for a while, newer versions of macOS won't allow these older versions to run — double-clicking the app gives an error message stating that you need a newer version. What to do?

For iMovie, the simple answer is:

1. Right-click iMovie.app and select _Show Package Contents_
2. Navigate to _Contents > MacOS_
3. Right-click "iMovie" and select "Open"

Doing so bypasses a block that's built into the OS which actively prevents older versions of these apps from launching. And although it does launch, the Terminal shows that several components within the app are failing to load properly. Can they be fixed?

Of course they can. In the spirit of [those who have gone before me](https://medium.com/@cormiertyshawn895/deep-dive-how-does-retroactive-work-95fe0e5ea49e#d038), I'll show how to fix the above errors and restore double-click launching to iMovie 10.1.12 on macOS 12 Monterey.

## Solve the launch errors

You will need:

- iMovie 10.1.12 in `/Applications`
- iMovie 10.3.8 somewhere else, e.g. `/Users/Shared`
- [Suspicious Package](https://www.mothersruin.com/software/SuspiciousPackage/)
- [Python 2.7.18 installer for macOS](https://www.python.org/downloads/release/python-2718/)

### Fix the MotionEffect plugin

One error occurs because Python 2 is not present, which was removed from macOS as of Monterey 12.7.3. Its absence prevents title previews from displaying and may also cause iMovie to crash when you open a project.

    Error loading /Applications/iMovie.app/Contents/PlugIns/MediaProviders/MotionEffect.fxp/Contents/MacOS/MotionEffect:  dlopen(/Applications/iMovie.app/Contents/PlugIns/MediaProviders/MotionEffect.fxp/Contents/MacOS/MotionEffect, 0x0109): Library not loaded: '/System/Library/Frameworks/Python.framework/Versions/2.7/Python'
      Referenced from: '/Applications/iMovie.app/Contents/Frameworks/Ozone.framework/Versions/A/Ozone'
      Reason: tried: '/System/Library/Frameworks/Python.framework/Versions/2.7/Python' (no such file), '/Library/Frameworks/Python.framework/Versions/2.7/Python' (no such file)

Although installing the final release of Python 2.x for macOS would fix this, I recommend just extracting the one component you need:

1. use Suspicious Package to open `python-2.7.18-macosx10.9.pkg`
2. click _All Files_ and select `Python.framework` in _Library > Frameworks_
3. select _File > Export "Python.framework" to Downloads_
4. in Terminal, enter `sudo mv ~/Downloads/Python.framework /Library/Frameworks/` along with your password when prompted

Try launching iMovie again; you should find it's much more stable.

### Restore camera access

These errors indicate that this iMovie's older MIO framework can't communicate with the system's newer CoreMediaIO framework.

    MIO failed to find a suitable CoreMediaIO
    MIO has no CMIO framework to resolve symbol: kCMIOSampleBufferAttachmentKey_PulldownCadenceInfo
    MIO has no CMIO framework to resolve symbol: kCMIOSampleBufferAttachmentKey_NoDataMarker
    MIO has no CMIO framework to resolve symbol: kCMIOSampleBufferAttachmentKey_HDV2_VAUX
    MIO has no CMIO framework to resolve symbol: kCMIOSampleBufferAttachmentKey_SMPTETime
    MIO has no CMIO framework to resolve symbol: kCMIOSampleBufferAttachmentKey_DiscontinuityFlags
    MIO has no CMIO framework to resolve symbol: kCMIOSampleBufferAttachmentKey_HostTime

Fortunately, this can be fixed by replacing `MIO.framework` inside the app bundle with a newer version. Assuming you have iMovie 10.3.8 (the last supported version for Monterey) in `/Users/Shared`, in Terminal run:

    cd /Applications/iMovie.app/Contents/Frameworks/
    mv MIO.framework /tmp/
    cp -a /Users/Shared/iMovie.app/Contents/Frameworks/MIO.framework .

If you relaunch iMovie and click _Import Media_, you should now see that devices like **FaceTime HD Camera** are listed.

## Hack the app launcher

The app won't launch when double-clicked because an entry in `/System/Library/CoreServices/CoreTypes.bundle/Contents/Resources/Exceptions.plist` prevents an app with the `CFBundleIdentifier` "com.apple.iMovieApp" from launching if its `CFBundleVersion` is 382785 or lower (ours is 347479). So, all we need to do is change that value, which can be done using _plutil_:

    plutil -replace CFBundleVersion -string "382786" /Applications/iMovie.app/Contents/Info.plist

This breaks codesigning, so remove the signature:

    codesign --remove-signature /Applications/iMovie.app

Now try double-clicking the app. Hey look, it launches!

What about other apps from the same timeframe? Well, Keynote 9.1, Numbers 6.1, and Pages 8.1 all seem to launch just fine. GarageBand 10.3.5 failed with a code signature error, but `codesign --remove-signature /Applications/GarageBand.app` took care of that. The excellent [Retroactive](https://github.com/cormiertyshawn895/Retroactive) can handle Aperture, iPhoto, and iTunes; as for the pro apps… well, let me know how it goes.
