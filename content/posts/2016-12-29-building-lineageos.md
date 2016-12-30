---
layout: "post"
title: "Building LineageOS"
description: "A complete tutorial on building LineageOS and derivatives"
categories: ["android"]
tags: ["android","cyanogenmod","lineageos","tutorial"]
---
# This is a work-in-progress - don't take it all to heart

<p id="whatyouneed">&nbsp;</p>
## Requirements
You're going to need:

- a Debian-based OS, such as Ubuntu (16.10+ is preferable)
- a reasonable knowledge of the command line
- 8GB+ RAM
- Dual-core+ CPU
- Lots of disk space (~50GB for the actual source code, ~30GB per device and however much you want for cache)
- 10mbps+ internet (if you want to get *anything* done)
- sudo privileges

<p id="whatstheword">&nbsp;</p>
## Some information/Glossary
- Python is an easy-to-understand high level programming language often used for automation
- Git is a version control system. It basically tracks any changes to files and folders
  - So if someone makes a typo, you don't have to download 50GB+ worth of files again after they correct it
  - Also, if someone does something malicious, it can be tracked down to the date and time it was added to the code and by whom
- Repositories (often shortened to repos) are basically collections of files (with Git information) such as _android_device_huawei_angler_, the repo that contains _device_ information for the _angler_ device (Nexus 6P) made by _Huawei_
- breakfast, brunch and lunch are scripts made by Google which make building Android much easier. It means you just pick the device you want to build for, and it does it, rather than having to build each file individually

<p id="tools">&nbsp;</p>
## Installing some tools
For both 32 and 64-bit systems you're going to need to install these packages:
```bash
sudo apt install bc bison build-essential curl flex git gnupg gperf libesd0-dev liblz4-tool libncurses5-dev libsdl1.2-dev libwxgtk3.0-dev libxml2 libxml2-utils lzop maven openjdk-8-jdk pngcrush schedtool squashfs-tools xsltproc zip zlib1g-dev
```
In addition, for 64 bit systems you're going to need:
```bash
sudo apt install g++-multilib gcc-multilib lib32ncurses5-dev lib32readline6-dev lib32z1-dev
```

Secondly, we're going to need a Python script, written by Google which manages multiple Git repositories. We'll install it to your personal binary folder.

```bash
# make the directory. ~ is shorthand for /home/yourusername
mkdir -p ~/bin

# Download the repo files
curl https://storage.googleapis.com/git-repo-downloads/repo > ~/bin/repo

# Make it executable
chmod a+x ~/bin/repo

# Add your personal binary folder to your PATH (a list of folders to search through when looking for commands)
# The code (which launches every time you open a terminal), checks whether ~/bin exists and if so, adds it to the path
echo '# set PATH so it includes users private bin if it exists' >> ~/.profile
echo 'if [ -d "$HOME/bin" ] ; then' >> ~/.profile
echo '    PATH="$HOME/bin:$PATH"' >> ~/.profile
echo 'fi' >> ~/.profile

```

<p id="decisions">&nbsp;</p>
## Making decisions
<p id="decisions-wheregoesthecode">&nbsp;</p>
### Where are you going to put this code?
Ideally on 1TB SSD, but, unless you're rich or prepared to spend lots of money... no.
An SSD would be ideal, but finding a large size one isn't a cost-effective option.

On a different disk to your OS's disk is your best choice. That way your computer shouldn't crap out too badly when it's building.
I'm going to assume for simplicity, you've chosen to build it on the same disk.

<p id="decisions-devices">&nbsp;</p>
### What devices am I building for?
I can't make this decision for you, but you can. Be aware, each device will take more disk space, and will also mean you will take more time when you build for _all_ devices sequentially

<p id="gettingthecode">&nbsp;</p>
## Getting the code
Here, you want to close and reopen your terminal, to ensure the change we just made is taken into account
```bash
# make the folder to store all the Android code
# This is where you can choose to store all this somewhere else
mkdir -p ~/android/system

# Move into the directory, ready to start downloading
cd ~/android/system/

# Tell repo what we want to download
# init = initialise the repositories
# https://github.com/LineageOS/android.git = the list of repositories to download
# -b cm-14.1 =  the branch or version of code we are downloading
#               this will change depending on the version *you* want. This is for cm-14.1 (CyanogenMod 14.1 based on Android 7.1.1)
repo init -u https://github.com/LineageOS/android.git -b cm-14.1

# Start the download
repo sync
```
At this point, (unless you have gigabit internet), go get a coffee. This is gonna take a while. When it's done, you'll be returned to the shell prompt again

<p id="devicecode">&nbsp;</p>
## Getting the devices
The code we just downloaded is the _general_ or _common_ code for Android. Put simply, it's the stuff that's the same on all the devices.  
To get specific files for a device (such as angler), we need to tell repo which device you want.  
We can either do this simply, but not future-proofed, or the slightly more complicated way, which is much more fun.

<p id="devicecode-simpleway">&nbsp;</p>
### Simple way
Just replace yourdevicecodenamehere with your device's codename. For instance, the Nexus 6P's is _angler_, the Nexus 9's is _flounder_ and Motorola Moto G (2013)'s is _falcon_

```bash
source build/envsetup.sh
breakfast yourdevicecodenamehere
```

<p id="devicecode-complicatedway">&nbsp;</p>
### Complicated way (the proper way)

<p id="devicecode-complicatedway-whatsamanifest">&nbsp;</p>
#### What's a manifest?
Repo decides what repositories to download by looking through manifests. The main manifest (downloaded when we initialised repo) is present in `~/home/yourusername/android/system/.repo/manifest.xml` and contains loads of repos needed for building common Android files.  
You _could_ edit this file, but since this is updated when new things are added to Android, your changes would always be being overwritten. So the better option is to use _local manifests_. If you add your own `.xml` files in `~/home/yourusername/android/system/.repo/local_manifests/`, you can add and remove your own repos to the main list, without editing the main manifest.

<p id="devicecode-complicated-organisation">&nbsp;</p>
#### Organisation
For instance, my `local_manifests` folder looks a little like this:

File              | What does it contain?
---               |:---
`angler.xml`      | Device specifics for angler (Nexus 6P)
`falcon.xml`      | Device specifics for falcon (Motorola Moto G - 2013)
`flounder.xml`    | Device specifics for flounder (Nexus 9 - WiFi)
`flounder_lte.xml`| Device specifics for flounder_lte (Nexus 9 - LTE)
`qualcomm.xml`    | Qualcomm common files (used by falcon and serranoltexx)
`roomservice.xml` | A default file made by the repo tool
`serranoltexx.xml`| Device specifics for serranoltexx (Samsung Galaxy S4 Mini - Intl.)
`shared.xml`      | Some common Android files needed by falcon, serranoltexx, which aren't in the default Android source

<p id="devicecode-complicatedway-whyohwhy">&nbsp;</p>
#### Why on earth might you do this?
If you want to make changes to device specific files, or even common Android things, you can `fork` the repository you'd like to change. You can make your changes in this repo (which you own) and then incorporate those changes into your builds.

<p id="devicecode-complicatedway-forkoff">&nbsp;</p>
#### Forking the repo and making changes (only if you want to make changes)

You first need to create a [GitHub](https://github.com/) account. Then go to [LineageOS's Github](https://github,com/LineageOS), find the repository you want to fork, and click fork in the top right. This will create a copy in your account, where you can make changes online.  

To add the changes to your source code, create your manifest in _local_manifests_ `touch ~/android/system/.repo/local_manifests/whatever-name-you-want.xml`. Then edit it `nano ~/android/system/.repo/local_manifests/whatever-name-you-want.xml`
```xml
<?xml version="1.0" encoding="UTF-8"?>
<manifest>
        <remote         name="github"
                        fetch=".."
                        review="review.lineageos.org" />

        <project        path="where/the/files/need/to/go"
                        name="GitHubUsername/name_of_repo"
                        remote="github"
                        revision="cm-14.1"/>
</manifest>
```
For instance: (I chose to not fork, as I'm not making changes for my own purposes)
```xml
<?xml version="1.0" encoding="UTF-8"?>
<manifest>
        <remote         name="github"
                        fetch=".."
                        review="review.lineageos.org" />

        <project        path="device/huawei/angler"
                        name="LineageOS/android_device_huawei_angler"
                        remote="github"
                        revision="cm-14.1"/>

        <project        path="kernel/huawei/angler"
                        name="LineageOS/android_kernel_huawei_angler"
                        remote="github"
                        revision="cm-14.1"/>

        <project        path="vendor/huawei"
                        name="TheMuppets/proprietary_vendor_huawei"
                        remote="github"
                        revision="cm-14.1"/>
</manifest>
```
After you've made any changes to your manifest or want to update the code from LineageOS, run:
```bash
repo sync
```

<p id="finallyready">&nbsp;</p>
## First build

<p id="doitallforme">&nbsp;</p>
## Automation
