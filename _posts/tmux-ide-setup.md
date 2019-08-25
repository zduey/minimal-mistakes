---
title: Using Tmux + VIM as an IDE

categories:
    - resources
tags:
    - powerline
    - vim
    - tmux
---
- dotfiles directory
- Use git submodules for vim8 package management
- create a symlink from dotfiles/vim to ~/.vim
- Install powerline with sudo apt-get install powerline-fonts
- Install tmux

There are an uncountable (well, not technically) number of blog posts about
programming environments, so unfortunately this post is just adding to the fray.
My intention is not to argue that this setup is the one that *you* should choose, 
I merely want to elaborate on my current setup and why I chose it. If you happen
to have a similar set of needs and contraints, then maybe it is a setup that could
work for you as well.

## Requirements
These days, almost all of my work is done via a remote server, which immediately
pulls emacs/vim/nano to the top of the editor stack since those are available on
almost all machines by default. I have no interest in  diving into the vim/emacs
fray, so suffice it to say I'm a vim user and am happy with it. Quite honestly,
I've never put them time into learning emacs, so I cannot provide a justification
for choosing one over the other. I happened to learn Vim first and haven't looked
back (yet).

I also tend to have longer-running jobs that I like to be able to check on/fix/restart when 
I'm at home. I don't want to have to put a lot of effort into re-creating my programming 
environment from a different machine. 

Finally (and probably most importantly), I really hate using
a mouse. There were a few weeks where I actually hung my mouse over one 
of my monitors so that the shame of having to reach for it would instead encite me
to open a new tab in my browser and find a shortcut/hotkey for doing whatever task
I was about to click my way into.  This hatred of clicking  ruled out a number of
traditional IDE options. Don't get me wrong, I know there are plenty of IDEs out
there  with a large number of hotkeys that can largely keep you from ever having to 
touch the mouse (Eclipse, PyCharm, Sublime, VS Code, etc). In general, what I don't
like about these solutions that one of the following is usually true:
1. You have to pay if you want the good stuff (e.g. remote interpreter in PyCharm)
2. You can easily fall into the trap of becoming overly dependent. For example, If 
   you ever found yourself without the comfort of your usual menus for pushing code 
   changes, running tests, etc. you might not know how to complete those tasks with 
   just a terminal in front of you.
3. Integration with a terminal can be a bit clunky or require switching applications.

## Setup

