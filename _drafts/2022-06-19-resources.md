---
title:  "Where Are My Resources?"
date:   2022-06-19 12:00:00 -0500
categories: javafx
logo: /assets/logos/JavaFXLogo.png
permalink: /javafx/resources
ScreenSnap: /assets/posts/StarterFX.png
GitHubDialog: /assets/posts/StarterFX-GitHub.png
Diagram: /assets/posts/MVCI.png
excerpt: One of the toughest problems beginners encounter is resources that won't load.  Here's how to figure out how to organize your resources and how find them with your code.
---

# Introduction


# Project Organization

First off I'm going to assume you're using some kind of build engine like Gradle, Maven or (gasp!) Ant.  If you're not, you should be as these handle all your dependencies and organize your project in a nice predictable way.  To keep things simple, I'm going to talk specifically about projects built with Gradle, but Maven is also very similar.  

## On the Source Side

Gradle will expect that your project is divided into Source Sets.  A typical project will have a "main" Source Set, and a "test" Source Set.  Your production code and resources would reside in the "main" Source Set, while unit tests and related resources would go in the "test" Source Set.  

Within a Source Set, the project is divided into code sections, and a "resource" section. It's a best practice to have one code section for each programming language in your project.  Most JavaFX projects will have a "java" code section, but they could also have a "kotlin" section or a "groovy" section.  Inside of the each language section you will have folders that correspond to the individual packages in your project.

There will only be on "resource" section in a Source Set.  It's really just a folder, and the files contained within it can be in whatever format that the programmer wants to use.  

## On the "Build" Side

Part of the Gradle build process is to copy all of the classes after they have been compiled into a director structure starting with "build".  There will be a subdirectory called "classes" and all of the compiled classes will be found in there.  There will also be a subdirectory called "resources".  Under that there will be a subdirectory each for "main" and whatever other Source Sets are built.  Under these subdirectories will be folders that follow the structures that were used in the "resource" folders of each Source Set on the source side.

It is possible to configure a Gradle build to do all kinds of wild and wonderful things when copying the files from source to build.  Files can be selectively copied, renamed, moved to different locations.  Files can be duplicated into different directories, duplicated and renamed - almost anything.  We're not going to worry about those things here, because if you're doing that in your Gradle build, you're probably not going to be reading this article.

## What This Means

The biggest thing to keep in mind about all of this is that your code is going to run against the file and folder structure in the "build" subdirectory.  This means that the real source of truth for how you should be looking for your resources is contained in the "build" subdirectory.

# Resource Organization

While you can put your resources into the "resources" section in any way that you want, you should keep in mind that the resource methods in Java are designed to facility organizing your resources by package.  This might be useful to you, but it's also probably the thing that's causing you the most trouble with finding your resources.

## The Key Consideration - Uniqueness

When deciding how you are going to organize your resources, you have one key question to ask:

What is the scope in which a resource is going to be used?  

This is because you can only have one resource with the same name in particular context, which roughly corresponds to a file in a directory.  For a simple project with an uncomplicated layout, this question is largely moot because you don't have opportunities for name collisions.  However, when you start to use things like sub-projects, or intend to overwrite resources from libraries, this becomes more important.  Also, if you intend to use a standard resource name, but implemented in different contexts, this can become more complicated.

## Possible Schemes

Everything in /resources
: This is the simplest possible organizational scheme.  Just dump all of your resource files into the root directory of /resources.

Divided Up By Type 
: The next step up is to organize your resources by type.  So create a subdirectory off /resources for each resource type.  This would mean you might have directories like /resources/images, /resources/css or /resources/sounds.  This makes it a little easier to keep track of what resources you have, but doesn't really have any programming impacts over using a single directory.

Divided Up By Package
: This

Divided Up By Package and Type
: This is combination of the previous two methods.  Within each package, have a subdirectory for

Which method is best?

There really is no answer to this because the main driver is context.  The best guide might be to simply to use the simplest possible scheme that works well for the project.  For instance, if you're building a library that is intended to be used by other projects, it would probably be best to consider "root" to be the base, unique, package for that library.  That way you would avoid collisions with resources in the utilizing projects unless they deliberately chose to override your resource files.

In the same way, if you're building a project that uses other libraries, you might want to keep all of your project's resources off the root directory, in order to avoid collisions with resources from sub-projects that are organized by package.
