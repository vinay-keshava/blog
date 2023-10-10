---
title: "Interception Vimproved"
date: 2023-09-18T11:23:17+05:30
weight: 10
description: "Setting up Interception Vimproved on Debian Bookworm GNU/Linux"
tags: ["vim", "interception-tools", " gnu/linux" , "gnu","keyboard"]
draft: true
type: post
---

## Setting up Interception Vimproved on  Debian Bookworm

I started learning vim few months ago and wanted to vim key bindings like shortcuts on my laptop after trying a mechanical hackable keyboard.

This blog post shows how to setup interception-vimproved using interception-tools on Debian Bookworm.

Step 1: Install the dependencies to build interception-vimproved on Debian Bookworm GNU/Linux
 
``` 
$ sudo apt install interception-tools meson libyaml-cpp-dev cmake 
```
interception-tools is a small set of tools for input events of devices,that can be used to customize the behaviour of input keyboard mappings.

The advantage of interception-tools operates at lower level compared to xmodmap by using libevdev and libudev.

Step 2: Clone interception-vimproved repository and build
```
$ git clone "https://github.com/maricn/interception-vimproved"

$ cd interception-vimproved

$ sudo make install
``` 

Step 3: Create a file called udevmon.yaml in /etc/interception and paste the following contents into the file /etc/interception/udevmon.yaml
``` 
- JOB: "interception -g $DEVNODE | interception-vimproved /etc/interception-vimproved/config.yaml | uinput -d $DEVNODE"
  DEVICE:
    NAME: ".*((k|K)(eyboard|EYBOARD)|TADA68).*"
``` 

Step 4: Reload udevmon using systemctl
``` 
$ sudo systemctl restart udevmon
``` 

To change any keybindings or to add new mappings the config file is present in config.yaml located in /etc/interception-vimproved/

my config.yaml has the below shortcuts
``` 
- intercept: KEY_CAPSLOCK
  ontap: KEY_ESC
  onhold: KEY_LEFTCTRL

- intercept: KEY_ENTER
  # not necessary: ontap: KEY_ENTER is inferred if left empty
  onhold: KEY_RIGHTCTRL

# this is a layer. hold space (onhold) contains several remappings
- intercept: KEY_SPACE
  onhold:

  # special chars
  - from: KEY_E
    to: KEY_ESC

  # alternative syntax
  - {from: KEY_D, to: KEY_DELETE}
  - {from: KEY_B, to: KEY_BACKSPACE}

  # vim home row
  - {from: KEY_H, to: KEY_LEFT}
  - {from: KEY_J, to: KEY_DOWN}
  - {from: KEY_K, to: KEY_UP}
  - {from: KEY_L, to: KEY_RIGHT}

  # vim above home row
  - {from: KEY_Y, to: KEY_HOME}
  - {from: KEY_U, to: KEY_PAGEDOWN}
  - {from: KEY_I, to: KEY_PAGEUP}
  - {from: KEY_O, to: KEY_END}

  # number row, to F keys
  - {from: KEY_1, to: KEY_F1}
  - {from: KEY_2, to: KEY_F2}
  - {from: KEY_3, to: KEY_F3}
  - {from: KEY_4, to: KEY_F4}
  - {from: KEY_5, to: KEY_F5}
  - {from: KEY_6, to: KEY_F6}
  - {from: KEY_7, to: KEY_F7}
  - {from: KEY_8, to: KEY_F8}
  - {from: KEY_9, to: KEY_F9}
  - {from: KEY_0, to: KEY_F10}
  - {from: KEY_MINUS, to: KEY_F11}
  - {from: KEY_EQUAL, to: KEY_F12}

  # xf86 audio
  - {from: KEY_M, to: KEY_MUTE}
  - {from: KEY_COMMA, to: KEY_VOLUMEDOWN}
  - {from: KEY_DOT, to: KEY_VOLUMEUP}

  # mouse navigation
  - {from: BTN_LEFT, to: BTN_BACK}
  - {from: BTN_RIGHT, to: BTN_FORWARD}
``` 

Arch has interception-tools already packaged here is the [link](https://wiki.archlinux.org/title/Interception-tools)
```
:wq
```

