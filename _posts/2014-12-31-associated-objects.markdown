---
layout: post
title: "Associated Objects"
description: Associated Objects
headline: Use with caution
modified: 2014-12-31
category: ios associated objects
tags: []
image: 
  feature: some-image.jpg
comments: true
mathjax:
---

One main difference between a class and a category in Objective-C is that a class can store data by creating properties and instance variables, whereas a category cannot.  This is only partially true, thanks to associated objects.

{% highlight Objective-C %}
#import <objc/runtime.h>
{% endhighlight %}

Objective-C’s runtime has several great features, including associated objects.  It contains several C functions that allow you to gain access to the lower level implementation of the language. These APIs should not be preferred if there is an ‘Objective-C way” to do it, especially because this can lead to a maintenance nightmare…so use it cautiously!

One feature of the runtime functions allow you to store an arbitrary value with an existing object.  Think of it like this: the runtime allows you to set values for keys to any object, like NSMutableDictionary does, but using C functions.  It also supports ‘get’ and ‘remove all,’ as shown in the below snippet:

{% gist vinnybad/ef68620c7ebf1e4f4034 %}

Rule of thumb: Don’t use this if you can get away with creating a @property or an instance variable.  There are many reasons for this, but one of the main ones is you lose the ability for the compiler to auto-complete property names.

Where can it help?

To illustrate why associated objects are worth learning let’s take a look at where they can be useful: UIImageView category in AFNetworking.  We all love the setImageWithURL: method that Mattt Thompson has created.  You can just pass in a reference to a NSURL object and it takes care of all of the asynchronous downloading of the image, and the setting of the image on success.  Great stuff.

But what happens when you have to load a table view with hundreds of images asynchronously?  Can you still use this method?

Let’s take a look at what needs to happen if you were to re-implement this and it was used in a table view:

Assume:

- You’re re-implementing the setImageWithURL: method
- each cell contains one image

Needs:

- images need to be downloaded asynchronously
- on success of the asynchronous call, the image needs to be set as the image into it’s appropriate UIImageView
- the image shouldn’t be set into a cell that has been recycled
- the user could flick the screen to the bottom of the table view, and it shouldn’t abuse bandwidth

Needs 1 and 2 are reasonable.  Where we see the problem getting tougher is when you encounter needs 3 and 4.  If you were to implement this method yourself, you could create a request to download the image, add a success block that sets the downloaded image, and see things work…kind of.  If you scroll quickly on the table view and a cell is recycled, you could see an older image load asynchronously on an incoming cell.  Since the cell still lives even after the user scrolls, the success block of the request sets the downloaded image to the UIImageView…which is bad if the user has scrolled and the cell has been recycled!

AFNetworking uses associated objects to get around this.  When setImageWithURL: is called, it associates an AFHTTPRequestOperation with the UIImageView.  When the cell is recycled, the implementor of tableView:cellForRowAtIndexPath: changes the url for the UIImageView via setImageWithURL:.  At this point, the initially associated AFHTTPRequestOperation is retrieved and canceled, and a new request is made to load the image asynchronously (we aren’t considering any caching optimizations in this post).  Because the request was canceled, the success block which sets the image finally to the UIImageView never runs…so the old image will never appear!

You may ask: since the UIImage can only be set on the main thread since it’s a UI update, isn’t it possible for AFNetworking to dispatch the success block on the main thread right as there is another call to setImageWithURL:?  In this case, only one of the two blocks will ever run at one time since they’re both running in the same queue.  Even so, the success block checks to see if the stored request matches the current request and only then proceeds in setting the image, thereby avoiding the condition altogether.

Since we want the ability to set an image with a URL on a UIImageView, and categories don’t support storing data, it’s necessary to drop to the runtime level.  In this next section, we’ll take a look at some concerns with over-using this building-block of Objective-C.

Concerns

Naming collisions: There’s a problem with over-using this feature though since any developer / library author can associate objects to any class…name collisions!  As class names can collide, keys can also collide, so a prefix is used with associated object keys as well (af_ is used by AFNetworking but it’s recommended a 3-letter prefix suffixed with an underscore should be used).  Worst of all, the compiler won’t warn you of any collisions.

Loss of auto-complete: you can gain this though by creating a method that just associates / retrieves the value, but it’s an additional step.

Hard-to-maintain code: This is perhaps the biggest reason to not use associated objects if it can be avoided.  Use it only if it’s absolutely needed.  If a novice developer uses this function to associate objects onto a completely separate object in potentially multiple places, things can get hairy really quickly.

