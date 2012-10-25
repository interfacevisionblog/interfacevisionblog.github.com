---
layout: post
title: "Developers: Give Each Other a Fighting Chance"
description: ""
category: Software
tags: [API, Error Handling, Error Messages, Framework]
author: Eric Hosick
author_twitter: erichosick
---
{% include JB/setup %}

## Introduction

Figuring out an API can be difficult. Using one that is fraught with bugs and bad error messages is almost impossible.

## My Plead – Give Each Other a Fighting Chance

Development is hard enough without having APIs fraught with bugs and no good error messages. It seems to me, that over the years, APIs and frameworks aren’t getting better. Instead, they are becoming more fragile than ever before. There are probably a lot of reasons for this but I think one of the main reasons is a misunderstanding of what software development really is.

## Quick Point – Seeing Software Development as Automation of a System

Software Development is no more than the automation/simulation of systems (that is a strong statement to make so take a second to think about it.). The development process takes a system that is/can be done manually and automates that system via software run on a computing device.

A real world system, like manufacturing a cell phone, has an optimal process: the process to follow when there are no problems in production. However, in every system, there are situations that occur which are not optimal: parts not shipped on time, power goes out, machines lose their calibration, cost of material goes up, etc.

As such, not only is the optimal process of a system written down, but also actions to take when events occur within a system that are non-optimal.

## The Solution – Software Developers must Consider non-optimal Aspects of the System

When automating a system, as engineers, we have to take into account all of the aspects of the system we are automating: not just the optimal outcomes. Further, we have to view the computing device itself, and the software running on it, as another part of the system being automated. That means, unexpected outcomes, such as out of memory errors, must now also be considered within that system (even at the business level #1).

So, when writing an API, make sure to put as much, if not more, thought into the non-optimal aspects of the system. Further, provide adequate “error handling” or “error messages” so that the users of the API are able to act accordingly.

A good API considers both the expected and unexpected events within the system being automated.

The reason why I put “error handling” and “error message” in quotes is that, in a system, there is no such thing as an “error” as such #2. There are unexpected outcomes that require an alternate process. The only reason why we call them errors in software development is because we, as software developers, have failed to understand the purpose of software development: to automate systems.

Really, if you are going to write an API, you have an obligation as an engineer to consider all aspects of a system you are automating and provide the right exception and/or information allowing users of your API to make the correct business decisions.

## Conclusion

The APIs of today need to be more developer friendly. This can be achieved by software developers realising that the software they write must automate both the optimal and non-optimal processes within the system: including those non-optimal processes injected into the system because it is now using said computing device.

Developers really need to start providing APIs that are not fragile and provide adequate information when “errors” occur. This give the users of their APIs a fighting chance in using the API successfully.

Give your fellow/fellowet developers a chance to be successful in their development efforts.

## The Story – Why This Post

I was trying to use a console based API (I will not flame the API) and was getting an error message “File not found”. That would be a fine error message except for the part where the command line options required four different files. I did try to pass each one separately, and in each case the “File not found” error did not occur: replaced by the error “Missing option xyz”. There was also a debug option which was also of no help.

What good, in this case, is the error message “File not found”? What could I learn from it? Nothing… In fact, I spent more time trying to figure out what was wrong than I spent writing this post.

The bad part is that this API was created by developers for developers. So, the question becomes…

Do developers actually care about helping other developers become successful?

1. As an example, I may want to restart a process if it starts taking up too much memory or I might want to start a new server instance running in my cloud computing environment. This is a business decision made by the product owner.
2. I realise that there are edge cases that are difficult to find. I also know that no systems are closed, and as such it is not practical to cover all edge cases. However, knowing those edge cases and not accounting for them should not be called an error. It should be called an incomplete API and a failure to automate that system.