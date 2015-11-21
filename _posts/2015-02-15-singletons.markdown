---
layout: post
title:  "Singletons"
date:   2015-02-15 00:00:00 -0600
categories: ios associated objects 
---
One of the more influential patterns in software engineering is a Singleton.  This topic covers how singletons were created in pre-modern Objective-C, how they’re created now using GCD, and ends with how to never have to write that code ever again.  To read up more about the singleton pattern, I urge you to check Wikipedia (http://en.wikipedia.org/wiki/Singleton_pattern).

Before GCD was released, most people used to resort to not implementing singletons properly, like so:

{% gist vinnybad/655a34701f176daaf807 %}

The keen observer could realize that this code is not thread-safe.  It’s entirely possible that two threads could invoke the ‘shared’ method at the same time, thus potentially creating more than one instance of the object…which could lead to weird issues.

Is it over-kill to consider making it thread-safe?  Not really, especially because it’s not always known how or where exactly exactly the singleton will be used by the next developer.  A rule of thumb to remember: “You write code not just for yourself, but for others…so make sure it’s written well.”  This leads us to the @synchronized block implementation:

{% gist vinnybad/9d318f6d391db7c75b48 %}

@synchronized is a working solution and thread-safe, but when compared to Grand Central Dispatch (GCD), it’s less performant.  GCD defines a C function called dispatch_once, which will ensure that a block will execute only once for a given predicate.  Think of a predicate like a key:

{% gist vinnybad/97f1c152af0a8bf27964 %}

Remembering this code for each new project is not fun or necessary.  Only if there were a way to solve it across the board, for every case and every project you’ll ever work on:

{% gist vinnybad/c67dfc27f8a860fccd1a %}

This macro not only simplifies your code, it has already been debugged...plug and play!
