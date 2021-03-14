---
layout: post
title: WSL and ManimGL
description: How to configure WSL to work with manimGL
author: Karl Jan Clinckspoor
date: 12 March 2021
tags:
    - wsl
    - linux
    - manim
    - python
---

# Getting ManimGL to work on WSL

## On manim

[manim](https://www.github.com/3b1b/manim) is an animation package written in Python by Grant
Sanderson, who makes educational math videos of extreme quality. Currently, there are several
versions of manim, an old one, mostly used to compile the old videos, a new one, which uses OpenGL
to show the videos as they're made, not requiring them to be finished before seeing, and the
community edition, which does require finishing the videos, but it's better documented and easier to
beginners (or so they say). I'm definitely a total beginner at this moment.

## On Linux

In my experience, it is easier to use Python tools in Linux. On the other hand, I am forced to use
Windows for work and leisure (and I have to admit, I like it, I find it stable and unobtrusive). The
traditional ways of dealing with this are either:

1. Dual boot Linux and Windows

   This has given me significant headaches. It's annoying to have to switch, sometimes your boot
   loader will simply stop working (I blame Windows), and you'll have to spend an hour fixing it,
   and it's difficult to access files between OSs (primarily Windows->Linux through).

2. Use a Virtual Machine (VM)

   This is a cool alternative. You can run Windows or Linux inside each other, have shared
   clipboards, shared folders and, if you're ready to configure, you can even dedicate a graphics
   card to your VM, as shown by a video on LinusTechTips. A disadvantage is that you'll want to give
   the VM quite a lot of RAM if you intend to work on something serious and not have it lag. Another
   disadvantage is that accessing your files in the VM can still be a bit painful if something bad
   happens to your VM. Still, it's great, and if you have the required RAM, much more convenient 
   than dual booting.
   
## On WSL

Recently, Microsoft has released the Windows Subsystem for Linux (WSL). With a bit of software 
magic, they made it possible to run an actual Linux Kernel on a very low level (AFAIK, it's not 
"in" Windows, it's kinda "parallel" to it). This makes it possible to more or less seamlessly 
join the best of both worlds.

## Getting it running

Normally, WSL runs only in the terminal, and here's the catch: `manimGL` *requires* a graphical 
environment right off the bat, to display the animations in real time. If you try to run that, 
you'll get quite a few errors. In order to get it, or any graphical program, to show on Windows 
through WSL is to use a X-Server on Windows and connect it to WSL. I tried this before with 
VcXsrv, but I found it to be clunky and unintuitive. I found that some people recommended 
MobaXterm, and decided to give that a go.

Assuming you have `manimgl` installed, and (MobaXTerm)[https://mobaxterm.mobatek.
net/download-home-edition.html] installed and running (including the X-server), you will need to 
configure your `DISPLAY` variable. To do this, on your `.bashrc` or `.zshrc`, add the following 
line ((thanks to nick janetakis)[https://nickjanetakis.
com/blog/using-wsl-and-mobaxterm-to-create-a-linux-dev-environment-on-windows]):

```bash
export DISPLAY="$(/sbin/ip route | awk '/default/ { print $3 }'):0"
```

Be sure to save and reload your shell. Now, when you ran MobaXTerm for the first time, Windows 
probably asked about your firewall preferences. I had it set to Private only, and didn't work 
at all. I went to the `Firewall and network protection`, then to `Allow an application through 
the firewall`. This can also be accessed through `Control Panel->System and Safety->Windows 
Defender Firewall->Allowed applications`. Allow yourself to change the settings and find 
`xwin_mobax.exe`, probably at the bottom of the list. I had to entries, one with Private checked,
another one with `Public` set. I naively set both entries to both Private and Public (do it at 
your own risk), then clicked ok.

Last think, in MobaXterm, I went to `Settings->Configurations->X11` and in `X11 remote access`, 
I set it to full. This was to stop it from requesting some permissions continuously (again, do 
it at your own risk - I barely know what I'm doing).

Now, to test. I used `sudo apt install firefox`, then typed `firefox` and voilá! It appeared!

Going back to manim, if you have the main repo cloned in WSL, you can use its test scripts. 
`manimgl example_scenes.py`, then select `01`, and a huge window with the animation appeared! I 
was ecstatic when this happened. Took me a few hours to install everything and figure out how to 
get it running.

## Conclusion

I'm liking WSL a lot, especially now that I can use graphical programs. Setting PyCharm and 
VSCode to access WSL is easy, as is using `pyenv`, `pyenv virtualenv`, `jekyll` and 
other Linux-inclined tools. Now, for my latest project, I got manim to work, and am looking 
forward to exploring its possibilities.

I know this post is a bit sparse on actual instructions. I might update it in the future with a 
series of commands to get every part running, from scratch.