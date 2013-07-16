---
layout: post
title: PyCrawler updated to v.1.3
---

PyCrawler is a simple but functional crawler written in Python, [here] you can read the original post about it with all its features listed.

This update deals with duplicate pages: it is not uncommon that 2 different URIs lead to the same page, especially if for any reason the URLs used maintain any state info or non influential parameters (for example, without login, all the links may redirect to the home page of a web site, but that is not noticeable by looking at the URL only).

So, the solution usually enforced is to check the page content and somehow keep track of it; of course, comparing each page's content to every other page already crawled would take forever, so we need a more efficient solution: computing a digest for pages' content, and keeping the signature for each page in a hash table for constant time lookup.

Of course, two concerns still remains:

1. Hash tables and digest means collisions: we need to choose a digest long enough to avoid collisions as much as we can, and decide a policy to resolve them when they happens.

2. Computing the digest isn't free and isn't constant-time: the longest the input, the more the computation will take; the more complicated the algorithm, the slower it is.

To try to balance performance and collisions, I decided to use *sha256*, and to ignore collisions.

*sha512* would have (probabilistically) guaranteed no collisions at all, but it would also have been much more computational intensive, slowing down the crawler; *sha256*, on the other hand, didn't noticeably slow down the crawler at all, and the chances to have a collision over 1 million documents would be smaller than 0.001% anyway, so, considering that it's very unlikely to crawl a single domain with several hundred of thousands of pages, I guess it's safe for our purposes to ignore collisions.

The alternative would have been for each collision, to retrieve again the collided page, and compare the two contents in detail; that would have required either keeping in memory a copy of each page crawled (crazy, given the unlikelyhood of collisions), or issue a new http request each time a collision happens (perhaps keeping in a cache the content of colliding pages for a while).
However, since collisions are so unlikely, unless you need the guarantee of a 100% accuracy for some reason, it is safe to skip this step.