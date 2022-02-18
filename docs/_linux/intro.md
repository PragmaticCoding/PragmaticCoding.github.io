---
title: The Absolute Beginners Guide to Linux
short_title: Introduction
permalink: /linux/intro
excerpt: A guide to show you just information to get you going with Linux, or to manage your new Raspberry Pi.
---

# Introduction

The purpose of this series is to introduce you to the wonders of Linux through the Command Line Interface (CLI).

Why would you need to know this?

## Headless Servers

A great many Linux systems are deployed as what are known as "headless servers".  This means that they are connected to a network, but do not have a keyboard, mouse or monitor attached to them.  The only way to communicate with them is to connect over the network, and then use whatever user interface they have available.

It *is* possible to configure a headless server with a network compatible graphic interface, like X-11, but, to be honest, this is rarely done and often not worth the trouble.

## Utilities and Tools

Unix has been around for far longer than any of the common GUI desktops.  That meant that for decades, those old "green screen" tubes where all we had, and that meant a 80x24 character display.  There are hundreds, if not thousands, of incredibly useful - even irreplaceable - utilities and tools that are designed for the CLI.  The entire ecosystem has been designed to work together and the result is nothing short of amazing.

There were times in the past when I would actually transfer files from my Windows workstation to a Unix server just so that I could bring those tools and utilities into play and solve problems that simply couldn't be done on Windows.

# Linux and Distributions

All versions of Linux are based on a common "Linux Kernel".  Like any other piece of software, it's been maintained, updated and improved over the years, and there are a number of different versions in use at any given time out in the wild.

But what about Debian, Arch, Ubuntu, Red Hat and all the others?

These are called "distributions".  They're all Linux, and they all start off from some version of the Linux kernel, but then they branch out.

There's lot's of other software that goes into making up a complete operating system besides the kernel.  There are different philosophies about how the user should interact with the system, how additional functionality should be implemented and approaches to updating and installing components.  These different approaches are what mostly defines the differences between the distributions.

Some distributions are designed to be full-featured, even heavyweight, general purpose GUI desktop environments.  Others are designed to be stripped down to the bare essentials, so that they can run on cheap, low-powered CPU's with minimal memory and storage.  An example of this would be a distribution called, "BusyBox".  BusyBox is everywhere.  Chances are you've used something like a storage device or a WiFi router, or an MP3 player which has been based on BusyBox.  Pretty much anything which has a CPU can run BusyBox.

Others are designed to run on out of date laptops, Linux can run fine on older systems that can no longer run the latest version of Windows.  Others, like Mint, are designed to provide a gentle transition from Windows, with a user experience that feels very much like Windows.

Some distributions like Manjaro or Ubuntu are forks from older distributions, where some developers wanted to take those older distributions in a slightly different direction from the original distribution.

The good thing is that, for most of the things we're going to look at, none of that matters.  The concepts and the commands are going to be pretty much the same for every version of the Kernel and for every distribution.
