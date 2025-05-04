---
title: The ignominious finale of QuickTime 7
---

It started with a seemingly simple question: "Which version of QuickTime should I install on my retro 10.5 Leopard for Intel installation?"

Quick recap: back in the early days of Mac OS X, when dinosaurs like the WMV format still roamed the internet and H.264 had not yet completely taken over, QuickTime was a separate component within OS X that, like iTunes and Java, had its own update cadence apart from regular OS updates. Upon the release of Mac OS X 10.6 Snow Leopard and its deprecation in favour of QuickTime X, its updates were rolled into those for the OS while support for previous OS X versions and Windows were eventually dropped.

The obvious answer would be "the latest version". According to these documentation pages that are still online (and [this 2017 snapshot](https://web.archive.org/web/20170702062047/https://support.apple.com/en_US/downloads/quicktime)), the latest for each platform is:

- [QuickTime 7.6.4 for Tiger and Leopard](https://support.apple.com/en-us/docs/software/pl171)
- [QuickTime 7.7.9 for Windows](https://support.apple.com/en-us/docs/software/pl170)

But digging into the [history of QuickTime 7 releases](https://en.wikipedia.org/wiki/QuickTime#QuickTime_7.x) shows that 7.6.6, 7.6.9, and 7.7 were released for Mac OS X Leopard. Also, no QuickTime updates are shown in Software Update on Leopard; unless they're installed manually, it'll stay at 7.2.1. So what happened?

From what I've found, the 7.7 release had some nasty bugs that affected pro users in particular. Although Apple did eventually release a stealth update to 7.7 by uploading an updated installer with the same version number to their support site, its only difference was a newer installer certificate that expired in 2019 instead of 2012 (a mere eight months after the original installer's release). Apple never officially acknowledged the issue and simply removed versions after 7.6.4 from their support site's search results.

Here's a timeline:
- 2010-12-07: QuickTime 7.6.9 is released.
    - [info page from 2011-07-19](https://web.archive.org/web/20110719050227/http://support.apple.com/kb/DL761)
- 2011-08-03: QuickTime 7.7 is released. Shortly thereafter, various forum threads begin to [describe problems with the update](https://www.digitalrebellion.com/blog/posts/critical_problems_with_quicktime_7.7_and_quicktime_7.6.6_build_1787).
    - This [7.7 info page from 2012-01-11](https://web.archive.org/web/20120111175044/http://support.apple.com/kb/DL761) (and this [older URL from 2012-02-10](https://web.archive.org/web/20120210023529/http://support.apple.com:80/downloads/QuickTime_7_6_for_Leopard)) shows `0deb99cc44015af7c396750d2c9dd4cbd59fb355` for the download's SHA1 value.
    - The installer package within the [original `QuickTime770_Leopard.dmg`](https://download.info.apple.com/Mac_OS_X/041-0741-20110803.1VdPF/QuickTime770_Leopard.dmg) has a modification date of 2011-07-15.
- 2011-12-19: an Apple employee [posts in this issue thread](https://discussions.apple.com/thread/3241946?answerId=3241946021&sortBy=rank#3241946021) requesting more information.
- At some point in early 2012, a new installer with the same version number is uploaded.
    - This [7.7 info page from 2012-08-03](https://web.archive.org/web/20120803192255/http://support.apple.com/kb/DL761) (and this [older URL from 2012-05-18](https://web.archive.org/web/20120518134655/http://support.apple.com:80/downloads/QuickTime_7_6_for_Leopard)) shows `6cfc51c2c23413f86a05c63b7dc2c2c2690fad28` for the download's SHA1 value.
    - The installer package within the [modified `QuickTime770_Leopard.dmg`](https://download.info.apple.com/Mac_OS_X/041-4967.20110803.1VdPF/QuickTime770_Leopard.dmg) has a modification date of 2012-03-13.

In conclusion: 7.6.9 is probably fine, but there's little point in going past 7.6.4. The issues wrought by 7.7 were fixed for Snow Leopard and Lion in subsequent updates, but Apple chose not to bless holdouts on Leopard (and therefore PowerPC) with the same fixes. This is just another aspect of how the PowerPC-Intel transition was relatively sudden and harsh compared to the Intel-ARM transition.

Side note: although this [QuickTime 7 Player installer](https://support.apple.com/en-us/106391) for Snow Leopard and later is versioned as 7.6.6, it contains only the Player application, and not any system-level components.

## Released versions

In then process of writing this post, I ended up assembling a list of every Mac and Windows release of QuickTime 7 with links to their archived downloads and info pages. Hyper-fixation will do that.

### QuickTime 7.0

#### 7.0
April 29, 2005: Initial release for Mac OS X 10.3.9 Panther and included with Mac OS X 10.4 Tiger.
- info: [Mac](https://web.archive.org/web/20050507234217/http://www.apple.com/quicktime/download/mac.html)
- download: [Mac](https://web.archive.org/web/20050507063909if_/http://appldnld.m7z.net:80/qtinstall.info.apple.com/simca/us/OSX/QuickTimeInstallerX.dmg)

#### 7.0.1
- [security](https://web.archive.org/web/20081222084631/http://support.apple.com/kb/TA23292)
- info: [Mac](https://web.archive.org/web/20050610023707/http://www.apple.com/support/downloads/quicktime701.html)
- download: [Mac](https://web.archive.org/web/20050604000128if_/http://appldnld.m7z.net:80/qtinstall.info.apple.com/smaltblue/us/OSX/QuickTimeInstallerX.dmg)

#### 7.0.2
September 7, 2005: First stable release for Windows 2000 and XP.
- info: [Mac(1)](https://web.archive.org/web/20051001110045/http://www.apple.com/support/downloads/quicktime702.html) [Mac(2)](https://web.archive.org/web/20060217182733/http://www.apple.com/support/downloads/quicktime702formac.html) [Windows](https://web.archive.org/web/20051102073851/http://www.apple.com/support/downloads/quicktime702forwindows.html)
- download: [Mac[not archived]](https://web.archive.org/web/20051012171246if_/http://appldnld.m7z.net/qtinstall.info.apple.com/cisitalia/us/OSX/QuickTimeInstallerX.dmg) [Windows](https://web.archive.org/web/20051012171246if_/http://appldnld.m7z.net/qtinstall.info.apple.com/cisitalia/us/win/QuickTimeInstaller.exe)

#### 7.0.3
- [security](https://web.archive.org/web/20110521212408/http://support.apple.com/kb/TA23702)
- info: [Mac/Windows](https://web.archive.org/web/20051204041810/http://www.apple.com/support/downloads/quicktime703.html)
- download: [Mac](https://web.archive.org/web/20051016025538if_/http://appldnld.m7z.net/qtinstall.info.apple.com/boots/us/OSX/QuickTimeInstallerX.dmg) [Windows](https://web.archive.org/web/20051018052138if_/http://appldnld.m7z.net/qtinstall.info.apple.com/boots/us/win/QuickTimeInstaller.exe)

#### 7.0.4
January 10, 2006: First universal binary release (PowerPC/Intel) for Mac OS X.
- [security](https://web.archive.org/web/20110226133647/http://support.apple.com/kb/TA23845)
- info: [Mac/Windows](https://web.archive.org/web/20060113125043/http://www.apple.com/support/downloads/quicktime704.html)
- download: [Mac](https://web.archive.org/web/20070814215429if_/http://appldnld.m7z.net/qtinstall.info.apple.com/snape/us/OSX/QuickTimeInstallerX.dmg) [Windows](https://web.archive.org/web/20110811151048if_/http://appldnld.m7z.net/qtinstall.info.apple.com/snape/us/win/QuickTimeInstaller.exe)

### QuickTime 7.1

#### 7.1
- [security](https://web.archive.org/web/20081229060821/http://support.apple.com/kb/TA24130)
- info: [Mac](https://web.archive.org/web/20060521031711/http://www.apple.com/support/downloads/quicktime71.html) [Windows](https://web.archive.org/web/20061108005408/http://www.apple.com/support/downloads/quicktime71forwindows.html)
- download: [Mac[not archived]](https://web.archive.org/web/20070624122454if_/http://appldnld.apple.com.edgesuite.net/qtinstall.info.apple.com/lupin/us/OSX/QuickTimeInstallerX.dmg) [Windows](https://web.archive.org/web/20070624122454if_/http://appldnld.apple.com.edgesuite.net/qtinstall.info.apple.com/lupin/us/win/QuickTimeInstaller.exe)

#### 7.1.1
Mac-only release.
- info: [Mac](https://web.archive.org/web/20060613190024/http://www.apple.com/support/downloads/quicktime711.html)
- download: [Mac[not archived]](https://web.archive.org/web/20061124115427if_/http://appldnld.apple.com.edgesuite.net/qtinstall.info.apple.com/diggery/us/osx/QuickTimeInstallerX.dmg)

#### 7.1.2
Mac-only release.
- info: [Mac](https://web.archive.org/web/20060703153009/http://www.apple.com/support/downloads/quicktime712.html)
- download: [Mac[not archived]](https://web.archive.org/web/20061124115427if_/http://appldnld.apple.com.edgesuite.net/qtinstall.info.apple.com/buckbeak/us/osx/QuickTimeInstallerX.dmg)

#### 7.1.3
- [security](https://web.archive.org/web/20090125081719/http://support.apple.com/kb/TA24355)
- info: [Mac(1)](https://web.archive.org/web/20061206001825/http://www.apple.com/support/downloads/quicktime713.html) [Mac(2)](https://web.archive.org/web/20070310183551/http://www.apple.com/support/downloads/quicktime713formac.html) [Windows](https://web.archive.org/web/20070309142434/http://www.apple.com/support/downloads/quicktime713forwindows.html)
- download: [Mac](https://web.archive.org/web/20061124115427if_/http://appldnld.apple.com.edgesuite.net/qtinstall.info.apple.com/mcgonagail/us/osx/QuickTimeInstallerX.dmg) [Windows](https://web.archive.org/web/20061116103906if_/http://appldnld.apple.com.edgesuite.net/qtinstall.info.apple.com/mcgonagail/us/win/QuickTimeInstaller.exe)

#### 7.1.5
- [security](https://web.archive.org/web/20090511000525/http://support.apple.com/kb/HT2243)
- info: [Mac](https://web.archive.org/web/20070307083105/http://www.apple.com/support/downloads/quicktime715formac.html) [Windows](https://web.archive.org/web/20070308094652/http://www.apple.com/support/downloads/quicktime715forwindows.html)
- download: [Mac](https://web.archive.org/web/20250424152152if_/http://appldnld.apple.com/QuickTime/061-2833.20070301.uH75q/QuickTime715.dmg) [Windows](https://web.archive.org/web/20160222104156if_/http://appldnld.apple.com/QuickTime/061-2831.20070301.qTS15/QuickTimeInstaller.exe)

#### 7.1.6
May 1, 2007: Final release for Windows 2000.
- [security](https://web.archive.org/web/20090331083817/http://support.apple.com/kb/HT2244)
- info: [Mac](https://web.archive.org/web/20070504053721/http://www.apple.com/support/downloads/quicktime716formac.html) [Windows](https://web.archive.org/web/20070503231633/http://www.apple.com/support/downloads/quicktime716forwindows.html)
- download: [Mac](https://web.archive.org/web/20211011053037if_/http://appldnld.apple.com/QuickTime/061-3149.20070501.iNqt4/QuickTime716.dmg) [Windows](https://web.archive.org/web/20190619192906if_/http://appldnld.apple.com/QuickTime/061-3155.20070501.qbBt4/QuickTimeInstaller.exe)

#### Security Update for QuickTime 7.1.6
May 29, 2007: Final update for Windows 2000.
- [security](https://web.archive.org/web/20090331165921/http://support.apple.com/kb/TA24733)
- info: [Mac](https://web.archive.org/web/20070601200927/http://www.apple.com/support/downloads/securityupdatequicktime716formac.html) [Windows](https://web.archive.org/web/20070601205701/http://www.apple.com/support/downloads/securityupdatequicktime716forwindows.html)
- download: [Mac](https://web.archive.org/web/20140629160115if_/http://supportdownload.apple.com/download.info.apple.com/Apple_Support_Area/Apple_Software_Updates/Mac_OS_X/downloads/061-3447.20070529.mU8y6/SecUpdQuickTime716.dmg) [Windows](https://web.archive.org/web/20180103172512if_/http://supportdownload.apple.com/download.info.apple.com/Apple_Support_Area/Apple_Software_Updates/Mac_OS_X/downloads/061-3451.20070529.o08v3/SecUpdQuickTime716.msi)

### QuickTime 7.2

#### 7.2
July 11, 2007: First release for Windows Vista.
- [security](https://web.archive.org/web/20090125173745/http://support.apple.com/kb/TA24829)
- info: [Mac](https://web.archive.org/web/20070823114821/http://www.apple.com/support/downloads/quicktime72formac.html) [Windows](https://web.archive.org/web/20070824054518/http://www.apple.com/support/downloads/quicktime72forwindows.html)
- download: [Mac](https://web.archive.org/web/20200301121906if_/http://appldnld.apple.com/QuickTime/061-2912.20070710.Fby76/QuickTime720.dmg) [Windows](https://web.archive.org/web/20180321164047if_/http://appldnld.apple.com/QuickTime/061-2915.20070710.pO94c/QuickTimeInstaller.exe)

#### Compatibility Update for QuickTime 7.2 for Mac
- info: [Mac](https://web.archive.org/web/20070921025708/http://www.apple.com/support/downloads/compatibilityupdateforquicktime72.html)
- download: [Mac](https://web.archive.org/web/20120127134331if_/http://supportdownload.apple.com/download.info.apple.com/Apple_Support_Area/Apple_Software_Updates/Mac_OS_X/downloads/061-3870.20070911.U8t5N/QT72CompatibilityUpdate.dmg)

#### Security Update for QuickTime 7.2 for Windows
- [security](https://web.archive.org/web/20090427130352/http://support.apple.com/kb/HT2168)
- info: [Windows](https://web.archive.org/web/20071017162951/http://www.apple.com/support/downloads/securityupdateforquicktime72forwindows.html)
- download: [Windows](https://web.archive.org/web/20250424163506if_/http://download.info.apple.com/Mac_OS_X/061-3937.20071003.oNy6R/SecUpdQuickTime720.msi)

#### 7.2.1
October 26, 2007: Included with Mac OS X 10.5 Leopard.

### QuickTime 7.3

#### 7.3
November 5, 2007: First update for Mac OS X 10.5 Leopard.
- [security](https://web.archive.org/web/20090524025935/http://support.apple.com/kb/HT2047)
- info: [Panther](https://web.archive.org/web/20071106154457/http://www.apple.com/support/downloads/quicktime73forpanther.html) [Tiger](https://web.archive.org/web/20071106154502/http://www.apple.com/support/downloads/quicktime73fortiger.html) [Leopard](https://web.archive.org/web/20071106154452/http://www.apple.com/support/downloads/quicktime73forleopard.html) [Windows](https://web.archive.org/web/20071214092143/http://www.apple.com/support/downloads/quicktime73forwindows.html)
- download: [Panther](https://web.archive.org/web/20250424204451if_/http://appldnld.apple.com/QuickTime/061-3763.20071101.Vb5Gh/QuickTime730_Panther.dmg) [Tiger](https://web.archive.org/web/20250424171058if_/http://appldnld.apple.com/QuickTime/061-3392.20071101.6Gb2S/QuickTime730_Tiger.dmg) [Leopard](https://web.archive.org/web/20250424203802if_/http://appldnld.apple.com/QuickTime/061-3922.20071101.9Gb6t/QuickTime730_Leopard.dmg) [Windows](https://web.archive.org/web/20180321164517if_/http://appldnld.apple.com/QuickTime/061-3394.20071101.Xc5Tg/QuickTimeInstaller.exe)

#### 7.3.1
- [security](https://web.archive.org/web/20090123193623/http://support.apple.com/kb/HT1738)
- info: [Panther](https://web.archive.org/web/20080116052514/http://www.apple.com/support/downloads/quicktime731forpanther.html) [Tiger](https://web.archive.org/web/20080117105543/http://www.apple.com/support/downloads/quicktime731fortiger.html) [Leopard](https://web.archive.org/web/20080114010643/http://www.apple.com/support/downloads/quicktime731forleopard.html) [Windows](https://web.archive.org/web/20080112112947/http://www.apple.com/support/downloads/quicktime731forwindows.html)
- download: [Panther](https://web.archive.org/web/20200228095643if_/http://appldnld.apple.com/QuickTime/061-4126.20071213.wgD6H/QuickTime731_Panther.dmg) [Tiger](https://web.archive.org/web/20200228095642if_/http://appldnld.apple.com/QuickTime/061-4127.20071213.3Ujvo/QuickTime731_Tiger.dmg) [Leopard](https://web.archive.org/web/20200228095643if_/http://appldnld.apple.com/QuickTime/061-4130.20071213.Xy54EQ/QuickTime731_Leopard.dmg) [Windows](https://web.archive.org/web/20180103164202if_/http://appldnld.apple.com/QuickTime/061-4128.20071213.8w6nZ/QuickTimeInstaller.exe)

### QuickTime 7.4

#### 7.4
- [security](https://web.archive.org/web/20090402065837/http://support.apple.com/kb/HT1521)
- info: [Panther](https://web.archive.org/web/20080304005622/http://www.apple.com/support/downloads/quicktime74forpanther.html) [Tiger](https://web.archive.org/web/20080316013128/http://www.apple.com/support/downloads/quicktime74fortiger.html) [Leopard](https://web.archive.org/web/20080309144809/http://www.apple.com/support/downloads/quicktime74forleopard.html) [Windows](https://web.archive.org/web/20080315205526/http://www.apple.com/support/downloads/quicktime74forwindows.html)
- download: [Panther](https://web.archive.org/web/20200301091333if_/http://appldnld.apple.com/QuickTime/061-4024.20080115.ScV4G/QuickTime740_Panther.dmg) [Tiger](https://web.archive.org/web/20200301091359if_/http://appldnld.apple.com/QuickTime/061-4021.20080115.p9m3F/QuickTime740_Tiger.dmg) [Leopard](https://web.archive.org/web/20200301091346if_/http://appldnld.apple.com/QuickTime/061-4015.20080115.uYrtH/QuickTime740_Leopard.dmg) [Windows](https://web.archive.org/web/20160221222101if_/http://appldnld.apple.com/QuickTime/061-4027.20080115.3gGs6/QuickTime740Installer.exe)

#### 7.4.1
- [security](https://web.archive.org/web/20090225150627/http://support.apple.com/kb/HT1323)
- info: [Panther](https://web.archive.org/web/20080313082428/http://www.apple.com/support/downloads/quicktime741forpanther.html) [Tiger](https://web.archive.org/web/20080317222857/http://www.apple.com/support/downloads/quicktime741fortiger.html) [Leopard](https://web.archive.org/web/20080319081432/http://www.apple.com/support/downloads/quicktime741forleopard.html) [Windows](https://web.archive.org/web/20080306020437/http://www.apple.com/support/downloads/quicktime741forwindows.html)
- download: [Panther](https://web.archive.org/web/20200301091333if_/http://appldnld.apple.com/QuickTime/061-4281.20080207.Thphr7/QuickTime741_Panther.dmg) [Tiger](https://web.archive.org/web/20200301091333if_/http://appldnld.apple.com/QuickTime/061-4282.20080207.Tgr7r/QuickTime741_Tiger.dmg) [Leopard](https://web.archive.org/web/20200301091333if_/http://appldnld.apple.com/QuickTime/061-4248.20080207.LprtQ6/QuickTime741_Leopard.dmg) [Windows](https://web.archive.org/web/20160211164948if_/http://appldnld.apple.com/QuickTime/061-4283.20080207.WnQT74/QuickTimeInstaller.exe)

#### 7.4.5
- [security](https://web.archive.org/web/20090118063145/http://support.apple.com/kb/HT1241)
- info: [Panther](https://web.archive.org/web/20080407111402/http://www.apple.com/support/downloads/quicktime745forpanther.html) [Tiger](https://web.archive.org/web/20080407081857/http://www.apple.com/support/downloads/quicktime745fortiger.html) [Leopard](https://web.archive.org/web/20080407074509/http://www.apple.com/support/downloads/quicktime745forleopard.html) [Windows](https://web.archive.org/web/20080418115739/http://www.apple.com/support/downloads/quicktime745forwindows.html)
- download: [Panther](https://web.archive.org/web/20231226004643if_/http://appldnld.apple.com/QuickTime/061-4330.20080402.bgt5W/QuickTime745Panther.dmg) [Tiger](https://web.archive.org/web/20250424173812if_/http://appldnld.apple.com/QuickTime/061-4331.20080402.gt4RX/QuickTime745Tiger.dmg) [Leopard](https://web.archive.org/web/20250424173920if_/http://appldnld.apple.com/QuickTime/061-4321.20080402.TQlp3/QuickTime745Leopard.dmg) [Windows](https://web.archive.org/web/20250424173947if_/http://appldnld.apple.com/QuickTime/061-4318.20080402.n7Ypc/QuickTimeInstaller.exe)

### QuickTime 7.5

#### 7.5
June 9, 2008: Final release for Mac OS X 10.3 Panther.
- [security](https://web.archive.org/web/20091223082203/http://support.apple.com/kb/HT1991)
- info: [Panther](https://web.archive.org/web/20080726124041/http://www.apple.com/support/downloads/quicktime75forpanther.html) [Tiger](https://web.archive.org/web/20080726122928/http://www.apple.com/support/downloads/quicktime75fortiger.html) [Leopard](https://web.archive.org/web/20080726123134/http://www.apple.com/support/downloads/quicktime75forleopard.html) [Windows](https://web.archive.org/web/20080726122223/http://www.apple.com/support/downloads/quicktime75forwindows.html)
- download: [Panther](https://web.archive.org/web/20250424180039if_/http://appldnld.apple.com/QuickTime/061-4450.20080609.Vftr4/QuickTime75_Panther.dmg) [Tiger](https://web.archive.org/web/20250424175746if_/http://appldnld.apple.com/QuickTime/061-4451.20080609.nJu0o/QuickTime75_Tiger.dmg) [Leopard](https://web.archive.org/web/20210314170547if_/http://appldnld.apple.com/QuickTime/061-4458.20080609.Qqp4r/QuickTime75_Leopard.dmg) [Windows](https://web.archive.org/web/20160213062409if_/http://appldnld.apple.com/QuickTime/061-4459.20080609.gb6Yk/QuickTimeInstaller.exe)

#### 7.5.5
- [security](https://web.archive.org/web/20091223075247/http://support.apple.com/kb/HT3027)
- info: [Tiger](https://web.archive.org/web/20080913104559/http://www.apple.com/support/downloads/quicktime755fortiger.html) [Leopard](https://web.archive.org/web/20080913104555/http://www.apple.com/support/downloads/quicktime755forleopard.html) [Windows](https://web.archive.org/web/20080913060106/http://www.apple.com/support/downloads/quicktime755forwindows.html)
- download: [Tiger](https://web.archive.org/web/20250424181759if_/http://appldnld.apple.com/QuickTime/061-4702.20080909.re43e/QuickTime755_Tiger.dmg) [Leopard](https://web.archive.org/web/20250424181811if_/http://appldnld.apple.com/QuickTime/061-4709.20080909.AqspI/QuickTime755_Leopard.dmg) [Windows](https://web.archive.org/web/20210211041157if_/http://appldnld.apple.com/QuickTime/061-4710.20080909.bnh76/QuickTimeInstaller.exe)

#### Compatibility Update for QuickTime 7.5.5 for Leopard
- info: [Leopard](https://web.archive.org/web/20081225055052/http://support.apple.com/downloads/Compatibility_Update_for_QuickTime_7_5_5)
- download: [Leopard](https://web.archive.org/web/20140629073741if_/http://supportdownload.apple.com/download.info.apple.com/Apple_Support_Area/Apple_Software_Updates/Mac_OS_X/downloads/061-5786.20081117.plUt5/CompatibilityUpdateQT755.dmg)

### QuickTime 7.6

#### 7.6
- [security](https://web.archive.org/web/20091223080555/http://support.apple.com/kb/HT3403)
- info: [Tiger](https://web.archive.org/web/20090311081151/http://support.apple.com/downloads/QuickTime_7_6_for_Tiger) [Leopard](https://web.archive.org/web/20090125122021/http://support.apple.com/downloads/QuickTime_7_6_for_Leopard) [Windows](https://web.archive.org/web/20090125111856/http://support.apple.com/downloads/QuickTime_7_6_for_Windows)
- download: [Tiger](https://web.archive.org/web/20191004153945if_/http://appldnld.apple.com/QuickTime/061-5368.20090121.C13C9E/QuickTime76_Tiger.dmg) [Leopard](https://web.archive.org/web/20210707122646if_/http://appldnld.apple.com/QuickTime/061-5375.20090121.CCE8B/QuickTime76_Leopard.dmg) [Windows](https://web.archive.org/web/20180623090153if_/http://appldnld.apple.com/QuickTime/061-5376.20090121.BD7E9/QuickTimeInstaller.exe)

#### 7.6.2
- [security](https://web.archive.org/web/20091223083511/http://support.apple.com/kb/HT3591)
- info: [Mac](https://web.archive.org/web/20090605104947/http://support.apple.com/downloads/QuickTime_7_6_2_for_Mac) [Windows](https://web.archive.org/web/20090605160454/http://support.apple.com/downloads/QuickTime_7_6_2_for_Windows)
- download: [Tiger](https://web.archive.org/web/20210707191016if_/http://appldnld.apple.com/QuickTime/061-612.20090601.mDeSi/QuickTime762Tiger.dmg) [Leopard](https://web.archive.org/web/20210212013134if_/http://appldnld.apple.com/QuickTime/061-6629.20090601.NgyW5/QuickTime762Leopard.dmg) [Windows](https://web.archive.org/web/20190809071337if_/http://appldnld.apple.com/QuickTime/061-6118.20090601.Pq3V9/QuickTimeInstaller.exe)

#### 7.6.4
September 9, 2009: Final release for Mac OS X 10.4 Tiger.
- [security](https://web.archive.org/web/20091223075946/http://support.apple.com/kb/HT3859)
- info: [Mac](https://web.archive.org/web/20090916051613/http://support.apple.com/downloads/QuickTime_7_6_4_for_Mac) [Windows](https://web.archive.org/web/20090914113444/http://support.apple.com/downloads/QuickTime_7_6_4_for_Windows)
- download: [Tiger](https://web.archive.org/web/20191004153945if_/http://appldnld.apple.com/QuickTime/061-6742.20090909.TgQt4/QuickTime764_Tiger.dmg) [Leopard](https://web.archive.org/web/20191004153945if_/http://appldnld.apple.com/QuickTime/061-6738.20090909.tq764/QuickTime764_Leopard.dmg) [Windows](https://web.archive.org/web/20210424214608if_/http://appldnld.apple.com/QuickTime/061-7180.20090909.QTWdw/QuickTimeInstaller.exe)

#### 7.6.5
Windows-only release.
- [summary](https://web.archive.org/web/20100802052523/http://support.apple.com/kb/HT3524)
- [info page[not archived]](https://web.archive.org/web/20090101000000*/http://support.apple.com/kb/DL837)
- download: [Windows](https://web.archive.org/web/20160223052221if_/http://appldnld.apple.com/QuickTime/061-7356.20091118.Puyt6/QuickTimeInstaller.exe)

#### 7.6.6
- [security](https://web.archive.org/web/20100914090839/http://support.apple.com/kb/HT4104)
- info: [Leopard](https://web.archive.org/web/20100811025911/http://support.apple.com/kb/DL761) [Windows](https://web.archive.org/web/20100731062725/http://support.apple.com/kb/DL837)
- download: [Leopard](https://web.archive.org/web/20210426071120if_/http://appldnld.apple.com/QuickTime/061-7507.20100330.Qt657/QuickTime766Leopard.dmg) [Windows](https://web.archive.org/web/20110820053945if_/http://appldnld.apple.com/QuickTime/061-7793.20100330.Vfrt5/QuickTimeInstaller.exe)

#### 7.6.7
Windows-only release.
- [security](https://web.archive.org/web/20100816054657/http://support.apple.com/kb/HT4290)
- [info page[not archived]](https://web.archive.org/web/20100101000000*/http://support.apple.com/kb/DL837)
- download: [Windows](https://web.archive.org/web/20111028122206if_/http://appldnld.apple.com/QuickTime/061-8896.20100812.Bhyt5/QuickTimeInstaller.exe)

#### 7.6.8
Windows-only release.
- [security](https://web.archive.org/web/20100918182713/http://support.apple.com/kb/HT4339)
- [info page[not archived]](https://web.archive.org/web/20100101000000*/http://support.apple.com/kb/DL837)
- download: [Windows](https://web.archive.org/web/20101113081242if_/http://appldnld.apple.com/QuickTime/061-8938.20100915.Qts3T/QuickTimeInstaller.exe)

#### 7.6.9
- [security](https://web.archive.org/web/20110106021820/http://support.apple.com/kb/HT4447)
- info: [Leopard](https://web.archive.org/web/20110719050227/http://support.apple.com/kb/DL761) [Windows](https://web.archive.org/web/20110511015003/http://support.apple.com/kb/DL837)
- download: [Leopard](https://web.archive.org/web/20111113150747if_/http://appldnld.apple.com/QuickTime/061-9558.20101207.Qtgrt/QuickTime769Leopard.dmg) [Windows](https://web.archive.org/web/20110116221829if_/http://appldnld.apple.com/QuickTime/041-0025.20101207.Ptrqt/QuickTimeInstaller.exe)

### QuickTime 7.7

#### 7.7
August 3, 2011: Final release for Mac OS X 10.5 Leopard. All subsequent releases are Windows-only or integrated into Mac OS X updates.
- [security](https://web.archive.org/web/20140327102631/http://support.apple.com/kb/HT4826)
- info: [Leopard (original)](https://web.archive.org/web/20111031065844/http://support.apple.com/kb/DL761) [Leopard (fixed)](https://web.archive.org/web/20140224021003/https://support.apple.com/kb/DL761) [Windows](https://web.archive.org/web/20110903200032/http://support.apple.com/kb/DL837)
- download: [Leopard (original)](https://web.archive.org/web/20210227155558if_/https://download.info.apple.com/Mac_OS_X/041-0741-20110803.1VdPF/QuickTime770_Leopard.dmg) [Leopard (fixed)](https://web.archive.org/web/20181025155820if_/http://supportdownload.apple.com/download.info.apple.com/Apple_Support_Area/Apple_Software_Updates/Mac_OS_X/downloads/041-4967.20110803.1VdPF/QuickTime770_Leopard.dmg) [Windows](http://appldnld.apple.com/QuickTime/041-0743.20110803.4Rgtg/QuickTimeInstaller.exe)

#### 7.7.1
- [security](https://web.archive.org/web/20140327101628/http://support.apple.com/kb/HT5016)
- info: [Windows](https://web.archive.org/web/20120115010354/http://support.apple.com/kb/DL837)
- download: [Windows](https://web.archive.org/web/20120104015336if_/http://appldnld.apple.com/QuickTime/041-3089.20111026.Sxpr4/QuickTimeInstaller.exe)

#### 7.7.2
- [security](https://web.archive.org/web/20140327100936/http://support.apple.com/kb/HT5261)
- info: [Windows](https://web.archive.org/web/20120709141238/http://support.apple.com/kb/DL837)
- download: [Windows](https://web.archive.org/web/20121013075046if_/http://appldnld.apple.com/QuickTime/041-4337.20120425.sxIv8/QuickTimeInstaller.exe)

#### 7.7.3
- [security](https://web.archive.org/web/20140327095149/http://support.apple.com/kb/HT5581)
- info: [Windows](https://web.archive.org/web/20130509022403/http://support.apple.com/kb/DL837)
- download: [Windows](https://web.archive.org/web/20130510203942if_/http://appldnld.apple.com/QuickTime/041-6453.20121107.Aitr5/QuickTimeInstaller.exe)

#### 7.7.4
- [security](https://web.archive.org/web/20140327100611/http://support.apple.com/kb/HT5770)
- info: [Windows](https://web.archive.org/web/20130727203706/http://support.apple.com/kb/DL837)
- download: [Windows](https://web.archive.org/web/20131103100615if_/http://appldnld.apple.com/QuickTime/041-9648.20130522.Rftg5/QuickTimeInstaller.exe)

#### 7.7.5
- [security](https://web.archive.org/web/20140317194631/http://support.apple.com/kb/HT6151)
- info: [Windows](https://web.archive.org/web/20140329031657/http://support.apple.com/kb/DL837)
- download: [Windows](https://web.archive.org/web/20140723094352if_/http://appldnld.apple.com/QuickTime/091-8571.20140218.8MjJw/QuickTimeInstaller.exe)

#### 7.7.6
October 22, 2014: Final release for Windows XP.
- [security](https://web.archive.org/web/20141028071418/http://support.apple.com/kb/HT6493)
- info: [Windows](https://web.archive.org/web/20141110211408/http://support.apple.com/kb/DL837)
- download: [Windows](https://web.archive.org/web/20160222053425if_/http://appldnld.apple.com/QuickTime/031-08466.20141022.Xwlnm/QuickTimeInstaller.exe)

#### 7.7.7
This and subsequent releases [refuse to install on Windows 10](https://www.reddit.com/r/Windows10/comments/3c1u6f/quicktime/), although there are [workarounds](https://www.edugeek.net/forums/topic/140033-installing-quicktime-on-windows-10/).
- [security](https://web.archive.org/web/20160412032259/https://support.apple.com/en-us/HT204947)
- info: [Windows](https://web.archive.org/web/20150810150336/http://support.apple.com/kb/DL837?locale=en_US)
- download: [Windows](https://web.archive.org/web/20150810171116if_/http://secure-appldnld.apple.com/QuickTime/031-17469-20150630-3cf8c404-1a97-11e5-aaac-4e5ebe268ff7/quicktimeinstaller.exe)

#### 7.7.8
- [security](https://web.archive.org/web/20160414092914/https://support.apple.com/en-us/HT205046)
- info: [Windows](https://web.archive.org/web/20150918014144/http://support.apple.com/kb/DL837?locale=en_US)
- download: [Windows](https://web.archive.org/web/20150927224352if_/https://secure-appldnld.apple.com/quicktime/031-27600-20150820-F20FB1EF-6710-46BD-99B3-7DCF1253B310/QuickTimeInstaller.exe)

#### 7.7.9
January 7, 2016: Final release for Windows Vista & Windows 7.
- [security](https://web.archive.org/web/20160414091323/https://support.apple.com/en-us/HT205638)
- info: [Windows](https://web.archive.org/web/20160412183820/https://support.apple.com/kb/DL837?locale=en_US)
- download: [Windows](https://web.archive.org/web/20160110232959if_/https://secure-appldnld.apple.com/quicktime/031-43075-20160107-c0844134-b3cd-11e5-b1c0-43ca8d551951/quicktimeinstaller.exe)

## Sources

- Wikipedia entry for [QuickTime](https://en.wikipedia.org/wiki/QuickTime#QuickTime_7.x)
- Apple's lists of security updates: scroll to the bottom of the [current list of security releases](https://support.apple.com/en-us/100100); the [oldest is archived here](https://web.archive.org/web/20080923162341/http://docs.info.apple.com/article.html?artnum=25631)
- the installers' actual URLs were obscured by [Apple's download process](https://web.archive.org/web/20050930230712/http://www.apple.com/quicktime/download/mac.html), but careful searching found [many of them on tweakers.net](https://tweakers.net/downloads/zoeken/?keyword=Apple+QuickTime) and this [Avid forum post](https://community.avid.com/forums/t/44210.aspx), which helped me track down the Internet Archives' snapshots of Apple's various CDN servers over the years:
    - [qtinstall.info.apple.com](https://web.archive.org/web/*/http://qtinstall.info.apple.com/*)
    - [appldnld.m7z.net/qtinstall.info.apple.com](https://web.archive.org/web/*/http://appldnld.m7z.net/qtinstall.info.apple.com/*)
    - [appldnld.apple.com.edgesuite.net/qtinstall.info.apple.com](https://web.archive.org/web/*/http://appldnld.apple.com.edgesuite.net/qtinstall.info.apple.com/*)
    - [content.info.apple.com](https://web.archive.org/web/*/http://content.info.apple.com/*)
    - [appldnld.apple.com.edgesuite.net/content.info.apple.com](https://web.archive.org/web/*/http://appldnld.apple.com.edgesuite.net/content.info.apple.com/*)
    - [download.info.apple.com](https://web.archive.org/web/*/http://download.info.apple.com/Mac_OS_X/*)
    - [supportdownload.apple.com/download.info.apple.com](https://web.archive.org/web/*/http://supportdownload.apple.com/download.info.apple.com/Apple_Support_Area/Apple_Software_Updates/Mac_OS_X/*)
    - [appldnld.apple.com](https://web.archive.org/web/*/http://appldnld.apple.com/QuickTime/*)
    - [secure-appldnld.apple.com](https://web.archive.org/web/*/https://secure-appldnld.apple.com/QuickTime/*)
