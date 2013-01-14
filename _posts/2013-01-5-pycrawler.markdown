---
layout: post
title: PyCrawler
---

PyCrawler is a simple but functional crawler written in Python.

It's a very humble crawler indeed, and I'm not making any claim this is going to solve your crawling problems: it's just a spare time project, built in a couple of hours for fun and as a challenge. It only crawls static content, and ignores url fragments.

Nonetheless, I tried to give it a general and robust structure and I think it is a good basis to build a decent crawler.
At the time of writing, it has the following properties:

*    Breadth-first crawling

*    Parallel crawling (the number of crawlers running in different threadsin parallel is totally customizable at runtime)

*    Keeps track of the static resources referenced, and builds a map of the website crawled (starting from any page retrieved)

*    Crawls only pages within one single domain (determined by the starting page)

*    Can be set at runtime so that the number of pages crawled will be limited

*    Supports an elementary mechanism of polite crawling

[Here](http://mlarocca.github.com/PyCrawler/) you can find complete documentation, while you can also take a look at its [source code](http://github.com/mlarocca/PyCrawler/)

## Update: 2013/01/13


I introduced the concept of depth in the crawling process, allowing to specify a predefined maximum depth for the web pages to crawl, in alternative or together with a limit for the number of pages retrieved. Consequently, the interface of start_crawling method has slightly changed (See documentation).

Basically, the depth of a page is its distance from the starting page, i.e. the number of intermediate pages (plus one) in the chain of references that goes from the starting page to the current page. This setting allows a different way to limit the crawling, especially for huge sites, while allowing to expand the limit uniformly (radially) around the starting page.
