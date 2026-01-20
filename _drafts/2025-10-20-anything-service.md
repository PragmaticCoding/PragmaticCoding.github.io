---
title:  "Linux: Run (Almost) Anything as a Service"
date:   2025-10-20 12:00:00 -0500
categories: javafx
logo: /assets/logos/JavaFXLogo.png
permalink: /linux/anything_as_service
ScreenSnap0: /assets/elements/ConstrainedProperty0.png
ScreenSnap1: /assets/elements/ConstrainedProperty1.png
ScreenSnap2: /assets/elements/ConstrainedProperty2.png
ScreenSnap3: /assets/elements/ConstrainedProperty3.png
ScreenSnap4: /assets/elements/ConstrainedProperty4.png
ScreenSnap5: /assets/elements/ConstrainedProperty5.png
ScreenSnap6: /assets/elements/ConstrainedProperty6.png
ScreenSnap7: /assets/elements/ConstrainedProperty7.png
ScreenSnap8: /assets/elements/ConstrainedProperty8.png
ScreenSnap9: /assets/elements/ConstrainedProperty9.png
ScreenSnap10: /assets/elements/ConstrainedProperty10.png

Diagram: /assets/elements/ListProperties.png
OLArticle: /javafx/elements/observable-classes-lists
ABGuide: /beginners/intro
GitHubLink: https://github.com/thomasnield/ConstrainedProperty

JavaDocOL: https://openjfx.io/javadoc/23/javafx.base/javafx/collections/ObservableList.html
JavaDocFXC: https://openjfx.io/javadoc/23/javafx.base/javafx/collections/FXCollections.html

excerpt: "A quick tutorial about how to congigure any persistent process as a service using systemd."
---

# Introduction

Years back, I worked on systems that were hosted on Unix.  Not Linux, but System V Unix.  Mostly HP-UX.  

We viewed these systems as our "Swiss Army Knives" in the Data Centre, because they were always up - sometimes going years between reboots - and had all the tools we needed to perform the day to day tasks that came up.  We wrote scripts in `ksh` when we could, and programs in C when we had to.  We used utilities like `sed`, `awk` and `curl` to do all kinds of things you'd never think to do today because now we have systems to do these things properly.

As an example, we had issues with the A/C in our computer room that went on for years.  So we went out and bought a thermometer with a network adapter.  This thermometer had an IP address and hosted a web page.  You could surf to it and see the current temperature.  Then we when to one of our Unix servers and we wrote a script that would use `curl` to grap the web page and then a bunch of other utilities to parse the HTML and extract the text string that held the temperature.  If it was over some set amount, like 72F, then it would use mailx to send an email an admin group.

For this, we had `cron`.  We set it up so that it ran every 15 minutes or so, and then we just forgot about it...until the A/C went on the fritz.  At some point we created a monthly maintenance routine for the network admin to perform, and it included dropping the threshold temperature down to 50F just to make sure that the script still worked and the email was sent.

Of course, nowadays, you'd never do any of this.  Any network aware thermometer that you could buy would have a built-in REST API or support some modern version of SNMP, and there would be a plug-in for whatever network management dashboard tool you might be using.  But 20+ years ago, we didn't have those things and all of this stuff was strickly DYI.

## /etc/inittab

The example above used `cron` because the job itself ran in a few milliseconds and then terminated.  We just needed it to be launched a few times each hour, which is exactly what `cron` is good at.

Sometimes, however, you needed to run something that had to keep running and never stop.  You could do this at the console with the `&` option:

```
> somecommand &
```
This would run `somecommand` in the background, and the prompt would return right away.  It was usually a good idea to send any output to a log file of some sort:
```
> somecommand > /var/log/somecommand.out 2>&1 &
```
However, if you subsequently logged out of your terminal session, any background tasks you had running would end.  This is because would all be sent a "hang-up" or a "hup" signal.  You could avoid this by using `nohup`:
```
> nohup somecommand > /var/log/somecommand.out 2>&1 &
```
This was fine if you wanted to manually start up a job and have it keep running, and was usually the best way to develop, test and debug one of these commands.  

But what if you wanted the job to start every time the system booted?

Those Unix systems used something called `init` as the root controlling every process on the system.  `init` could do all kinds of things, and its main function was to launch various functions as the system was booted.  To control that, we had `/etc/inittab`.  Those systems defined something called "run levels", which described the various stages of the system being up and running.  There was a run level for single user mode, and one for multiuser, and then one for multiuser with networking enabled.  You'd specify which run level you wanted your program to run in, and when the system reached that level, it would execute your command.  Entries looked like this:

```
x783:3:respawn:/usr/local/bin/somecommand > /var/log/somecommand.out 2>&1
```
Here, "respawn" means that if `somecommand` were to terminate, `init` would restart it.  The "x783" was an identifier, and the "3" meant that it was to be executed when the system reached Run Level 3.

## Most Linux Distros Use systemd

The vast majority of Linux distros have moved away from `init`, and now use something called `systemd`.  Apparently, `systemd` is somewhat divisive, and some people love it while others hate it.  We won't worry about that, though, we'll just look at how to deal with it.

What is clear is that setting up a program to run in `systemd` is not a simple as just adding a line to `/etc/inittab`.  Usually, when you add a new package using `apt`, or `apt-get` or `podman` that implements a service, the install process will handle all of the `systemd` details so that you don't have to worry about it.

What we're going to look at here is how to take some other program, one that doesn't automatically install as a service, and turn it into a service with `systemd`.

# Deploying Jekyll as a Service

As an example I'm going to show how to deploy something that I use when creating this website as a service: Jekyll.

## My Website Workflow

I create this website using an editor called `Pulsar`, which is a fork of the now defunct `Atom`.  You can think of it as much like `VSCode`, except it's better - mostly because it's not `VSCode`.  It supports `Git` and `GitHub` nicely, and does all the things I need it to do.

The website itself is written in markdown, using the `Liquid` templating language and `Kramdown`. Styling is provided through the `MinimalMistakes` template, which I have customized to a small degree.  

To build the static site, I use `Jekyll`, which has dictated most of the other decisions about markdown.  `Jekyll` itself is compatible with whatever `GitHub Pages` uses (the `Jekyll` website says that it *is* `Jekyll` that they use), which was actually the key criteria in everything.

I need to stress that my interest in all things HTML, CSS, JavaScript and web is pretty much zero, and will never, ever be non-zero.  I view all of this stuff as nothing more than the toolbox I need to master just enough to get my website looking the way that I want.  I'm not an expert on how any of these parts work.  Also, a vast amount of the design decisions I made were back at the beginning, when I knew even less than I know today.  I'm stuck with the results of those decisions unless I want to go back and rewire everything I've done over the past few years.

When I'm typing away on an article like this, I like to see it as it will appear on the website from time to time.  `Jekyll` can handle this for me, I run `Jekyll` with the "server" option and it persists and monitors the directories with the markdown and styling files.  If it detects a file added, removed or modified, then it rebuilds the site.  Additionally, as long as it is running it acts as a web server, and I can surf to `http://localhost:4000` with a browser to see the rendered site.  

Finally, when I'm happy with the work I use `Pulsar's` integration with `Git` to push the changes to `GitHub`.  Then a workflow runs on `GitHub` and it runs it's own version of `Jekyll` to build the static site data and publish the site.  

This has work fairly well for the past four years or so.  However, it's a bit of a pain that I have to fire up the `Jekyll` server on my local machine when I want to work on the website.  Yes, I can just leave it running, but it feels like a loose end.

## Deploying to a Homelab

Recently, I've started working on a Homelab.  I have a couple of Lenovo M910Q's running `Proxmox` as a hypervisor, and I've started deploying some services like DNS/AdBlocking, HomeAssistant and JellyFin.  It's virtually trivial to spin up a new Linux container (LXC) to host a single service.  

It occured to me that `Jekyll` was an ideal candidate for deployment in the Homelab.

One of the wonderful things about Unix/Linux is that networking is baked-in from the get-go.  You can freely mount drives between Linux systems using `NFS` and everything is virtually seemless.  Yes, it is possible to programitically determine that a file system is `NFS` mounted if you really want to, but for the most part you can treat an `NFS` mounted file system exactly the same way you would a local file system.  Networking speed will certainly be a factor, but for many applications it is a very small, even negligible, factor.

I spun up a new `LXC` running `Ubuntu`.  I chose `Ubuntu` because that's what I running on my workstation (well, `Kubuntu`, but it's practically the same thing with regards to `Jekyll`).  I know that `Jekyll` works well with `Ubuntu` and the installation procedure will be the same.  In case you're interested, I create most of the other VM's and LXC's with `Debian` because it's a little bit more generic.  

Once the container was up and running, I made sure that `NFS` was installed.  It appears that `Ubuntu` comes with `NFS` fully installed, which `Debian` does not.  It also didn't require me to do any messing around with `/etc/hosts.allow` or any security stuff to enable `NFS` access.  

The next step was to create an `NFS` export directory for the website data.  I chose `/mnt/BlogData`, and then changed the ownership and permissions to make sure that my workstation could write to it once it was exported.  Then I added an entry into `/etc/exports` for that directory, specifying my workstation as a read/write client.  

Next, I `NFS` mounted the directory on my workstation, and I used a recursive `cp` command to copy the website data from my workstation to the new server container.  Once that was all done, I created an entry in `/etc/fstab` on my workstation so that it would automatically mount the website directory when it booted.

With that done, the next step was to install `Jekyll` and get it building the website on the server from the command line.

As far a I know, `Jekyll` is a `Ruby` application, and you use an application called `Bundler` to load `Gems` which it needs to run.  Most of the heavy lifting of this stuff for my website was done years ago, and the website data contains the `GemFiles` that tell `Ruby`, through `Bundler` what it needs to load.  The process of getting it up and running involved installing `Ruby` and some related packages using `apt-get`, setting some environment variables, and then using `Ruby` to install `Jekyll` and `Bundler`.

At this point, I could now use `Bundler` to run `Jekyll`, which downloaded a bunch of stuff it needed, and then proceeded to build the static site and present a web server.  

Now we can thing about creating a service from this.

## Creating the Service

### systemctl

The fundamental command to deal with `systemd` services is `systemctl`.  The general format is:

```
>systemctl {command} {service_name}
```
Common commands that you might use are:

start
: Starts the specified service.

stop
: Stops the specified service.

restart
: Stops, and then starts the specified service.

status
: Shows whether the specified is running or stopped, and some other useful information.

enable/disable
: Tells the `systemd` whether or not the specified service should be started at boot-time.

You'll need to use `systemctl` a lot as you create your service.  So you should become comfortable with how it works.

Most of the functionality of `systemctl` is privileged, so you'll either need to run it as `root` or (preferrably) use `sudo`.

Let's look at the status for the `cron` service:

```
$ sudo systemctl status cron
● cron.service - Regular background program processing daemon
     Loaded: loaded (/usr/lib/systemd/system/cron.service; enabled; preset: enabled)
     Active: active (running) since Tue 2025-10-21 14:04:13 UTC; 2 days ago
       Docs: man:cron(8)
   Main PID: 216 (cron)
      Tasks: 1 (limit: 18897)
     Memory: 1.2M (peak: 3.4M)
        CPU: 2.606s
     CGroup: /system.slice/cron.service
             └─216 /usr/sbin/cron -f -P

Oct 23 18:15:01 Blog CRON[14246]: pam_unix(cron:session): session closed for user root
Oct 23 18:24:01 Blog CRON[14274]: pam_unix(cron:session): session opened for user root(uid=0) by root(uid=0)
Oct 23 18:24:01 Blog CRON[14275]: (root) CMD (cd / && run-parts --report /etc/cron.hourly)
Oct 23 18:24:01 Blog CRON[14274]: pam_unix(cron:session): session closed for user root
Oct 23 18:25:01 Blog CRON[14277]: pam_unix(cron:session): session opened for user root(uid=0) by root(uid=0)
Oct 23 18:25:01 Blog CRON[14278]: (root) CMD (command -v debian-sa1 > /dev/null && debian-sa1 1 1)
Oct 23 18:25:01 Blog CRON[14277]: pam_unix(cron:session): session closed for user root
Oct 23 18:35:01 Blog CRON[14284]: pam_unix(cron:session): session opened for user root(uid=0) by root(uid=0)
Oct 23 18:35:01 Blog CRON[14285]: (root) CMD (command -v debian-sa1 > /dev/null && debian-sa1 1 1)
Oct 23 18:35:01 Blog CRON[14284]: pam_unix(cron:session): session closed for user root
```
This has a lot of useful information.  We get a brief description of the service, something about "loaded", and whether or not it's actually running and how long it has been running for.  It also tells us where to get the documentation, and the id of its running process.  At the bottom, we get to see the last `journal` entries for this service in the system logs.  

### systemd Configuration Files
