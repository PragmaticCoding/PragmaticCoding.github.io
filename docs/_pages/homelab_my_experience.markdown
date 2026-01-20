---
layout: single
title: Homelab: Why Listen to Me?
toc: true
toc_label: "Contents"
toc_icon: "brain"
toc_sticky: true
header:
  overlay_image: /assets/images/brain.jpg
  overlay_filter: 0.4

sidebar:
    nav: "master"
permalink: /homelab/my_experience
skip_link: true
---
# What Do I Know About This?

I've always thought of myself as a programmer first, but my career has always been as "A programmer would does other IT stuff".

# It Started at the Very Beginning of My Career

My first real job, in 1986, was as a PC programmer for a large dairy company.  The core computing that ran the operations was written in COBOL and ran on Digital VAX computers.  There were about 40 COBOL programmers and 2 of us PC programmers.  PC's were pretty new-fangled in 1986, and we were doing a lot of work in dBase, as well as writing entire systems in Lotus 1-2-3.  

However, we also became pretty much involved in any of the PC related issues that came up, even if they didn't involve programming.  One of the things was RS232 serial communications.  You have to remember that, back in 1986, everything was RS232 serial.  People accessed the VAX through dumb terminals connected via RS232.  Modems used RS232, as did many printers.  

After the 10th time or so that I had asked one of the techies to make me an RS232 of some sort, they got fed up.  After all, they were mostly concerned with VAX stuff, and PC issues were a distraction.  So they gave me the cabling kit and a quick lesson.

And that was it.  I was making cables, then I was debugging cables, and then, before I knew it I was an bit of an RS232 expert.  Woo-hoo!

Now I was the programmer who did RS232!

That knowledge went with me to my next job, and I used it there.  Remember, back then everything was RS232.  I had moved from PC programming to working with some odd-ball minicomputer systems at this point, and they used dumb terminals connected via RS232.  I even spent a week once wiring an office expansion with Cat-5 cable that we rigged up for RS232.

At some point the platforms that we were working with transitioned from the custom mini-computers to RISC based Unix systems.  None of the programmers at the software company that I worked for at the time had much interest in Unix.  So I spent time learning as much as I could about Unix.  This was mostly HP-UX and IBM's AIX.  

Now I was the programmer that did RS232 and Unix!

# Networking

The last job I had lasted 24 years and started in 1996.  When I started the company only had about 50 people and I was the only programmer with an IT manager who just did techie stuff. When I walked into the computer room (more like a closet, actually) it had exactly 3 servers: A Novell Netware server, an RS6000 and a thing called a "KarlBridge" that connected our Ethernet network to the SNA network of our parent company's AS400.  

Those servers and some other equipment were connected to each other by 10Base-2 coax, which was truly a horrible invention.  The connectors were always flakey, and would spontaneously disconnect if you looked at them the wrong way.  We developed the "ping and wiggle" technique.  You started a ping command on the RS6000 to some other device and turned the monitor around until you could see it from behind the computers.  Then you went to each device, grabbed the coax on either side of the connector and wiggled it a bit.  If the console on the RS6000 started scrolling ping results, then you had found the faulty connection!

The rest of the PC's and printers in the office shared a couple of 10Base-T hubs.  Not switches, they weren't available yet, but hubs.

After about 1 year, the original IT manager was gone and his replacement, like me, was primarily a programmer.  But there was nobody else, so we automatically became responsible for the entire IT infrastructure.

Over time, this morphed into having to design the entire data centre infrastructure as the company grew and technology evolved.  Virtually everything new that the company decided to do had some IT component that we had to research and implement.  We did use consultants to give guidance and advice when we were moving into major changes, but ultimately we had to make the decisions ourselves.

Now I was the programmer that did RS232, Unix and data centre planning!

# IT Infrastructure Since 1996

Back in 1996, we had MS Windows, but it was Windows 3.1, which was not quite the same thing.  Windows 95 was a big change from that.

Back in 1996, virtually all corporate email was internal.  You couldn't send an email to someone in another company, hardly anyone had personal email accounts.  We implemented an email server called "Mercury" on our Novell Netware server somewhere around 1997/1998.  When we eventually replaced the Netware server with Windows Active Directory and file servers, we also implemented Lotus Notes and used that as an email server.  We hosted our own email server with Lotus Notes for nearly 20 years.  

At some point we needed to be able to email - both internally and externally - from our Unix server.  Back then, we needed to implement and configure "sendmail", which we relayed through our Lotus Notes email server.  It worked quite well.

Back in 1996, almost nobody had a website.  We didn't, but by 1998 we did.  And it wasn't just a static site, it connected to our main application and customers could log in and do business transactions.  This was really cutting edge at the time.  

# Who are the Experts?

There are lots of people out there that have way more experience with corporate IT infrastructure than I do.  And lots of them have experience with way more complicated IT infrastructure than I do.  

But here's the thing...

Corporate IT infrastructure is NOT a homelab.

From what I've seen, over the past 35 years, is that there are people that I would call "Techies": the people who are experts at configuring Cisco switches, at planning backup schedules, configuring Active Directory, designing VLANs, managing VMWare, configuring workstations, doing penetration tests, and a whole host of really technical operations.

While I admire these people and the thechnical skills that they have, I have noticed that they are almost 100% focused on those technical tasks and very, very rarely have an interest in the business context in which these technologies are used.

This can be a bit of an issue when we talk about homelabs, because you, the one building it, have to be the total package.  You need to learn the technical skills, but you need to understand the context that things you build are going to be used in.  
