---
layout: post
title: Set up Linux
---

## User management ([source](https://wiki.archlinux.org/index.php/Users_and_groups))

To list users currently logged on the system

    # who

To list all existing user accounts

    # passwd -Sa

To add a new user

    # useradd -m -G additional_groups -s login_shell username

`-m/--create-home`  
creates the user home directory as `/home/username`.  
`-G/--groups`  
introduces a list of supplementary groups which the user is also a member of.  
`-s/--shell`  
defines the path and file name of the user's default login shell. See /etc/shells for valid login shells.  

If an initial login group is specified by name or number, it must refer to an already existing group. If not specified, the behaviour of useradd will depend on the `USERGROUPS_ENAB` variable contained in `/etc/login.defs`. The default behaviour (`USERGROUPS_ENAB yes`) is to create a group with the same name as the username.  

When the login shell is intended to be non-functional, for example when the user account is created for a specific service, `/usr/bin/nologin` may be specified in place of a regular shell to politely refuse a login.  

To add a new user named `archie`, creating its home directory and otherwise using all the defaults in terms of groups, folder names, shell used and various other parameters:

    # useradd -m archie

To display the default value for the login shell of the new account

    # useradd --defaults

To protect the newly created user archie with a password

    # passwd archie

The above useradd command will also automatically create a group called `archie` and makes this the default group for the user `archie`. Making each user have their own group (with the group name same as the user name) is the preferred way to add users.  

## Sudo ([source](https://wiki.archlinux.org/index.php/Sudo))

The configuration file for sudo is `/etc/sudoers`. It should always be edited with the `visudo` command.  

To allow a user to gain full root privileges when he/she precedes a command with sudo, add the following line:

    USER_NAME   ALL=(ALL) ALL

To allow a user to run all commands as any user but only on the machine with hostname HOST_NAME:

    USER_NAME   HOST_NAME=(ALL) ALL

To allow members of group `wheel` sudo access:

    %wheel      ALL=(ALL) ALL

### Passing aliases

If you use a lot of aliases, you might have noticed that they do not carry over to the root account when using sudo. However, there is an easy way to make them work. Simply add the following to your `~/.bashrc` or `/etc/bash.bashrc`:

    alias sudo='sudo '

## xinit ([source](https://wiki.archlinux.org/index.php/Xinit))

### xinitrc  

`~/.xinitrc` is handy to run programs depending on X and set environment variables on X server startup. If it is present in a user's home directory, startx and xinit execute it. Otherwise startx will run the default `/etc/X11/xinit/xinitrc`.  

This default xinitrc will start a basic environment with `Twm`, `xorg-xclock` and `Xterm` (assuming that the necessary packages are installed). Therefore, to start a different window manager or desktop environment, first create a copy of the default xinitrc in your home directory:  

    $ cp /etc/X11/xinit/xinitrc ~/.xinitrc

Then edit the file and replace the default programs with desired commands. Remember that lines following a command using `exec` would be ignored. For example, to start `xscreensaver` in the background and then start `openbox`, use the following:

    ~/.xinitrc

    ...
    xscreensaver &
    exec openbox-session

### Usage

To run Xorg as a regular user, issue:

    $ startx

## i3

[How do I start i3?](https://faq.i3wm.org/question/6126/how-do-i-start-i3/index.html)

> The easiest way to get started is to edit (or create, if misisng) ~/.xinitrc. If it didn't exist, simply put exec i3 in there. If it exists, check the bottom for some exec call, comment it out and put exec i3 instead.
