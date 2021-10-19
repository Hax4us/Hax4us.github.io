---
layout: post
title: Install Java 11 In Termux
cover-img: "/assets/img/2021/2021-10-19/cover-1.jpg"
tags: [termux]
---
__Openjdk-17__ is already available in Termux Official Repository but still many of us want to install __Openjdk-11__ , because of many reasons.

### Requirements :
- aarch64 device 

### Steps :
1. Download JDK tarball 
```bash
wget https://github.com/Lzhiyong/termux-ndk/releases/download/openjdk/openjdk-11.0.12.tar.xz
```
2. Create new directory for jdk
```bash
mkdir $PREFIX/lib/jvm
```
3. Extract into new created directory
```bash
tar -xf openjdk-11.0.12.tar.xz -C $PREFIX/lib/jvm
```
4. Create a symlink with name java into $PREFIX/bin
```bash
ln -sf $PREFIX/lib/jvm/openjdk-11.0.1/bin/java $PREFIX/bin/java
```

### Tip :
* You can create more symlinks for javac, keytool, jarsigner, etc.
