---
title: The Absolute Beginners Guide to Linux
short_title: Introduction
permalink: /linux/bash
excerpt: A first look at the Bash shell and the most important things you need to know about it.
---

# Introduction

"Bash" stands for "Bourne Again Shell", which is a punny way of saying that it's a re-imagining of the Bourne shell.  None of which has anything to do with spy novels.

Back in the early days of Unix, there were three main shells:  The Bourne shell("sh"), the C shell("csh"), and the Korn shell("ksh").  The Bourne shell was always the vanilla, "standard" shell, while the C shell had a syntax resembling the C programming language and the Korn shell was packed with a lot more complexities and features than the Bourne shell.

Personally, I always used the Korn shell, and I have to say that the Bash shell feels very much like the Korn shell.  They are very similar in functionality.  

We're going to look at the Bash shell here, and not worry about the others.  Bash is the default that you'll find in most Linux environment nowadays, anyways.

# What Is a Shell?

A shell is just a program, like any other, except that it's interactive.  It accepts what you type in, executes programs for you and handles the input and output to and from those programs, and it displays stuff on the screen.  It's also special because it's the program that is invoked by the "login" program after you've typed in the correct password.  If you kill off your shell, your connection to the system will be dropped.

# Command Structure

Linux commands have a fairly common structure:

``` shell
$ command [option][option]...[option] arguments
```
That probably seems pretty cryptic:

command
: This is the name of the command.  Something like `ls`, or `cd` or `mkdir`.

[option]
: Options are generally expressed with a hyphen (`-`), and one or two letters.  That's the Unix way.  You just need to know what the letters mean.  Some utilities support alternate forms with double hyphens and long names.  If an option has an argument that goes with it, then it will generally be preceded by a space.

arguments
: Most utilities are going to assume some quantity of information is going to always be needed.  Like a file name to work on, or a search string.  Arguments don't have any special tags, and you generally just need to know what they are and what order they go in.

## Some Examples

Here's the command to search for a string in a file:

``` shell
$ grep "abc" fred.data
```
The command is "grep" and there are no options, but two arguments.  The first is the string to search for, and the second is the file to search in.  This command will print every line in `fred.data` that has the string "abc" in it.

We can add an option:
``` shell
$ grep -v "abc" fred.data
```
Here we've added the `-v` option, which reverses the sense of the search.  Now it will print every line in `fred.data` that does NOT have "abc" in it.

If the options don't have any parameters, you can string them together behind a single `-`.  Here we'll add the option to count the lines instead of printing them:

``` shell
$ grep -vc "abc" fred.data
```

# Standard Files

Every program that runs in Linux is passed pointers to three "files", which all normally point to the terminal.  You really, really need to understand these:

Standard In(stdin)
: This is an input file.  Data will be read from here.

Standard Out(stdout)
: This is an output file, data will be written to here.

Standard Error(stderr)
: This is an output file, any data related to errors that happen should be written to here.

Simple, right?  Many, many Linux utilities are designed to read their data from `stdin`, write the results on `stdout` and write any errors on `stderr`.

What's so important about that?

You can manipulate all of these files through the shell.  Let's look at the key operations:

## Redirecting stdout to Disk File

The first thing you can do is that you can tell the shell to take all of the output from a utility and redirect it to a disk file.  This allows you to save the output from a program.  You do this by using the "redirect out" operator `>`.  Let's look at this in action:

``` shell
dave@MyComputer:~/demo
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

dave@MyComputer:~/demo
$ ls /opt/turtl > test.out

dave@MyComputer:~/demo
$ cat test.out
blink_image_resources_200_percent.pak
content_resources_200_percent.pak
content_shell.pak
icon.png
icudtl.dat
libffmpeg.so
libnode.so
LICENSE
LICENSES.chromium.html
locales
natives_blob.bin
pdf_viewer_resources.pak
resources
snapshot_blob.bin
turtl
ui_resources_200_percent.pak
version
views_resources_200_percent.pak
```
In this example we've done a listing of `/opt/turtl`, once to see what it looks like, and then again but saving the results to `test.out`.  Then we've used the `cat` command to print the results to the screen.  The `cat` command is intended to be used to concatenate two files together, joining them one after the other and writing the results to `stdout`.  But if you just give it one file then it will essentially just write that file to `stdout` - in this case, the screen.

One thing you'll see is that the you get two columns of output just running the `ls` command without any redirection, but you get only one column if you redirect it to a file.  So the `ls` program is aware that `stdout` isn't going to be the screen.  In most cases, if you redirect the output of `ls` to a file, you're going to find it's easier to deal with in a single column if you're going to run that output through some other utility.

## Redirect stdin From a Disk File

Now we have some test data, let's run it through a utility to do something with it.  For this example, we'll use `wc`, a program that counts the lines, words and characters in a file.

`wc` can actually be told to read a file, but if you don't specify a file, then it assumes it's `stdin`.  Let's try both ways:

``` shell
dave@MyComputer:~/demo
$ wc test.out
 18  18 318 test.out

dave@MyComputer:~/demo
$ wc
abt
efge otd
to utean onot
      3       6      27
```
Here we've run `wc` twice.  The first time we told it to count `test.out`.  The results were 18 lines, 18 words and 318 characters.  Then we just entered the command `wc` with no arguments, so it assumed that we meant `stdin`.  In case though, `stdin` is just the terminal, so it drops down a line and just blinks at you.  Then I typed a few random letters and hit `<Enter>`, and it just blinked at me some more.  So I typed a few more lines of random stuff, and I hit `<Ctrl>D`.  

`<Ctrl>D` is the code for "End of File", or in technical terms `EOF`.  

That signaled to `wc` that the end of the input had come and it generated the results: 3 lines, 6 words and 27 characters.

Now lets redirect `stdin` to come from `test.out`.  You do this with the "redirect in" operator, `<`.  Notice how it's just the opposite angle bracket from redirect out.  Think of them as arrows.  "Out" points away from the command, "In" points towards it.

The command looks like this:

``` shell
dave@MyComputer:~/demo
$ wc < test.out
 18  18 318
```
And it gives the same results as `wc test.out`.

## Redirecting stdout From One Command into stdin of a Second

This is probably the most common use of all of this redirection stuff.  Do in and out at the same time to put the output of one command into the input of another command.  This is called "piping", and it's done using the "piping symbol", `|`.  

First we'll look at using `cat` to read a file from disk and write it to `stdout`, and then we'll pass it to `wc` as its input:

``` shell
dave@MyComputer:~/demo
$ cat test.out | wcresources
     18      18     318resources
```
As it stands, this isn't terribly useful, since we know that `wc` can just take a filename as an argument.  But there are commands that only read `stdin`, and this is a viable technique to use them with a disk file.  Just pipe it through `cat`.

More often, though, you are actually going to run a command that does something interesting, and then run it through one or more commands to manipulate the data and end up with even more interesting results.  

So we'll try that.  Instead of using `test.out`, we'll create the `ls` listing from scratch, then run it through `wc`.

``` shell
dave@MyComputer:~/demo
$ ls /opt/turtl | wc
     18      18     318
```
It's the same results as before, just as you would expect.

But what if we only wanted to know how many files were in `/opt/turtl`?  How would we do that?  In that case, we only want the first number that comes out of `wc`, the line count.  Here we can use another utility called `cut`, which is used for pulling particular fields or columns out of an input file.  

But we have a problem.  `cut` can use the spaces as a delimiter between the values, but there's lots of extra spaces in there which are probably going to change if the numbers get bigger or smaller.  You can see that there are less spaces between the "18" and the "318" than between the two "18"'s.  We can use another utility, `tr` (which stands for "translate"), to get rid of extra spaces.

Let's try it:

``` shell
dave@MyComputer:~/demo
$ ls /opt/turtl | wc |tr -s " "
 18 18 318
```
That looks better.  The `-s` option is the "squish" option which removes repeated instances of whatever characters are specified, in this case, it's spaces.

So now we're ready to run it through `cut`:

``` shell
dave@MyComputer:~/demo
$ ls /opt/turtl | wc |tr -s " "|cut -d" " -f2
18

```
And there you go.  The `-d` option specifies the delimiter, and the `-f` says what field to grab.  Since we have a leading space, we want field 2, which is the first "18".

This is just a demonstration to show piping, but in real life you'd just use the `-l` option of `wc` to report just the lines:

``` shell
dave@MyComputer:~/demo
$ ls /opt/turtl | wc -l
18
```
It's always good to read the man pages!
