---
layout: post
title: Reengineering TubeHound, a video search aggregator
---

##AKA Considerations about choosing the right technology stack for your application.

![thound](../images/thound.jpg)

###What

A video search aggregator, is a tool that makes your life easier when you need to query online videos. There are many ways to aggregate your searches, and to help you find what you are looking for.
The idea behind [__TubeHound__](http://thound.herokuapp.com/) is that sometimes, when you are looking for videos about a certain topic, you might as well be interested in videos which do not directly match your search, but are nonetheless connected to the ones that do. For example, when you are looking for a [trailer](http://thound.herokuapp.com/search/monuments%20men/1) of a certain movie, you might be interested to watch trailers of similar movies. Or, if you are looking for a tutorial by a specific content producer, say a [cooking tutorial](http://thound.herokuapp.com/search/Gordon%20Ramsay's%20Ultimate%20Cookery%20Course%20S01E01/1) from your favourite chef, you are probably interested in acknowledging the existence of other chefs who release tutorials, and maybe, once you notice them, you might be interested in searching more results about them.

[__TubeHound__](http://thound.herokuapp.com/), for every query you submit, shows you many results related to the videos matching you search terms, and allows you to refocus your search with a couple of mouse clicks: the goal is making navigation among your results as easy as following your stream of thoughts.

###Why

When I first thought about creating [__TubeHound__](http://thound.herokuapp.com/), in its earliest version, I was just unhappy with the way video hosting's search engines work.
Then, I read about [__Meteor__](https://www.meteor.com/), and this looked like the perfect project to start learning about it.
So, basically it was born both as an experiment and a training project.

It was indeed an enriching experience, but along the road I also realized how __Meteor__ was an overkill for this kind of project (and how, at the same time, it made some other operations much harder).

After completing a [beta version](http://thound.meteor.com) in late 2012, I had to abandon its development for a few months, during which __Meteor__ changed enough to make maintenance on the project quite complicated. The problem was that __Meteor__ was in a very early stage, and so its interface and even its core were changing very quickly; to be completely honest it had happened even during those few months I actively developed with it that I had been forced to change my implementation to update the framework to its latest release. So, on the whole, I'd say that was also a very instructive experience: never _ever_ use an unstable product for a production project, or even for a hobby project you'd like to maintain and develop during the years.

Recently, I started using [__RactiveJs__](http://www.ractivejs.org/), and it immediately looked obvious to me that it was a perfect choice to reengineer [__TubeHound__](http://thound.herokuapp.com/), because it was lightweight, and it provided reactive templates that could be managed with little or no effort.
So I decided to start over, from scratch (almost).

###How

Once I decided to reengineer my app, the first step was to find out which features added the most value to it, and which ones were worth implementing. So I tried to apply the Pareto Principle, i.e. "find the 20% of your features that add 80% value to your business (or project)".
For [__TubeHound__](http://thound.herokuapp.com/), it meant trimming off functionalities like user management, registration, and playlists. The plan is to add them later, especially the playlist as a sort of memory for your searches, but they are not the main goal of the website, not its added value.

Then, I had to choose the technology stack:

For the back-end:

* Python on Google App Engine for everything concerning YouTube APIs, querying and caching (exploiting GAE support for memcached).
* NodeJs for routing, serving the static content (and possibly handle users registration/login etc...)

This division is targeted to ease the load on the server taking care of the video queries, avoiding to burden it with further request for static content; moreover, a user management system would require permanent storage, and it would have no sense to include it in the same database as the one used for query related issues: the only meaningful solution would be to split the two functions into two separate lanes, working on two different databases, each one tuned for the specific needs emerging from its purpose. If you are interested in getting more on this topics, I suggest taking a look at [this thorough book](http://www.amazon.com/Scalability-Rules-Principles-Scaling-Sites-ebook/dp/B00503D1TY).

For the front end, I chose [__jQuery__](http://jquery.com) and [__Bootstrap__](http://getbootstrap.com/) to take care of DOM manipulation and layout, [__RequireJS__](http://requirejs.org/) to structure the AMD modules and organize dependencies, and [__RactiveJs__](http://www.ractivejs.org/) to add reactivity to the page.


###When

Or I should rather say "how long": well, it took me just 2 weeks to create the new version of [__TubeHound__](http://thound.herokuapp.com/). Although this is a simplified version of the original one, and I was able to re-use some functions (very few, actually) from the previous version, I was stunned by how simple the development was, and how fast I could create a functioning version - even if I had to create a completely new design!
The development of the original version had taken a lot more time. Excluding the Python back-end, which remained the same, it required at least 2 months to learn __Meteor__ and create a working example.

###Differences

It's not that __Meteor__ isn't a great tool. It is, indeed. Just - as every tool - you need to find the right job to employ it. Or, complementary, for every job you need to carefully choose the right tool - if you are familiar with _Maslow's hammer law_, you might have already figured out this is a textbook case.

According to Maslow's hammer law, in fact, "if you have a hammer, everything looks like a nail"! 
Of course, the result of using a hammer to open your can of soup can be a messy, possibly painful, disaster; and indeed the difference between using __Meteor__ instead of something like __Ractive__ (or __Angular__) is pretty much the same.

__Meteor__ is really great when you need a full-stack solution to develop an application that supports multi user real-time interaction, like a shared document manager, a shared painting tool, an online multi-player game, or such things, because it makes very easy to update shared data and seamlessly and instantly refresh all of the remote clients, leveraging web sockets.

If you just need reactivity on a low-interactive front-end, then using __Meteor__ it's probably an overkill.
__Ractive__, on the other end, makes it very easy to connect a model (an object) to a view, and update the view every time the model beneath it changes. While __Meteor__ (as version 0.5, might have changed in newer versions) refreshed the whole template for every single change to one of its elements, __RactiveJs__ handles this kind of situations and redraws only the nodes that are actually changed. To be fair, somewhere around version 0.5 the _#isolate_ tag was added to __Meteor__ templates to manually achieve the same result - however it isn't as clean nor as effective as what you get in __Ractive__ for free.

I personally also find the way __Ractive__ lets you use expressions inside templates (and define helper methods) much easier and immediate, but perhaps this is more of a personal preference. And again, it's worth saying this is not a post against __Meteor__, rather a careful consideration about _"when to use what"_.

Another great tool you could use instead of __Ractive__, in fact, is [__AngularJs__](http://angularjs.org/). You can read a bit more about differences and similarities between the two of them online, perhaps starting from [here](http://blog.ractivejs.org/posts/whats-the-difference-between-angular-and-ractive) and - why not? - [here](http://mlarocca.github.io/01-22-2014/pathsjs_ractive.html).

###Conclusions

When you start a new project, you should definitely take some time pinning down your requirements and figuring out the right technology stack for your needs - every minute you spend on this, will likely save you dozens of hours of frustration later.

In the meantime, please try [__TubeHound__](http://thound.herokuapp.com) and let us know if you like it, what would you like to change or what you think add or may add value to it.
