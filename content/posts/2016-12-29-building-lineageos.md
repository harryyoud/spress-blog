---
layout: "post"
title: "Building LineageOS"
description: "A complete tutorial on building LineageOS and derivatives"
categories: ["android"]
tags: ["android","cyanogenmod","lineageos","tutorial"]
---

<p id="wtfisit">&nbsp;</p>
## Wtf is LineageOS?
Android, being open-source, means people can take the code, make their modifications and publish it. CyanogenMod was a group of people who did this, bringing many features into Android we now take for granted. After some trouble with commercialisation, the project rebranded as LineageOS in December 2016. At the time of writing (Late December 2016), the project is basically just the code from cm-14.1. Bigger changes will come when a new Android version is released to the Android Open Source Project.

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
sudo apt install bc bison build-essential curl flex git gnupg gperf libesd0-dev liblz4-tool libncurses5-dev libsdl1.2-dev libwxgtk3.0-dev libxml2 libxml2-utils lzop maven openjdk-8-jdk pngcrush schedtool squashfs-tools xsltproc zip zlib1g-dev android-tools-adb android-tools-fastboot
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

Last thing, you need to ensure OpenJDK (and not Oracle's JDK) is set to be the default java.

```bash
sudo update-alternatives --config java
```
You should see something like this:
Type the selection number of the line showing "java-8-openjdk(-amd64)" with "manual mode" and press enter
```
There are 3 choices for the alternative java (providing /usr/bin/java).

  Selection    Path                                            Priority   Status
------------------------------------------------------------
  0            /usr/lib/jvm/java-8-openjdk-amd64/jre/bin/java   1071      auto mode
  1            /usr/lib/jvm/java-8-openjdk-amd64/jre/bin/java   1071      manual mode
* 2            /usr/lib/jvm/jdk1.7.0/bin/java                   1         manual mode
  3            /usr/lib/jvm/jdk1.8.0/bin/java                   1         manual mode

Press enter to keep the current choice[*], or type selection number: 3
```
and do the same for javac:
```bash
sudo update-alternatives --config javac
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

<p id="decisions-ccache">&nbsp;</p>
### Use CCACHE?
CCACHE is amazing. When you get round to building, it checks to see what has changed and only builds files that are different from the build. This saves unbelievable amounts of time, as some files, hardly ever touched, only need to be built once!

To turn it on:
```bash
cd ~/android/system/
echo "export USE_CCACHE=1" >> ~/.bashrc
# replace 50G with the amount of disk space you want to spare for this.
# The larger the better, if you can afford it
prebuilts/misc/linux-x86/ccache/ccache -M 50G
```

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
## Device-specific code
The code we just downloaded is the _general_ or _common_ code for Android. Put simply, it's the stuff that's the same on all the devices.  
To get specific files for a device (such as angler), we need to tell repo which device you want. As well as the open-source device specifics, you need some proprietary files (things like camera libraries or sound FX libraries). These can be either extracted from a device already running stock android: (with Developer Settings enabled - tap on Build number in Settings > About seven times), but are best downloaded online.

Just replace yourdevicecodenamehere with your device's codename. For instance, the Nexus 6P's is _angler_, the Nexus 9's is _flounder_ and Motorola Moto G (2013)'s is _falcon_

```bash
source build/envsetup.sh
breakfast yourdevicecodenamehere
```
You should see some errors, at which point you want to download the proprietary code for your manufacturer.
All this can be done in the next section, once we go through some terms.

<p id="devicecode-manifest">&nbsp;</p>
### Proprietary files

<p id="devicecode-manifest-whatsamanifest">&nbsp;</p>
#### What's a manifest?
Repo decides what repositories to download by looking through manifests. The main manifest (downloaded when we initialised repo) is present in `~/home/yourusername/android/system/.repo/manifest.xml` and contains loads of repos needed for building common Android files.  
You _could_ edit this file, but since this is updated when new things are added to Android, your changes would always be being overwritten. So the better option is to use _local manifests_. If you edit `~/home/yourusername/android/system/.repo/local_manifests/roomservice.xml`, where you can add and remove your own repos to the main list, without editing the main manifest.

<p id="devicecode-manifest-whyohwhy">&nbsp;</p>
#### Why on earth might you do this?
If you want to make changes to device specific files, or even common Android things, you can `fork` the repository you'd like to change. You can make your changes in this repo (which you own) and then incorporate those changes into your builds.

#### How do I make a manifest then?
Edit roomservice: `nano ~/android/system/.repo/local_manifests/roomservice.xml`
You should see some content already here, based loosely on this:
```xml
<?xml version="1.0" encoding="UTF-8"?>
<manifest>
<remote name="github" fetch=".." review="review.lineageos.org" />
<project path="where/the/files/need/to/go" name="LineageOS/name_of_repo" remote="github"/>
</manifest>
```

This contains all the device-specific code (excluding the proprietary stuff). To download the proprietary stuff, wander over to [TheMuppets GitHub](https://github.com/TheMuppets) and search for your manufacturer. Take note of the name of the repository.
Then add this before `</manifest>` at the bottom of the file:
```xml
<project path="vendor/MANUFACTURER" name="TheMuppets/NAME_OF_REPO" remote="github"/>
```
Take a look at my local manifest for some examples [here](https://github.com/harryyoud/LinOS_Manifests)

After you've made any changes to your manifest (which we've done) or want to update the code from LineageOS, run:
```bash
# From your ~/android/system directory
repo sync
```

<p id="finallyready">&nbsp;</p>
## First build
Before you do anything, make sure your codebase is up-to-date:
```bash
# From your ~/android/system directory
repo sync
```
Then simply:
```bash
# Replace DEVICECODENAME with the device's codename in lowercase
brunch DEVICECODENAME
```

Now it's time for another coffee :-)  
When it's done (returned you back to the shell prompt), you should fine the build under `~/android/system/out/target/product/DEVICECODENAME/` called something like `cm-14.1-20161231-UNOFFICIAL-DEVICECODENAME.zip`.  
So that's everything... right? RIGHT?
Well... no. This is great, but wouldn't it be good if you didn't have to do this every time you want a new build?  

<p id="doitallforme">&nbsp;</p>
## Automation
There are loads of scripts out of there which wrap around the brunch and lunch command. I love reinventing the wheel (who doesn't?) so I made my own and named it `midnightsnack`. You can take a look [here](https://github.com/harryyoud/midnightsnack). It's run at 12:05am every morning by cron (the Linux scheduling daemon). You can add things to cron like this:
```bash
crontab -e
```
and add:
```
# minute hour dayOfMonth month dayOfWeek command
# * means every. so the 5th minute of the 12th hour of every day of every month regardless of the day of week
5 0 * * * /bin/bash /home/harry/projects/midnightsnack/main.sh
# /bin/bash is the shell (or interpreter) which runs the main.sh file
```
It basically goes like this:

1.  Import includes (functions, settings, error descs) and set vars
2.  CheckVariablesExist
3.  LogHeaders
4.  Repo Sync
5.  DeviceLoop
     5a. LogHeaders
     5b. Lunch
     5c. Make
     5d. Find and rename zip
     5e. MD5sum
     5f. Upload zip, then rename
     5g. Delete Build zip
6.  TarGZ logs
7.  Drop email with successes and fails with attached logs

<p id="whatnow">&nbsp;</p>
## All done
So builds happen every night for all the devices you chose. What now?

- Port LineageOS to another device? - _I might write a tutorial for this one day_
- Make some changes
  - Disable encryption?
  - Add an app into the system partition?
  - Bypass some SafetyNet measures?
  - Remove root?
