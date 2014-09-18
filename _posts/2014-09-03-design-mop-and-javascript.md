---
layout: post
title: "Message-Oriented Programming With Javascript"
description: "We show how to do message-oriented programming by taking advantage of Javascript's prototyping."
category: Design
tags: [Interface Vision, javascript, composition, message-oriented Programming]
author: Eric Hosick
author_twitter: erichosick
---

## Introduction

We are going to talk about Javascript and message-oriented programming (MOP). Javascript is a beautiful language and is a perfect candidate for message-oriented programming.

## Why Message-Oriented Programming

The purpose of message-oriented programming is to standardize the behavioral interface of objects. This means, invoking the behavior of any object is the exact same: irrelevant of the behavior we are trying to invoke.

Invoking the addition of two numbers is the same as invoking the behavior to push data through a socket. As we will see below, this also means programming becomes the composition of data structures.

Finally, we are able to fully disconnect mechanism (the framework) from the business behavior (using the framework).

## What is Message-Oriented Programming

We use abstractions when we program. The types of abstractions we use generally fall under different programming methodologies.

In object-oriented programming, we use objects as the core abstraction along with attributes, methods, variables and parameters (to name a few).

In functional programming, the core abstraction is functions along with parameters and variables (to name a few).

In message-oriented programming (not to be confused with messaging systems and frameworks), messages and properties are the only abstractions. There are no methods, functions, parameters, variables and so on. A property of a message contains a single message or a composition of messages.

A composition of messages is really a data structure. This means that our programs are actually data structures and, as such, can be manipulated just like you would any data structure. Want to duplicate and run part of a program in it's own thread? Just copy the program from that point, as you would any data structure, and run it.

Want to store part of your program? Just point at any part of your program and save it as you would any data structure.

## Why Abstractions Are Important

How we form mental models of the real world is shaped by the abstractions we use to describe the real world. An object-oriented mental model of a real world system will feel very different from a mental model formed by a functional approach to describing a real world system. The same can be said about different maths.

Messages are, in our opinion, a great way of forming a mental model of real world systems because they are conceptually simpler than functions or objects and require fewer abstractions (no methods, functions, parameters, variables and so on.).

## Implementation

### Prototypal Inheritance

We need to be able to create instances of messages and we are going to do this using [prototypal inheritance](http://javascript.crockford.com/prototypal.html).

The general pattern for prototypal inheritance looks something like this:

    function object(o) {
      function F() {}
      F.prototype = o;
      return new F();
    };

However, the behavioral interface of messages is going to be the exact same so let's be a little more descriptive of what it means to create a message.

    function message(behavior) {
      function Msg() {}
      msg.prototype = behavior;
      return new Msg();
    };

### Common Behavior For All Messages

In our existing C# framework, we came up with a list of common behavior across all message. In this post, we'll only provide a few of these common behaviors for brevity.

For now, let's consider go, asNum and asStr as our common behavior.

* go - Accessing this property causes the message's behavior to run.
* asNum - Accessing this property causes the message's behavior to run as a numeric message.
* asStr - Accessing this property causes the message's behavior to run as a string message.

The behavioral interface for any message will look something like this:

    var Message = {
      get go() {
        // code goes here
        return something;
      }
      get asNum {
        // code goes here
        return numeric;
      }
      get asStr {
        // code goes here
        return string;
      }
    };

You can also add data to the interface which will be shared for that message type.

### Data And Messages

So, we have a way to define the behavior for all messages in our framework. However, how do we get information required by a message into that message to run the message's behavior? After all, we aren't using parameterized functions.

What if we pass both behavior and data to create a message. Something like this:

    function msg(behavior, data) {
      var Msg = function() {}; 
      Msg.prototype = behavior; // shared behavior and data (if any)
      var msg = new Msg();
      msg.data = data; // instance specific data
      return msg;
    };

So, now within one of our behavior properties we could do something like this:

    ...
    get asNum {
      return this.data.val; // where example data was { val: 5 }
    };

To create a new Numeric message we could do something like this:

    var Num = {
      get go() { return this.asNum; },
      get asNum() { return this.data.val; },
      get asStr() { return this.data.val.toString(); }
    };

    var newMessage = msg (Num, {val:4})
    
    newMessage.go; // returns 4
    newMessage.asNum; // returns 4
    newMessage.asStr; // returns "4"

Notice that any data required for a message is located neatly in one property.

### Describing Addition Using Messages

To build up an understanding of message-oriented programming, we're going to add two numbers using messages.

This will look like we aren't gaining anything by using MOP to implement something as simple as addition but just stick with us.

Programming in MOP is done through the composition of messages. It is very similar to creating a data structure.

To add we will need two messages. A message that knows how to Add and a message that represents a Numerical data type.

    // Behavior for a numerical value (same as above)
    var Num = {
      get go() { return this.asNum; },
      get asNum() { return this.data.val; },
      get asStr() { return this.data.val.toString(); }
    };

    // Behavior for addition
    var Add = {
      get go() { return this.asNum; },
      get asNum() { return this.data.left.asNum + this.data.right.asNum; },
      get asStr() { return this.data.left.asStr + this.data.right.asStr; }
    };

To compose addition we do the following:

    var addMsg = msg( Add, {
      left: msg (Num, {val:23} ),
      right: msg( Num, {val:44} )
    });

Finally, to use the message we could do one of the following:

    addMsg.go; // returns 67;
    addMsg.asNum; // returns 67;
    addMsg.asStr; // returns "2344"

What is really cool about this is that you can run the same message as different primitives. In this case, we are able to run the Add message as a string or a numeric.

One more example.

    var addMsg = msg( Add, {
      left: msg( Sub, {
        left: msg (Num, {val:0} ),
        right: msg( Num, {val:44} )
      }),
      right: msg( Num, {val:44} )
    });

### Adding Two Numbers Entered Into a Form

Here is where we can start seeing some of the strengths of message-oriented programming.

Let's create some messages that are able to access HTML input elements:

    // Access an HTML Input Element using JQuery
    var FormFieldGet = {
      get go() { return this.asNum; },
      get asNum() { return $(this.data.id).val(); },
      get asStr() { return $(this.data.id).val().toString(); }
    };

    // Update an HTML Input Element using JQuery
    var FormFieldSet = {
      get go() { return $(this.data.id).val(this.data.val.go); },
    };

Please note that these messages are part of a framework. We can look at them as mechanisms that we use to compose business behavior.

We have html as follows:

    <form>
      <input type="text" name="left"> +
      <input type="text" name="right"> =
      <input type="text" name="result">
    </form>

When programming using MOP, we will take a different approach to solving problems than we would using traditional OOP or functional programming methodologies. We need to think about composition of behavior. Taking a bunch of small messages and hooking them up to get new types of behavior.

In this specific case, we need to update a field on a form so we will be using a FormFieldSet message:

    msg( FormFieldSet, {
      id: "result",
      val: ???
    }).go;

Calling go let's us run the message immediately.

But what do we put in place of val? What value are we trying to set? The value we want is the addition of two things.

    msg( FormFieldSet, {
      id: "result",
      val: msg( Add, {
        left: ????,
        right: ????,
      }),
    }).go;

The things we want to add are the other two form fields. So, let's access them and get their values:

    msg( FormFieldSet, {
      id: "result",
      val: msg( Add, {
        left: msg( FormFieldGet, { id:"left" } ),
        right: msg( FormFieldGet, { id:"right" } ),
      }),
    }).go;

OR

    // Using custom Prototyes for FormFieldSet, Add,
    // FormFieldGet instead of the shared msg Prototye
    
    FormFieldSet ({
      id: "result",
      val: Add ({
        left: FormFieldGet ({ id:"left" }),
        right: FormFieldGet ({ id:"right" }),
      }),
    }).go;

And we are done.

Please note that what you are looking at is 100% business behavior. The structure of the code itself looks very different form the code we used to create the messages in the first place.

There is almost a complete disconnect between how we describe business behavior and the mechanisms that do the work for us. We have 100% encapsulation and a 100% decoupled system (because all messages have the exact same behavioral interface). This is the holy grail of object-oriented programming promised so many decades ago.

Take a moment to look at how clean that Javascript looks. Very consistent in look. Very dry.

The javascript program itself is also a data-structure that can be easily persisted or even traversed. During run time, we can alter the behavior of our program by changing the message composition (this is actually very different from code generation).

We could dump that javascript in a key/value store for easy re-use.

And the code looks so clean. So pure.

## Conclusion

Javascript is a very versatile, and awesome, language that supports message-oriented programming. Using MOP, we are able to compose programs that can be manipulated as if they were data-structures.

There is a lot more we can do with this to clean things up. For example, our msg() function could do boxing of primitive data types for us making our message configurations easier to read. We would place this in a library.

If you find our work on message-oriented programming interesting, please follow us [@interfaceVision](http://www.twitter.com/interfaceVision) and/or [@erichosick](http://www.twitter.com/erichosick).

