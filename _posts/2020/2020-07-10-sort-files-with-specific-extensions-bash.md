---
layout: post
title: Find files with Specific Extension Bash
tags: [bash, termux, linux]
---
Today i am going to show you how can we make a deep scanner by which we can scan whole directory including subdirectories and fetch all files of given extension.

Here is Code :)

<script src="https://gist.github.com/Hax4us/445383bc968f5b42bb2cae67b8808d61.js"></script>

### Explanation : 

1. In Line 3, i declared a array `database` which will save our fetched files name.
2. In Line 5, introduced a new function with the name of `scanner` which expects two values as arguments i.e __PATH__ to be searched & __EXTENSION__ of required files.
3. Line 27, take input from user (_path & extension_)
4. Line 29, function `scanner` called with required arguments
5. Line 6 to 8, inside `scanner` function, two `local` variables are declared & initialised. One is `path_to_be_scanned` which is initialised to `$1` (path to be scanned) and Second is `file` initialised to empty `String`. Now third one is `ext` initialised to __EXTENSION__ of file.
6. Line 10, `cd` into given __directory__.
7. Line 12, so now we are looping through all files & directories in current directory. (`*` is just like a universal set which consists all elements )
    > if you are more curious guy then just search `regex` on google
8. Line 13, so now check if `file` is real __file__ or __directory__.
```shell
# -d returns true if given argument is directory otherwise false
if [ -d DIR ]; then
    echo "DIR is a directory & exists"
else
    echo "DIR is a not a directory or not exists"
fi
```
9. Line 15, i am skipping one folder for scanning to save our time but you can scan it too if you want.
10. Line 16, ok so if folder name is not __Android__ then call `scanner` again with current value of `file` and `ext`. Always `cd` back for each recursive call.
11. Line 21, now i am checking if `file` contains given `ext` or not. If yes then append it to `database` array. 
> if you never heard about recursion then you should keep reading this post :)

### Example For Better Understanding
* Content of directory `$HOME/bash` ( See Attached Screenshot )
<center><img src="/blog/assets/img/ss1.jpg" width="80%"></center><br />
* After running script with required inputs i got output like below
<center><img src="/blog/assets/img/ss2.jpg" width="80%"></center><br />
* As you can see i got my all `.sh` files within given directory (including subdirectories test1, test2, test3, test4). But how ?

#### Behind The Scene
* So after taking input, `scanner` called with `$HOME/bash` as `path` and `sh` as extension of file to be searched.
* Now after `cd` into `$HOME/bash`, magic begins with maggy masala magic ;)
* Ok so first i got a list of all things within current directory i.e `$HOME/bash` by `*` a.k.a asterisk and feed that list to `for` loop.
```bash
# for loop now looks like
for file in debug.sh sort.sh test1 test2 test3 test4 wget.sh; do
...
done
```
* Now i checked if `$file` is a __file__ or a __directory__. So in first run of loop `$file` is `debug.sh`. 
```bash
# Offcourse debug.sh is not a directory
if [ -d debug.sh ];  then
... # will not run.
fi
```
* Now i am checking if __extension__ of `debug.sh` is `sh` or not by simply trimming `$file` name using `${file##*.}`. It will return anything after `.` from `debug.sh` i.e `sh` 
```bash
if [ sh = sh ]; then
    database+=(debug.sh)
fi
```
* Now in second run of loop, `$file` is `sort.sh` and the same process will be done.
* In third run of loop when `$file` is `test1` which is a folder, `[ -d test1 ]` will return `0` or `true`. Then if its not a folder with name of __Android__ then call again `scanner` with current value of `file` variable i.e `test1`.
* Now inside `scanner` function `path_to_be_scanned` assigned to `test1` & `ext` is same as before i.e `sh`.
* After `cd` into `test1` folder, `for` loop looks like
```bash
for file in t1 t2 t3 t4; do
    ...
done
```
* They all are files, but not with `sh` extension, so nothing happens. Neither `[ -d t1 ]` nor `[ * = sh ]` will be execute.
* Now after looping four times (t1,t2,t3,t4) function ends and will return to it's previous state where execution was stopped. (See below diagram)
<center><img src="/blog/assets/img/stack.png" width="80%"></center><br />
<center><img src="/blog/assets/img/stack2.png" width="80%"></center><br />
* So program was on line 16 when `scanner` was called second time. And one more thing, first called `scanner` function comes in __paused__ state while executing `scanner` second time. And when second `scanner` will do all his work then current `scanner` function will be wiped out from memory ( stack ) and then our paused `scanner` will resume from Line 17
> yes your Operating system ( more specifically CPU ) maintains a stack where it stores recursive functions along with data like local variables. You should learn about stack if you are confused :)
* We are on Line 17 now, it's time to `cd` back from `test1` folder and then check if `test1` has `sh` extension but it does not make sense because we know that `test1` is a folder not file, so it's your homework to not check folders for extension. Well so `if` part will not execute.
* Now fourth run of `for` loop, `$file` is `test2`, which is a folder and same thing will happen now ( recursive call)
* Fifth run of loop, `$file` is `test3`, which is also a folder or directory, ok so it's time to call our hero again `scanner test3 sh` and now inside `scanner`, `for` loop looks like
```bash
for file in my.sh; do
...
done
```
* `my.sh` is a file so no recursive call takes place and we shifted to second `if` condition
```bash
if [ sh = sh ]; then
    database+=(my.sh)
done
```
* Now `for` loop ends and there is nothing to execute so second `scanner` wiped out again and control returns to our first initial called function `scanner`.
* At last when initial called `scanner` ends ( means there is nothing to do ) , this also wiped out from stack and stack is empty and we are ready to go for next instruction in script which is `echo ${database[@]}`

Ok so this was my best ( i think 😬).
Thank you for reading.
