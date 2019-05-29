---
title: Distributed Tigerbrew Formula Testing using Jenkins
excerpt_separator: <!--more-->
---

Mac developers familiar with the [Homebrew](https://brew.sh) binary package manager for macOS may also have heard of [Tigerbrew](https://github.com/mistydemeo/tigerbrew), a fork tailored for PowerPC and and early Intel Macs running Mac OS X 10.4 or 10.5. <!--more-->Despite being computational history by now, they still have a community of their own working to keep them running and useful. As one such member, I've been looking into putting my collection of old Macs to work testing Tigerbrew formulae automatically.

## Overview

Homebrew's original continuous integration (CI) [server farm](https://jenkins.brew.sh) tests each incoming pull request on the [homebrew/core](https://github.com/Homebrew/homebrew-core) tap using [Jenkins](https://jenkins.io), a Java-based CI server that's been around for several years now. I'm using its ability to do builds by simultaneously dispatching jobs to a collection of nodes, then collect the results and logs. (This is different from testing each incoming change, in that we're testing just the formulae that are in Tigerbrew currently.) To fully test a formula, we'll need to include all arch/OS combos from that era, which requires separate Macs for each:

* Tiger on G3, G4, G5, Intel
* Leopard on G4, G5, Intel

Since Homebrew 2.0 now requires at least OS X 10.9 Mavericks, I'll eventually want to test on OS X 10.6–10.8 as well, once Tigerbrew adds support. Being Intel-only, those nodes I'll at least be able to virtualize.

I also have have an early Intel iMac acting as a Jenkins host and node controller. It and each node is plugged into an ethernet switch with an uplink to the router, which has DHCP reservations configured so that each machine gets a consistent and easy-to-type IP address.

## Nodes

I'm starting with these Macs as build nodes:

1. **Tiger G3**: 10.4.11 on iMac DV (PowerPC 750, 1GB)
2. **Tiger G4**: 10.4.11 on Mac mini Late 2005 (PowerPC 7447A, 1GB)
3. **Tiger G5**: 10.4.11 on Power Mac G5 (not yet acquired)
4. **Tiger Intel**: 10.4.11 on Mac mini 2007 (Intel Core 2 Duo, 3GB)
5. **Leopard G4**: 10.5.8 on Xserve 2002 (dual PowerPC 7455, 2GB)
6. **Leopard G5**: 10.5.8 on Power Mac G5 (not yet acquired)
7. **Leopard Intel**: 10.5.8 on Mac mini 2007 (Intel Core 2 Duo, 3GB)

### Node Setup

Each build node gets a clean OS install and a consistent configuration. I started by booting and installing OS X from a system disc to a reformatted hard drive.

- I had some old FireWire hard drives handy, so by formatting one using Apple Partition Map and cloning each installation disc to an appropriately sized partition using [SuperDuper!](https://www.shirt-pocket.com/SuperDuper/) and then booting each node off that instead, I was able to speed up the installation process considerably.
- Leopard installations need just the Mac OS X 10.5 Install DVD, which works for both PowerPC and Intel. But the Mac OS X 10.4 Install DVD works only for PowerPC; the first Intel Macs each shipped with their own system-specific build of Tiger. As I didn't have the original discs for the Intel mini nor a copy of Tiger Server (which will install to both PowerPC and Intel), but did have the original discs for an early Intel iMac, the solution I came up with was to:
    1. boot the iMac off its original Tiger DVD
    2. boot the mini in Target Disk Mode and connect it to the iMac using FireWire
    3. install Tiger to the mini
    4. reboot the iMac off of the mini
    5. install the 10.4.11 update, which upgraded the mini's Tiger system to a version it could boot from.

For each installation I clicked "Custom Install" in order to skip printer drivers and extra languages, and include X11. Then after using the Setup Assistant to set the same username and password on each node, I ran Software Update repeatedly until all updates were installed.

- On another drive I have local copies of all Apple software updates for each system, which I'll list in another post.
- Tiger's last X11 is 1.1.3, and while Leopard's last version via Software Update is 2.1.6, it can be updated to [XQuartz 2.6.3](https://www.xquartz.org/releases/XQuartz-2.6.3.html).
- [Remote Desktop Client 3.4](https://support.apple.com/kb/dl1350) is the newest for Tiger and Leopard, which won't appear in Software Update.
- When installing Xcode, I clicked "Custom Install" and unchecked all extra options except "Command Line Support" ([Xcode 2.5](https://download.developer.apple.com/Developer_Tools/xcode_2.5_developer_tools/xcode25_8m2558_developerdvd.dmg) on Tiger) or "UNIX Development" ([Xcode 3.1.4](https://download.developer.apple.com/Developer_Tools/xcode_3.1.4_developer_tools/xcode314_2809_developerdvd.dmg) on Leopard), then made sure to launch Xcode once to ensure it works and accept the EULA.
- Installing Java 6 for the Jenkins client is covered in an [earlier post](../2018/java-6-tiger-leopard.html).
- To get my public key installed on each node, I ran `ssh-copy-id -i $USER_NAME@$HOST_NAME` for each from my local Mac.
- I also cloned each completed installation to a partition on another external drive to avoid having to repeat the process should the need arise to reinstall.

I wrote this [bash script](https://gist.github.com/EricFromCanada/34997fc7cdca00cc35053c79309414f9) to automate the majority of each node's configuration tasks. You can run it directly from the web with:

    bash -c "$(curl -fsSkL https://gist.githubusercontent.com/EricFromCanada/34997fc7cdca00cc35053c79309414f9/raw/04fe8c8baebf0b303df2beb4941fd75dacd0c11a/tigerbrew-node-config.command)"

Only these two changes had to be done manually, since automating them would require [extra](https://github.com/xfreebird/kcpassword/blob/master/enable_autologin) [tools](https://www.pyehouse.com/cscreen/):

- _System Preferences > Accounts:_ under Login Options, enable automatic login
- _System Preferences > Displays:_ set to 1024 x 768

### Tigerbrew Installation

Just follow the instructions to [install Tigerbrew](https://github.com/mistydemeo/tigerbrew/blob/master/README.md) and update the PATH:

    ruby -e "$(curl -fsSkL raw.github.com/mistydemeo/tigerbrew/go/install)"
    echo 'export PATH=/usr/local/sbin:/usr/local/bin:$PATH' >> ~/.bash_profile
    source ~/.bash_profile

Next, install a few must-have tools:

- _(Tiger only)_ `brew install apple-gcc42` - installs GCC 4.2, since Tiger only provides GCC 4.0
- `brew install curl` - installs a newer curl that can handle modern TLS protocols used by most servers these days
- `brew install git` - installs git, required for updating Tigerbrew, which wasn't included with Xcode back then
- also [consider installing](https://github.com/mistydemeo/tigerbrew/wiki/New-formulae-in-Tigerbrew) `cctools`, `ld64`, `python`

Compiling and installing all these may take a while. When done, running `brew update` will convert your installation to a git repository, and `brew doctor` should return no errors.

## Master

I'm starting with an iMac5,1 for the master, maxed to 3GB of RAM and running OS X Lion, its last officially supported version. (Later version are possible with hackery, but aren't necessary in this case.) With Jenkins installed, it'll be the front end for the entire operation. Once its public SSH key is installed on each node, it will copy and launch the agent binary on each node automatically. The catch is that the agents require the same minimum version of Java as what runs on the master, so we'll install the last version of Jenkins that ran under Java 6.

### Master Setup

Using a [bootable OS X installer USB stick](http://osxdaily.com/2011/07/08/make-a-bootable-mac-os-x-10-7-lion-installer-from-a-usb-flash-drive/) I created long ago, I installed OS X Lion and all software updates.

- On Lion, the newest XQuartz version [2.7.11](https://www.xquartz.org/releases/XQuartz-2.7.11.html) still works, and its last Remote Desktop Client is [3.7.1](https://support.apple.com/kb/dl1710).
- I installed [Xcode 4.6.3](https://download.developer.apple.com/Developer_Tools/xcode_4.6.3/xcode4630916281a.dmg) by mounting the disk image and copying Xcode into the Applications folder. The [Command Line Tools](https://download.developer.apple.com/Developer_Tools/command_line_tools_os_x_lion_for_xcode__april_2013/xcode462_cltools_10_76938260a.dmg) are also required, which are a separate download for this version.
- For Jenkins I installed Apple's [Java for OS X 2015-001](http://supportdownload.apple.com/download.info.apple.com/Apple_Support_Area/Apple_Software_Updates/Mac_OS_X/downloads/031-29055.20150831-0f779fb2-4bf4-11e5-a8d8-/javaforosx.dmg). This installs 32- and 64-bit versions of Java 1.6.0_65-b14-468, as shown by `/usr/libexec/java_home -V`.
    - As [noted in my last post](./apple-java-2015-vs-2017.html), Apple has replaced on its support site all references to the 2015 installer with [Java for OS X 2017-001](https://support.apple.com/kb/dl1572), which installs _only_ the JDK, and no longer includes key system components for running applications with Java 6. If you try to run Jenkins 1.609.3 on OS X Lion with only 2017-001 installed, it will crash repeatedly with `Symbol not found: _JRSCopyOSJavaSupportVersion` errors. (OS X Mountain Lion does not have this issue, but that requires a Mac with 64-bit EFI.) The link above is a direct download for the 2015 version.

### Tigerbrew and Jenkins Installation

Tigerbrew is installed as shown above, along with `curl` and `git`, followed by running `brew update`. This is to give the master a Tigerbrew installation to run formula queries against.

Install [Jenkins 1.609.3](http://archives.jenkins-ci.org/osx-stable/jenkins-1.609.3.pkg) for OS X, using the default options. This creates a "jenkins" user under which all of Jenkins' actions will run, with its home directory set to `/Users/Shared/Jenkins`.

1. Secure the "jenkins" user by unlocking the _System Preferences > Users & Groups_ preference pane, selecting the new user under "Other Users", and clicking "Reset Password" to set a password, which is initially blank. You can also set a full name and picture.

1. Open a new Terminal and switch users with `su - jenkins` using the password you just created. You should now be in its newly-created home directory at `/Users/Shared/Jenkins`.

1. Create a new SSH key pair in `~/.ssh/`, which Jenkins will use to access the nodes:

        mkdir -p ~/.ssh
        ssh-keygen -t rsa -f ~/.ssh/id_rsa -C "jenkins@Tigerbrew-Master"

1. Copy the public key into each node, either with `ssh-copy-id` or this command, setting `$USER_NAME` and `$HOST_NAME` as needed:

        cat ~/.ssh/id_rsa.pub | ssh $USER_NAME@$HOST_NAME '[ -d "~/.ssh" ] || mkdir -p ~/.ssh; cat - >> ~/.ssh/authorized_keys'

    This will also populate the "jenkins" user's `~/.ssh/known_hosts` with that node's hostname, which you'll be using again later. Ensure that `ssh $USER_NAME@$HOST_NAME` logs you into each node from the master without needing a password.

1. Open the Jenkins interface at <http://localhost:8080> and go to _Manage Jenkins > Manage Plugins_. Since our version is now quite out of date, we need to manually install older plugin versions to get the functionality we need. Use the Advanced tab to upload these files, in order:

    - <https://updates.jenkins.io/download/plugins/description-setter/1.10/description-setter.hpi> (latest)
    - <https://updates.jenkins.io/download/plugins/token-macro/1.12.1/token-macro.hpi>
    - <https://updates.jenkins.io/download/plugins/build-name-setter/1.6.7/build-name-setter.hpi>
    - <https://updates.jenkins.io/download/plugins/structs/1.13/structs.hpi>
    - <https://updates.jenkins.io/download/plugins/credentials/2.1.11/credentials.hpi>
    - <https://updates.jenkins.io/download/plugins/ssh-credentials/1.13/ssh-credentials.hpi>
    - <https://updates.jenkins.io/download/plugins/ssh-slaves/1.17/ssh-slaves.hpi>
    - <https://updates.jenkins.io/download/plugins/workflow-step-api/1.14.2/workflow-step-api.hpi>
    - <https://updates.jenkins.io/download/plugins/workflow-basic-steps/1.14.2/workflow-basic-steps.hpi>
    - <https://updates.jenkins.io/download/plugins/junit/1.20/junit.hpi>
    - <https://updates.jenkins.io/download/plugins/script-security/1.25/script-security.hpi>
    - <https://updates.jenkins.io/download/plugins/matrix-project/1.13/matrix-project.hpi> (latest)
    - <https://updates.jenkins.io/download/plugins/resource-disposer/0.12/resource-disposer.hpi> (latest)
    - <https://updates.jenkins.io/download/plugins/ws-cleanup/0.32/ws-cleanup.hpi>

    Jenkins can be restarted anytime at <http://localhost:8080/restart>.

1. Then set some settings:

    - _Manage Jenkins > Configure System_
        - _Usage:_ **Only build jobs with label restrictions matching this node** (as we don't want the master to run any jobs)
        - _Jenkins URL:_ set this to the URL you use to access the Jenkins interface. (If they don't match, you'll be told your reverse proxy is broken.)
    - _Credentials > System > Global credentials > Add Credentials_
        - _Kind:_ **SSH Username with private key**
        - _Username:_ the user account name that all your nodes were set up with
        - _Private Key:_ **From the Jenkins master ~/.ssh**
        - _Description:_ `jenkins@Tigerbrew-Master`

1. Create a new node. For this example, I'm adding the iMac named "Tiger G3".

    - _Manage Jenkins > Manage Nodes > New Node_
        - _Node name:_ `Tiger G3` as a Dumb Slave
        - _Remote root directory:_ `/Users/Shared/jenkins-data` (this is where the agent process will keep its working files; parent directory must be writable)
        - _Labels:_ `macos ppc tiger` (allowing us to restrict jobs to certain nodes)
        - _Usage:_ **Only build jobs with label restrictions matching this node**
        - _Launch method:_ **Launch slave agents via SSH**
            - _Host:_ `Tiger-G3.local` (use the same hostname listed in the "jenkins" user's `~/.ssh/known_hosts` as mentioned earlier)
            - _Credentials:_ select the username you added in the previous step
            - _Host Key Verification Strategy:_ **Known hosts file** (this requires that you've previously logged into the node under the "jenkins" user, causing its key to be added to `/Users/Shared/Jenkins/.ssh/known_hosts`)

    The process should now launch on the node, and will be visible in the node's Dock.

1. Create a new job. With this, we'll be able to kick off a build on all nodes by providing a formula name.

    - _New Item_
        - _Item name:_ `Tigerbrew Builds Test` as a Multi-configuration project
        - check _This build is parameterized_ and add a String parameter
            - _Name:_ `FORMULA`
            - _Description:_ `Enter the name of the formula to test.`
        - under _Configuration Matrix_, add a Slaves axis
            - _Name:_ `Macs`
            - _Node/Label:_ check all nodes under _Individual nodes_ (but not the master)
        - under _Build Environment:_
            - check _Delete workspace before build starts_
            - check _Set Build Name_ and set to `${ENV,var="FORMULA"}`
                - click "Advanced…" and uncheck _Set build name after build ends_
        - under _Build_, add an _Execute shell_ build step using this script, which will be run on each node with each job.
            ```sh
            # set PATH so brew command can be found
            source ~/.bash_profile
            # check for updates
            brew update
            # print the formula's version, which will be added to the build's description
            echo "[version]" $(brew info $FORMULA | tail -n +1 | head -n 1)
            # print node's configuration
            brew --env
            brew config
            # download the formula's dependencies and then its sources
            [[ $(brew deps $FORMULA) ]] && brew fetch --retry --build-from-source $(brew deps $FORMULA)
            brew fetch --retry --build-from-source $FORMULA
            # install dependencies and then the formula itself
            brew install --verbose --only-dependencies $FORMULA
            brew install --verbose --build-bottle $FORMULA
            ```
        - under _Build_, add a second build step _Set build description_ and set to `\[version\] (.*)`

    Click "Save" when done.

When the name of a formula to build is submitted via the job's _Build with Parameters_ sidebar link, the script above is run on each node, whose console output can be viewed live by clicking through to each node listed in the sidebar.

This is only proof-of-concept at this point. A full pipeline for picking which formulae to test, optimal handling of dependencies, producing bottles, and testing of formula updates is still in the works, so stay tuned.
