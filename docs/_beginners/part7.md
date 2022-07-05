---
title: Part 7 - What Next?
short_title: Part 7
permalink: /beginners/part7
screenshot_1: /assets/beginners/part6a_1.png
excerpt: Now we've completed the core programming for our "Create" function, where do go next.  We'll look at the idea of a "Product Backlog", and talk about how we should prioritize our next steps.
---

# The Application So Far

Now we have an application that allows the user to enter in an account number and name, and then to save it in our database.  It looks like this when it's running:

![ScreenShot]({{page.screenshot_1}})

# Where Do We Go From Here?

What should we do next?  

Typically, in a Agile development scenario, you would get feedback from the Product Owner who is responsible for setting the priorities on the "Product Backlog".  The Product Backlog is just a big list of possible updates to the application, which might be new features, bug fixes or changes to something that's already been done.  The Product Owner always works from the position that what's been delivered so far *is* the final product, unless they ask for something to be done.

In our case, we have no Product Owner, but we can speculate about how a reasonable Product Owner would decide about the priorities for development.

# Our Product Backlog

The Product Backlog is just a list of things that we might do to our application.  It's a dynamic thing, and every time we update our application and show it to the users, they'll come up with new ideas that go into the Product Backlog.  We'll keep our list to just things that seem likely at this stage.

## Divide the Name Field

Just having everything in one field is a bit loose.  What's the format?  "Fred Smith"?  What about "Mr. Fred Smith"? Maybe "Smith, Fred"?  It's probably a good idea to break it down into three fields: a salutation, the first name and the last name.

## Database Latency

Just about every database that you can use has at least the possibility of latency and blocking.  Our simulated database has none of that, and our application doesn't have any way to handle it.  This is a big problem, and we need to deal it before we can go "live".

## Empty Records

Let's run the application and hit "Save" without entering anything in the TextFields:

``` console
> Task :run
Saving account:  Name:  Result: 1
```

That's not good.  We shouldn't be able to save empty records.

## Duplicate Records

If you run the application, enter an account number and name and then hit save several times, you'll get console output that looks like this:

``` console
> Task :run
Saving account: 1234 Name: Fred Smith Result: 1
Saving account: 1234 Name: Fred Smith Result: 2
Saving account: 1234 Name: Fred Smith Result: 3
Saving account: 1234 Name: Fred Smith Result: 4
```

Oops!  It looks like we saved the same record in our database 4 times!

Even worse, if we change the name each time, we get this:

``` console
> Task :run
Saving account: 1234 Name: Fred Smith Result: 1
Saving account: 1234 Name: George White Result: 2
Saving account: 1234 Name: Robert Bland Result: 3
Saving account: 1234 Name: Sara Brown Result: 4
```

Yikes!  Now we have 4 different records for account "1234", each for a different person.  

We should fix this.

## More Fields

There's very little practical use for an application that only stores the customer name.  We'll need to add fields for address, phone number and email.

## Look and Feel

This application is still pretty ugly, and it will only get worse as we expand it.  Maybe we should do some work to clean up the look and feel before we go any further?

## Retrieve

So far, we've only done the "Create" part of "CRUD".  We really cannot do "Delete" or "Update" until we've implemented "Retrieve", so that should be the next major feature that we add.

# The Decision

At this point, our application works but it relies on the users behaving themselves and not creating duplicates or empty records.  These things can corrupt our database, so they need to be addressed before we expand the feature set.

Even more importantly though, is that our application wouldn't work very well with a real database that introduces latency and potentially blocks while waiting for a response.  From that respect, our application is "non-functioning" and needs to fixed before we do anything else.
