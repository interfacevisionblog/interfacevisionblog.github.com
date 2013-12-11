---
layout: post
title: "SipCoffee - A Composite Based Programming Language"
description: "A very simple programming language which uses composition exclusively."
category: Design
tags: [Language, Persistence, Scripting]
author: Eric Hosick
author_twitter: erichosick
---

## Introduction

Interface Vision is a Gui based visual object language and fully composable framework. Programming is done by hooking up parts: either visually or coding in C#.

We've designed a language, SipCoffee, to simplify the syntax of describing the hooking up of Parts.

## Examples

Hello world in SipCoffee:

<div id='id-s1-1-top'>&nbsp;</div>
    Application (
      using Library ( name "Vision.Console" )
      action WriteLine ( text "Hello World" )
    )

###### Source-1.1: Hello World in SipCoffee. {#id-s1-1}

Hello written 10 times

<div id='id-s1-2-top'>&nbsp;</div>
    Application (
      using Library ( name "Vision.Console" )
      action BetweenInclusive (
        start 1
        end 20
        incrementBy 2 
        action WriteLine ( text "Hello" )
      )
    )

###### Source-1.2: Hello written 10 times in SipCoffee. {#id-s1-2}

The numbers 1, 2, 4, 8, 16 and 32 written on seperate lines

<div id='id-s1-3-top'>&nbsp;</div>
    Application (
      using Library ( name "Vision.Console" )
      action ForEach (
        items 1 2 4 8 16 32
        action WriteLine ( text CurrentItem () )
      )
    )

###### Source-1.3: A for each in SipCoffee. {#id-s1-3}

Write out the first name of users Jane, Smith and Joe: each on a new line.

<div id='id-s1-4-top'>&nbsp;</div>
    Application (
      using Library ( name "Vision.Console" )
      action ForEach (
        items
          Part ( properties
            StringNamed ( name "firstName" value "Jane")
            StringNamed ( name "lastName" value "First")
          )
          Part ( properties
            StringNamed ( name "firstName" value "Smith")
            StringNamed ( name "lastName" value "Between")
          )
          Part ( properties
            StringNamed ( name "firstName" value "Joe")
            StringNamed ( name "lastName" value "Last")
          )
        action WriteLine (
          text Property ( name "firstName" part CurrentItem() )
        )
      )
    )

###### Source-1.4: A for each in SipCoffee against Parts. {#id-s1-4}

Write out the first name of users with data coming from a database: each on a new line.

<div id='id-s1-4-top'>&nbsp;</div>
    Application (
      using Library ( name "Vision.Console" )
      action ForEach (
        items SqlConnect (
          database "someDatabase"
          command SqlQuery (
            sql "SELECT firstName, lastName FROM users"
          )
        )
        action WriteLine (
          text Property ( name "firstName" part CurrentItem() )
        )
      )
    )

###### Source-1.4: A for each in SipCoffee against Parts. {#id-s1-4}

Syntactically, the language is simple.

Parts are upper case and properties are lower case.

Each Part is a class [1](#id-1) contained within the Interface Vision framework.

Methods/Functions are not part of the language (which greatly simplifies the syntax).

## Source-1.1 Explained

[Source-1.1](#id-s1-1-top) contains 3 parts named Application, Library and WriteLine. 

Application has two properties: using and action. The using property contains an instance of Library and the action property contains an instance of WriteLine.

Library has a name property which is the name of the library to load. In this case, the name property is set to a string primitive type and contains the value "Vision.Console".

WriteLine has a text property, which is the text to write, and have the value "Hello".

## Parsing Expression Grammar

A pseudo parsing expression grammar is as follows:
  
<div id='id-g1-1-top'>&nbsp;</div>
    a) PROPERTY <- property primitive+
    b) PROPERTY <- property PART+
    c) PART <- Part ( PROPERTY* )

###### Grammar-1.1: Parsing expression grammar for SipCoffee. {#id-g1-1}

### Primitives

A primitive is a string or numeric. Examples being:

* Strings - "Hello"
* Floats - 34.56f
* Reals - 3.456
* Integers - 34
* Longs - 56l

### Collections

A collection is defined by simply listing the items in the array separated by white space.

* array of strings - "Hello" "And" "GoodBye"
* array of integers - 1 2 5 6 12 656
* array of floats - 23.0f 345.4f 63.346f
* array of Parts - User ( name "Jane" ) User ( name "Toan" ) User ( name "Frank" )

## Conclusion

That's it. A very simple language based on composition of behavior.

## Footnotes

{#id-1} 1. Actually, a Part can be a class or a composition of other parts. But this isn't relevant for the SipCoffee language.


