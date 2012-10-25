---
layout: post
title: "You see Coding as a Loosing Game if You Focus on Testing"
description: ""
category: Agile
tags: [TDD, BDD, MVP]
author: Eric Hosick
author_twitter: erichosick
---
{% include JB/setup %}

## Introduction

Focusing on testing is like assuming we are going to lose the game and we only want to loose by a little bit: we assume there will be bugs – a negative approach.

The thing we should focus on is the User Experience (UX). Testing should be our second line of defence.

## Focus on the Features and UX

Focusing on the UX, the behaviour, the features is a positive approach. We focus on the wining game: delivering exactly what the stakeholders want (1).

* **Behaviour Driven Development (BDD)** assures we are implementing only what the stakeholders want (1). We are “testing” the features before we even think about testing code. This is our first line of defence.
* **Test Driven Development (TDD)** provides the robustness and engineering – it is our second line of defence.

## Use Both BDD and TDD

behaviour Driven Development, using a DSL like gherkin, provides the glue between the stakeholders (aka the UX) and the code. This glue drives our development and, following red/yellow/green/refactor, assures that every line of code written is directly related to implementing the behaviour (aka the UX) (2).

BDD does not assure robustness and there may be overlooked edge cases that would require additional behaviour (aka UX).

Test Driven Development, as our second line of defence, is where the engineering comes in assuring the software system is robust. Additional behaviour may be discovered, such as what to do with errors, and TDD is a good way to discover such edge cases. TDD feeds back into BDD allowing for the discovery of additional behaviour perhaps missed when mocking out that initial UX.

## Conclusion

Don’t focus on testing your code, focus on implementing only the behaviour asked for by the stakeholder.

Use BDD to assure that the minimum lines of code are written.

Use testing to assure robustness, good engineering and as a feedback loop to fill in missing behaviour.

1. Business still needs to figure out what stakeholders want to assure minimal behaviour (Minimal Viable Product). This process should be as painful as possible. The more pain, the less behaviour. The less behaviour, the less code. Less code means less bugs.
2. The ability to regression test our behaviour is a huge benefit but pales in comparison to the benefits from assuring the minimum amount of code is being written.
