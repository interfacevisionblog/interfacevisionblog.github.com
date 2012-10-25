---
layout: post
title: "MonoDevelop With A Custom Version of Mono on the Macintosh"
description: ""
category: Language
tags: [C#, computer language, hacking, mono Runtime, monoDevelop, .NET runtimes, Mono / .NET 3.5, Mono / .NET 4.0, Mono 2.8.1 (Default), Runtime version]
author: Eric Hosick
author_twitter: erichosick
---
{% include JB/setup %}

## Introduction

This blog post will tell you how to develop using MonoDevelop with your own custom version of C# Mono.

## Status and Code Completion

Currently, everything seems to work and I am able to compile programs using my own version of Mono in MonoDevelop. Code completion using a custom compiled Mono seems to be intermittent. I’ve seen code completion include new properties for the object class but many times it doesn’t.

## What you Will Need to run MonoDevelop

You will need to install mono on your mac if you don’t already have it. Be sure to install the correct binary based on your mac. Intel, PowerPC or Universal. You can get mono here: [http://www.go-mono.com/mono-downloads/download.html](http://www.go-mono.com/mono-downloads/download.html).

Of course, you need to install MonoDeveop. You can get MonoDevelop here: [http://monodevelop.com/Download](http://monodevelop.com/Download).

## Compile Your Own Version of Mono

Follow the directions on getting mono to compile here [http://www.mono-project.com/SourceCodeRepository](http://www.mono-project.com/SourceCodeRepository). 

## Setup Where Mono Will Compile To

Note: When I wrote this, Mono was still supported by Novell. The above process to compile mono may be better than what I've provided below.

You can read here on how to compile mono for the Macintosh – [http://www.mono-project.com/Compiling_Mono_on_OSX](http://www.mono-project.com/Compiling_Mono_on_OSX)

We will place our version of mono where MonoDevelop also looks for the unmodified version of Mono.

Create a directory where your mono version will be installed:

    $ sudo mkdir /Library/Frameworks/Mono.framework/Versions/2.8.1-mine

You will need to change the owner to get make install to work.

    $ sudo chown monoUser /Library/Frameworks/Mono.framework/Versions/2.8.1-mine

We can now configure, make and install our own version of mono.

    $ cd ~/projects/mono/mono-2.8.1
    $ ./configure –prefix=/Library/Frameworks/Mono.framework/Versions/2.8.1-mine/ –with-blib=embedded –enable-nls=no
    $ make
    $ make install

To verify, change to your new install:

    $ cd /Library/Frameworks/Mono.framework/Versions/2.8.1-mine/bin
    $ ./mono -V

and you should see that your version of mono was compiled today:

Mono JIT compiler version 2.8.1 (tarball Sun Dec  5 13:04:58 ICT 2010)
Copyright (C) 2002-2010 Novell, Inc and Contributors. www.mono-project.com

NOTE: The mono package comes with everything you need to make and install mono. Make sure you have the correct binary type (Intel, PowerPC or Universal). All libraries need to be compiled using the same binary type. The mono package has done this for you. If you use something like Macports you may end up with a library using the wrong binary type giving you the errors described on the bottom of [http://www.mono-project.com/Compiling_Mono_on_OSX](http://www.mono-project.com/Compiling_Mono_on_OSX).

## Using Your Version of Mono

Run MonoDevelop and go to MonoDevelop -> Preferences. Select .NET Runtimes and your version will be there automatically! You just need to set it as the default library (this doesn’t seem to stick at the time of the writing of this document).

If you have a project open you can go to Project -> Active Runtime and see your version listed there. Select this version and you will be using your version of the .NET runtime.

## What Next?

Well, go to ~/projects/mono/mono-2.8.1/mcs/class/ or /YOUROOT/projects/mono/mono-2.8.1/mcs/class/corlib/System if you want to start with Object.cs and start hacking!

# Amazement and Awe

I just want to say that after working with proprietary system for most of my life, I am totally inspired and awed by the ability to take something as big as Mono and simply hack at the root of the system (Object.cs). I really want to thank everyone who has spent their time adding to this amazing product! Thank you!

Also special thanks to Michael Hutchinson for helping me with this.