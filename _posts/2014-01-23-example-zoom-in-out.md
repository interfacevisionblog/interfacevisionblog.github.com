---
layout: post
title: "Zooming And Scaling"
description: "Using the pinch gesture in IOS and OSX to zoom in and out on controls."
category: Design
tags: [Code re-use, Interface Vision, resize]
author: Eric Hosick
author_twitter: erichosick
---

## Introduction

Visually representing an entire program could start looking really cluttered. A general rule of design is to keep the number of elements in a group to around 5 +/- 2 [1](#id-1).

A program may require hundreds of elements. Our plan is to allow our users to zoom-in and zoom-out on different parts of their program: seeing more detail as they zoom in and less as they zoom out.

Imagine Google maps, but instead of seeing more detail about roads as you zoom in, you see more details about your program.

So, we needed to add zoom to our demo.

### The Results

<iframe width="746" height="420" src="http://www.youtube.com/embed/eiB8Vpltouc?vq=hd1080" frameborder="0" allowfullscreen="allowfullscreen"> </iframe>

## IOS vs OSX Drawing

### Coordinate Systems

IOS and OSX use different [coordinate systems](https://developer.apple.com/library/ios/documentation/general/conceptual/Devpedia-CocoaApp/CoordinateSystem.html). For OSX, (0,0) defaults in the lower left hand corner. For iOS, Android and Windows, (0,0) is in the upper left hand corner. We can not "flip" iOS but we can "flip" OSX [2](#id-2).

We are bringing up this difference between the coordinate system because, throughout the vision framework, we need to take into account OSX's inverted coordinate system (we want to encapsulate this type of minutia within our development framework so our users can focus on making software).

[Source-1.1](#id-s1-1-top) contains the code necessary to flip the OSX coordinate system.

<div id='id-s1-1-top'>&nbsp;</div>
    using System;
    using System.Xml.Serialization;

    namespace Vision.Ux.Gui {

      [Register("ViewDrawable")] public class ViewDrawable : MonoMac.AppKit.NSView {

        /// Other Code 

        /// <summary>
        /// True causes the coordinate system to flip with regards to native coordinate system.
        /// False keeps the operating system's native coordinate system to be used.
        /// </summary>
        public override bool IsFlipped {
          get { return true; }
        }

      }
    }
###### Source-1.1: We need to flip the drawing coordinates for OSX. {#id-s1-1}

Without overriding IsFlipped and returning true, the output is "flipped" as shown in [Figure-1.1](#id-f1-1-top).

<div id='id-f1-1-top'>&nbsp;</div>

<p class="pagination-centered"><img class="img-polaroid" src="/assets/img/example_zoom_osx_flipped.png"><img></p>

###### Figure-1.1: For OSX, (0,0) is in the lower left hand corner. {#id-s1-1}

### Events

Interface Vision has it's own [event system]({% post_url 2014-01-04-example-events-basic %}) which we need to send all operating system native events to. The coordinate provided in NSEvent of OSX is based on (0,0) being in the lower left hand corner. Overriding IsFlipped and setting it to true only flips the drawing system. It does not 'flip' the native iOS event system.

[Source-1.2](#id-s1-2-top) contains the code required to support pinch in OSX.

<div id='id-s1-2-top'>&nbsp;</div>
    using System;
    using System.Xml.Serialization;

    namespace Vision.Ux.Gui {

      [Register("ViewDrawable")] public class ViewDrawable : MonoMac.AppKit.NSView {

        /// Other Code 

        public override void MagnifyWithEvent (NSEvent theEvent) {
          // base.MagnifyWithEvent (theEvent); // We will consume the event here.
          NSView parentViewNative = view.parentView.adaptedPart as NSView;
          if ( null != parentViewNative ) {
            // IsFlipped = true (see below) but pos of event is not flipped so we need to flip it
            Pos2fNat posInViewFound = new Pos2fNat {
              native = this.ConvertPointFromView (theEvent.LocationInWindow, null)
              }; 
            Pos2fNat posInWindowFound = new Pos2fNat {
              native = parentViewNative.ConvertPointFromView (theEvent.LocationInWindow, null)
              };
            eventManager.insert = new GesturePinch {
              eventStep = Event.EventStepProcessing,
              posInView = posInViewFound,
              posInWindow = posInWindowFound,
              view = this.view,
              scale = (1 + theEvent.Magnification),
              velocity = theEvent.Magnification };
          }
        }

      }
    }
###### Source-1.2: We need to flip the drawing coordinates for OSX. {#id-s1-2}

The value of theEvent.LocationInWindow zooming in the upper left part of the screen returns something like:

    ( X=37.41797, Y=945.9766 )

We need to use the following code to "flip" the Location sent by the event:

    ConvertPointFromView (theEvent.LocationInWindow, null)

which gives us:

    ( X=37.41797, Y=56.02344 )
    
The ConvertPointFromView also converts the absolute position of the event location to the logical position within the view.

The [Pinch Gesture](https://developer.apple.com/library/ios/documentation/UIKit/Reference/UIPinchGestureRecognizer_Class/Reference/Reference.html) in iOS contains both a scale and velocity value.

The UIPinchGestureRecognizer.Velocity value in iOS is similar to the NSEvent.Magnification value. However, iOS seems to be about 100 times more sensitive than OXS in it's velocity value.

An equivalent scale/magnification between iOS and OSX is:

    (UIPinchGestureRecognizer.Velocity/100 + 1) ≈ NSEvent.Magnification

Just for your enjoyment, [Source-1.3](#id-s1-3-top) shows the code necessary to turn an iOS gesture event into an Interface Vision event with the velocity adjusted:

<div id='id-s1-3-top'>&nbsp;</div>
    using System;
    using System.Xml.Serialization;
    using MonoTouch.UIKit;
    using MonoTouch.Foundation;
    using MonoTouch.ObjCRuntime;
    using MonoTouch.CoreGraphics;

    namespace Vision.Ux {

      [Serializable()] public class GesturePinchRecognizer : GesturePinch {

        protected void gestureAction (UIPinchGestureRecognizer theEvent ) {
          if ( null == theEvent ) {
            UIView parentViewNative = view.parentView.adaptedPart as UIView;
            if (null != parentViewNative) {
              Pos2fNat posInViewFound = new Pos2fNat {
                native = theEvent.LocationInView(theEvent.View)
              };
              Pos2fNat posInWindowFound = new Pos2fNat {
                native = theEvent.LocationInView (parentViewNative)
              };
              eventManager.insert = new GesturePinch {
                eventStep = eventStepFound,
                posInView = posInViewFound,
                posInWindow = posInWindowFound,
                view = this.view,
                scale = theEvent.Scale,
                velocity = theEvent.Velocity/100 + 1,
                };
            }
          }
        }

        protected bool initialized = false;
        [XmlIgnore] public override IPart withPart {
          get {
            if (!initialized) {
              initialized = true;

              UIView view = parentView.adaptedPart as UIView;
              if (null != view) {
                UIPinchGestureRecognizer gesture = new UIPinchGestureRecognizer (gestureAction);
                view.AddGestureRecognizer (gesture);
              }
            }
            return base.withPart;
          }
        }
      }
    }
###### Source-1.3: Converting iOS events into Vision events.{#id-s1-3}

In this case, we adjust the velocity provided by iOS so it is close to the magnification value of OSX.

We are also still using theEvent.LocationInView but not for flipping the coordinate system. In this case, we only need it to convert the absolute position of the event location to the logical position within the view.

### Scaling and Zooming Controls

There are a few approaches to scale controls:

#### NSView.ScaleUnitSquareToSize

This is only available in OS X.

Use:

    NSView.ScaleUnitSquareToSize (scaleWidth, scaleHeight).

Although this led to a quick solution initially, there were a few issues. Primarily, and this is also the case for Transformations, everything within the view is scaled. If you want to, for example create resize handles, those controls would also scale.

Resources and information we found:

* [GCZoomView](http://apptree.net/gczoomview.htm) - "GCZoomView is a simple NSView subclass that adds a set of standard zoom commands, such as Zoom In, Zoom Out, Zoom To Fit, Zoom to any arbitrary scale, etc."
* [Discussion](http://listarc.com/showthread.php?1574653-scaleUnitSquareToSize) - "Yes. Rather than using bounds scaling and leaving your drawables oblivious to zoom, it is commonly helpful to instead build your architecture to support drawing zoomed into a on unscaled bounds coordinate system. Often you'll want to draw things in unscaled units (resize handles, selection loops, and the like)."
* [Discussion 2] (https://groups.google.com/forum/#!msg/cocoa-dev/oprMTlUg4-A/FbJjnpCY4LoJ) - Some example code on zooming and keeping the zoom positioned relative to the mouse/finger coordinates.

#### Use Transformations

Available for both iOS and OSX.

    NSView p_view = new NSView ();
    p_view.WantsLayer = true; // FOR OSX
    p_view.Layer.Transform = CATransform3D.MakeScale(scaleWidthFound, scaleHeightFound, 1.0f);

This can lead to fuzzy looking images still requiring some custom logic. However, it does work for both iOS and OSX. This also has the issue of scaling controls like resize handles.

#### Roll Out A Custom Solution

We wanted to simply use one of the zooming solutions provided by either iOS or OSX. These work great if you are trying to zoom a single entity (say an image). But, when you need to support zooming of a hierarchy of controls, this just doesn't seem to cut it.

So, we've rolled out our own solution. This isn't as crazy as it sounds. We had an existing CSS Layout Engine that had hundreds of tests. We added more tests building out scaling as part of the layout engine.

### The Configuration - Scaling

We needed to change our configuration to support zoom through the pinch gesture by scaling our controls. Scaling the controls had an affect on our existing move and resize code. When resized or moved, controls scaled by a factor of two would visually move twice as fast as the mouse pointer/finger.

Even more interesting is that we support scaling within scaling. So, a parent view may be zoomed at 1.5 times while the child view is zoomed at 1.25 times. This means that the controls in the child view need to resize based on the scale factors of all parent controls.

In the [resizing windows]({% post_url 2014-01-16-example-resize-window %}/#position-delta-config) post, we had refactored our code so the configuration to move and resize controls was re-usable. We need to update this configuration so it also takes into account scaling.

Let's first update our shared delta configuration to adjust for the scaling.

the left of the window:

    PartNamedString ( nameStr "posDeltaX"
      part Divide (
        left Add (
          left PropDynamicGet ( nameStr "left" part PropScopeGet ( nameStr "viewFocus") )
          right Subtract (
            left PropGet ( nameStr "x" part PropScopeGet ( nameStr "position") )
            right PropGet ( nameStr "x" part PropScopeGet ( nameStr "positionPrior") )
          )
        )
        right PropScopeGet ( nameStr "curScaleWidth" )
      )
    )

the top of the window.

    PartNamedString ( nameStr "posDeltaY"
      part Divide (
        left Add (
          left PropDynamicGet ( nameStr "top" part PropScopeGet ( nameStr "viewFocus") )
          right Subtract (
            left PropGet ( nameStr "y" part PropScopeGet ( nameStr "position") )
            right PropGet ( nameStr "y" part PropScopeGet ( nameStr "positionPrior") )
          )
        )
        right PropScopeGet ( nameStr "curScaleHeight" )
      )
    )

Basically, we divide our delta by the scale. We get the current scaled width and height from the scope properties named curScaleWidth and curScaleHeight.

Now all we need to do is figure out how to use the correct scales based on a move or resize action.

So, how do we define the curScaleWidth and curScaleHeight properties?

#### Where Interface Vision Really Shines

In our [resizing windows]({% post_url 2014-01-16-example-resize-window %}/#shared-resize-move), we were able to share the configuration to resize and move windows by nesting Scope.

The behavior to calculate the deltaX and deltaY needs to be altered based on action being taken: move/resize or zoom.

We add to this nested scope the curScaleWidth and curScaleHeight properties.

For moving and resizing scaleHeightCalc and scaleWidthCalc are added:

    insert Scope (
      properties HashTable (
        insert StringKeyString ( keyString "scopeHeight" withString = "top" )
        insert StringKeyString ( keyString "scopeWidth" withString = "left" )
				insert PartNamedString ( keyString "curScaleHeight"
          part PropGet ( nameStr "scaleHeightCalc"
            part = new PropGet ( nameStr = "parentUxControl" part PropScopeGet ( nameStr "uxViewFocus" ) )
          )
        )
        insert PartNamedString ( keyString "curScaleHeight"
          part PropGet ( nameStr "scaleWidthCalc"
            part = new PropGet ( nameStr = "parentUxControl" part PropScopeGet ( nameStr "uxViewFocus" ) )
          )
        )
      )
      part PropScopeGet ( nameStr "viewResMov" )
    )

For zooming  scaleHeightCalc and scaleWidthCalc are different:

    insert Scope (
      properties HashTable (
        insert StringKeyString ( keyString "scopeHeight" withString = "top" )
        insert StringKeyString ( keyString "scopeWidth" withString = "left" )
        insert PartNamedString ( keyString "curScaleHeight"
          part PropGet { nameStr = "scaleHeightCalc" part PropScopeGet ( nameStr "uxViewFocus" ) )
        )
				insert PartNamedString ( keyString "curScaleWidth"
          part PropGet { nameStr = "scaleWidthCalc" part PropScopeGet ( nameStr "uxViewFocus" ) )
        )
      )
      part PropScopeGet ( nameStr "viewResMov" )
    )

This may sound a bit too forward, but it is the ability for Interface Vision to do what we just did that makes it just so amazing as a technology and framework.

The behavior for moving and scaling are different but there are no conditions to choose which behavior to use specific to scaling. We are able to alter the behavior of posDeltaX and posDeltaY by simply configuring different behavior in the curScaleHeight and curScaleWidth scope properties. From the perspective of deltaX and deltaY, the property is just a property with some value. The actual behavior used to calculate the value returned by the property is 'unknowingly' used by the deltaX and deltaY configuration.

Also cool to note, configuration for supporting scaling required no additional conditions: just adjustments to existing configurations. It is advantageous to lower the number of conditions in a program for different reasons (less logic to test, easier to read, etc.). In our case, it is greatly advantageous because it greatly simplifies the visual representation of programs.

### The Configuration - Zooming

Finally, we are ready to zoom-in and out using the pinch gesture.

The configuration of the events within the event manager are very similar to the configuration of panning and resizing.

    EventManager (
      // ... 
      insert EventMonitor (
        eventToMonito GesturePinch ( eventStep "Begin" )
        action PropScopeGet ( nameStr "behaviorPinch" )
      )
      insert EventMonitor (
        eventToMonito GesturePinch ( eventStep "Processing" )
        action PropScopeGet ( nameStr "behaviorPinch" )
      )
      insert EventMonitor (
        eventToMonito GesturePinch ( eventStep "Finished" )
        action PropScopeGet ( nameStr "behaviorPinch" )
      )
    )

Our action, in all three cases, causes the behavior named 'behaviorPinch' to run. We don't need to really worry about the beginning, processing and finished steps of the event.

The configuration (named behaviorPinch) to handle the actual scaling is also similar to the pan behavior within our global scope.

    Scope (
      // ... Other configurations
      insert PartNamedString ( keyString "behaviorPinch"
        part ArrayList ( callBehavior true
          insert PropScopeSet ( nameStr "uxViewFocus"
            source PropGet ( nameStr "view"
              part PropScopeGet ( nameStr "eventCurrent" )
            )
          )
          insert PropSet ( nameStr "withFloat"
            part PropDynamicGet ( nameStr "scale-height"
              part PropScopeGet ( nameStr "uxViewFocus" )
            )
            source = new Multiply (
              left PropDynamicGet ( nameStr "scale-height"
                part PropScopeGet ( nameStr "uxViewFocus" )
              ) 
              right PropGet ( nameStr "velocity"
                part PropScopeGet ( nameStr "eventCurrent" )
              )
            )
          )
          insert PropSet ( nameStr "withFloat"
            part PropDynamicGet ( nameStr "scale-width"
              part PropScopeGet ( nameStr "uxViewFocus" )
            )
            source = new Multiply (
              left PropDynamicGet ( nameStr "scale-width"
                part PropScopeGet ( nameStr "uxViewFocus" )
              ) 
              right PropGet ( nameStr "velocity"
                part PropScopeGet ( nameStr "eventCurrent" )
              )
            )
          )
        )
      )
    )

In this case, we are adjusting the scale-width and scale-height dynamic-properties of the view that has focus:

    PropDynamicGet ( nameStr "scale-width" part PropScopeGet ( nameStr "uxViewFocus" ) )


These values are used within the CSS Layout engine to scale the final values for width, height, left, top of controls (it also scales margin, border and padding values). There is actually no CSS property for scaling in the official CSS3 standards making the layout engine non-compatible with CSS3.

## Conclusion

Although we wanted to use scaling solutions provided by iOS and OSX, we just couldn't make it work with the features we needed for our customers. That isn't to say we won't be enabling our users to use iOS and OSX's Transformation features within their configurations. That will be provided.

The solution worked really well by updating the CssLayoutEngine to support scaling.

If you find our work interesting, please follow us [@interfaceVision](http://www.twitter.com/interfaceVision) and/or [@erichosick](http://www.twitter.com/erichosick).

## Next Step

The [next step]({% post_url 2014-01-28-example-templates-and-factories %}) in our goal of creating Interface Vision's Gui based visual development environment is to dynamically add controls by using factories (instead of 'hard coding' our configuration).

The [prior step]({% post_url 2014-01-16-example-resize-window %}) was to do further refactoring and allow resizing of windows.

## Footnotes

{#id-1} 1. You may want to checkout this interesting article on [The Magical Number Seven Plus Or Minus Two](http://en.wikipedia.org/wiki/The_Magical_Number_Seven,_Plus_or_Minus_Two).

{#id-2} 2. This isn't exactly true. It is possible to use transformation matrices to "flip" the layer we are drawing on using Apple's [Core Animation framework](https://developer.apple.com/library/ios/documentation/Cocoa/Reference/CoreAnimation_functions/Reference/reference.html).



