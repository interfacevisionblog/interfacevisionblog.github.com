---
layout: post
title: "Specifications and Quality"
description: ""
category: Agile
tags: [ bdd, bdd specifications, requirements, quality, QA]
author: Eric Hosick
author_twitter: erichosick
---
{% include JB/setup %}

## What are Specifications?

Specifications are “a detailed description of the design and materials used to make that something”.

Generally that “something” is a product and the detailed description is the features leaving us with:

> Specifications describe the features of a product.

## A Product Without Specifications Has No Quality

Two customers can have completely different expectations of what a good quality tire is:

* This tire is of low quality. I have to replace them every 30,000 kilometres. My other tires lasted me 90,000 kilometres.
* This is a high quality tire. I can go much faster around corners.

In both cases, the tire is soft providing better road traction allowing a car to go around corners faster. The side affect of this is that the tires have to be replaced more often.

This means there is no way to say the tire has quality because “quality” now becomes subjective to the user of the product. I could go on about what headaches this is going to lead to but may leave that for another post.

## Product Without Specifications May Not Possible to Produce!

I am asked to deliver a hole in a 3mm thick piece of aluminium that is 12.5mm in diameter. I proceed to drill a hole that is 12.51mm in diameter. I am told that the product does not meet the specifications. They needed a hole that is exactly 12.5mm in diameter.

Unfortunately, no matter how hard I try, I will not be able to make that hole exactly 12.5mm in diameter. There will always be a measuring device that will show that the hole I made is not exactly 12.5mm.

I could say “Yes, but it is 12.5000003mm. That is good enough.”

They disagree.

Subjective has just reared it’s ugly head. Even worse, in this case, I will not be able to make the product I committed to.

## Creating Specifications Allows for Quality

Writing down specifications allows you to:

* Know if the product can be made.
* Assure you understand the vision (even if it is your own).
* Estimate and do project management.
* Assure you haven’t missed anything.
* Verify the product.

The last point is really important.

## Specifications ARE Testable

Wow! This is amazing because you are able to test and verify that what you are making matches with what was asked for:

> Specifications are used to verify that features are correct.

You must have a way to measure the specifications to assure that you have met the quality expected by the customer. That is, the features of the product being made have actually been made.

To do this, you must be able to measure or test that you have actually delivered the specifications.

## How does Agile fit into all of this?

I will go into detail in other blog entries but Agile provides tools to provide specifications and assure quality while minimising costs:

* Current log and backlog contains specifications managed by product owner.
* Specifications created in detail only when they will be developed in the current iteration (don’t need to spec out an entire product in detail up-front).
* For software, Behaviour Driven Development (BDD) allows for one-to-many correlations between features and implementation.
