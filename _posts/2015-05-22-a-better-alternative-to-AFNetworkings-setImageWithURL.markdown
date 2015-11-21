---
layout: post
title:  "A Better Alternative to AFNetworking's setImageWithURL:"
date:   2015-05-22 00:00:00 -0600
categories: ios AFNetworking images networking
---
Any iOS developer that has been around for a while is familiar with AFNetworking and it’s amazing capabilities.  There is one area where AFNetworking falls short though: UIImageView+AFNetworking.h’s setImageWithURL:.  It does not allow for persisting images to a disk cache with zero configuration…after all, it’s mainly a networking library and not a image library.

Enter Haneke: “a lightweight zero-config image cache for iOS, in Objective-C” (as stated on their github page).

Caching

Haneke is built for downloading and caching images.  By default, there is a 2-level cache (L1: in-memory, L2: least recently used (LRU) disk cache).  This allows for an “offline mode” version of your app with minimal effort.  The caching parameters are also configurable to allow for larger caches, etc.

Invisible Optimizations

The most wonderful parts of Haneke are invisible: the optimizations under it all.  Hanke is optimized to show images in table views and collection views.

When using UIImageViews in such situations, the images that are downloaded are most likely all of the same size.  Even if they are not, they are most likely rendered in the same size because of the way cell reuse works.  Leveraging this ‘constraint,’ Haneke resizes images that were downloaded and creates a separate cache for images of that size for faster access.

This allows for larger images of a particular size to be handled separately.  In practice, these images of the same size are used for the same purpose.  Using an LRU cache eviction policy (it uses NSCache internally), this works well in practice because images that are least used of a particular size (aka particular purpose) are evicted.

Conclusion

By far, Haneke is a great alternative to AFNetworking’s UIImageView category.  It performs a lot of under-the-hood optimizations with no extra code and requires the same amount of code to use as AFNetworking’s above-mentioned category.

For the 80% of people, Haneke will be enough.  When hundreds or thousands of images need to be handled and 60 FPS is required, there are alternatives such as FastImageCache that can be used…but that requires much more setup and a deeper understanding of how everything works.
