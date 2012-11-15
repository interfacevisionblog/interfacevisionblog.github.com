---
layout: post
title: "Should the Class Object Be Enumerable?"
description: ""
category: Framework
tags: [Interface Vision, Aggregate, Collection, Objects, Framework, Architecture]
author: Eric Hosick
author_twitter: erichosick
---
{% include JB/setup %}

## Introduction

We've spent a lot of time thinking about framework. One question we asked ourselves was:

> Should our class object be enumerable?

[Jonathan Swift](http://en.wikipedia.org/wiki/Jonathan_Swift) has something to say on this:

> So nat'ralists observe, a flea
> Hath smaller fleas that on him prey;
> And these have smaller fleas to bite 'em.
> And so proceeds [Ad infinitum](http://en.wikipedia.org/wiki/Ad_infinitum)

### Our Thoughts

We started by asking ourselves if all things in the Universe are composed of other things. The answer is apparently a big yes (though it could be that at some point, at some quantum level, this is not the case).

<div class="pagination-centered img-polaroid">
  <p>
    <img src="/assets/img/object-aggregate-tortugues.jpg" alt="">
  </p>
</div>

Being that all things in the real world are composed of other things, it made sense for us to add this behavior to our class object.

### An Example

Let's look at what a foreach loop looks like within our framework:

    ...
    object information = new Thing();
    foreach ( object o in information) {
      o.action = Empty.instance;
    }
    ...

In this case, we have some information of type object. We do not know if that information is an actual aggregate or a single instance (though we do allow you to query if an object is an actual aggregate by calling isAggregate). However, this doesn't matter and we are able to traverse the returned object instance.

Traditionally, a developer would have to write something like this:

    ...
    object information = new Thing();
    if ( information is AggregateThing) {
      foreach ( object o in (information as AggregateThing)) {
        o.action = Empty.instance;
      }
    } else {
      o.action = Empty.instance;
    }
    ...

### We Are Anti-If

From our example usage, you can see that we do not need an if statement.

Personally, I'm a fan of the [Anti-if Campaign](http://www.antiifcampaign.com/). Treating all objects as aggregates allows us to remove quite a few ifs from our framework and also allows users of our framework to use less ifs.


### Implementation

Treating object as an instance is relatively easy.

The length of object is always 1.

    ...
    public virtual int length {
      get { return 1; }
    }
    ...

Making the instance enumerable requires a little extra work (note this is for C#):

    public virtual IEnumerable enumerable {
      get {
        IEnumerable result = new ObjectEnumerable() {  
          theEnumerator =  new ObjectEnumerator() { singleItem = this } as IEnumerator
        } as IEnumerable;
        return result;
      }
    }

Implementing the indexers is a little more difficult requiring design decisions.

    public virtual IItem this[int position] {
      get {
        return this;
      }
      set {
        throw new Exception("No can do.");
      }
    }

In the example code, for get, we are simply returning the instance. Should we throw an exception if the index is greater than 0? We are of the mind set that exceptions are bad. So, it might be better to just ignore the index itself and always return this.

Setting also requires some consideration. We really can't set the single instance. Overwriting the properties of the existing object with the new object's properties is probably totally out of the question.

Throwing an exception is an option, as was done, but perhaps simply ignoring the set is the best option.

Design decisions.

### Systems Thinking

In systems thinking, every part of that system must be required to transform some input into a desired output. If you are able to remove any part of that system, and still have the same desired output, then it's not a system. You have to remove that part of the system.

Of course, this is based on the context of the system and desired output.

From the standpoint of systems thinking, a coded object is not necessarily always an aggregate. In this way, we have violated a principal of systems thinking.

### Conclusion

All objects in the Universe are composed of other objects so, within our framework, object should also be enumerable.

Treating object as an enumerable allowed us to greatly simplify our framework and usage of our framework.

### References

Image from [http://en.wikipedia.org/wiki/File:Giambologna_tortugues.jpg](http://en.wikipedia.org/wiki/File:Giambologna_tortugues.jpg).

