---
layout: post
title: Convert Smali To Dex
subtitle: Easy way To Compile Smali Back To Dex
tags: [misc, termux]
---
If you have some smali files & want to convert it into single dex then you can use smali program written by __JesusFreke__.

I will convert smali to dex in my termux.

## Steps :
1. Download smali program
```bash
wget https://github.com/JesusFreke/smali/releases/download/v2.0b6/smali-2.0b6.jar
```
2. Now let's say i have a smali file with name __Test.smali__ then 
```bash
# Output will be saved as out.dex in current directory
java -jar smali-2.0b6.jar Test.smali
```
3. If you have multiple smali files then put all together in a single folder ( let's folder name is __smali_folder__ ) then
```bash
java -jar smali-2.0b6.jar smali_folder
```

