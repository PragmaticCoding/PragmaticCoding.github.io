---
layout: single
title: Homelab on a Shoestring Budget
toc: true
toc_label: "Contents"
toc_icon: "brain"
toc_sticky: true
header:
  overlay_image: /assets/images/brain.jpg
  overlay_filter: 0.4

sidebar:
    nav: "master"
permalink: /homelab/shoestring
skip_link: true
---
# Building a Homelab on a Shoestring Budget

Eventually the cost of my ISP/television package became too much for me to stomach.  The two were coming from the same vendor, and they haven't yet built out fibre to the house in our neighbourhood.  This meant that I was paying too much for 25Mbps internet and a ton for TV packages that I wasn't really using much.  So I switched to cable internet from another ISP, and signed up for IPTV.  The cost savings from the first month alone paid for IPTV box.

But there was a bit of a gap.  A small handful of programming that we did use wasn't available via IPTV, and recording from the IPTV was much more problematic than with the ISP TV.  So I decided to set up a PC with Jellyfin.

I had an old, cast-off, all-in-on PC sitting around, so I decided to re-purpose it.  It had a 4th generation Core i3 processor, so it wasn't particularly powerful, but it was free.  I installed Jellyfin and PiHole on it.  It worked.

Then, for a lark, I tried to install NextCloud on it.

That was too much, and it choked.  

It was clear then, that I had to come up with a solution that would work.

The thing is that I'm pathologically cheap.  So I wanted to come up with a way to build something good, but in a cash-flow constrained manner.  What I wanted was something that I could build over time, and keep the costs down to the level where it didn't really affect my monthly budget much.

This page is going to be the blog of that project.  I'll keep up a dialogue here as it progresses, and I'll provide links to articles with more details, or detailed instructions as appropriate.  

## Why Listen to Me?

For the past 40 years, I've been a programmer who does other IT stuff.  

[Read More](/homelab/my_experience){: .btn .btn--info}

## What's My Philosophy and Approach?

Over the years of being involved in designing IT infrastructure, I've developed a pragmatic approach to it.    

[Read More](/homelab/approach){: .btn .btn--info}

# Homelab on a Shoestring

OK?  Then let's get started...

My starting point was to build something that would be immediately useful, but constrained to be accessible only when connected to my home network.  This way I wouldn't have to concern myself with security issues too much at first, and I could concentrate dealing with technical issues involved in getting the stuff to work.  Hopefully I wouldn't be too far along before I was ready to think about external access, and I could do a comprehensive security audit and assessment at that point.  -

## Picking the Platform

The very first issue was to figure out how to actually deliver these services.  It took very little time to determine that using virtual machines in a hypervisor was going to be the way to go.  Furthermore, it appears that just about everyone was using something called Promox to do this.  It's free, and appears to work really well.

[Read About Proxmox and Virtualization](/homelab/proxmox){: .btn .btn--info}

## Picking the Hardware

The next question was to decide what hardware to run it on.  From what I see on various social media sites, there are three approaches:

1. Use one big, powerful computer.
1. Use decommissioned enterprise hardware.
1. Use a cluster of small computers.

I think that there are significant issues with using a single, powerful computer.  The first is that it will be expensive and require a significant cash outlay in one lump.  It is possible to start with a bare-bones system and build it out over time, adding memory storage and other upgrades, but you are still stuck with a significant up-front cost.

The second issue is about reliability.  Or, more specifically, about availability.  My goal is to eventually replace a whole array of cloud services which are, for the most part, fairly reliable.  It's not going to be acceptable if it goes down.  For me, this means that redundancy is important.

[Read About High Availability](/homelab/lonovo_tiny){: .btn .btn--info}

A lot of your redundancy issues could be dealt with by implementing a couple of cast-off enterprise servers.  These often have redundant everything, from power supplies to network cards to fans to storage systems.  Not to mention hot-swap drives and components, ILO management and sophisticated HA technologies.

It doesn't take much surfing around to find lots of comments about the downside to repurposing decommissioned enterprise-grade servers.  For one, they are very often shockingly underpowered compared to even modern desktop equipment.  Shocking because these things were once used to run entire businesses.  

The next big issue is that they aren't really designed for home use.  For one, they generally use a ton of power.  A ton of power.  Along with that power consumption comes a lot of heat and noise.  There's a reason that most data centres have a huge air conditioning bill.  If you've ever been in a large data centre, you would understand just how loud this equipment can be.  Even just a few units can be significant if it's sitting in your apartment living room.  

I decided to go with option 3: A bunch of small, inexpensive computers.  So small and cheap, in fact, that if I needed more power I could just go buy another and add it to my cluster when I needed it.  

I was happy to see that Dell, HP and Lenovo all make teeny little "1 Litre" computers that run quietly while using very little electricity and have more than enough computing power to do what I need.  I settled an the Lenovo M910Q.

[Read About the Lenovo M910Q Tiny](/homelab/lonovo_tiny){: .btn .btn--info}

I bought two of these units for about $130 CAD each and proceeded to put them into a Proxmox cluster with my initial all-in-one PC.  Then I was ready to get started.

## Implementing DNS and Ad-Blocking
