---
layout: post
title: PyCharm, WSL and external tools
description: How I reconciled using WSL2, PyCharm and Black
author: Karl Jan Clinckspoor
date: 11 April 2021
tags:
- python
- wsl
---

As I alluded to in [a previous post]({% post_url 2021-03-12-configuring-wsl2-manimgl %}), I like 
using Windows Subsystem for Linux (WSL) and Python. I also enjoy using PyCharm to write more 
structured programs (for exploratory data analysis, it's Jupyter Notebooks all the way), more 
than VSCode, for PyCharm, despite being much more of a resource hog, it has better support for 
fetching documentation (IMO), has superior debugging and a better interface to set new ways to 
run your tests (I like clicking and not editing JSON manually, OK?). However, VSCode has one 
thing that's much better than PyCharm, which is its handling of WSL.

I like to apply code formatting, and the [black](https://github.com/psf/black) code formatter is 
one of my favorites. Setting it to run in PyCharm is relatively painless, provided you follow 
[their instructions](https://black.readthedocs.io/en/stable/editor_integration.
html#pycharm-intellij-idea), but if you do that, it won't work with WSL. And, to complicate 
things, I have it installed in a specific conda environment in a pyenv environment. What a mess.

Nevertheless, I got it working. Go to `Settings > Tools > External Tools`, add a new entry to `black`
and then add this:

* `Tool Settings - Program`: `wsl`
* `Tool Settings - Arguments`: `--exec /bin/sh -c "/home/karl/.pyenv/versions/3.8.8/envs/manimCE/bin/python3.8 -m black $FileRelativePath$"`
* `Tool Settings - Working Directory`: `$ProjectFileDir$`
* `Advanced Options`: Check `Synchronize files after execution`. If you want to check if there were
    any problems, check `Open console for tool output`

To run this automatically every time a file is saved, you can use the tutorial provided in the 
documentation, and use the same program and arguments I provided.

If this breaks when I'm doing something more complicated, I'll update this.