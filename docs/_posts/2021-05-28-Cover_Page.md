---
title:  "What About the Cover Page?"
date:   2021-05-28 12:00:00 -0500
categories: agile
logo: /assets/logos/Agile.png
excerpt: There's a lot of ways that software development projects can fail, or partially fail.  Some of these happen when the users don't understand as much as you think they do about what you're building, how it will look and how it will work.  
---
This is what happened to us:

"What about the cover page?"

"Pardon?"

"Yes, we need to be able to print a cover page to send along with the cheque when we mail it out."

This was the first we had heard about this - at the final demo of the product.  How did this happen?

## The Situation

The company I worked for as Lead Developer routinely contracted out a lot of legal work to external law firms.  A lot.  Which meant that we also received, processed and paid a lot of legal bills.  

In case you don't know, lawyers bill by the hour, and they log their entire day in 6 minute (1/10 of an hour) increments.  They have software that tracks it all, and when you get your bill it can have pages and pages of entries like, "6 minutes - Phone call with Jones about settlement" or, "24 minutes - Review documents".  Things like that.  The legal profession calls them "dockets".

We had spent years building the software to take our company from receiving our legal bills on paper and scanning them, to having them sent by email, to having them uploaded through a portal.  But somebody still had to look at those dockets and figure out how they translated into our coding system to track the expenses.  We needed to classify the money spent into things like "Investigation", "Trials", or "Pleadings and Motions".  

The final step in our evolution from paper invoices was to have the legal firms do the classification for us and enter those amounts in our web portal, then upload a PDF of the invoice and docket as supporting documentation.  It would save us a lot of time and take out the guess-work that our in-house staff had been doing.

Of course, building a web application for outside users is complicated and required that we build different entry screens for different kinds of legal bills - each with its own back end processing.  We did the most common types of bills first, then the easiest uncommon things.  We left the less common, complicated types of bills to the end.

Finally we came to "Third Party Invoices".  

Sometimes our lawyers would need to hire expert witnesses, or investigators and other kinds of professionals to do some work.  The law firms really didn't like our existing system where they paid the subcontractors directly and then we reimbursed them for the amount.  It meant that they were out the money while they waited for us to pay them.  They would rather that we just pay those subcontractors directly.

This was very different from all of the other invoices.  We needed to provide a way for the administrators at the law firms to enter the name and the address of the payee, and they could only apply the amount to a single billing code in our system - from a smaller list of special codes.  We had re-engineer the interface into our legacy billing system to allow us to override the name and address too, which was no small job.  

Finally, having the ability to generate a cheque for an arbitrary name and address is a completely different situation from generating a cheque for someone already set up as a supplier in the system.  It presents an entirely new challenge from the perspective of setting up financial and fraud controls.  So we needed to make sure we thought of all that as we built it.

## The Process

We were trying to be as Agile as possible.  We organized our work into Sprints, set goals for the Sprints and touched base with the users as much as possible.  Unfortunately, we were never able to get this  particular business unit to agree to partial implementations as we built anything.  They were happy to come and see demos and to discuss requirements and ideas, but they didn't want to deploy anything until it was all done.

We spent almost two months building this.  

During this time we had numerous meetings with all of the internal stakeholders.  We would have meetings with everyone involved in the project in the room at one time.  This included our internal administrators and their management; the people managing the work the outside law firms were doing and who approved the legal bills; their managers; and even the VP of the department.  We also routinely had all of the Development Team in these meetings; programmers, BA's and testers.

We always felt it was important to have the whole Team understand as much of the business aspect of what they were building as possible.  So we encouraged the Team to participate in these meetings, make suggestions and ask questions.

All through the development I felt like we had a good handle on what we needed to build.  I also felt that the users did too.  I can vividly remember sitting in meetings while the users tossed ideas around and argued amongst themselves about technical or process details.  They would discuss the implications of some approach, and offer alternatives or improvements to the ideas.

It seemed liked the users had a solid grasp of how the software was going to work.  They had examined the implications and ramifications of deep elements the design and had developed a 360 degree view of what we were building.

I was fooled.

## What About the Cover Page?

Well, it turns out you can't just stuff a cheque in an envelop and put it in the mail.  You need to add a cover page that tells that person why you are sending them money.

It seems obvious now.

We had spent all that time building the software, and we had arrived at our final demo and deployment planning meeting.  We were ready to talk about issues around training, coordinating with the Finance department (who print the cheques), and getting the message out to the law firms about the change.  We were done!

We did the demo to the same people who had been in all of the meetings all along.  The same people who had argued about technical details and helped us to design the whole thing.

Virtually the first question that anyone had after we completed that final demo was, "What about cover page?".  Immediately, all the other users in the room were nodding their heads.  Where's the step where we create the cover page?

From a developer perspective this was no small thing.  We had engineered a business process and now we needed to inject a whole new step.  This was a brand new document, and we would need to build link to our automated document creation system, not to mention design the document and the automation for it.

This wasn't a disaster, but it was a set-back.  There would be no discussions about training and deployment that day.  We had more work to do.

## What Went Wrong?

When I described this to my boss at the time, he asked, "Why didn't the BA shadow the users?  See how the manual process works".  That's a fair question.

This was essentially a brand new process.  We didn't cut third party cheques for expenses - at least not routinely - before this.  So there was no one to shadow.

But that's not really the point.  There are potentially a variety of techniques that might have revealed the need for cover letters, but we didn't employ them.  We didn't use them because we had a fully engaged customer who seemed (collectively) to understand what we were building.

I can tell you, for a fact, that everyone on the Development Team had a shared vision of what they were building, how it would work and what it would look like when it was completed.

And that's the problem.

As developers we had that clear, shared vision for the product.  We assumed that the customer did too.

### The First Incontrovertible Rule of Software Engineering

This was not unique.  I've seen examples before where the customer's reaction upon seeing the finished product was, "What's this?  This is not what you described in the meeting".  Yet every developer that was in that meeting would agree that what they delivered is exactly what was discussed.

This leads to the most import rule about building systems:

Customers don't engage with and understand your design until you give them working software.
{: .notice--danger}

There's something about vapourware that puts up an impenetrable barrier that stops most non-developers from picturing how it will work.  But once you give them actual working software, especially when you say, "This is it.  This is what you get.", that barrier dissolves and they can start really thinking about what that software does and how it does it.

This is really, really hard for us developers to anticipate.  It's what we do all day, every day.  We envision what a good solution would look like, what potential problems there might be and the implications of one approach over another.  We assume that the other people on the Team are doing the same things, constantly - because they are.

It's easy to forget that this is a skill that you developed.  Often over years.  As a developer, it may be the most important skill you have.  Other people don't have this skill and never will.

## How Do You Deal With This?

Our biggest mistake was letting the customer talk us into a "Big Bang" release.  We could have found an approach that would have allowed us to deliver a partial implementation that was actually useful itself.  For instance, we could have built some internal screens to allow users to enter and code up these invoices that came to us as PDF.  This would have allowed us to get useful working software into the users' hands a lot faster, before we spent time building pages on the extranet for outside counsel to use.

The Waterfall approach would say that we had a failure in our requirements gathering, and that by improving our methodologies there we could ensure success in the future.  But experience has shown that this is a fallacy.  One of the reasons for this is that your customers can't understand your design before you build it.  So they can't anticipate the things you'll need to know to build it correctly.  And no matter what you do, the customer is always going to be the subject matter expert on business process and you need their insight to figure out the requirements.

A core idea of Agile is that everyone's ignorance is greatest at the start of any project, and the best approach is one that allows everyone to learn as the project progresses.  For customers, that means they need to get their hands on working software as soon as possible.  Because without working software, they won't understand what you're building.
