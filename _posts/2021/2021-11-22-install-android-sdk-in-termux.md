---
layout: post
title: Install Android SDK In Termux
tags: [termux]
---
To install Android SDK in Termux , you will have to follow some simple steps.

### Requirements :

- Java [`apt install openjdk-17`]

### Steps :

1. First of all you will have to download Android SDK

    ```bash
    wget https://github.com/Lzhiyong/termux-ndk/releases/download/android-sdk/android-sdk.tar.xz
    ```

2. Extract into __share__ directory

    ```bash
    tar xf android-sdk.tar.xz -C $PREFIX/share
    ```

3. Now make a wrapper for __sdkmanager__ 
    
    ```bash
    #!/data/data/com.termux/files/usr/bin/bash

    /data/data/com.termux/files/usr/share/android-sdk/tools/bin/sdkmanager \
        --sdk_root=/data/data/com.termux/files/usr/share/android-sdk "$@"
    ```
    and save it with name `sdkmanager` in `$PREFIX/bin` 
    
4. That's it :)

