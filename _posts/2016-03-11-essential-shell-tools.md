---
layout: post
title: 'Essential shell tools'
author: Christopher Batey
comments: true
tags:
- shell
---

I spent far too much of my life on a computer using the default terminal
software and shell tools. Over the past year that has changed and now have a long list
of tools I can't believe I lived without:

[Tmux](https://tmux.github.io/): The terminal multiplexor. Where I used to use screen on jump boxes I
wouldn't have thought of using anything similar locally a few years ago. The main
features I rely on: 

* Splitting and tabbing terminals in a standard way regardless
of operating system
* Multiple sessions for context switching e.g I have a work session and personal session

[Autojump](https://github.com/wting/autojump): A faster way to navigate. This
tool is simply awesome. It stores your cd history then allows you to do `j
pattern` and it will move you into the directory you move to the most that
matches the pattern. Within hours of installing you'll be hooked.

[SCM Breeze](https://github.com/ndbroadbent/scm_breeze): My life is versioned! I
spend hours a day interacting with git. Why oh why did I spend so much time
typying git commands! For example `git checkout -b blah` becomes `gcb blah`
. Another: `git commit -m
"something interesting"` becomes `gcmsg "something intersting"`. Watch the gifs
on the home page to see even more powerful examples.

[My repos](https://myrepos.branchable.com/): Allows you to mange many repos as
one. Simply enter a repo you have checked out do `mr register` and it will be
added to your `.mrconfig`. Then you can push all repos with `mr push` and update
all your repos with `mr update`. I move between a laptop, desktop and client
workstations a lot so this tool is simply life changing. Combine it with `vcsh`
so you can manage all your dot files as separate repos and you're laughing.

[Zsh](http://www.zsh.org/): Stop using bash now! Then install `oh-my-zsh`.

[Ansible](ihttps://www.ansible.com/): I now completely auomate the installation
of my desktop and laptop. Where as I don't think Ansible is powerful enough in
the work place for managing thousands of servers it is perfect for managing my
two desktops and laptop. My initial though was "Only slightly more complicated
than writing shells cripts to install things but a lot more powerful". I have
recently switched from apt based distros to dnf based distros so my home
automation isn't complete but here is what I use to keep my desktops and laptop
consitent: [home-ansible](https://github.com/chbatey/linux-home-office).

Hopefully you find some of these tools as useful as I do.




