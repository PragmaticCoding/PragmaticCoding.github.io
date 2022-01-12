---
layout: single
title: A Career Spent Programming In the Fringes
short_title: Fringes
toc: true
toc_label: "Contents"
toc_icon: "brain"
toc_sticky: true
header:
  overlay_image: /assets/images/brain.jpg
  overlay_filter: 0.4

sidebar:
    nav: "master"
permalink: /fringes/
IBM1130: /assets/images/IBM1130.JPG
DPS6: /assets/images/DPS6.jpg
HP9000: /assets/images/HP9000.jpg
---

![IBM 1130]({{page.IBM1130}}){:class="img-responsive" style="float: right" width="240" padding-right="20px"}
I learned to program in FORTRAN on punch cards with an IBM 1130 system (that's one just like it in the picture), but by the time I was finished with school and out looking for a job we had these fancy new things called "Personal Computers".  

My first real job was for a large dairy company, in their brand new "End User Computing" group.  What that really meant was that two of us were building stuff on PC's in an IT department which had about 40 COBOL programmers.  So, out on the fringe right away.

We were working with the tools of the day, which largely meant Lotus 123 (back then building spreadsheets was a programmer's job) and dBase.  Towards the end of my time in that job, we came across a copy of a database manager called "Revelation".  I have no idea who puchased it, or when, but we started building a system to track ingredients in frozen novelties.

#### Revelation

When I left, I found a contract working for the government on a project to improve the quality of beef, pork and sheep through genetic selection.  I worked mostly on the sheep programme.  Technicians would go to farms during lambing season and weigh newborn lambs, then they'd be weighed again when they were weened and again a couple of months later.  All of the information would come back to our office, and our systems were used to track and analyze the data.  The results were used by the farmers to decide which lambs to keep for breeding.

The amount of data we processed was enormous for the time, especially since we were doing it all on PC's.  The workstations were IBM XT's, and the servers were a pair of state-of-the-art Compaq 386's.  All running on a Token Ring network.  I remember that the database files were so big that they exceeded the capacity of the tapes we were using for backups, so we needed to use tape systems that allowed a single file to span tapes.  The files were probably about 90MB or so.

We used Revelation in that job because it was the only DBMS that could handle the volume of data that we had on a PC based system.  It also worked brilliantly in a multi-user environment, unlike the alternatives - dBase and Clipper.  One thing that really sticks in my mind, after all these years, is that we never had any performance issues, even with a fair number of data entry clerks running full tilt all day and the huge amount of data that we had.

#### Ultimate

By the time I was ready to move on, I had discovered that Revelation was a variant of something called, "The Pick Operating System", but designed to run under DOS on a PC.  Most of the other variants were built to run on bigger servers, and actually did comprise the operating system.  They all had self-important names, like "Ultimate", "Prime", "Reality" and "Universe".

![Another Image]({{page.DPS6}}){:class="img-responsive" style="float: left" width="260" padding="20px"}
It turns out that systems built on these operating systems were really cost effective compared to mainframes, and smaller companies or factories could afford to install one of these systems at a really reasonable price.  This created a market for specialist software companies, who created turnkey systems to run a business on these platforms.  

This beast to the left is a Honeywell-Bull DPS6, which Ultimate modified to become their flagship computer.  It was over 6 feet tall, and the unit on the left is a reel-to-reel tape drive.

In my area, there seemed to be a lot of Ultimate shops, and I ended up working for a few of those.

Eventually, I ended up working for Ultimate Canada, which had originally been an Ultimate VAR and a software house building heavy manufacturing and distribution software.  They had been bought out by Ultimate to become their regional office, while still maintaining the software business.

#### Universe

![Another Image]({{page.HP9000}}){:class="img-responsive" style="float: right" width="200" padding="20px"}
At this point, we were in the early 1990's, and Hewlett-Packard and IBM came out with their spiffy new RISC based servers running their versions of Unix.  These systems were an amazing price breakthrough, and ran about 10 times faster than the proprietary hardware the Pick-based systems were running on at about the same price.  This was the death knell for most of those companies.

There was a mad scramble to port these systems over to Unix, and most of the big players in the market failed miserably.  Ultimate completely botched the transition and was gone in a couple of years.  Some of the employees at Ultimate Canada bought the rights to the manufacturing software and started a new company.

The big winner from this was a company called VMark, which made "Universe".  Universe had always run on Unix systems, and it worked well.  The way that Revelation worked under DOS.  Virtually all of the other companies had failed by 1994.  

So we transitioned customers over to Universe when we could, and kept on working with our software.  Eventually I ended up employed by bank, working primarily on their mortgage system - in Universe.

In 1996 I found a job working for a small professional liability insurance company which had just bought a system from a small software vendor out of New Jersey.  The system ran on an RS6000 in Universe.  This company thought they needed some in-house programming expertise, so they hired me.  And I stayed there for almost 25 years - and so did that system.
