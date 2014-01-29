---
layout: post
title: "Templates and Factories for a Gui Based Language"
description: "A composition of parts can be viewed as a template. A factory can copy a template letting us dynamically create a window."
category: Design
tags: [Code re-use, Interface Vision, resize]
author: Eric Hosick
author_twitter: erichosick
---

## Introduction

In all our prior examples, our configuration had windows pre-defined within the 'canvas'. Instead of configuring the windows within the canvas, we need a way to add to our configuration dynamically.

To add to our configuration dynamically, we're going to use factories which 'build' things from Templates.

### The Results

<iframe width="746" height="420" src="http://www.youtube.com/embed/eiB8Vpltouc?vq=hd1080" frameborder="0" allowfullscreen="allowfullscreen"> </iframe>

## Templates

In computing we consider a template to be:

> a preset format for a document or file, used so that the format does not have to be recreated each time it is used.

Traditionally, software frameworks have to code out the Template functionality to add it to the framework.

In Interface Vision, Templates are an emergent aspect of the framework. How so?

Any composition of parts, even if that composition contains only one part, can be used as a Template.

In prior examples, our [complete application]({% post_url 2014-01-13-example-window-move %}/#complete-application) had the windows that showed up on our canvas composed within the canvas:

    UxViewDrawNat ( keyString "canvas" styling ".canvasDoc"
      properties HashTable (
        insert StringKeyString ( keyString "Css"
          withString "{ width: 4000px; height: 4000px; scale-width:100%, scale-height:100% }"
        )
      )
      uxActions ArrayList ( callBehavior true
        insert GesturePinchRecognizer ()
      )
      uxControls ArrayList ( callBehavior true
        insert UxViewDrawNat ( keyString "window01" styling ".windowFrame"
          properties HashTable (
            insert StringKeyString ( keyString "Css"
              withString "{ top: 50px; left: 50px; width: 200px; height: 200px; scale-width:100%, scale-height:100% }"
            )
          )
          uxActions ArrayList ( callBehavior true
            insert GesturePanRecognizerNat ( touchesMinNum 1 )
          )
          uxControls ArrayList ( callBehavior true
            insert UxRectRoundNat ( keyString "winBtnMove" styling ".winBtnMove"
              uxControls UxImageNat ( keyString "winBtnMoveImage"
                properties HashTable ( insert StringKeyLong ( keyLong  "draggable" ) )
                image ImageNat ( fileName Vision.Core.String ( withString "list.png" ) )
              )
            )
            insert04 UxRectRoundNat ( keyString "winBtnResize" styling ".winBtnResize"
              uxControls UxImageNat ( keyString "winBtnResizeImage" 
                properties HashTable ( insert StringKeyLong ( keyLong "resizable" ) )
                image ImageNat ( fileName Vision.Core.String ( withString "resize_0.png" ) )
              )
            )
          )
        )
        insert UxViewDrawNat ( keyString "window02" styling ".windowFrame"
          properties HashTable (
            insert StringKeyString ( keyString "Css"
              withString "{ top: 250px; left: 350px; width: 200px; height: 300px; scale-width:100%, scale-height:100% }"
            )
          )
          uxActions ArrayList ( callBehavior true
            insert GesturePanRecognizerNat ( touchesMinNum 1 )
          )
          uxControls ArrayList ( callBehavior true
            insert UxRectRoundNat ( keyString "winBtnMove" styling ".winBtnMove"
              uxControls UxImageNat ( keyString "winBtnMoveImage"
                properties HashTable ( insert StringKeyLong ( keyLong  "draggable" ) )
                image ImageNat ( fileName Vision.Core.String ( withString "list.png" ) )
              )
            )
            insert04 UxRectRoundNat ( keyString "winBtnResize" styling ".winBtnResize"
              uxControls UxImageNat ( keyString "winBtnResizeImage" 
                properties HashTable ( insert StringKeyLong ( keyLong "resizable" ) )
                image ImageNat ( fileName Vision.Core.String ( withString "resize_0.png" ) )
              )
            )
          )
        )
      )
    )

As you can see, this is problematic because the configurations for window01 and window02 are the exact same (other than the top, left, width and height css values). What we want to do is make window configurations reusable.

We make our configuration reusable by placing the configuration within a Factory. We don't need to explicitly define the configuration as a template.

## Factories

A Factory is used to create new instances of parts during run time.

### The FactoryInstance Part

The FactoryInstance Part, when given a Template, will create an exact copy of the template. For example:

    FactoryInstance (
      part ConsoleWrite (
        text String (
          withString "Hello"
        )
      )
    )

This configuration does not cause "Hello" to be written to the console. Instead, the configuration returns a new ConsoleWrite part pre-configured with a String Part having "Hello" as the text to write. So, in this case, ConsoleWrite is being used as a template.

## Tap Gesture Creates a Window

For this example, every time a user taps somewhere on the 'canvas', we will create a new window. This will replace our existing behavior (which is drawing two windows in known locations).

### Tap Gesture Event

The configuration for adding the Tap Gesture to the [event system]({% post_url 2014-01-04-example-events-basic %}) is as follows:

    EventManager (
      properties HashTable (
        // other configured events like pan and pinch
        insert EventMonitor ( eventToMonitor GestureTap ( eventStep "finished" tapsRequiredNum 1 touchesRequiredNum 1 )
          action PropScopeGet ( nameStr "behaviorTap" )
        )
      )
    )

In this configuration, if a user taps then the 'behaviorTap' configuration will run.

## Tap Gesture Behavior

When a user taps on the 'canvas', we want to create a window at the position the user tapped. This will require us to:

* 1) Create a new instance of the window
* 2) Set the new window's top position to the y value of the tap position within the view
* 3) Set the new window's left position to the x value of the tap position within the view
* 4) Add the newly created window to the view.
* 5) Notify the view that it's contents have changed so it can redraw.

### Using a Factory to Create a Window

Let's take the window01 (or window02) configuration and use the configuration as a Template by placing it in a FactoryInstance Part.

    insert PropScopeSet ( nameStr "newWindow" 
      source FactoryInstance (
        part UxViewDrawNat ( styling ".windowFrame"
          properties HashTable (
            insert StringKeyString ( keyString "Css"
              withString "{ top: 0px; left: 0px; width: 200px; height: 200px; scale-width:100%, scale-height:100% }"
            )
          )
          uxActions ArrayList ( callBehavior true
            insert GesturePanRecognizerNat ( touchesMinNum 1 )
          )
          uxControls ArrayList ( callBehavior true
            insert UxRectRoundNat ( keyString "winBtnMove" styling ".winBtnMove"
              uxControls UxImageNat ( keyString "winBtnMoveImage"
                properties HashTable ( insert StringKeyLong ( keyLong  "draggable" ) )
                image ImageNat ( fileName Vision.Core.String ( withString "list.png" ) )
              )
            )
            insert UxRectRoundNat ( keyString "winBtnResize" styling ".winBtnResize"
              uxControls UxImageNat ( keyString "winBtnResizeImage" 
                properties HashTable ( insert StringKeyLong ( keyLong "resizable" ) )
                image ImageNat ( fileName Vision.Core.String ( withString "resize_0.png" ) )
              )
            )
          )
        )
      )
    )

The only changes to the initial Window configuration is that we set the top and left to 0px (because we will be overriding these anyway).

We use the result of the Factory and place it in a Scope property named 'newWindow'. You will see, in the next step, that we place the result of the factory in this scope property so we can update the top/left values before placing the window into the view.

### Updating Top and Left

We've created a copy of our window configuration and now we need to set the top and left values to the x,y position of the tap gesture.

    insert PropSet ( nameStr "withFloat"
      part PropDynamicGet ( nameStr "left"
        part PartHold ( part PropScopeGet ( nameStr "newWindow" ) )
      )
      source PropGet ( nameStr "x" part PropScopeGet ( nameStr "position" ) )
    )
    insert PropSet ( nameStr "withFloat"
      part PropDynamicGet ( nameStr "top"
        part PartHold ( part PropScopeGet ( nameStr "newWindow" ) )
      )
      source PropGet ( nameStr "y" part PropScopeGet ( nameStr "position" ) )
    )

The most interesting part of this configuration is the usage of PartHold:

    PartHold ( part PropScopeGet ( nameStr "newWindow" ) )

Part's in our framework automatically run when used. The scope-property 'newWindow' contains our newly created window which is a UxViewDrawNat part. The behavior of this part is to add itself to the parent view. We don't want to actually run the behavior of the part. We just want to get a reference to it so we can update it's top and left position.

This is where the PartHold Part comes in. The PartHold part puts a Part's behavior "on hold" allowing you to access the part without running it.

### Add the Newly Created View

We've created the window and updated the windows top,left position. Now we need to place the part into the array of controls of the parent view.

    insert PropSet ( nameStr "insert"
      part PropGet ( nameStr "uxControls" part PropScopeGet ( nameStr "uxViewFocus" ) )
      source PartHold ( part PropScopeGet ( nameStr "newWindow" ) )
    )


## Complete Solution

The complete configuration for the tap event is provided:

    Scope (
      properties HashTable (
        // other parts within the global scope
        insert PartNamedString ( keyString "behaviorTap"
          part ArrayList ( callBehavior true
            insert When (
              condition IsEqString (
                leftString "canvas" 
                right WithKeyString ( part PropScopeGet ( nameStr "uxViewFocus" ) )
              )
              action Scope (
                properties HashTable (
                  insert PartNamedString ( keyString "newWindow" )
                )
                part ArrayList ( callBehavior true
                  // 1. Create window
                  insert PropScopeSet ( nameStr "newWindow" 
                    source FactoryInstance (
                      part UxViewDrawNat ( styling ".windowFrame"
                        properties HashTable (
                          insert StringKeyString ( keyString "Css"
                            withString "{ top: 0px; left: 0px; width: 200px; height: 200px; scale-width:100%, scale-height:100% }"
                          )
      							  	)
                        uxActions ArrayList ( callBehavior true
                          insert GesturePanRecognizerNat ( touchesMinNum 1 )
                        )
                        uxControls ArrayList ( callBehavior true
                          insert UxRectRoundNat ( keyString "winBtnMove" styling ".winBtnMove"
                            uxControls UxImageNat ( keyString "winBtnMoveImage"
                              properties HashTable ( insert StringKeyLong ( keyLong  "draggable" ) )
                              image ImageNat ( fileName Vision.Core.String ( withString "list.png" ) )
                            )
                          )
                          insert UxRectRoundNat ( keyString "winBtnResize" styling ".winBtnResize"
                            uxControls UxImageNat ( keyString "winBtnResizeImage" 
                              properties HashTable ( insert StringKeyLong ( keyLong "resizable" ) )
                              image ImageNat ( fileName Vision.Core.String ( withString "resize_0.png" ) )
                            )
                          )
                        )
                      )
                    )
                  )

                  // 2. Update Left
                  insert PropSet ( nameStr "withFloat"
                    part PropDynamicGet ( nameStr "left"
                      part PartHold ( part PropScopeGet ( nameStr "newWindow" ) )
                    )
                    source PropGet ( nameStr "x" part PropScopeGet ( nameStr "position" ) )
                  )

                  // 3. Update Top
                  insert PropSet ( nameStr "withFloat"
                    part PropDynamicGet ( nameStr "top"
                      part PartHold ( part PropScopeGet ( nameStr "newWindow" ) )
                    )
                    source PropGet ( nameStr "y" part PropScopeGet ( nameStr "position" ) )
                  )

                  // 4. Add to control
                  insert PropSet ( nameStr "insert"
                    part PropGet ( nameStr "uxControls" part PropScopeGet ( nameStr "uxViewFocus" ) )
                    source PartHold ( part PropScopeGet ( nameStr "newWindow" ) )
                  )

                  // 5. Get the view to redraw
                  insert PropGet ( nameStr "redraw" part PropScopeGet ( nameStr "uxViewFocus" ) required false )
                )
              )
            )
          )
        )
      )
    )



## Conclusion

Configurations within our framework can be used as Templates allowing us to duplicate any configuration we come up with. In our example, we used a Factory to create new windows that were then added to the 'canvas'.

If you find our work interesting, please follow us [@interfaceVision](http://www.twitter.com/interfaceVision) and/or [@erichosick](http://www.twitter.com/erichosick).

## Next Step

The next step in our goal of creating Interface Vision's Gui based visual development environment is to create a view that can easily display collections (aggregates).

The [prior step]({% post_url 2014-01-23-example-zoom-in-out %}) in our goal of creating Interface Vision's Gui based visual development environment was to allow us to zoom in and out of a view using the pinch gesture.

