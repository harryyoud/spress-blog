---
layout: "post"
title: "Building LineageOS"
description: "A complete tutorial on building LineageOS and derivatives"
categories: ["android"]
tags: ["android","cyanogenmod","lineageos","tutorial"]
draft: true
---

## Requirements
You're going to need:
- a Debian-based OS, such as Ubuntu (16.10+ is preferable)
- a reasonable knowledge of the command line
- 8GB+ RAM
- Dual-core+ CPU
- Lots of disk space (~50GB for the actual source code, ~30GB per device)
- 10mbps+ internet (if you want to get *anything* done)
- sudo privileges

## Some information/Glossary
- Python is an easy-to-understand high level programming language often used for automation
- Git is a version control system. It basically tracks any changes to files and folders
  - So if someone makes a typo, you don't have to download 50GB+ worth of files again after they correct it
  - Also, if someone does something malicious, it can be tracked down to the date and time it was added to the code and by whom
- Repositories are basically collections of files (with Git information) such as android_device_huawei_angler, the repo that contains device information for the angler device (Nexus 6P) made by Huawei
- breakfast, brunch and lunch are scripts made by Google which make building Android much easier. It means you just pick the device you want to build for, and it does it, rather than having to build each file individually

## Installing some tools
For both 32 and 64-bit systems you're going to need to install these packages:
```bash
sudo apt install bc bison build-essential curl flex git gnupg gperf libesd0-dev liblz4-tool libncurses5-dev libsdl1.2-dev libwxgtk3.0-dev libxml2 libxml2-utils lzop maven openjdk-8-jdk pngcrush schedtool squashfs-tools xsltproc zip zlib1g-dev
```
In addition, for 64 bit systems you're going to need:
```bash
sudo apt install g++-multilib gcc-multilib lib32ncurses5-dev lib32readline6-dev lib32z1-dev
```

## Making decisions
### Where are you going to put this code?
Ideally on 1TB SSD, but, unless you're rich or prepared to spend lots of money... no.
An SSD would be ideal, but finding a large size one isn't a cost-effective option.

On a different disk to your OS's disk is your best choice. That way your computer shouldn't crap out too badly when it's building.
I'm going to assume for simplicity, you've chosen to build it on the same disk.

## Getting the code
First, we're going to need a Python script, written by Google which manages multiple Git repositories. We'll install it to your personal binary folder.
```bash
# make the directory. ~ is shorthand for /home/yourusername
mkdir -p ~/bin
# make the folder to store all the Android code
# This is where you can choose to store all this somewhere else
mkdir -p ~/android/system
# Download the repo files
curl https://storage.googleapis.com/git-repo-downloads/repo > ~/bin/repo
# Make it executable
chmod a+x ~/bin/repo
```

## Getting the devices
## First build
## Automation
