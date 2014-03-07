---
layout: post
title: You'd better stay away from the "this" trap when defining a JavaScript module
---

# JavaScript this pointer is actually evil... especially if you are going to export modules.

## Let's start with an example

{% highlight javascript %}

function declareModuleTest () {

  return {
    funA: function funA () {
      console.log("A",this);
    },
    funB: function funB () {
      console.log("B", this);
      return this.funA();
    }
  }
}

var module = declareModuleTest();

function testThis (callback) {
  callback();
}

testThis(module.funB);

{% endhighlight %}

What is the expected outcome, in your opinion? What do you think it will be the value of **_this_** inside _funB_, when called from **_testThis_**?
Chances are you haven't bet on "page will crash", but actually this is the case: an exception will be thrown at this line

{% highlight javascript %}
    return this.funA();
{% endhighlight %}

because the value of **_this_** will either be **_null_** or **_window_**, depending on if you are using [**strict mode**](http://ejohn.org/blog/ecmascript-5-strict-mode-json-and-more/).

If we had called **_funB_** as **_module.funB()_**, there would have been no problem, and the execution would proceed as expected: let's see why!


##A step back: The this pointer

The Internet is crammed with literature on the infamous "this" pointer. It actually is one of those [bad parts](http://yuiblog.com/blog/2007/06/08/video-crockford-goodstuff/) highlighted by [Douglas Crockford](http://javascript.crockford.com/); the main reason of its evilness is that function context depends on the way you call it.

There are, in fact, 4 different ways in which you can invoke a function, producing different values for the this pointer:


* Using **_new_** keyword: in this case, the function is invoked as a constructor, and **_this_** is assigned the reference of the newly created Object (that, by the way, will be returned at the end of the method).

* As an object method: If an object instance **_obj_** has a method **_m_**, and you call it as **_obj.m()_**, then the this pointer inside **_m_** will be the reference to **_obj_**.

* As a standalone function: Here the behaviour depends on if you are using strict mode (i.e. _EcmaScript 5_) or not; before _ES5_, the this pointer passed to the function invoked would have been a reference to the global object (i.e. **_window_**): this was one of the most common bug sources in JavaScript. In _ES5_ this nonsense has been partially corrected: now this will point to null inside these functions - This is still far from the ideal behaviour, however: it would have been better passing a reference to the object inside which the function is called (which would have also solved our problem above in the first place).

* Using apply/call: by using this methods of the **_Function_** prototype, we can set the reference stored in this inside the function body by passing it as a first parameter to the method: funB.apply(module) or funB.call(module) would do the trick. (The difference between apply and call is that the former takes the arguments for the function as an array as its second argument, while for the latter the actual parameters must be listed after the first parameter).

## So... why isn't it working?

Let's try another test first:

{% highlight javascript %}

function declareModuleTest () {

  return {
    funA: function funA () {
      console.log("A",this);
    },
    funB: function funB () {
      console.log("B", this);
      return this.funA();
    }
  }
}

var module = declareModuleTest();

var testThis = module.funB;
testThis();

{% endhighlight %}

Do you think that anything changes now? Of course not!

Turns out that, if you assign a method to a variable (either by explicitily assigning it to a var or by passing it as a parameter), the Function object you are storing won't retain the information about its context, so when invoked it will be invoked as... a function (since it is one!).

Therefore the this pointer will be inconsistent with what the way we'd expect the function to be used.
Since we have no control on how a user of our libraries will use it, using references to **this** in public methods (methods that, directly or indirectly, can be called from outside our module) is simply **UNSAFE**.

## A simple solution

The solution, however, is really at hand:

{% highlight javascript %}
function declareModuleTest () {
  var moduleTest = {
    funA: function funA () {
      console.log("A", this, moduleTest);
    },
    funB: function funB () {
      console.log("B", this, moduleTest);
      return moduleTest.funA();
    }
  };
  return moduleTest;
}

var module = declareModuleTest();

var testThis = module.funB;
testThis();
{% endhighlight %}

simply declare the module as a private module attribute, and replace every occurrence of **_this_** with a reference to that attribute.

To further improve the design of our module, I'd suggest a little improvement made possible by strict mode:

{% highlight javascript %}
function declareModuleTest () {
  "use strict";
  var moduleTest = {
    funA: function funA () {
      console.log("A", this, moduleTest);
    },
    funB: function funB () {
      console.log("B", this, moduleTest);
      return moduleTest.funA();
    }
  };
  if (Object.seal instanceof Function) {
    Object.seal(moduleTest);
  }
  return moduleTest;
}

var module = declareModuleTest();

var testThis = module.funB;
testThis();
{% endhighlight %}

This way, you make sure your module can't be tampered with by users (which you might care, especially if external users act as middlemen at some point).

If you target only newer browsers versions, and you either check at the very beginning for _ES5_ compatibility, or you are fine with causing crashes in earlier versions (definitely do not take it as an advice! :D), you can remove the test on **_Object.seal_**.
