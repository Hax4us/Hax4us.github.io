---
layout: post
title: Install Flutter In Termux
subtitle: Easy way to install Flutter in Termux.
tags: [termux]
---
I think this is the first post in whole internet about installing flutter in Termux 😌.

### Requirements

- Android SDK ( Install from [here](https://github.com/hax4us/Apkmod) )

### Installation Steps

* Download & execute installer script 

    ```
    curl -s https://raw.githubusercontent.com/Hax4us/flutter_in_termux/master/install.sh | bash -s
    ```
    That's it :)

### Must Read

* This is only for `arm64/aarch64` devices.

* You can now login into debian by

    ```
    proot-distro login flutter
    ```
    and then you can use `flutter` commands as usual like you can create & build sample projects 
    ```
    flutter create example-project
    cd example-project
    flutter build apk --release --target-platform android-arm64
    ```

Join us on Telegram [t.me/hax4us_group](https://t.me/hax4us_group)
