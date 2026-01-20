---
title:  "Service Continuity and Recovery"
date:   2025-10-20 12:00:00 -0500
categories: homelab
logo: /assets/logos/JavaFXLogo.png
permalink: /homelab/continuity
M910Q: /assets/homelab/M910Q-Front-Back.png
Stack: /assets/homelab/M910Q-stack.jpg
M910_Inside_Top: /assets/homelab/M910Q-top.jpg
M910_Inside_Bottom: /assets/homelab/M910Q-bottom.jpg
M910Q_Specs: /assets/homelab/ThinkCentre_M910_Tiny_Spec.PDF
M710Q_Specs: /assets/homelab/ThinkCentre_M710_Tiny_Spec.PDF
Stack1k: /assets/homelab/ThinkCentre_M710_Tiny_Spec.PD
Stack1: /assets/homelab/M910Q-Front-Back.png



Diagram: /assets/elements/ListProperties.png
OLArticle: /javafx/elements/observable-classes-lists
ABGuide: /beginners/intro
GitHubLink: https://github.com/thomasnield/ConstrainedProperty
InstallationLink: https://pve.proxmox.com/pve-docs/chapter-pve-installation.html

JavaDocOL: https://openjfx.io/javadoc/23/javafx.base/javafx/collections/ObservableList.html
JavaDocFXC: https://openjfx.io/javadoc/23/javafx.base/javafx/collections/FXCollections.html

excerpt: "A look at the Lenovo M910Q Tiny computers I've chosen as homelab servers."
---

# Introduction

In the corporate world, we used to call this "Business Continuity and Recovery", but for homelabs, we can replace "Business" with "Service" and the concept maps over very well.

I remember in the late 1990's or early 2000's there was a major fire in a business district in downtown Montreal.  A large number of businesses, if not all of them, suffered complete loss of their facilities.  IIRC, one of them was the corporate office of a large, cross-country bread company.  Back in those days, most companies were still hosting their own data centres on-site, so many of these companies lost their corporate IT infrastructure in the fire.

Some months later, I saw an article where they had done a survey of the impact of that fire on those businesses.  What they found was that if a company didn't have its IT services back up and running in about 4-5 days, there was a 90% chance they would never come back.  In other words, if they couldn't recover quickly the business would go under.

So, for businesses, service continuity and recovery is an existential issue.  It's not going to be for a homelab.  Even if you lose your house and all your data and services, your family is going to continue on somehow.  But it's still a very big concern.  One you should plan for from the very beginning, and it should certainly impact the choices that you make about how you implement your homelab.

# Continuity and Recovery Are NOT the Same Thing

Let's take a look at what these terms mean...

"Service Continuity" refers to how your services can continue to perform in the face of technical challenges.  This can mean equipment failure, external service (like power or internet providers), human error, or any number of other things.

Generally speaking, any technology that refers to "redundancy" or "high availability" is aimed at providing service continuity.  For intstance "RAID", which stands for "**Redundant** Array of Inexpensive/Independant Disks" is a technology aimed at preserving service continuity in the failure of a single disk.

You'll often see people say "RAID is not a backup strategy".  That's because RAID is a continuity technology, and backups are a "Service Recovery" technology.

"Service Recovery" refers to how your services can be restored after a loss of service.  In a virtualized environment, virtually all of your services are essentially just data, which means that "Service Recovery" is synonymous with "Data Recovery".  Yes, if your house is destroyed in a fire, you'll probably need to get new hardware to run the services, but, to be honest, that's the easy part.

# A Closer Look at Service Recovery

In many ways, recovery is easier to deal with than continuity, so we'll look at it first, since it really concerns backups.

## Recovery Point and Recovery Time

Well, I say "concerns backups", but nobody really cares about that.  What they care about is "restore from backup".

In the business world, we were always concerned with two key factors about restoring data:

Recovery Point

: This is how close restoring from a backup will bring you to the point where the data loss occurred.  Generally speaking, you want your recovery point to be as close to current as possible.

Recovery Time

: This is how much time it takes you to restore from a backup after the failure occurs.  Clearly you want this to be as short as possible.

## Causes of Service or Data Loss

It's important to understand what kind of events you are attempting to recover from, because these will often impact how you plan and what you expectations for recovery will be.  Let's start with a simple list of some easy-to-identify causes:

* Equipment failure
* Software failure
* Facility destruction
* Human error

The first item is what everyone probably thinks about as the most likely and, therefore, most important danger.  What if your disk drive fails and you lose your data?  If you are using RAID, then this won't necessarily mean you need to restore from backups, as you can replace a failed drive and propagate the redundant data back out to the new unit.  But what if the RAID controller fails, or if two drives fail?  It can happen, especially if you don't have monitoring on your RAID array and miss when the first drive fails.  

If your house burns to the ground, you're probably going to have bigger problems on your mind than your homelab.  However, if the information that you need for insurance purposes was stored in your homelab, you should probably have a plan for restoring that service outside of your home.

Finally, let's take a look at human error.  In 35 years of working in IT, I saw a relatively small number of data loss events due to hardware or software failure, but I saw many, many requests to restore data that was lost due to human error.

The thing that makes human error data loss different from other forms of data loss is that it's very often not noticed until much later in time.  It's one thing if you see smoke coming out of a drive bay, it's a completely different thing when somebody accidentally saves a file on top of a completely different file and doesn't notice at the time.  You might not become aware of the data loss until months later.  

This last item shows the complexity of "recovery point".  This is because the recovery point concerns the "point where the data loss occured", not "the point where you became aware of the data loss".  

Think about it.  If having a recovery point that is just a shade behind, "right now", is the goal, then a hot backup is the way to go.  Changes in the live data are copied to the backup within milliseconds, and if the live hardware fails you can flip to the hot backup with virtually no data loss.  But what if someone accidentally deletes a file?  Within milliseconds it's also deleted from the hot backup.  The same thing goes for mirrored drives or RAID5.  The moment you delete the primary data, the redundant data is also deleted.
