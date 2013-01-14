---
layout: post
title: Combining effective encapsulation and inheritance in JavaScript
---

While developing a plugin for the D3 graphic library, I found myself facing one of the most common conundrum for a JavaScript programmer: how on Earth is it possible to get a good compromise between encapsulation and extensibility?

Don't get me wrong, there are many different known techniques to safely and effectively keep private attributes and methods really private (mostly involving closure and local vars), as well as there are many more to mimic inheritance, from pseudo-classical to prototypal or parasitic ones.
A very good source of inspiration for all of these techniques can be found on _Douglas Crockford_ web sites or, if you have a little bit more time, in some of his great videos on _YUITheather_ (a must for any JS developer).

But as far as I know there have been only a few attempts to cope with both problems at the same time: there is no "protected" encapsulation in JavaScript, and none of them looks particularly easy or satisfying to me.
And that's a bummer, I mean, web development today is all about mash-ups, and every JS engineer  at some point will have to exploit mash-ups, so she will be (well, at least he really SHOULD BE) necessarily concerned about the methods that his objects are going to expose to consumers.

The most common solution starts, or at least it should, from embracing an Object Capability System.
In such a paradigm, Objects can communicate with other objects only if they have a reference to them (as usual), but an Object A can obtain a reference to an Object C only in one of the following 3 ways:

1. By creation (A creates C)
2. By construction (during construction of A it is endowed with a reference to C)
3. By introduction (An object B to whom A has a reference introduces C to A)


More on this topic [here](http://www.youtube.com/watch?v=zKuFu19LgZA) _(around 42:00)_

Moving on: from objects to methods. Even if we (as Object C) have been introduced to A, we don't want A to be able to mess with our internal state: hence, we start using private attributes and methods.
If we were to use prototypal inheritance, we could create C Objects using a factory-like method like the following one:

{% highlight javascript linenos %}
	function Object_C(...args){
	    var privateAttribute;
	    function privateMethod(){
		...
	    }
	    prototype = {
		publicAttribute: someValue,
		publicMethod: function(){
		    ...
		}
	    }
	    var new_Object_C = Object.create(prototype);
	    return new_Object_C;
	}
{% endhighlight %}

So far so good, right?
But what if we want to extend Object_C, creating a new "class" D, possibly overriding some of its public methods, while maintaining access to its protected ones? What if we need to override such methods and yet keep using their super version inside the new ones? And finally, what if we want A to be using either C and D objects without noticing the difference?

Of course, we could achieve all of this by writing a separate factory for D, maintaining the same external interface as C while rewriting all its methods altogether.
It is possible, but - _let's face it_ - **it would be a mess**, and it wouldn't be much of a DRY implementation: if we play it smart we could reuse C public methods in D, but even so any change to C private methods would imply that the same change has to be copied to D ones.
While it might be harsh but possible for a short inheritance tree and if the same developer is taking care of both "classes", if the inheritance tree contains dozens of classes, or more than one people have been working on them over time, well, here there is your recipe for bugs!
Over time, you will most likely have a bunch of objects out of synch, until they fail spectacularly, obviously during a presentation in front of your clients (remember: software is a bitch, and JS software is the most bitchy of them all!)

So, let's see how you could get out of this mess.
I propose a simple pattern, built upon one of the most famous design patterns, together with some extra care and good practices taken from other languages.

What if we could wrap any object we want to introduce to the external mashup world inside a proxy that masks all the protected stuff from outside our trust circle, and handle only this wrapper to 3rd parties?
This way we could use public properties inside our libraries, fine-tuning all the writable, enumerable and configurable properties according to our inheritance needs, while hiding these objects altogether from whoever loads our libs;
we then expose an interface to the world that lets others creates these objects only through it, and by doing so handing over only the proxy wrappers to the consumers.
So, the only question is: how do we tell this proxy which properties should be exposed to the external world and which should be "protected"?
The answer is: there is no easy way using JavaScript, there is no such a construct, of course, otherwise we wouldn't be messing around with proxies and stuff, right?
But, thanks to ECMAscript 5, we might have a loophole: enumerable properties! We could create a proxy that will expose only the public functions we declare in our objects, if we agree to consider a method public if and only if it is an enumerable property of its owner. We can then make all protected methods not enumerable (and probably not writable as well, but that's another story), and so we can build an automated way to systematically add to the proxy a... proxy method linking to each one of the public methods in our objects.

**EDIT** Over the web you can find some good libraries that uses a similar idea but relies on naming conventions only: the risk is to move the concern from behaviour to aestetic, and might trick you if you use 3rd-party libs that uses a different naming convention. I think we can do better using ECMAScript 5 new features.
    To make things a little bit clearer, I'd also suggest to have protected methods' names starting with __ (two underscores) in addition to declarying them non enumerable. But just _be aware that this naming convention isn't your main concern *(enumerability is)*_, and it's just for sake of clarity.




Basically, all we need is adding a single method to the Object class:

{% highlight javascript %}
    /** createSafeProxy()
    
        Creates and returns a safe proxy for the object passed
        that will wrap around it and expose only those methods
        that are declared as enumerable.
        
        @param {boolean, default=false} CanDestroy 
                         States if the proxy consumer has the authority to call destroy 
                         on the original object
        
        @return {Object} A proxy wrapping this object;
        @throw  Any exception the original object pseudo-constructor might throw.
      */
    function createSafeProxy(canDestroy){
        
        var property;
        var proxy = Object.create(null);
        var obj = this; //We must retain the "this" pointer to 
                        //the current object to use it inside different contexts
        
        for (property in obj){
            //DO NOT check hasOwnProperty: the proxy must work for obj's prototype methods as well
            //ONLY enumerable methods will be added to proxy's interface
            if (Function.isFunction(obj[property])){
                
                //If it's a method not marked as protected, it is added to the proxy interface;
                proxy[property] = ( 
                    function(p){
                        return  function(){
                                    var result;
                                    if (obj){
                                        result = obj[p].apply(obj, 
                                                    Array.prototype.slice.apply(arguments, [0]));
                                        //Special care is needed to support method chaining
                                        if (result === obj){
                                            //obj.property returns obj itself,
                                            //but we must return this proxy instead;
                                            return proxy;
                                        }else{
                                            return result;
                                        }
                                    }else{
                                        throw "Null reference: the object has been already destroyed";
                                    }
                                };
                    })(property);
            }
        }
        
        //Adds a wrapping destroy method to allow withdrawal of the privileges 
        //given up introducing  the consumer to obj;
        proxy.destroy = function(){
                            try{
                                if (canDestroy){
                                    obj.destroy();  //Destroys the original object 
                                }                   //but only if authorized to
                            }finally{
                                obj = null;
                            }
                        };
                            
        return proxy;
    }

    if (!Object.prototype.createSafeProxy){
        Object.defineProperty(Object.prototype, "createSafeProxy", {
                                value: createSafeProxy,
                                writable: false,
                                enumerable: false,
                                configurable: false
                            });
    }
{% endhighlight %}

One more thing we need to stress out: in order to have inheritance properly working, we need to handle original objects to the ones that are inheriting from them, but to avoid any possible encapsulation violation, the consumer should be able to access only their wrapped-in-proxy versions.
We therefore need to decouple this two aspects, and the perfect way to do so is through a factory. The factory will have internal private (pseudo)constructors for all our objects, and will expose only methods that creates a new object by returning its proxified version.

How cool is that, right? But, you know, everything comes with a price, and this time the price is that you can't have any public attribute in your classes. Is that bad? Well, not really: you just need to define the appropriate getters and setters methods for this attributes, being careful to declare them as public methods, and you're good to go! Even better, this forces you to abide with the strongest version of encapsulation best practice: no attribute should be public, but should only be accessed trough getters and setters - call it Ruby style, if you prefer.

Let's see a full example, to (hopefully!) help clarifying a bit.
First we can add a shortcut to "Object" object to help with the creation of public and protected methods (please take a look at it in the source code [here](https://github.com/mlarocca/)); it's also good to have a few more utility functions, like one to get super version of an overridden method.
With those tools in our belt, here is what we could obtain:

{% highlight javascript %}
var Factory = (
    function(){
        /** Object of type A
            @private
            
            Private (to Factory) pseudo-constructor for A-type objects
            */
        function Object_A(arg1, arg2){
            var obj_A = Object.create(Object);    //The new Object that will be returned
            
                //Private attribute: it will be visible to Object_A only
            var privateAttribute1;
                //Protected attribute: it will be visible to Object_A 
                //and every Object directly inheriting from it
            obj_A.protectedAttribute1 = "A";  //["A" is a placeholder for any value you might want]
                
                //Public attribute: it will also be visible to Object_A 
                //and every Object directly inheriting from it...
            obj_A.publicAttribute1 = 1;  //[1 is a placeholder for any value you might want]
            
            //...but by defining a proper getter and setter, consumers will be able to modify it                        
            obj_A.addPublicMethod("setPublicAttribute1", 
                                    function(val){
                                        //Special care is needed to ensure inheritance-compliancy
                                        if (this.hasOwnProperty("publicAttribute1")){
                                            this.publicAttribute1 = val;
                                        }else{
                                            //If publicAttribute1 is inherited, 
                                            //we must look up for it in the super classes;
                                            this.superMethod("setPublicAttribute1", val);
                                        }
                                        return this;
                                    });
            obj_A.addPublicMethod("getPublicAttribute1", function(){return this.publicAttribute1;});
            
            obj_A.publicMethod1 = function(m_arg1){
                try{
                    console.log("Obj " + this.protectedAttribute1 + ": " + arg1 + ", " + arg2 + ";", "Method 1: " + m_arg1);
                }catch(err){
                }
            }

            obj_A.addProtectedMethod("protectedMethod1", function(){
                //Protected method: you won't be able to call it through a proxy!
                try{
                    console.log("Protected Method 1");
                }catch(err){
                }
             });
            
            Object.seal(obj_A);   //You might want to do so, otherwise there is no real encapsulation
                                  //WARNING: using freeze instead of seal would prevent any assignment
                                  //objects's public attributes
            
            return obj_A;
        }
        
        /**
            Private (to Factory) pseudo-constructor for B-type objects
            */
        function Object_B(arg1, arg2, arg3){
            var superObj = Object_A(arg1, arg2);    //The new Object that will be returned
            var obj_B = Object.create(superObj);
            
                //Private attribute: it will be visible to Object_B only
            var privateAttribute1;
                //Protected attribute: it will be visible to Object_A and every Object directly inheriting from it
                //Overrides obj_A.protectedAttribute1
            obj_B.protectedAttribute1 = "B";  //["B" is a placeholder for any value you might want]
                
            //Public attribute 1, together with its getters and setters, is inherited from : it will also be visible to Object_A and every Object directly inheriting from it...

            
            obj_B.publicMethod2 = function(m_arg1){
                //Call protected method 1, just to show it can; 
                obj_B.protectedMethod1();
            }
            
            Object.seal(obj_B);   //You might want to do so, otherwise there is no real encapsulation
                                  //WARNING: using freeze instead of seal would prevent any assignment
                                  //objects's public attributes
            
            return obj_B;
        }                    
        
        var FactoryObj = {
            Object_A: function(){
                        return Object_A.apply(null, Array.prototype.slice.apply(arguments, [0])).createSafeProxy();
                      },
            Object_B: function(){
                        return Object_B.apply(null, Array.prototype.slice.apply(arguments, [0])).createSafeProxy();
                      }                                  
        };
        
        Object.freeze(FactoryObj);  //We don't want anybody messing with our factory!
        
        return FactoryObj;
    })();
{% endhighlight %}

And... that's about it!
Please feel free to drop a line if you have suggestions or you spotted a bug!
