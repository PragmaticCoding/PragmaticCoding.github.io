---
title: The Absolute Beginners Guide to Linux
short_title: Getting Started
permalink: /linux/start
excerpt: How to get started.
---

# Introduction

This is the first in a series of articles that will tell you just about everything you really need to know about Linux in order to use it do useful stuff.  We're going to concentrate on the command line interface (or the "CLI"), since that's the part that seems to be the hardest to master.

Understanding the CLI in Linux will allow you to do all kinds of things you couldn't do otherwise, even though most Linux versions come with a great GUI nowadays.  Of course if you are managing a headless server, or a Raspberry Pi, the CLI may be the only choice you have.

## The Structure of Linux

Linux consists of a few basic parts.  It doesn't matter what version of Linux you use, all of these parts will be there is some fashion:

Kernel
: This is the part that actually runs the computer.  It will manage memory and cpu and control processes.  Generally speaking, this is the part that makes Linux Linux, and not some other variant of Unix.  

Shell
: This is the program that you're talking to when you type in commands at the prompt.  There are a variety of shells available for Linux, but the most common is Bash ("Bourne Again Shell"), which is a variant of the Bourne Shell (also known as "sh").  There are others, but you don't need to know about them.

Utilities
: This is a huge suite of programs, many of which have been around since the 1970's, which do things for you.  For the most part, when you type in a command in the Shell, the first word is going to be either the name of a utility, or a built in command in the shell.

# How to Get to the Command Line Interface

If you're looking at GUI desktop on your Linux system, you just need to run a program called "Terminal".  You should be able to find it in the menus or the dock pretty easily.

If you have a headless server, meaning that you don't have keyboard and monitor hooked up to your Linux system, then you'll need to connect to it using SSH.  If you are using Windows, your best bet is to download and install a utility called PuTTY.


# How to Get Help

Probably one of the most important commands in Linux is the one that will give you help about a command.  Virtually every utility in Linux has an entry in the "Manual", also known as the "Manual Pages", and most often, the "Man Pages".  This will explain why the command to show you information about commands is called `man`.

Let's start off by getting some information about `man` itself.  Try this:

```
$ man man
```
And you should get something like this:

```
MAN(1)                           Manual pager utils                          MAN(1)

NAME
       man - an interface to the system reference manuals

SYNOPSIS
       man [man options] [[section] page ...] ...
       man -k [apropos options] regexp ...
       man -K [man options] [section] term ...
       man -f [whatis options] page ...
       man -l [man options] file ...
       man -w|-W [man options] page ...

DESCRIPTION
       man  is  the system's manual pager.  Each page argument given to man is nor‐
       mally the name of a program, utility or function.  The manual  page  associ‐
       ated  with  each of these arguments is then found and displayed.  A section,
       if provided, will direct man to look only in that  section  of  the  manual.
       The default action is to search in all of the available sections following a
       pre-defined order (see DEFAULTS), and to show only  the  first  page  found,
       even if page exists in several sections.

       The  table  below  shows  the  section numbers of the manual followed by the
       types of pages they contain.

       1   Executable programs or shell commands
       2   System calls (functions provided by the kernel)
       3   Library calls (functions within program libraries)
       4   Special files (usually found in /dev)
       5   File formats and conventions, e.g. /etc/passwd
       6   Games
       7   Miscellaneous (including macro packages and conventions),  e.g.  man(7),
           groff(7)
       8   System administration commands (usually only for root)
       9   Kernel routines [Non standard]

       A manual page consists of several sections.
```
Plus a lot more information which you can page through.

The top part, the " SYNOPSIS" will tell you how to enter the command to run the utility, with as many forms as it will understand.  Further down in the listing, there will be a detailed description of all of the options, what they mean and how to use them.
