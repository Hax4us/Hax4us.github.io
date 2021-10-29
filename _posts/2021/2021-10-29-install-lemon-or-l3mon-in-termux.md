---
layout: post
title: Install LEMON Or L3MON In Termux
subtitle: A quick soltion for installing lemon
tags: [termux]
---
Well if you struggling while installing L3MON in termux then here is a solutiom for you :)

### Requirements :

- apkmod ( Install from [here](https://github.com/hax4us/Apkmod) )

- Lemon Deb package [(Get from Telegram)](https://t.me/hax4us_group)

### Installation Steps :

1. Download lemon deb package from my telegram group

2. Then install apkmod from above link

3. Now install our deb package
```bash
dpkg -i lemon_2.1-2_all.deb
```


### Tips :
* To start server 

    ```bash
    lemon
    ```

* If you started server for first time then prees __CTRL+C__ and edit a file `maindb.json` 

    ```bash
    vi $PREFIX/share/lemon/server/maindb.json
    ```
    and copy this md5 hash `098f6bcd4621d373cade4e832627b4f6` and paste it just after password field like this
    <img src="/assets/img/2021/2021-10-29/1.jpg" />

* Now save and start server again using command `lemon` and goto `127.0.0.1:22533` from browser and type `admin` in username and `test` in password. 

* However yon can always change the password if you know how to do :)
