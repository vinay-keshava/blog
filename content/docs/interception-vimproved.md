---
title: Interception Vimproved
next: /docs/selfhosting/nextcloud
---


### Setting up Interception Vimproved on  Debian Bookworm

I started learning vim few months ago and wanted to try vim key bindings like shortcuts on my laptop after trying a mechanical hackable keyboard.

[Interception Vimproved](https://github.com/maricn/interception-vimproved) is a plugin for [interception-tools](https://gitlab.com/interception/linux/tools) which combines both caps2esc and space on hold work as special ```fn``` key.
This blog post shows how to setup [interception-vimproved](https://github.com/maricn/interception-vimproved) using interception-tools on Debian Bookworm.

#### Step 1: Dependencies 

Installing Dependencies to build interception-vimproved on Debian Bookworm GNU/Linux
 
```bash
$ sudo apt install interception-tools meson libyaml-cpp-dev cmake 
```
interception-tools is a small set of tools for input events of devices,that can be used to customize the behaviour of input keyboard mappings.

The advantage of interception-tools operates at lower level compared to xmodmap by using libevdev and libudev.

#### Step 2: Clone & Build

Clone interception-vimproved repository and build
```bash
$ git clone "https://github.com/maricn/interception-vimproved"

$ cd interception-vimproved

$ sudo make install
``` 
Clone the git repository and change the directory, and then launch a ```make install``` command to build.
#### Step 3: Configuration

Create a new file called udevmon.yaml in /etc/interception and paste the following contents into the file /etc/interception/udevmon.yaml
```bash
- JOB: "interception -g $DEVNODE | interception-vimproved /etc/interception-vimproved/config.yaml | uinput -d $DEVNODE"
  DEVICE:
    NAME: ".*((k|K)(eyboard|EYBOARD)).*"
``` 

```udevmon.yaml``` is like a job specification for ```udevmon```,specifying that it matches with ```(k|K)(eyboard|EYBOARD))``` input device.

{{< callout type="info" >}}
  I haven't tested this for an External Keyboard Input device,but works fine for the built-in keyboard of the laptop.
{{< /callout >}}
#### Step 4: Reload udevmon 

Reload udevmon using systemctl
```bash 
$ sudo systemctl restart udevmon
``` 

#### Hack around the config
To change any keybindings or to add new mappings the config file is present in config.yaml located in /etc/interception-vimproved/ when a ```sudo make install``` is launched the config file is 
copied to ```/etc/interception-vimproved/config.yaml```.

my config.yaml has the below shortcuts
```bash{filename="/etc/interception-vimproved/config.yaml"}
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

[Interception-tools](https://tracker.debian.org/pkg/interception-tools) is packaged on debian,interception-vimproved is not, 
that is the reason we are building the source of interception-vimproved,hopefully i'll try packaging it !.

Arch has interception-tools already packaged here is the [link](https://wiki.archlinux.org/title/Interception-tools)

```bash
:wq #until next time
```

