---
layout: post
title: "Configuration re-use"
description: "How we re-use configurations in Interface Vision"
category: Design
tags: [Code re-use, Interface Vision, shared]
author: Eric Hosick
author_twitter: erichosick
---

## Introduction

A goal of programming is to create code that is re-usable.

That is, we are able to generalize a behavior required in a software system, write code to implement that behavior, and then re-use that code throughout the software system.

Interface Vision is able to configure behavior that can be re-used throughout the program.

## The Approach

In our post on [draggable controls]({% post_url 2014-01-13-example-window-move %}), we had some configurations that were redundant. That is, the exact same configuration was found in more than one place within the configuration.

We want to place this duplicated configuration in some area within our program and make it accessible to other areas of our program. This should allow us to re-use that configuration as needed.

It turns out we don't need to do anything to support re-use within Interface Vision. Re-use is an emergent feature ([emergent behavior](http://en.wikipedia.org/wiki/Emergent_behavior)) of our technology.

Let's see how it works.

## Behavior Re-use in Interface Vision

If you recall, we were able to [declare variables]({% post_url 2014-01-13-example-window-move %}#declaring-variables) in Interface Vision as follows:

    Scope (
      properties HashTable (
        insert PartNamedString ( keyString "eventCurrent" )
        // other variables within the scope
      )
    )

In this case, we have declared a scope variable named eventCurrent.

What we didn't mention is that a variable, and really any property for that matter, can contain more than just information. In fact, a variable (or property) can also contain behavior.

{#configuring-properties}
## Configuring Properties So They Can Contain Behavior

Let's take the behavior we want to make re-usable and place that behavior in Scope variables:

    Scope (
      // .. other scope variables 
      insert PartNamedString ( keyString "behaviorPanBegin"
        part ArrayList ( callBehavior true
          insert PropScopeSet ( nameStr "eventBegin" source PropScopeGet ( nameStr "eventCurrent" ) )
          insert PropScopeSet ( nameStr "eventPrior" source PropScopeGet ( nameStr "eventCurrent" ) )
          insert PropScopeSet ( nameStr "viewFocus"
            source PropGet ( nameStr "view"
              part PropScopeGet ( nameStr "eventCurrent" )
            )
          )
          insert PropScopeSet ( nameStr "positionPrior"
            source PropGet ( nameStr "posInWindow"
              part PropScopeGet ( nameStr "eventCurrent" )
            )
          )
          insert PropScopeSet ( nameStr "controlFocus" sourceRequired false
            source UxControlAtPoint ( returnView false
              position PropGet ( nameStr "posInView"
                part PropScopeGet ( nameStr "eventCurrent")
              )
              shapes PropScopeGet ( nameStr "viewFocus" )
            )
          )
        )
      )
      insert PartNamedString ( keyString "behaviorPanProcEnd"
        part ArrayList ( callBehavior true
          insert PropScopeSet ( nameStr "positionPrior"
            source PropGet ( nameStr "posInWindow" part PropScopeGet ( nameStr "eventCurrent" ) )
          )
          insert When (
            condition IsNotNil ( part PropDynamicGet ( nameStr "draggable" required false part PropScopeGet ( nameStr "controlFocus" )) )
            action ArrayList ( callBehavior true
              insert PropSet ( nameStr "withFloat"
                part PropDynamicGet ( nameStr "top" part PropScopeGet ( nameStr "viewFocus") )
                source Add (
                  left PropDynamicGet ( nameStr "top" part PropScopeGet ( nameStr "viewFocus") )
                  right Subtract (
                    left PropGet ( nameStr "y" part PropScopeGet ( nameStr "position") )
                    right PropGet ( nameStr "y" part PropScopeGet ( nameStr "positionPrior") )
                  )
                )
              )
              insert PropSet ( nameStr "withFloat"
                part PropDynamicGet ( nameStr "left" part PropScopeGet ( nameStr "viewFocus") )
                source Add (
                  left PropDynamicGet ( nameStr "left" part PropScopeGet ( nameStr "viewFocus") )
                  right Subtract (
                    left PropGet ( nameStr "x" part PropScopeGet ( nameStr "position") )
                    right PropGet ( nameStr "x" part PropScopeGet ( nameStr "positionPrior") )
                  )
                )
              )
              insert PropScopeSet ( nameStr "eventPrior" source PropScopeGet ( nameStr "eventCurrent" ) ) 
              insert PropScopeSet ( nameStr "positionPrior" source PropScopeGet ( nameStr "position" ) ) 
            )
          )
        )
      )
      // .. other scope variables       
    )

We've defined two scope variables named 'behaviorPanBegin' and 'behaviorPanProcEnd'. The "Part" contained within the part property of PartNamedString are the same configurations we had defined in our prior post.

## Using Properties With Behavior

Original, the above behavior was configured within our EventManager. Let's see how we access that behavior now that it is defined outside of the EventManager:

    EventManager ( keyString "PropEventManager"
      properties HashTable {
        insert EventMonitor (
          eventToMonitor GesturePanEventNat ( eventStep "Begin" touchesMinVal 1 )
          action PropScopeGet ( nameStr "behaviorPanBegin" )
        )
        insert EventMonitor (
          eventToMonitor GesturePanEventNat ( eventStep "Processing" touchesMinVal 1 )
          action PropScopeGet ( nameStr "behaviorPanProcEnd" )
        )
        insert EventMonitor (
          eventToMonitor GesturePanEventNat ( eventStep "End" touchesMinVal 1 )
          action PropScopeGet ( nameStr "behaviorPanProcEnd" )
        )
    )

That's it. All we have to do, for the action of the EventMonitor Part we are defining, is access the scope variable by calling PropScopeGet. Using PropScopeGet causes the configuration located within the part property to run.

## Conclusion

Configuring behavior that is re-usable is really easy in Interface Vision. 
If you find our work interesting, please follow us [@interfaceVision](http://www.twitter.com/interfaceVision) and/or [@erichosick](http://www.twitter.com/erichosick).

## Next Step

The [next step]({% post_url 2014-01-16-example-resize-window %}) is to do further refactoring and allow a window to be resized.

The [prior step]({% post_url 2014-01-13-example-window-move %}) was to create a configuration to allow us to drag controls around on the screen..

