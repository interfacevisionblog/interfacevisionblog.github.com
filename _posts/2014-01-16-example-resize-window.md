---
layout: post
title: "Resizing Windows and Configuration Re-Use"
description: "This time we are talking about resizing windows and more on re-using configurations in Interface Vision."
category: Design
tags: [Code re-use, Interface Vision, resize]
author: Eric Hosick
author_twitter: erichosick
---

## Introduction

I was planning on blogging about zooming in and out but it's been a challenge (which I will talk about in detail when we do get it working).

So, we've taken a quick detour and decided to add resizing of windows. This was surprisingly simple in Interface Vision. We also did some refactoring to take advantage of the ability for properties to contain behavior.

### The Results

<iframe width="746" height="420" src="http://www.youtube.com/embed/mhfJohCRCrg?vq=hd1080" frameborder="0" allowfullscreen="allowfullscreen"> </iframe>

## Calculating How Much We Moved Our Mouse/Finger

In our post on [draggable controls]({% post_url 2014-01-13-example-window-move %}), we needed to calculate the difference between where the mouse/finger is and where it was. We could then add this delta to the windows left top corner giving us the new window position.

Our initial configuration, using [SipCoffee]({% post_url 2013-12-19-design-composition-based-language %}) to adjust:

the left of the window:

    Add (
      left PropDynamicGet ( nameStr "left" part PropScopeGet ( nameStr "viewFocus") )
      right Subtract (
        left PropGet ( nameStr "x" part PropScopeGet ( nameStr "position") )
        right PropGet ( nameStr "x" part PropScopeGet ( nameStr "positionPrior") )
      )
    )

the top of the window.

    Add (
      left PropDynamicGet ( nameStr "top" part PropScopeGet ( nameStr "viewFocus") )
      right Subtract (
        left PropGet ( nameStr "y" part PropScopeGet ( nameStr "position") )
        right PropGet ( nameStr "y" part PropScopeGet ( nameStr "positionPrior") )
      )
    )

Resizing a window will also require knowing the distance a Mouse/Finger moved. So, let's turn the delta behavior into something that is re-usable.


{#position-delta-config}
First, let's add the delta behavior to the program's scope calling them 'posDeltaX' and 'posDeltaY' so they can be re-used in other parts of the configuration:

    Scope (
      properties HashTable (
        ... // other properites here
        insert PartNamedString ( keyString "posDeltaX"
          part Subtract (
            left PropGet ( nameStr "x" part PropScopeGet ( nameStr "position") )
            right PropGet ( nameStr "x" part PropScopeGet ( nameStr "positionPrior") )
          )
        )
        insert PartNamedString ( keyString "posDeltaY"
          part Subtract (
            left PropGet ( nameStr "y" part PropScopeGet ( nameStr "position") )
            right PropGet ( nameStr "y" part PropScopeGet ( nameStr "positionPrior") )
          )
        )
      )
    )

and now let's re-factor our Add to use the "named" behavior:

for left:

    Add (
      left PropDynamicGet ( nameStr "left" part PropScopeGet ( nameStr "viewFocus") )
      right PropScopeGet ( nameStr "posDeltaX" )
    )

for top:

    Add (
      left PropDynamicGet ( nameStr "top" part PropScopeGet ( nameStr "viewFocus") )
      right PropScopeGet ( nameStr "posDeltaY" )
    )
    
This is looking a lot better. When Add.right is accessed, PropScopeGet is run. ProprScopeGet causes the behavior in posDeltaX or posDeltaY to run causing the subtraction.

How very interesting. The subtraction behavior has access to posDeltaX and posDeltaY even though it was defined outside of the Scope's part property. How does that work?

### The Scope Part Is Nest-able

One of the cool things about Scope Parts is that they are nest-able: scopes can contain scope.

<div id='id-f1-1-top'>&nbsp;</div>
<p class="pagination-centered"><img class="img-polaroid" src="/assets/img/example_scope_nesting.png"><img></p>
###### Figure-1.1: Scopes can nest allowing behavior to access properties of any scope 'above' them. {#id-f1-1}

This is really powerful because behavior, even configured outside of the scope, run within a scope has access to that scope's properties and any scope 'above' the behavior.


## Moving And Resizing Windows

To move a window, we take our deltas (posDeltaX and posDeltaX) and we add them to the top, left corner of the window.
To resize a window, we take that same delta and add it to the width and height of the window.

We can re-use the exact same logic as we did for moving a window but replace "top" (of the window) with "height" (of the window) and "left" with "width".

First, let's create a configuration, placed within the main scope, that can both resize and move a window:

    Scope (
      properties HashTable (
        ... // other properites here
        insert PartNamedString ( keyString "viewResMov"
          part ArrayList ( callBehavior true
            insert PropSet ( nameStr "withFloat"
              part PropDynamicGet ( name PropScopeGet ( nameStr "scopeHeight" )
                part PropScopeGet ( nameStr "uxViewFocus" )
              )
              source Add (
                left PropDynamicGet ( name PropScopeGet ( nameStr "scopeHeight" )
                  part PropScopeGet ( nameStr "uxViewFocus" )
                )
                right PropScopeGet ( nameStr "posDeltaY" )
              )
            )
            insert PropSet ( nameStr "withFloat"
              part PropDynamicGet ( name PropScopeGet ( nameStr "scopeWidth" )
                part PropScopeGet ( nameStr "uxViewFocus" )
              )
              source Add (
                left PropDynamicGet ( name PropScopeGet ( nameStr "scopeWidth" )
                  part PropScopeGet ( nameStr "uxViewFocus" )
                )
                right PropScopeGet ( nameStr "posDeltaX" )
              )
            )
          )
        )
      )
    )

The interesting part is that original configurations for PropDynamicGet had "hard coded" the name of the property to get: in this case "hard coded" to "top".

    PropDynamicGet ( nameStr "top" part PropScopeGet ( nameStr "viewFocus") )
    
The new configuration gets the name of the property using PropScopeGet:

    PropDynamicGet ( name PropScopeGet ( nameStr "scopeHeight" )
      part PropScopeGet ( nameStr "uxViewFocus" )
    )

{#shared-resize-move}

All we have to do then is run our configuration within a Scope that has an extended-property named "scopeHeight". Our original configuration for moving windows was [here]({% post_url 2014-01-14-example-shared-configuration %}#configuring-properties). Let's see what our changes have done:

    insert PartNamedString ( keyString "behaviorPanProcEnd"
      part ArrayList ( callBehavior true
        insert PropScopeSet ( nameStr "positionPrior"
          source PropGet ( nameStr "posInWindow" part PropScopeGet ( nameStr "eventCurrent" ) )
        )
        insert When (
          condition IsNotNil ( part PropDynamicGet ( nameStr "draggable" required false part PropScopeGet ( nameStr "controlFocus" )) )
          action ArrayList ( callBehavior true
            insert Scope (
              properties HashTable (
                insert StringKeyString ( keyString "scopeHeight" withString = "top" )
                insert StringKeyString ( keyString "scopeWidth" withString = "left" )
              )
              part PropScopeGet ( nameStr "viewResMov" )
            )
          )
        )
        insert When (
          condition IsNotNil ( part PropDynamicGet ( nameStr "draggable" required false part PropScopeGet ( nameStr "controlFocus" )) )
          action ArrayList ( callBehavior true
            insert Scope (
              properties HashTable (
                insert StringKeyString ( keyString "scopeHeight" withString = "height" )
                insert StringKeyString ( keyString "scopeWidth" withString = "width" )
              )
              part PropScopeGet ( nameStr "viewResMov" )
            )
          )
       )
    )        
        
We can now re-use the "viewResMov" behavior for both moving and resizing windows.

## Conclusion

We've been able to further refactor or configuration to improve on re-use by taking advantage of the nesting feature of Scope. We'll probably write a post on how nesting Scope is so much more awesome than relying on the function calling chain: passing information through parameters all the way down.

If you find our work interesting, please follow us [@interfaceVision](http://www.twitter.com/interfaceVision) and/or [@erichosick](http://www.twitter.com/erichosick).

## Next Step

The [next step]({% post_url 2014-01-23-example-zoom-in-out %}) in our goal of creating Interface Vision's Gui based visual development environment is to allow us to zoom in and out of a view using the pinch gesture.

The [prior step]({% post_url 2014-01-14-example-shared-configuration %}) allows us to 're-use' parts of a configuration.
