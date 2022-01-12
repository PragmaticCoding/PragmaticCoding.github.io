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
