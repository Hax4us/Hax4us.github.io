---
layout: post
title: Install Debian With XFCE In Termux
subtitle: Debian with full GUI setup :)
tags: [termux]
---
We can literally install any distro with the almighty tool `Proot` & there is also a package `proot-distro` available which is nothing but a wrapper around `proot`.

### Installing Debian ...
---

* Install `proot-distro`

    ```
    apt install proot-distro
    ```

* Install debian 

    ```
    proot-distro install debian
    ```

* Pretty easy right :)


### Setting up XFCE ...
---

* Login into debian

    ```
    proot-distro login debian
    ```

* Install essential packages

    ```
    apt install xfce4-goodies xfce4-terminal tightvncserver
    ```

* Copy whole code given below 

    ```
    #!/bin/sh                           
    if [ -f $HOME/.Xresources ]; then
        xrdb "$HOME/.Xresources"
    fi
    xsetroot -solid grey
    # Fix to make GNOME work
    export XKL_XMODMAP_DISABLE=1 
    /etc/X11/Xsession
    dbus-launch --exit-with-session startxfce4 &
    ```

    and now create a new file like `touch ~/.vnc/xstartup` & open it with any editor like `vim ~/.vnc/xstartup` and paste your copied code there then save & exit.

* Run vncserver

    ```
    vncserver :1
    ```

* Open your favourite VNC Viewer And yes :1 means server is on port **5901**.


### Tip
---

* You can kill vncserver by `vncserver -kill :DISPLAY` , DISPLAY is **1** in this case.

Join us on Telegram [t.me/hax4us_group](https://t.me/hax4us_group)
