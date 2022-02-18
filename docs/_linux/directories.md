---
title: The Absolute Beginners Guide to Linux
short_title: Directories and Listings
permalink: /linux/directories
excerpt: What you need to know about directories and listing files.
---

# Introduction

Just like Windows (rather, Windows is just like Unix - since Unix came first), Linux has the idea of dividing the disk space into folders, or "directories", in a hierarchal structure.  Every file in the system has a "name" and a "path".  The "path" is the sequence directories and subdirectories from the very top of the structure down to the file that you are referring to.  

The directory at the very top of the structure is called the "root" directory.

# Specifying Paths

Unlike Windows, Linux uses the forward slash ("/") to separate the directories and subdirectories.  A complete path always starts with the root directory which is indicated by a slash ("/").  A typical path would look something like this:

``` shell
/usr/local/bin/myscript.sh
```

# The "Working Directory"

When you are in the CLI of Linux, there is always the idea that there is a current "working directory" that will be used by default if you do not specify an exact path to a file.  

Most Linux distributions come "out of the box" with the prompt set up to show you what the current working directory is.  Something like this:

``` shell
dave@MyComputer:/usr/local/bin
$
```
If your distribution doesn't do this, you can set it up so that it does.  This will be covered in a later section.

## Changing the Current Directory

For this, we use the "cd" command.  Just like any other command, if you specify the complete path, it will take you right to it, otherwise it will assume that the path starts from your current working directory:

``` shell
dave@MyComputer:/var/opt
$ cd /usr/local
dave@MyComputer:/usr/local
$ cd include
dave@MyComputer:/usr/local/include
$ ll
```
You can go "up" one level by using the special indicator for the parent of a directory, "..".  You can do this multiple times to go up several levels:

``` shell
dave@MyComputer:/usr/local/include
$ cd ..
dave@MyComputer:/usr/local
$ cd include
dave@MyComputer:/usr/local/include
$ cd ../..
dave@MyComputer:/usr
$
```

One last trick for now.  You can go back to the last working directory that you had by specifying "-".  Like this:

``` shell
dave@MyComputer:/opt/turtl
$ cd /var
dave@MyComputer:/var
$ cd -
/opt/turtl
dave@MyComputer:/opt/turtl
$ cd -
/var
dave@MyComputer:/var
$
```

It's also important to know that the "cd" command is a part of the shell, and isn't a stand-alone utility.  So if you are looking for information about it in the man pages, you'll need to check the entry for the shell, not `cd`.

# Listing Files - The "ls" Command

Probably one of the most useful commands in Linux is the ls command, which will show you the contents of a directory.  It works like this:

``` shell
dave@MyComputer:/
$ ls /opt/turtl
blink_image_resources_200_percent.pak  locales
content_resources_200_percent.pak      natives_blob.bin
content_shell.pak                      pdf_viewer_resources.pak
icon.png                               resources
icudtl.dat                             snapshot_blob.bin
libffmpeg.so                           turtl
libnode.so                             ui_resources_200_percent.pak
LICENSE                                version
LICENSES.chromium.html                 views_resources_200_percent.pak

```

If you don't specify a directory, it assumes that the current working directory is what you want:

``` shell
dave@MyComputer:/opt/turtl
$ ls
blink_image_resources_200_percent.pak  locales
content_resources_200_percent.pak      natives_blob.bin
content_shell.pak                      pdf_viewer_resources.pak
icon.png                               resources
icudtl.dat                             snapshot_blob.bin
libffmpeg.so                           turtl
libnode.so                             ui_resources_200_percent.pak
LICENSE                                version
LICENSES.chromium.html                 views_resources_200_percent.pak

```

The standard form of the ls command just gives a list of file and directory names, which isn't often all that useful.  There is a "long" option, which gives more interesting information:

``` shell
dave@MyComputer:/opt/turtl
$ ls -l
total 118892
-rw-r--r-- 1 root root    25856 Aug 20 17:06 blink_image_resources_200_percent.pak
-rw-r--r-- 1 root root       15 Aug 20 17:06 content_resources_200_percent.pak
-rw-r--r-- 1 root root 10282921 Aug 20 17:06 content_shell.pak
-rw-r--r-- 1 root root     1472 Aug 20 17:06 icon.png
-rw-r--r-- 1 root root 10130560 Aug 20 17:06 icudtl.dat
-rw-r--r-- 1 root root  2882720 Aug 20 17:06 libffmpeg.so
-rw-r--r-- 1 root root 20499904 Aug 20 17:06 libnode.so
-rw-r--r-- 1 root root     1060 Aug 20 17:06 LICENSE
-rw-r--r-- 1 root root  1789407 Aug 20 17:06 LICENSES.chromium.html
drwxr-xr-x 2 root root     4096 Aug 20 17:06 locales
-rw-r--r-- 1 root root   239010 Aug 20 17:06 natives_blob.bin
-rw-r--r-- 1 root root   140979 Aug 20 17:06 pdf_viewer_resources.pak
drwxr-xr-x 3 root root     4096 Aug 20 17:06 resources
-rw-r--r-- 1 root root  1559008 Aug 20 17:06 snapshot_blob.bin
-rwxr-xr-x 1 root root 73936808 Aug 20 17:06 turtl
-rw-r--r-- 1 root root   151829 Aug 20 17:06 ui_resources_200_percent.pak
-rw-r--r-- 1 root root        6 Aug 20 17:06 version
-rw-r--r-- 1 root root    57761 Aug 20 17:06 views_resources_200_percent.pak

```

The first column of stuff, with the r's, w's and dashes are the permissions on the entry, we'll go into that later.  The two columns with "root" indicate the owner and the group for the file, also more on that later.  

Ignore the next number, it's not going to be of any interest to you.  Then we have the size, the last date/time that it was updated and, finally, the name.

Additionally, the very first character on the line will indicate if there is something special about the file, usually if it's a directory.  You can see that all of the entries in `opt/turtl` have nothing special about them except for `locales`, and `resources`, which are both directories.

You can also specify specific files to list.  For instance:

``` shell
dave@MyComputer:/opt/turtl
$ ls -l version
-rw-r--r-- 1 root root 6 Aug 20 17:06 version
dave@MyComputer:/opt/turtl
$ ls -l v*
-rw-r--r-- 1 root root     6 Aug 20 17:06 version
-rw-r--r-- 1 root root 57761 Aug 20 17:06 views_resources_200_percent.pak

```
Note that you can use wildcards to specify files here.  In the second case we specified `ls -l v*`.
