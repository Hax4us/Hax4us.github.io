---
layout: post
title: Install VSCode In Termux
subtitle: Yes it's possible with proot.
tags: [termux]
---
Let's install VSCode in our termux.

### Requirements 
---
* Stable internet
* Debian With XFCE desktop [Click here](https://hax4us.github.io/2022-01-23-install-debian-with-xfce-in-termux/)


### Downloding VSCode deb file
---

* login into debian

    ```
    proot-distro login debian
    ```

* download vscode (aarch64 or arm64)

    ```
    wget https://gitlab.com/Hax4us/dumbrepo/-/raw/master/vscode/code_1.63.2-1639561157_arm64.deb
    ```

    for armhf or arm
    ```
    wget https://gitlab.com/Hax4us/dumbrepo/-/raw/master/vscode/code_1.63.2-1639561157_arm.deb
    ```

* unpack deb (don't forget to change arm64 with arm if you have arm 32bit device)

    ```
    dpkg-deb -R code_1.63.2-1639561157_arm64.deb code
    ```

* copy into `/usr/share`

    ```
    cp -a code/usr/share/* /usr/share/
    ```

* Now you can see VSCode in Applications 

    <img src="/assets/img/2022/2022-01-23/01.jpg" />


### Tip:

* Always use VNC Viewer inbuilt backspace key instead of your keyboard's backspace key.


Join us on Telegram [t.me/hax4us_group](https://t.me/hax4us_group)
