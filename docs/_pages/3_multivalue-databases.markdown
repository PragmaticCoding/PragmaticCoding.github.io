---
layout: single
title:  "MV"
toc: true
toc_label: "Contents"
toc_icon: "brain"
toc_sticky: true
header:
  overlay_image: /assets/images/brain.jpg
  overlay_filter: 0.4

sidebar:
    nav: "master"
date:   2021-12-11 10:33:44 -0500
permalink: /pages/multivalue
categories: MultiValue
---
# MultiValue Databases

"MultiValue" is the modern name given to a class of database managers modelled after the [Pick Operating System](https://en.wikipedia.org/wiki/Pick_operating_system), designed way, way back in 1965.  Generally speaking, a MultiValue system isn't just a database, but has an integrated programming language (usually a variant of BASIC), a batch processing language, a command processor, a text editor, and a query language.  From the 1970's through to today, there's a substantial number of companies with MultiValue products on the market, and in its heyday, there were lots of software companies building vertical-market systems that used these databases.

The database itself is usually designed as a Link-Hash Filing System with variable-length delimited records.  In other words, data tables are very similar to a Java "Map" on disk, with the records composed of Strings with embedded delimiters to separate fields, values and subvalues.  With file systems, we are usually more concerned about how the data is stored on disk in order to optimize both speed and disk space, so the hashing function and division of the records into groups is usually more important than it would be with a Map stored in memory.

It's this idea of having three levels of delimiters which gives MultiValue its name.  A record can have any number of fields, and each field can have any number of values in it.  In turn, each value can be broken up into a number of subvalues.  

I recently started learning how to use MongoDB, and I was struck by how similar the approach of NoSQL is to MultiValue.  Of course, the records are stored in JSON-like structures instead of delimited records, but implications to your application design are pretty much the same.  Today, we don't really think twice about multi-valued data, but prior to about 2010 MultiValue was a pretty radical idea in computing.

# MultiValue Variants

The original MultiValue database was the "Pick Operating System", invented by Richard (Dick) Pick.  It was designed to track the inventory of parts for military helicopters in the mid 1960's.  Other companies started producing versions that were, in many ways, identical to Pick.  Many of them in 1970's and 1980's were designed to run as operating systems on proprietary hardware, but others were designed to run as databases on various operating systems.  This probably reached a peak in about 1992, and a partial list would include Pick, Prime INFORMATION, ADDS Mentor, The Ultimate O/S, Revelation, Reality, jBase, Universe and UniData.

At around this time, RISC based UNIX systems hit the market.  Up to this point, MultiValue systems had the advantage that they delivered outstanding performance on relatively modest hardware.  About $100K of hardware would purchase enough hardware to run a multi-user system that could easily support 10-20 full time users on dumb terminals.  That would be more than enough to run a medium sized business or even a large factory, and the alternative would have involved spending 10 times more on a VAX or an IBM mainframe.  

But the RISC based UNIX servers changed everything.  They delivered 10-20 times the performance of biggest of the MultiValue hardware platforms at less than 1/2 the price.  Overnight, the MultiValue operating system was essentially wiped out.  Most of the big players in this area disappeared in a few years, including Prime, Reality and Ultimate.  

The products that had been developed as databases running on other operating systems fared much better.  Universe had always run on UNIX and the company that developed it, VMark, swallowed up UniData making them the really big player in the market.  VMark changed it's name, was bought by Informix which was in turn bought by IBM.  So, for a while, IBM was actually one of the major players in the MultiValue marketplace.  They later sold UniVerse and UniDate (known as "U2") to Rocket Software.

# The File System

What really made Pick unique and worthwhile, back in the heyday, was the filing system.  It was a key/value system, although that term hadn't been coined until much more recently.  It worked like this:

When you created the file, you specified two things:  the hashing algorithm and the "size" (also known as the modulo).  The file was allocated a number of blocks (way back when, usually 512bytes each) equal to the modulo.  To store a record, the key was run through a numerical process based on the hashing algorithm that turned it into a number, then it was divided by the number of blocks and the remainder was taken.  This would give you a number between zero and the file size.  Your record would be stored in that block.  

To read you record, the same process was followed.  You supplied the key, and the system would figure out which block the data was in almost instantly, and then only a single disc read was required.  Then the system only had to look through that block of data in memory to fetch your record.  

The hashing algorithm was designed to spread the records evenly and somewhat randomly between all of the blocks in the file.  There were usually a dozen or so different algorithms to choose from depending on the nature of your keys.  

But what if you filled up one of these 512 byte blocks?  The system would add a new block to the end of the file, and then link it as a continuation of the block that filled up.  For this reason, Pick was always described as a "Link-Hash Filing System".  In Pick terms, the blocks were called "frames", and the linked lists of frame with the same hashing result were called "groups".  So if you created a file with a modulo of 101 (prime numbers always worked better to evenly distribute the records) then you would initially have 101 groups in your file, each of which had exactly 1 frame.  As you started adding records, you would eventually add more overflow frames, but you would always have 101 groups.  

This is what made Pick incredibly efficient compared to other databases.  If you had a record key and a well tuned database, you could read that record with just one or two disk reads.  And it didn't matter if you had 100 records, or 100 million records, the response time would stay almost flat.  

This meant that people were implementing databases in Pick systems on small machines that were unbelievably huge.  Literally, "unbelievably".  I worked on one PC based system back in the 80's that we had trouble backing up.  The problem was that our core data file was bigger than the capacity of a single tape, and we had to find a backup system that would allow a single file to span multiple tapes!  Other programmers that were working with dBase and Clipper had trouble believing that we were working with that much data - and we never had performance issues!

# Pick BASIC  

It's easier just to show you what Pick BASIC looks like:

```
OPEN "","SOME.FILE" AS TEST.FILE ELSE
  PRINT "OH OH!  SOMETHING WENT WRONG"
  STOP
END
MORE = TRUE
EXECUTE 'SELECT SOME.FILE WITH COUNTRY = "WAKANDA" AND WITH AGE > "23" AND WITH ANY ORDER.VALUE < "18.4"'
LOOP
  READNEXT TEST.KEY ELSE MORE = FALSE
WHILE (MORE = TRUE) DO
  READ TEST.REC FROM TEST.FILE, TEST.KEY THEN
    FIRST.NAME = TEST.REC<4,1>
    LAST.NAME = TEST.REC<4,2>
    NAME = LAST.NAME : ", " : FIRST.NAME
    TEST.REC<17> = NAME
    WRITE TEST.REC ON TEST.FILE, TEST.KEY
  END ELSE
    PRINT "UNABLE TO READ ":TEST.KEY:" FROM SOME.FILE"
  END
REPEAT
CLOSE TEST.FILE
```
Most of this is pretty easy to read and figure out.  The `EXECUTE` statement runs a TCL (Terminal Control Language) command from a program.  In this case the `SELECT` command performs a query which generates a list of record keys which are held in memory as an "active SELECT list".  The `READNEXT` statement pulls the next key from the active SELECT list.  If it comes to the end of the list, it executes the `ELSE` clause.  `READ` also has the same THEN/ELSE structure.  

The returned value from `READ` is called a "Dynamic Array" and it's just a database record.  The "angle bracket operators" are used for field/value/subvalue extraction and replacement from dynamic arrays.  What's field 4?  Who knows - check the dictionary.

I'm sure you get the idea.  Now imagine hundreds of programs with 3,000 - 5,000 lines of this stuff each.  

# Data Dictionaries

One of the coolest features of Pick is the "Data Dictionaries".    

A Data Dictionary is really just a data file, but one that has a particular format (well, actually about 5 different formats) and is usually attached to a data file by the database.  There's actually a Data Dictionary for Data Dictionaries so that you can list Data Dictionaries.  

The keys in a Data Dictionary are the field names, and the records contain information about the field number, data conversions for display and selection, justification, and manipulation of data in the field.  You can also create Data Dictionary items for grouping fields together and to grab data from other files for display and selection.  

Aside from the layouts of the Data Dictionaries, there's really no hard and fast rules about them.  You can have multiple Data Dictionary items for any field.  Usually, you would do this to display a field in different ways, or to do different kinds of manipulations on the same field for display and selection.  You can have fields with no Data Dictionary records at all, although you can't use those fields for display or selection.  

You can also completely different data types stored in the same field, with completely different Data Dictionary records for them.  This happens a lot with files of control data, where the programmers didn't want to create a separate file for each type of control record.

From a programmers perspective back in the old days, this was awesome.  You could modify the database structure at will, without needing any database administration tools, and you didn't even need to have a Data Dictionary item related to the data you were storing in your field.  In truth, most programming shops had rules and conventions that they'd all agree to follow, although it wasn't uncommon that you'd have to spend a fair bit of time rummaging about in program code to figure out what a particular field was for, because the Data Dictionary didn't have definitive answer.
