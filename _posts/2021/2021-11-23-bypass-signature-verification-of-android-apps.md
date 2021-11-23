---
layout: post
title: Bypass Signature Verification Of Android Apps
tags: [termux, apkmod, hacking]
---
Trust me , it's very easy to bypass or kill signature verification of any android app with the help of my tool __Apkmod__.

### Requirements :
- [Apkmod](https://github.com/Hax4us/Apkmod)

### Steps :

1. Let's assume , we have latest whatsapp apk so command would be

    ```bash
    apkmod --signature-bypass --killer=k2 -i /path/to/whatsapp.apk -o killed_whatsapp.apk
    ```
    > Apk must be untouched before killing signature verification.
    
    Now you can modify it or inject metasploit payload with it using apkmod.

2. Sign it ( Optional )

    ```bash
    apkmod -s -i killed_whatsapp.apk -o signed_whatsapp.apk
    ```

### Tips:

* There are two version available of signature killer , one is k1 and second one is k2, you will have to specify version like `--killer=k1` or `--killer=k2`.

* Always try `k2` first :)
