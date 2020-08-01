---
title: How to Port Calamares to Arch Linux
description: 
published: true
date: 2020-08-01T02:36:06.797Z
tags: python, archlinux, calamares, apps
editor: markdown
---

> 本文将在近期重写。

> This article is for reference ONLY.

Calamares is a distribution-independent installer framework. As distributions like Arch Linux don't provide graphical installer, we can try to adapt Calamares to them. To make things easier, we simply start by trying to provide a `pacstrap` part for Calamares, which allow users to install a minimal Arch Linux system, then we add the Calamares installer to our Archiso.

Getting Started
---------------

Make sure that you have Git installed, fetch the source code by cloning the Calamares repository, to speed up, add `--depth=1` option:

    git clone https://github.com/calamares/calamares.git --depth=1

Entering the repository, you will find lots of files and directories, don't panic! All you have to focus on are three files/directories: `settings.conf`, `src/branding/` and `src/modules/`. Let's try to deal with them one by one.

Step One: `settings.conf`
-------------------------

Firstly, you will find something like this:

    sequence:
    - show:
      - welcome
      - locale
      - ...
    - exec:
      - partition
      - mount
      - ...
    - show:
      - finished

This is the installation process of Calamares, consisting of the `show` part and the `exec` part. Unlike Arch Linux's official install way where users DIY their system from the beginning to the end, users should setup their system in the `show` part and the `exec` part should handle the rest of the work then `show` again to inform user whether the installation work is succeed or not, which is considered more user-friendly. Each part (`show` and `exec`) includes several modules like `welcome`, `locale` and so on. We will talk about them later.

Now go to ArchWiki and have a look at the Installation Guide, you will find that some of the modules can be safely removed because when you perform a installation by following the ArchWiki, you never do such things like `plymouthcfg` (In fact, plymouth is not supported by Arch Linux at all). However, you will need to add some part because Calamares doesn't provide such option, such as a `pacstrap` part. Simply add the line `- pacstrap` and do rest of the thing later in step three.

    sequence:
    - show:
      - welcome
      - locale
      - keyboard
      - partition
      - users
      - summary
    - exec:
      - partition
      - mount
      - pacstrap
      - networkcfg
      - packages
      - machineid
      - fstab
      - locale
      - keyboard
      - localecfg
      - users
      - networkcfg
      - hwclock
      - bootloader
      - umount
    - show:
      - finished

Let's move on. You will then see some of the boolean variables with notes above each of them. Simply set `true` or `false` according to the notes. Leave `branding` as `default`, you will configure it in step two.

Step Two: `src/branding/`
-------------------------

In fact, this step is optional if you don't care about branding at all. The problem is that your users will be faced with something strange like the distro name `Fancy GNU/Linux 2020.2 LTS "Turgid Tuba"` and the slide show texted `This is a customizable QML slideshow.` So it is not recommend to skip this step if you want to distribute the Calamares and Archiso to others.

Navigate to the `src/branding/` directory, read the `README.md` carefully, and enter the `default/` directory. There are two files that you have to pay attention. One of them is `branding.desc`. These entries should be modified to the actual one:

    strings:
        productName:         "@{NAME}"
        shortProductName:    Arch Linux
        version:             Rolling
        shortVersion:        Rolling
        versionedName:       Arch Linux
        shortVersionedName:  Arch Linux
        bootloaderEntryName: Arch Linux
        productUrl:          https://www.archlinux.org/
        supportUrl:          https://bbs.archlinux.org/
        knownIssuesUrl:      https://bugs.archlinux.org/
        releaseNotesUrl:     https://www.archlinux.org/

Another one is `show.qml`. It is used to control slide shows. Here we only add one page, with the image `squid.png` (already included in the directory) and text `Arch Linux is a lightweight and flexible Linux distribution that tries to Keep It Simple.`

        Slide {
    
            Image {
                id: background
                source: "squid.png"
                width: 200; height: 200
                fillMode: Image.PreserveAspectFit
                anchors.centerIn: parent
            }
            Text {
                anchors.horizontalCenter: background.horizontalCenter
                anchors.top: background.bottom
                text: "Arch Linux is a lightweight and flexible Linux distribution that tries to Keep It Simple."
                wrapMode: Text.WordWrap
                width: presentation.width
                horizontalAlignment: Text.Center
            }
        }

Step Three: `src/modules/`
--------------------------

This is the most difficult part. You should do configurations for all modules listed in `settings.conf`. In most of the time, you don't have to do programming work, but simply edit the `*.conf` file. For example, if you want to edit the `users` module, open `src/modules/users.conf` and edit it. Here is a sample file:

    ---
    defaultGroups:
        - users
        - wheel
    
    autologinGroup:  autologin
    doAutologin:     true
    setRootPassword: true
    doReusePassword: true
    
    passwordRequirements:
        nonempty: true
        minLength: -1
        maxLength: -1
        libpwquality:
            - minlen=0
            - minclass=0
    
    allowWeakPasswords: false
    allowWeakPasswordsDefault: false

For modules you created by yourself, like `pactrap`, you have to create a folder for it with the same name. Make sure you have already known a little bit about Python and Shell script, now start coding!

All we have to do is to call the real `pactrap` command. But it seems that if we really do that, the installation process may failed because of the behavior of Calamares. So we also have to port `pacstrap` for Calamares later. For now, we just name the script `pacstrap_calamares` and forget it. Let's create a file `main.py`.

    #!/usr/bin/env python3
    # -*- coding: utf-8 -*-
    
    import subprocess
    import libcalamares
    from pathlib import Path
    
    root_mount_point = libcalamares.globalstorage.value("rootMountPoint")
        
    def run():
        """
        Installing base filesystem. Please wait! It may take some time!
        """
    
        PACSTRAP = "/usr/bin/pacstrap_calamares"
        PACKAGES = "base grub efibootmgr linux linux-firmware vi nano networkmanager"
    
        subprocess.call(PACSTRAP.split(' ') + [root_mount_point] + PACKAGES.split(' '))
    

In fact, `subprocess` can always be used to write Shell script in Python. The only thing you have do is to save command into a string `foo`, then split it using `foo.split(' ')`, and put it into `subprocess.call()`. Here, we call `/usr/bin/pacstrap_calamares` and ask it to install some basic package `base grub efibootmgr linux linux-firmware vi nano networkmanager`. When `pacstrap_calamares` is running, Calamares will show `Installing base filesystem. Please wait! It may take some time!` on the screen.

Step Four: Build Calamares
--------------------------

We can build Calamares when we are building Archiso so the build process won't make your system dirty. Assume that you already have a Archiso profile. First, we add the needed packages to `packages.x86_64`.

    kf5
    qt5
    boost
    kpmcore
    yaml-cpp
    polkit-qt5

Assume that you have entered the chroot environment. To build Calamares, three steps are reqired: `cmake`, `make` and `make install`. Write all the steps down, and put it into `airootfs/root/`. Here is an example file:

    #/bin/bash
    cd /root/
    mkdir ./calamares/build/
    cd ./calamares/build/
    cmake ..
    make
    make install
    mkdir /etc/calamares/
    cp -p /root/archlinux-calamares/settings.conf /etc/calamares/
    rm -rf /root/archlinux-calamares/ /root/calamares/ /root/build_calamares.sh
    pacman -Rscun qt5-doc qt5-examples --noconfirm

Now we can ask `airootfs/root/customize_airootfs.sh` to run this shell script after a chroot environment is entered. Make sure the script is executable.

    cd /root
    ./build_calamares.sh

Almost done! Don't forget to create a `pacstrap_calamares` file for Calamares. Create a `airootfs/usr/bin/pacstrap_calamares` file. You don't have to worry about how to fill the file. Someone has already done that for you! And [here is the file](https://raw.githubusercontent.com/endeavouros-team/install-scripts/master/pacstrap_calamares). Thanks for the work done by Endeavour OS team.