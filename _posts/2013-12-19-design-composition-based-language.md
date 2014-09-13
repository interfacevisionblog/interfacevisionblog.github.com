---
layout: post
title: "A Programming Language Based on Composition of Messages"
description: "A very simple message-oriented programming language which uses composition of messages exclusively."
category: Design
tags: [Language, Persistence, Scripting, Composition]
author: Eric Hosick
author_twitter: erichosick
---

## Introduction

Interface Vision is a Gui based visual object language and fully composable framework created using Simple Interface Programming (SIP). Programming is done by hooking up messages (not to be confused with messaging frameworks): either visually or coding in C#.

However, the syntax in C# for hooking up messages looks funky.

So, we've designed a message-oriented language called SipCoffee.

We uses composition exclusively. There is no specific syntax to describe inheritance, for loops, if/else statements, variables, scope, methods, functions and so on. All of these are replaced with messages.

## Examples

### Hello World

Hello world in SipCoffee:

<div id='id-s1-1-top'>&nbsp;</div>
    Application {
      do WriteLine { text "Hello World" }
    }

###### Source-1.1: Hello World in SipCoffee. {#id-s1-1}

### Hello World Explained

[Source-1.1](#id-s1-1-top) contains two messages named Application and WriteLine. 

Application has the property do. The do property contains an instance of WriteLine.

WriteLine has a text property, which is the text to write, and has the value "Hello World".

### Hello World 10 Times

Hello World written 10 times on separate lines.

<div id='id-s1-2-top'>&nbsp;</div>
    Application {
      do For { start 1 end 20 by 2 
        do WriteLine { text "Hello World" }
      }
    }

###### Source-1.2: Hello written 10 times in SipCoffee. {#id-s1-2}

### A ForEach

The numbers 1, 2, 4, 8, 16 and 32 written on separate lines.

<div id='id-s1-3-top'>&nbsp;</div>
    Application {
      do ForEach {
        item List [ 1 2 4 8 16 32 ]
        do WriteLine { text CurrentItem {} }
      }
    }

###### Source-1.3: A for each in SipCoffee. {#id-s1-3}

### Another ForEach

Write out the first name of users Jane, Smith and Joe: each on a new line.

<div id='id-s1-4-top'>&nbsp;</div>
    Application {
      do ForEach {
        item List [
          Hash [
            KeyPair { key "firstName" value "Jane"}
            KeyPair { key "lastName" value "First"}
          ]
          Hash [
            KeyPair { key "firstName" value "Smith"}
            KeyPair { key "lastName" value "Between"}
          ]
          Hash [
            KeyPair { key "firstName" value "Joe"}
            KeyPair { key "lastName" value "Last"}
          ]
        ]
        do WriteLine {
          text HashRead { hash CurrentItem{} key "firstName" }
        }
      }
    }

###### Source-1.4: A for each in SipCoffee against Parts. {#id-s1-4}

### ForEach Using Sql

Write out the first name of users with data coming from a database: each on a new line.

<div id='id-s1-4-top'>&nbsp;</div>
    Application {
      do ForEach {
        item SqlConnect {
          database "someDatabase"
          command SqlQuery {
            sql "SELECT firstName, lastName FROM users"
          }
        }
        do WriteLine {
          text HashRead { hash CurrentItem{} key "firstName" }
        }
      }
    }

###### Source-1.4: A for each in SipCoffee against Parts. {#id-s1-4}

### ForEach Using Sql Explained

A ForEach message contains an item property. The item property contains a SqlConnect message configured with a database (named "someDatabase") and a command (a SqlQuery message with sql "SELECT firstName, lastName FROM users").

The ForEach message invokes the SqlConnect message. The SqlConnect message connects to the database and invokes the message located in the command property. This causes SqlQuery to invoke which runs the sql and returns a list of records to SqlConnect. Each record is a hash table with a key/value pair for each field.

SqlConnect passes that list back up to the ForEach message. The ForEach message is now able to iterate through the list of records.

The ForEach message, internally, stores the current item in the list. It then invokes the message located in the do property. In this case, ForEach invokes the WriteLine message.

We want to write out the first name of each user. This means we need to access that field within the current item (which happens to be a HashTable message).

This means we use the HashRead message in the text property of WriteLine. What hash table are we reading from? Well, the CurrentItem message is able to retrieve the current item from the ForEach message. This then becomes the hash table that the HashRead message uses. The key is then used to locate an entry in the hash table: in this case "firstName".

HashRead passes up to WriteLine the result of reading from the hash table (in this case a string) and the WriteLine message writes the final value of text to the console.

## The Syntax

Syntactically, the language is simple.

Messages are upper case and properties of messages are lower case. A property contains a message or composition of messages.

We use {} to define the contents of a message. We use [] to define special messages which manage collections of data (Hash, List and Array are examples of just such a Message).

## Parsing Expression Grammar

A pseudo parsing expression grammar is as follows:
  
<div id='id-g1-1-top'>&nbsp;</div>
    a} PROPERTY <- property primitive+
    b} PROPERTY <- property MESSAGE+
    c} MESSAGE <- Part { PROPERTY* } // Could also be () if people prefered that.
    d} MESSAGE <- Part [ MESSAGE ]

###### Grammar-1.1: Parsing expression grammar for SipCoffee. {#id-g1-1}

### Primitives are Messages

Primitives are things like string, numbers and dates (to name a few). Primitives are actually messages.

Examples being:

* Strings - "Hello" (String message)
* Floats - 34.56f (Float message)
* Reals - 3.456 (Real message)
* Integers - 34 (Int message)
* Longs - 56l (Long message)
* Dates - 1994-11-05T08:15:30-05:00 (Date message - Considering only allowing UTC dates to be stored)

### Collections

A collection is defined by simply listing the item in the array separated by white space (we could also separate items using commas).

* array of strings - [ "Hello" "And" "GoodBye" ]
* array of integers - [ 1 2 5 6 12 656 ]
* array of floats - [ 23.0f 345.4f 63.346f ]
* array of Parts - [ User { name "Jane" } User { name "Toan" } User { name "Frank" } ]

## Availability

SipCoffee works with the Interface Vision framework. We've created an initial persister that is able to save a program configured in Interface Vision as SipCoffee. The persister will itself be written in SipCoffee. When we get to that point, we will blog about it.

In the mean time, please follow us [@interfaceVision](http://www.twitter.com/interfaceVision) and/or [@erichosick](http://www.twitter.com/erichosick).

## Conclusion

That's it. A very simple message-oriented language based on composition.

## Next Step

The [next step]({% post_url 2013-12-31-example-window-basic %}) was to get a program to display a native window using SipCoffee.

