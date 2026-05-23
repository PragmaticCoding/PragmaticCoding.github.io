---
title:  "Self Hosting Self Assessment"
date:   2025-10-20 12:00:00 -0500
categories: homelab
logo: /assets/logos/JavaFXLogo.png
permalink: /homelab/considerations
M910Q: /assets/homelab/M910Q-Front-Back.png
Stack: /assets/homelab/M910Q-stack.jpg
Meme: /assets/homelab/WeDoThisBecause.jpg
M910_Inside_Bottom: /assets/homelab/M910Q-bottom.jpg
M910Q_Specs: /assets/homelab/ThinkCentre_M910_Tiny_Spec.PDF
M710Q_Specs: /assets/homelab/ThinkCentre_M710_Tiny_Spec.PDF
Stack1k: /assets/homelab/ThinkCentre_M710_Tiny_Spec.PD
Stack1: /assets/homelab/M910Q-Front-Back.png
10Base2_1: /assets/homelab/10Base2-1.jpeg
10Base2_2: /assets/homelab/10Base2-2.png
10Base2_3: /assets/homelab/10Base2-3.jpeg



Diagram: /assets/homelab/SnapCastDiagram.png
OLArticle: /javafx/elements/observable-classes-lists
ABGuide: /beginners/intro
GitHubLink: https://github.com/thomasnield/ConstrainedProperty
InstallationLink: https://pve.proxmox.com/pve-docs/chapter-pve-installation.html

JavaDocOL: https://openjfx.io/javadoc/23/javafx.base/javafx/collections/ObservableList.html
JavaDocFXC: https://openjfx.io/javadoc/23/javafx.base/javafx/collections/FXCollections.html

excerpt: 'Some things you should know before you get into self-hosting.  '
---

# Introduction

At first glance, self-hosting seems like an easy decision: Free yourself up from lots of monthly costs and the tyranny of cloud-based providers and be the master of your own destiny.

But what does it really take to make this happen?  Is it easy?  Do you have the time to do this?  Is it "plug and play"?

TLDR: It's not easy, it's not for everyone, and it takes time and effort.

![Meme]({{page.Meme}})

On the other hand, if you are the right kind of person it can be a lot of fun.

I've been doing this for about 6 months now, so it's been long enough for me to accumulate some experience, but not so long that I've forgotten what my expectations were.  So I can share my experience and you can decide for yourself if this is a path you want to follow.

## I'm Not a Techie, But...

I've always considered myself a programmer first, but a programmer who has always had to deal with infrastructure issues in a professional capacity throughout my career.  This has included hands-on hardware configuration, modification, installation and everything else, as well as strategic planning.  This later has involved making decisions about what kind of technologies we would deploy and how they would work with each other.  

So yes, I've always considered techie stuff to be "plumbing" to a large degree, but plumbing that had to be planned and executed properly to allow everything else to work.  So I've had to learn about technologies, learn about the choices available, and then figure out the oddball quirks about them to make them work the way we needed them to.

This has included all kinds of now obsolete technologies that you'll probably never have heard about.  And a lot of those technologies required way more fiddly technical tweaking to make them work, and work with each other, than their modern replacements do today.  And all of that fiddling has given me a pretty solid experience in the fundamentals, which serves well when doing all of this stuff in a homelab environment.

I also became the local *expert* in Unix and later Linux, and I've had about 30 years experience with all manner of *nix system administration and configuration.  

If you have less experience than me with these technologies, then you'll probably have a harder time with self-hosting than I have.  If you're a professional network admin with certifications in Cisco equipment and VMWare, then you won't - although I don't know why you'd even be reading this article.

### An Example: "Ping and Wiggle"

Back in the day, long before Cat5, we had the wonderment known as 10Base-2 cabling:

![10Base-2]({{page.10Base2_2}})  

I was having trouble finding good pictures of these things, but you can see here how they connect.  You could attach a cable to each side of the top of the "T", like this:

![10Base-2]({{page.10Base2_3}})  

This meant that you could create long chains of devices connected together.

Some of the devices that we used back then didn't have 10Base-T connectors (called "BNC"), but had these gazillion-pin connectors so you needed to get a transceiver for them called an MAU:

![10Base-2]({{page.10Base2_1}})  

This was extra awesome, because now you had two points of failure, and not just one.  And, as you can see, it stuck out 10cm from the back of the device, making it prone to getting knocked about.

10Base-2 was attractive because it didn't require any additional networking devices, like hubs or switches, to connect the computers together.  Just get some coax cabling and BNC connectors and off you go. You never saw these out in the user areas, though, because by the time desktop computers started to be networked regularly, we had 10Base-T.  But you did see 10Base-2 in technical areas, like computer rooms, quite a bit.

The problem with 10Base-2 was that it was flaky.  

Really, really flaky.

The coax cable was stiffer than you'd expect, and it was heavy, so it tended to put a lot of strain on the connectors even under ideal circumstances.  But if someone nudged it, even from across the room, it could cause a bad connection.  You really had to think twice before you went futzing about behind any of the servers.

10Base-2 worked just like christmas tree lights.  If one bulb went out, you lost the whole string.  Same with 10Base-2, if one connection when bad, then the whole network was down.

This meant that we were countinually troubleshooting network outages caused by 10Base-2 connections.

But how do you find the bad connection?  Just like the christmas lights, you can't just look at them and know which one's bad.

This is where we developed "Ping and Wiggle".

It was pretty simple.  Go to the console on any Unix server, log in and start up a `ping` to any other device on the network.  `Ping` runs continuously, displaying a line for each response it receives.  With the network down, it won't print anything.  

Turn the console around, so that you can see it from behind the servers.

Climb behind the server shelves (we mostly had tower servers back then) and go to the first 10Base-2 connector.  Grab the coax cables about 15cm on each side of the BNC adaptor. Give them a gentle wiggle whilst watching the console.  If you've got the bad connection, the console will erupt with `ping` responses.  The try to adjust the cable such that the `ping` responses continue after you've let go of the cables.

Carefully climb out from behind the servers.

I used this as an example because it's real, and you can understand it without needing any deep technical background.

I also used this as an example because it embodies the essence of network administration:

Everything is cobbled together

: In every installation, you'll have a mish-mash of technologies that you'll have to figure out how to make work together.  You can't always replace one technology with a newer, better technology because it may not exist.  We had to live with that 10Base-2 equipment because some of our servers didn't support anything else.

You have to create your own solutions

: The solution that works is the solution that you can invent with the tools and knowledge that you have. On the spot. Now.

Every component has quirks

: You learn what those quirks are and you learn how to aviod them, or to exploit them.

Nobody warned you this would happen

: In my last job, somebody before me decided to use 10Base-2 and there's a good chance that they didn't understand how flaky it was. Certainly, nobody ever said to me, "Watch out! There's 10Base-2 in the computer room!".  Then, one day, the network's down and guess who has to fix it?

In many ways, homelabs and self-hosting is much more like running a computer room back in the 1980's and 1990's than it is like modern network and systems administration. Back then, nobody had certifications or took courses on every different piece of technology.  You knew what you knew because you read the manuals and you had hands-on experience with the equipment.



# Considerations Before You Start

Let's take a look at the things that I have noticed once I started self-hosting, some of which I expected, and others that I did not...

## Proxmox Just Works

Of all of the elements that I've dealt with, Proxmox itself has been the least bothersome.  It just works the way you expect it to, and it is getting better all the time.

## It Doesn't Cost a Lot

My preference has been to go with older, less powerful, computers to use as homelab servers.  I have 3 Lenove M910Q Tiny computers, each with 16GB RAM and 256GB SSD drives.  The cost me about $125 CAD each.  My firewall is an HP T740 mini computer, chosen because it has a PCI riser that would let me install a 2 port network card.  It cost me $135 and another $25 for the network card.  These purchases were spread out over 3-4 months.

I've bought other components as I've needed them.  I now have 3 managed 1GB switches, each of which cost about $35. I just installed a $200 UPS.  If you keep reading you'll see that I bought WiFi access points and MoCa adaptors.  

I also implemented 4 SnapCast clients for whole-home audio.  These are $20 Raspberry Pi Zero's with $20 DAC cards - so about $40 for each client.  I had some audo equipment to use with them, but I also bought a class D amp for $45, and a $30 portable speaker for the bathroom SnapCast client.

In the near future I'll be deploying cameras outside the house, and I'll have to buy them and possibly some more networking equipment to support them.  It's also possible that the NVR software may require some server upgrades. However, I expect to recover a fairly high annual cost from my Google cameras.

But yes, you can spend way more than I have so far.  Lot's of people choose more modern, and more powerful, servers for Proxmox.

What I have noticed is that it takes much, much longer to configure and deploy each technology than it does to purchase them.  This means that there's no point buying anything until you're actually ready to use it.  In a practical sense, it takes months, not days or weeks, until you're ready to move on to the next project that requires new hardware.

## Everything Takes Longer Than You Expect

Maybe not everything, but almost everything.

Some of the items below explain why this happens.  

Another factor is that decisions that you made ealier, sometimes months earlier, will impact your ability to do things going forward.  Then you either have to deal with the implications of those earlier decisions, or go back and change them.  Either choice will cost you time.

In a much more general sense, self-hosting itself just takes up a huge amount of time.


## Online Documentation Stinks

My experience has been that virtually every piece of documentation for everything you might want to implement has one of the following problems:

* It's out of date.
* It's just wrong.
* It's missing key information.
* It's just bad.

Furthermore, when you start searching around in online forums or help sites, you'll find at least some of these:

* Threads that are years and years old.
* Conflicting advice.
* Snarky advice.
* Really, really technical advice you cannot understand.
* Advice for issues that are close, but not quite your issue.
* Wrong advice.

The other day I realized that an LXC that I was working on needed to use NFS, but I had forgotten to create it as privileged at the beginning of the process.  I wouldn't know where to start in the Proxmox documentation, so I did a web search for "Convert LXC from unprivileged to privileged". It came up with a page from the Proxmox forums from 4 or 5 years ago.  Apparently, you cannot directly convert it, but you can create a backup, then restore it setting a flag to make it privileged. There were some examples of commands to issue at the node's CLI to do this.

I'm not a fan of doing this kind of operation through the CLI of the node, so I kept on reading the thread in the hopes that someone might have added some notes a year or two later that had a GUI solution.  But the next thing I came upon was a post from someone who thought the best advice was to research what you trying to do better before you start, so that you don't have these problems.  Maybe true, but clearly not useful in the context of the thread. Responses back and forth on this went on for a while.

Eventually, I found a website article showing how to do it in a more modern version of Proxmox, where the GUI has all the prompts you need to correct the problem.  It turns out to be trivial.

You will have to deal with this kind of situation.  Many times.

## Everything is an Adventure in Exploration

This is the one thing you really need to be prepared for.  

Nobody - and I mean *nobody* - in the world is running the same hardware stack that you are, running the same containers and services, and making the same choices that you have made along the way.  Your homelab is unique in the world.

That means that every issue you come across is likely to have some elements to it that are unique to you.

And *that* means that you have to explore in order to find what works for you, your situation, and the approaches that you prefer to take.  

Here's an example: I reached the point where I was ready to start looking into implementing a reverse proxy.  While I had used NGINX at work a little bit, that was nearly 10 years ago and I lean towards more modern solutions if they are available.  So I decided to try Traefik, which a lot of people seem to think is good and it does a number of other things which might be useful.

However, I also lean towards using LXC over Docker containers when possible. LXC containers are super light, and I don't see the need for an extra layer of complexity if I can avoid it.  I'm not sure, but I think there is a Community-Script to install it directly in an LXC container.  So that's what I did.

I discover some time later, when I try to configure something a little bit more complex, that a direct install is fairly new, and that virtually everybody installs it through Docker.  Furthermore, configuration of Traefik is almost always done be customizing a Docker Compose file, and regenerating the container.  

This means that every single example of how to do anything close to what I wanted to do was presented as snippets of a Docker Compose file.  

Every. Single. One.

Yes, it is possible to puzzle over a Docker Compose file and see how the changes would reflect in the Traefik configuration files, but this appeared to be a mountain of work.

The choice seemed clear, reinstall Traefik as a Docker container, or find another reverse proxy.

I'm not unfamiliar with Docker, but not an expert.  Did I want to get into this just to get reverse proxy working?  Not really.

So I did some more research and decided to go with NGINX Proxy Manager, which is a web UI to configure NGINX. It seems to be working well, and none of the on-line resources demand that I use Docker.  


## Be Prepared For Unexpected Hardware Costs

When I getting ready to deploy my OPNSense firewall, my plan was to convert my NetGear Orbi mesh WiFi router to "AP Mode" and continue to use it.  Immediately before plugging the firewall into the network, I disabled the DHCP server in the Orbi router because I really didn't want any chance of having duplicate IP addresses in the network.

Unfortunately, as soon as I did this it bricked the Orbi router.  I don't know why, and factory resets and everything else I tried would not revive it.  

I carried on with the firewall install just so that I could have wired connectivity and Internet running.  Then I went on to Amazon and ordered a WiFi access point for about $150.  That arrived the next day, and I was up and running again.  I had some issues with coverage with a single AP, so I needed a second one, for another $150.  Later, I decided to move it to a new location to improve the coverage, but that required a pair of MoCa adaptors for another $150.  

In truth, it's now clear that running a consumer WiFi router in AP mode is suboptimal in any event.  So that $450 for access points and MoCa adaptors was going to happen eventually.  But I wasn't expecting it when it did happen.

## At Some Point Your Homelab Becomes "Mission Critical"

When you've just got a standard consumer grade WiFi router, probably supplied by your ISP, plugged into your ISP's modem, and all of your vital services are cloud-based, then your home network is not a vital part of your life in many ways.  You can still get to your Google Docs or Google Keep, you can still see your cloud stored photographs, you can still do a lot of things if the WiFi router dies.  And you can replace it with a quick trip to Best Buy.

Eventually, though, your homelab starts to replace those cloud services and now *you* are on the hook for making sure that they are up and running 24/7.  

And not just that, but they need to be available when you're not home.

What if you're on vacation, half-way around the globe, and your Immich server goes down?  Or your firewall goes down and you cannot connect through it to that Immich server?  Or the NGINX reverse proxy decides it doesn't want to connect to it any more?  Or a network switch misbehaves?  

These things are annoying enough when you can get to them to fix them, but what if you're away?  

Last week we had a couple of short power outages overnight while we were asleep.  I know that these were power failures because the control unit that runs the underfloor heating in our bathroom had the wrong time and had lost all of its programming.  

But the firewall was off when we woke up.  I realized that I had probably set the BIOS in the firewall computer to not automatically boot on power resumption.  A silly mistake, really, but one that was a pain to fix because I would have to rip it out of the homelab, bring it upstairs to den where I have a monitor with a DisplayPort port, hook up a keyboard and boot it into BIOS.  All of this time, our network would be completely down.  

And, while even with the nucience of moving the server, this was still a trivial operation I know that sometimes (actually shockingly often), operations that begin with me announcing that, "The network will be down for 5 minutes" can end up with the network being down for hours because something "came up".

So I put it off for a bit.

And then the power went out overnight again.

This is no big deal when I am home.  I just go down to the basement and click the button.

But what if I am not home?

So I ordered a UPS, and when I installed it and had everything down anyways, I took the firewall upstairs and configured the BIOS.

Then I noticed that when I shut down all the Proxmox servers nicely before unplugging them, they didn't come back up when the UPS was turned on.  At this point I realized that I had the BIOS in all of them set to "Return to previous state" when the power was resumed. That's potentially an issue if I configure automated shutdowns for power outages that drain the UPS.

The whole point of this is that "Mission/Life Critical" means that "Failure is not an option".

You need to be prepared to cope with that if you are going to self-host.

## Personally, I Lean Hard on Prior Experience

In a way, I find this the most frustrating aspect of self-hosting.  I've been able to get over (or around) every obstacle I've encountered because I came into this with a pantload of technical knowledge.  Here's the highlights:

* Understanding of basic networking
* Extensive programming skills including *nix scripting
* Extensive sysadmin knowledge for *nix systems
  * File systems, logical volumes and tools like symbolic links
  * NFS
  * Processes
  * Utilities like `curl`, `ping`, `traceroute`, `ip`, `ln`, `ps`, `awk`, `sed`, `df`, `du` and `find`
* Understanding of DNS
* Understanding of firewalls and VPN's

I find it frustrating because I wonder if it's possible for people without these skills to even contemplate getting into self-hosting.

# The Upside

## Less Reliance on Outside Providers

In early 2024 my Chromecast devices stopped working.  I had two: one was an audio-only device attached to my Tivoli radio; the other was an early video Chromecast that was used to stream Netflix and Amazon Prime Video in my home entertainment system.  Both were "old", at least in modern terms.

It turns out that both devices hit the market in 2014 and the security certificates that were issued for their servers back at Google expired after 10 years, or February 2024.  

This reminded me that the continued operation of both of these devices was entirely dependent on Google's interest in continuing to support them. And there's no guarantee of that going forward.  They *did* fix the problem this time, but that's pretty cold comfort.  And it's not like Google has never killed off a cool product before.

The loss of the audio Chromecast would be particularly annoying, since the device has been discontinued and any kind of replacement will involve adding in a device to turn the HDMI output from a video Chromecast into something that will go into an old-fashioned audio jack.  Or I replace the Tivoli radio.

We've been using the audio Chromecast, along with a bunch of Nest/Google Home devices to provide whole-home audio streaming from SomaFM as all-day background music. It's been OK, at best.  However, it drops several times each day, and sometimes has weird outages.  

So I decided to implement SnapCast for whole-home audio.  It's cheap, and synchronizes all of the clients so that they really are all locked together.  The clients are $20 Raspberry Pi's with $20 DAC cards.  Set-up took a little bit of research, but no big deal.

I researched the web API for the streaming component, Mopidy, and set up some bash scripts to start and stop SomaFM.  Then I loaded these into `cron` jobs that run in the morning and night.  Now I don't even need to turn it on in the morning.

Occasionally, and much less frequently than Google would glitch, the SomaFM feed will have some kind of disruption. Maybe once or twice a week.  Just today, I finished creating a new `cron` job that uses the Mopidy API to check to see if the SomaFM stream is running, and if not, to restart it.  This `cron` job runs every 3 minutes throughout the daytime.

Here's the important point.  Snapcast works, and will continue to work forever. Even if the project is abandoned tomorrow, it won't have any impact on my whole-home audio, because I have no dependency on them.  SomaFM might close shop tomorrow, and then I'll have to come up with a new streaming source, or set it up to run off a local library of music.  

Now, I don't care what Google does with its Chromecast devices.  

I cannot express how much joy this brings me.

## It's Better

Frankly, I'm shocked at how much better self-hosted solutions can be than cloud services.  Look at my experience with Snapcast in the previous section.

I'm having the same experience with VLAN's.  

With my old, standard-consumer, setup of a WiFi router into my ISP's modem I only had the option to have a single "Guest" SSID that I really only used for actual guests.

Now, with VLAN's and a firewall, I have separate subnets for guests, for all my "smart" devices, and for my Google cameras. This means that I can restrict the trusted areas of my network to the devices that are truly trusted.  All of the robot vaccuums, smart switches and plugs, printers and the garage door opener can spy on each other in their sequestered subnet for all I care.  

## It's Fun

For me, this is a key factor.

None of this is worthwhile if it's not fun.  

Does it get frustrating at times?  Yes.  

Does it make my head hurt at times?  Absolutely.

I got it into my head that I wanted to have all of my VPN use controlled through my firewall.  If I had a server that needed to connect to the outside world through a VPN, I wanted to have it use a VPN established by the firewall, not configured on the server itself.  One of the reasons that this is better is because the firewall maintains the VPN connection automatically.  I have seen that if there is some connectivity problem upstream, a local instance of `openvpn` on a server can fail to reconnect.

The idea is that you set up the VPN with a gateway on the firewall, then create firewall rules that direct any WAN traffic from specific local addresses through the VPN gateway after NAT.  

I set this up and tested it a few months back, and recently I was ready to set up my first real VPN'd server.  It almost worked.  The VPN part was fine, but I couldn't access anything local.  This server needed to be able to do that.

I `pinged`.  I `tracerouted`.  I watched the live firewall logs.  The ICMP packets were getting through at the firewall.  It all looked correct, but I never got any `ping` returns.

I took me days to figure it out.  I walked through the OPNSense setup guides, videos, stackoverflow.com.  Everything.

Eventually I solved.  I needed a rule that prevented NAT when connecting to internal addresses.

This was NOT mentioned in any online resource that I could find.  I don't know why.  How could anybody get this to work without doing this?  Maybe I've got some other quirk in my firewall configuration, something that I did wrong, that makes this necessary for me but not anyone else?  I don't know.

But when I figured this out I was on cloud nine.  There was a super sense of accomplishment.

I tried to explain how awesome it was to my wife.  But when I talk to her about this stuff, it's like the parents talking on those Peanuts cartoons, "Whah, waa, wa, waah, whah...".  Oh, well.

But that's my definition of, "fun".

## You'll Learn

There's no doubt that you'll learn stuff.  Is it otherwise useful stuff?  That depends on what you do in the other parts of your life.  If you do some of this stuff at work, then maybe so.  

But I don't think that's important.  Learning is always valuable, even if just for itself.

## Your Workstations are Just Software Platforms

It used to be that I built this website on my desktop computer in my den.  All of the data was local, and I ran a local instance of `Jekyll` so that I could see the work in progress by surfing to `localhost`.  Then I would push it up to GitHub and a workflow would publish it.

Not any more.

Now I have a `Jekyll` server in my Promox cluster.  The data is hosted on an NFS server that is shared by my desktop computer and that `Jekyll` server.  I save the changes in my editor, and `Jekyll` rebuilds the website.  I can surf to that server to see how it looks.  I still push from my desktop computer to GitHub, but the data itself is still coming from the NFS share.

The editor that I use is `Pulsor`, which is a fork of the now dead `Atom` project.  It supports a Git directly, and that push to GitHub comes from there.  I've created a `Gitea` server in my homelab, and I plan to move the local Git instance over there eventually.  I'm also going to redirect all of my Java and Kotlin projects over to Gitea in the near future.

The point of all of this is that the only component of this process that exists on my desktop computer is `Proton` itself.  The software that I use for content creation.

No data is on my desktop computer.

# Conclusion
