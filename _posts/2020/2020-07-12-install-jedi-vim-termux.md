---
layout: post
title: Install Jedi-Vim In Termux
cover-img: "/assets/img/vimjedi_preview.jpg"
tags: [vim, termux]
---
Let's see how to install jedi-vim plugin for python in termux. It's a cool plugin for pythonic guys.

### Features 
The Jedi library understands most of Python's core features. From decorators to generators, there is broad support.

Apart from that, jedi-vim supports the following commands

- Completion `<C-Space>`
- Goto assignment `<leader>g` (typical goto function)
- Goto definition `<leader>d` (follow identifier as far as possible, includes imports and statements)
- Goto (typing) stub `<leader>s`
- Show Documentation/Pydoc K (shows a popup with assignments)
- Renaming `<leader>r`
- Usages `<leader>n` (shows all the usages of a name)
- Open module, e.g. `:Pyimport os` (opens the os module)

### Requirements :
- Vim-Plug ( [Click here](https://www.hax4us.com/2019/09/how-to-install-plug-in-manager-for-vim.html?m=1) )
- Package `vim-python` ( `apt install vim-python` )

### Steps :
- Open/Create `.vimrc` file like this `vim ~/.vimrc` and edit your `.vimrc` file like below
<center><img src="/assets/img/vimrc.jpg" width="80%"></center><br />
- After editing & saving `.vimrc` file, open your vim by simple command `vim`
- Then type `:PlugInstall`
<center><img src="/assets/img/vim_pluginstall.jpg" width="80%"></center><br />
- Now hit `ENTER` and you will see something like this
<center><img src="/assets/img/vim_pluginstalling.jpg" width="80%"></center><br />
- After all done you can safely exit from `vim` as usual.
- Now you are ready to go, just create your `python` script and Enjoy :)
<center><img src="/assets/img/vimjedi_preview.jpg" width="80%"></center><br />
