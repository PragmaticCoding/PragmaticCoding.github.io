---
title:  "Handling Production Support on an Agile Development Team"
date:   2023-01-05 12:00:00 -0500
categories: javafx
logo: /assets/logos/JavaFXLogo.png
permalink: /agile/distractions
ScreenSnap: /assets/posts/StarterFX.png
GitHubDialog: /assets/posts/StarterFX-GitHub.png
Diagram: /assets/posts/MVCI.png
excerpt: Some thoughts on how to balance development work with production support.
---

# Introduction

Back in my first real programming job, in 1986, I had to keep time charts for my work.  Like everyone else, I hate time charts, but in this case I at least gained one interesting insight from the exercise.  I noticed that in some weeks, out of 40 or so working hours, I only logged 2 or 3 hours working on my primary project.  On top of that, those few hours were composed of small blocks of 15-30 minutes scattered throughout the week.  In reality, 15 minutes isn't even long enough to get your head back into the work and pick up from where you left off.

Where did the rest of the time go?

Support.  I was one of just a couple of programmers working on PC based systems (remember this was early on in the days of the PC), and we were also the defacto support technicians for any and all things PC related.  The rest of the department was full of COBOL programmers working on VAX systems.  So we got all of the PC support requests, and this could be anything from monitors that wouldn't turn on to helping with a spreadsheet.  And it added up.

I've learned over the years that this is pretty much the norm for just about any programmer working in development.  It may be that you have associated skills that get called on for non-project work, or it may just be that, most of the time, the best person to fix something is someone on the team that wrote it in the first place.  

It's the same today as it was in 1986.  It will be the same thing ten years from now.  Very few programmers have the luxury of putting aside all distractions and just getting on with work.

# Customer Expectations From Support

At another job I had, in the early 1990's, I was part of a small team doing application support for a niche software company.  They produced software for use in heavy industry, and the customers were generally factories and distribution warehouses.  Quite a few of the customers were auto parts manufacturers supplying the assembly plants for the big auto makers.  The company took support very seriously, and we had the authority to interrupt any meeting anywhere, any time, if we needed help from the developers to fix a problem - although we rarely needed to do anything like this.  

This was pretty intense, high-pressure work, and the company had decided that the support technicians needed to be able to work without distractions.  All good so far.  They had a support system set up to track the tickets, and when any customers called to report a problem the receptionist would take a few details down and enter a new ticket into the system.  The unbreakable rule was that no customer ever got transferred directly to us in Support when they called in.

We had a mandate to get back to the customers as soon as possible.  We all had dual monitors on our desks, with one screen permanently showing the ticket queue.  And we kept an eye on it, and we'd figure out amongst ourselves who could do the call-back to the customer.

We had metrics from the ticketing system that showed that something like 90% of the new tickets got call-backs within 5 minutes of them leaving a message with reception.  5 minutes!

And the customers hated it!!!

It drove them absolutely bonkers.  They'd beg the receptionist to patch them through to one of us.  They'd yell at the receptionist.  They really, really got frustrated.  Even though it was almost a certainty that we'd call them back in a couple of minutes.  And honestly, unless there are flames rushing through the factory and vats or something are blowing up, there's nothing so urgent that it can't wait 5 minutes at the most.

When we called the customers back, very often the problem they were having wasn't dire.  Sometimes it was super urgent and we'd drop everything to work on their issue.  But most of the time, we'd tell them that it would be a few hours before we could get on to their problem, but we'd look at it as soon as we could.  And just about all of the time, they were cool with that.

But, what the heck?  Freak out because you can't get patched through to us instantly, but then it's OK if it takes a few hours before we can even start to look at your problem?  It doesn't seem to make logical sense.

Except that it does.  At some level, what those customers really wanted was to hand off responsibility for some bad situation over to someone else who could deal with it.  Getting it solved wasn't the priority.  Explaining the problem to someone qualified to understand the nature of the problem, who understood the implications of the problem, and who could make a qualified determination of the urgency of the problem *was* the priority.  Sometimes they just needed to know that they hadn't caused a problem that would cost them their job.  

There's a really important lesson in there:  Fixing a problem is often nowhere near as important as managing the emotions and expectations of the people experiencing the problem.  

# Support Triage

When production support issues come into an Agile development team, the developers need to be able to determine if the issue needs to "jump the queue" and get fixed right away, or whether it can be added to the Product Backlog.

There are three questions that need to be asked...

Is the Fix Trivial?
: Can this issue be solved in a very small amount of time without involving lots of programmers and testers?  A very small amount of time can be interpreted depending on the team, but generally something taking 15 minutes would undoubtedly be trivial, and something that takes an hour might be trivial for some teams but not others.

Is the Fix Urgent?
:  Does this problem need to be solved now?  For instance, if you find a crucial problem with a year end procedure, but year end is 9 months away, then it's not urgent.  Also, urgency needs to be evaluated in the context of the length of your Sprints and the time to the end of the current Sprint.  An otherwise urgent item might be able to wait to be scheduled into the next Sprint if your very near the end of the current Sprint.

Is Fixing this Important?
: Does this problem really need to be fixed?  Is there a work-around, or are the impacts of the issue minimal?  A spelling mistake on an internally used screen may be annoying, but it may not have any real-world impact.  

That's all you need to ask.  Here's how you decide what to do:

If it's Trivial - Fix it Now
: Just get it out of the way.  The overhead of tracking the issue on the Product Backlog, discussing it in a meeting, flushing out requirements and prioritizing is probably going to be a bigger drag an development progress than just fixing it now without much ceremony.

If it's BOTH Important and Urgent - Fix it Now
: Really, you have no choice here.  You've determined that it's not trivial, but the impact of the problem is significant and needs to be fixed now.  Keep in mind that work on this issue is likely to have an impact on the deliverables for the current Sprint, so the Team may need to consult with the Product Owner to decide if something drastic needs to be done to the current Sprint.

If it's NOT Trivial, and is EITHER Not Important OR Not Urgent - Put it in the Product Backlog
: Once you've determined that it's not trivial, if it's either not important, or not urgent, then it should go on the Product Backlog so that it can be scheduled into a future Sprint.  The Product Owner can evaluate it in comparison with the other items on the Product Backlog and prioritize it accordingly.

# Bugs

This is a controversial idea:  In terms of Sprint scheduling, there's nothing special about a bug fix.  

When we started using Scrum, back in 2003, one of the biggest and unexpected benefits was that we stopped arguing with the customers over whether or not something was a bug.  We just didn't care.  If the customer wanted to call it a "bug", then fine, we'll call it that.  We didn't waste time arguing that it was actually a change in requirements, an enhancement, or a problem with the original specifications.

You see, for us it was just work.  It didn't matter where it came from, but if we were fixing a bug then we weren't available to build new functionality.  

In the old, pre-Agile world there was this idea that, somehow, the customers got bug fixes for free.  And I don't mean in terms of money, but in terms of scheduling.  Somehow the programmers were on the hook to fix a bug without impacting the big schedule for the project.

Also a bug that was discovered during the Sprint in which it was created isn't a bug, it's an unfinished story.  
