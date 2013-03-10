---
layout: post
title: Highlight extension for Jquery
---

When developing the [Tag Cloud Extension](http://mlarocca.github.com/01-14-2013/tagcloud.html) for Chrome I came across the idea to add a nice feature: custom highlighting for the tags, so that they could be easily spotted in their original context.
It sounded a very useful featue to have, so I decided to actually implement it. As I soon realized, the challenge was quite more complex than one could think, hence, before trying to reinvent the wheel, I decided to look for a library or even better a jQuery plugin that would implement text highlighting in a HTML page.

I found several proposal, and I spent a few days evaluating them and trying them out, but all of them were largely unsatisfactory and buggy implementations.
When I was about to give up, I finally stumbled upon the highlight.js jQuery plugin by [Johann Burkard](http://johannburkard.de/blog/programming/javascript/highlight-javascript-text-higlighting-jquery-plugin.html):
Finally an excellent, working solution, the best I could ever find and even desire.
This one the only one perfectly working solution I could find, an impressive job, really. The funny thing is, I later realized it was too part of a Tag Cloud lib, [DynaCloud](http://johannburkard.de/blog/programming/javascript/dynacloud-a-dynamic-javascript-tag-keyword-cloud-with-jquery.html).

Nonetheless, even this nice plugin lacked a few features I needed to use it in my extension project, but it represented a very good basis to start with, so I decided to extend it, and here it is what I came up with.

```javascript
/**
  * @author Marcello La Rocca marcellolarocca@gmail.com
  * Highlights arbitrary terms assigning up to 2 custom classes to it.
  * It is possible to use regular expressions as pattern and to choose to highlight only whole words matching it.
  * The highlightClassName parameter can be used to easily remove all the highlighting in a DOM elements with one single call,
  * while the specificClassName parameter allow for highlighting each pattern with a different css style (but it is optional).
  * For each highlighted piece of text, a span is created in the original HTML document and (up to) 2 classes 
  * (highlightClassName and specificClassName)are assigned to this new tag.
  * 
  * @param {String} pattern              The string [regular expression] to highlight.
  * @param {Boolean} wholeWordOnly       True iff only whole words matching pattern should be highlighted.
  * @param {String} highlightClassName   Name of the general class assigned to highlighted words: can be used for
  *                                      styling the highlighted text or just as a mean to remove highlighting 
  *                                      altogether with a single call.
  * @param {String} [specificClassName]  Name of the specific class that must be used to style the matching text.
  *
  * Based on 
  * <http://johannburkard.de/blog/programming/javascript/highlight-javascript-text-higlighting-jquery-plugin.html>
  * by Johann Burkard
  *
  */
jQuery.fn.highlight = function(pattern, wholeWordOnly, highlightClassName, specificClassName) {
    "use strict";

    var upperCasePattern = pattern.toUpperCase();
    var regex = wholeWordOnly ? new RegExp("(^" + pattern + "[\\W]+)|([\\W]+" + pattern + "[\\W]+)|([\\W]+" + pattern + "$)|(^"+ pattern + "$)", "gi") 
                              : new RegExp(pattern, "gi");
    
    function innerHighlight(node) {
        var nodesToSkip = 0;
        var pos;
        
        if (node.nodeType === Node.TEXT_NODE) {
            if (regex.test(node.data)) {
                
                //If the reg exp matches the content of the node, we need to find the index of pattern inside it
                pos = node.data.toUpperCase().indexOf(upperCasePattern);
                
                var spannode = document.createElement('span');
                spannode.className = highlightClassName + (specificClassName ? " " + specificClassName : "");
                var middlebit = node.splitText(pos);
                var endbit = middlebit.splitText(pattern.length);
                var middleclone = middlebit.cloneNode(true);
                spannode.appendChild(middleclone);
                middlebit.parentNode.replaceChild(spannode, middlebit);
                nodesToSkip = 1;
            }
        }
        else if (node.nodeType === Node.ELEMENT_NODE && node.childNodes && !/(script|style)/i.test(node.tagName)) {
            for (var i = 0; i < node.childNodes.length; ++i) {
                i += innerHighlight(node.childNodes[i]);
            }
        }
        return nodesToSkip;
    }
    
    return this.length && pattern && pattern.length ?   this.each(function() {
                                                            innerHighlight(this);
                                                        })
                                                    : this;
};

/**
  * @method removeHighlight
  *
  * @param {String} highlightClassName   Name of the class associated to highlighted words for which highlighting
  *                                      should be removed.
  */
jQuery.fn.removeHighlight = function(highlightClassName) {
    "use strict";

    return this.find("span." + highlightClassName).each(function() {                                                            
                                                            this.parentNode.replaceChild(this.firstChild, this);
                                                            try{
                                                                this.parentNode.normalize();
                                                            }catch(e){
                                                                //Nothing to do
                                                            }
                                                            
                                                        }).end();
};


jQuery.fn.getText = function() {
    "use strict";
    
    function innerGetText(node) {
        var texts = [];
        
        if (node.nodeType === Node.TEXT_NODE) {
                //If the reg exp matches the content of the node, we need to find the index of pattern inside it
                texts.push(node.data);
        }
        else if (node.nodeType === Node.ELEMENT_NODE && node.childNodes && !/(script|style)/i.test(node.tagName)) {
            
            for (var i = 0; i < node.childNodes.length; ++i) {
                texts.push(innerGetText(node.childNodes[i]));
            }
        }
        return texts.join(" ");
    }
    
    if (this.length > 0) {
        var results = [];
        var res;
        this.each(function() {
            results.push(innerGetText(this));
        });
        return results.join(" ");
    } else {
        return innerGetText(this);
    }
}; 
```

Besides a little refactoring of the code, the main differences are:

1. The text to highlight can be expressed as a pattern, or better its string representation (as in "\\w*" or "\\d+", for example);

2. It can be chosen to highlight only whole words exactly matching the pattern;

3. The CSS class assigned to the highlighted text is specified in each and every call, and thus different styles can be applied to different highlighted keyword or patterns in each page;

4. _removeHighlight_ takes the name of the class associated with the highlighting to remove as a parameter, so that, if more than one category (i.e. style) has been used, removal can be limited to single categories.

5. Highlighted text can be associated with up to two different classes of highlighting: our reccomendation is to use a general class not associated with css rules (parameter _highlightClassName_ ) to identify highlighting-related span tags in the document, and a second specific class (parameter _specificClassName_ ) to actually style the individual patterns highlighted (as said above, each pattern can be styled differently this way). This solution, together with the flexibilty of our version of removeHighlight, allow to remove all highlighting with one single call ( _removeHighlight(highlightClassName)_ ) or single highlight categories individually.


Of course the credit goes mostly to Johann for the great work he did on its version.