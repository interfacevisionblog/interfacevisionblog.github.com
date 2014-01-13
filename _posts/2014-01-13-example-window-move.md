---
layout: post
title: "Configurable Dragging of Gui Controls"
description: "How we got dragging of Gui controls to work in iOS and OSX on a scrollable view."
category: Design
tags: [Composition, Interface Vision, Window, Event, Events, Drag and Drop]
author: Eric Hosick
author_twitter: erichosick
---

## Introduction

This took a little longer to get working than we thought it would. There really are quite a few differences between iOS and OSX. Getting them to work using the same configuration was a bit of a challenge.

In this post, we are going to show how we used our [configurable event system] ({% post_url 2014-01-04-example-events-basic %}) to configure the dragging of controls.


### The Results

<iframe width="746" height="420" src="http://www.youtube.com/embed/hQPRShW7Qfc?vq=hd1080" frameborder="0" allowfullscreen="allowfullscreen"> </iframe>

### The Approach

We want a user to be able to drag controls around on a screen using a button attached to the window.

We will use the Pan Gesture with a single finger touch and apply the following logic:

* At the beginning of the Pan gesture, we set a prior event to the current event (automatically set by the event system). If there is a control at the position of the current event, we set a reference at this step.
* During the processing of the Pan gesture, we check if the current control is marked "draggable". If the control is draggable, we update the view's frame position based on the distance a mouse/finger moved.
* We then update the prior position to the current position (for the next pan gesture).
* During the ending of the Pan gesture, we also update the view's frame position for that last little movement.

### The Complete Configuration

[Configuration-1.1](#id-s1-1-top) contains the complete configuration in [SipCoffee] ({% post_url 2013-12-19-design-composition-based-language %}) for an application that runs on iOS and OSX devices that allows controls to be dragged around in a scrollable canvas. The complete configuration is broken down and explained in detail below.

Please note that this is a complete application, excluding the CSS, that is able to define the entire behavior of a cross platform application with a GUI user experience configured in around 110 lines [1](#id-1). 

<div id='id-s1-1-top'>&nbsp;</div>

    Scope (
      properties HashTable ( // Contains the 'variables' for this Scope.
        insert PartNamedString ( keyString "eventCurrent" ) // current event
        insert PartNamedString ( keyString "eventPrior" ) // prior event
        insert PartNamedString ( keyString "viewFocus" ) // view with focus
        insert PartNamedString ( keyString "controlFocus" ) // control with focus
        insert PartNamedString ( keyString "position" ) // current event position
        insert PartNamedString ( keyString "positionPrior" ) prior event position
        insert CssManager ( ... )
        insert EventManager ( keyString "PropEventManager"
          properties HashTable (
            insert EventMonitor (
              eventToMonitor GesturePan ( eventStep "Begin" touchesMin 1 )
              action ArrayList ( callBehavior true
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
            insert EventMonitor (
              eventToMonitor GesturePan ( eventStep "Processing" touchesMin 1 )
              action ArrayList ( callBehavior true
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
            insert EventMonitor (
              eventToMonitor GesturePan ( eventStep "End" touchesMin 1 )
              action ArrayList ( callBehavior true
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
          )
        )
      )
      part AppNat (
        action UxWindowNat (
          titleStr "Interface Vision"
          frame RectanglefFixed ( x 10 y 10 width 768 height 1024 )
          uxControls UxViewPrimaryNat ( styling ".windowNative"
            statusBarHiddenBool true
            uxControls UxViewScrollable ( styling ".canvas"
              scrollVerticalAllowedBool true
              scrollHorizontalAllowedBool true
              uxControls UxViewDrawNat ( styling ".canvasDoc"
                properties HashTable (
                  insert StringKeyString ( keyString "Css"
                    withString "{ width: 4000px; height: 4000px }"
                  )
                )
                uxControls ArrayList ( callBehavior true
                  insert UxViewDrawNat ( styling ".windowFrame"
                    properties HashTable (
                      insert StringKeyString ( keyString "Css"
                        withString "{ top: 50px; left: 50px; width: 200px; height: 200px }"
                      )
                    )
                    uxActions GesturePanRecognizerNat ( touchesMinVal 1 )
                    uxControls UxRectRoundNat ( styling ".windowButton"
                        uxControls UxImageNat (
                          properties HashTable ( insert StringKeyString ( keyString "draggable" ) )
                          image ImageNat ( fileName Vision.Core.String ( withString "list.png" ) )
                        )
                    )
                  )
                  insert UxViewDrawNat ( styling ".windowFrame"
                    properties HashTable (
                      insert StringKeyString ( keyString "Css"
                        withString "{ top: 50px; left: 270px; width: 200px; height: 200px }"
                      )
                    )
                    uxActions GesturePanRecognizerNat ( touchesMinVal 1 )
                    uxControls UxRectRoundNat ( styling ".windowButton"
                        uxControls UxImageNat (
                          properties HashTable ( insert StringKeyString ( keyString "draggable" ) )
                          image ImageNat ( fileName Vision.Core.String ( withString "list.png" ) )
                        )
                    )
                  )
                )
              )
            )
          )
        )
      )
    )

###### Configuration-1.1: A configured iOS and OSX application allowing controls to be dragged.{#id-s1-1}

There is some redundant configuration code that should be reused: specifically the Processing and End panning events. It is easy to re-use configurations and we'll write a post on this.


### Declaring and Using Variables

#### Traditional Approach

We need to keep some easily accessible information about the status of our dragging of windows. Traditionally, we would declare variables within some scope: a function, a method or passed as parameters. So, we may have something like:

    public class MoveWindow {
      Event eventCurrent = null;
      Event eventPrior = null;
      UxControl controlFocus = null;
      Pos2f position = null;
      Pos2f positionPrior = null;
      
      public void MoveWindow {
        // add code here.
      }
    }

or, maybe we just pass all the information into a function using parameters:

    public void MoveWindow ( Event eventCurrent, Event eventPrior, UxControl controlFocus, Pos2f position, Pos2f positionPrior ) {
      // add code here.
    }
    
Traditionally, we access variables by reading from them and setting them as follows:

    MoveWindow mw = new MoveWindow;
    mw.eventCurrent = // some logic
    mw.eventPrior = null;
    mw.position = eventCurrent.position;

#### How We Declare and Use Variables

Interface Vision is designed to be fully compose-able using Parts. As such, we define scope (see [How We do Scope] ({% post_url 2013-12-31-example-window-basic %}/#id-scope) ) and variables using Parts.

#### Scope Properties

Within our Scope we define variables using different Parts. In this case, we are using the PartNamedString part allowing us to define a part using a string as the variable name. There are also parts like LongKeyLong which allow us to define a long variable with a number as a variable name.

    Scope (
      properties HashTable ( // Contains the 'variables' for this Scope.
        insert PartNamedString ( keyString "eventCurrent" ) // current event
        insert PartNamedString ( keyString "eventPrior" ) // prior event
        insert PartNamedString ( keyString "viewFocus" ) // view with focus
        insert PartNamedString ( keyString "controlFocus" ) // control with focus
        insert PartNamedString ( keyString "position" ) // current event position
        insert PartNamedString ( keyString "positionPrior" ) prior event position
      )
      action TheAction (
        // behavior within the scope
      )
    )

The purpose of the variables is to allow us to drag a control around using Pan gestures.

* eventCurrent - The current event. This is set automatically when setting the property EventManager.insert to a new event.
* eventPrior - This contains the prior event.
* viewFocus - This is the view that had focus when the pan event began.
* controlFocus - This is the control that had focus, if any, when the pan event began.
* position - This is the (x,y) position on the view of the current event. This is a temporary variable used to keep the logic easier. This information is also contained within eventCurrent.
* positionPrior - This is the (x,y) position on the view of the prior event. This is a temporary variable used to keep the logic easier. This information is also contained within eventPrior.

Accessing variables within Interface Vision is also done using Parts [2](#id-2). For reading to and writing from Scope "variables" we have the following:

    PropScopeGet ( nameStr "eventCurrent" required false )

This reads a property defined within the current scope named "eventPrior". required being set to false will stop any warning from being generated if the variable does not exist.

    PropScopeSet ( nameStr "eventPrior" part PropScopeGet ( nameStr "eventCurrent" ) )

This sets a property defined within the current scope named "eventPrior" to the item located in part. In this case, the item located in part is PropScopeGet which, when used, will return a reference to the current event.

This is equivalent to the equal operator: destination = source.

#### Dynamic Properties

A dynamic property is a property that can be defined on any Part instance: even at run time.

    UxImageNat (
      properties HashTable (
        insert StringKeyString ( keyString "draggable" )
      )
    )

In this case, we have added to a UxImageNat Part a dynamic property named "draggable".

To access this dynamic property, we use the PropDynamicGet part:

    PropDynamicGet (
      nameStr "draggable"  // the name of the dynamic property
      part SomePart ( ) // the part that contains the dynamic property we want to get
    )

To set a dynamic property, we use the PropDynamicSet part (not used in our configuration).

#### Instance Properties

To access a property of an instance of a class, we use PropGet.

    PropGet ( nameStr "view"
      part PropScopeGet ( nameStr "eventCurrent" )
    )

In this case, we are getting the property named view of the Part located within the current event. The eventCurrent is actually a Part of type EventGesture which contains a view property.

Traditionally, we would access a property as follows:

    EventGesture someEvent;
    object eventView = someEvent.view; // view is a property of someEvent.

To write to a property of an instance of a class, we use PropSet.

    PropSet ( nameStr "withFloat"
      part PropDynamicGet ( nameStr "top" part PropScopeGet ( nameStr "viewFocus") )
      source PropDynamicGet ( nameStr "top" part PropScopeGet ( nameStr "viewFocus") )
    )

nameStr is the name of the property, part contains the instance that we want to write too. Source contains the value we want to place into the property, in this case, named 'withFloat'.

### Logic and Flow

A goal of Interface Vision is to make our framework really easy to use. If you look at [Configuration-1.1](#id-s1-1-top) you'll note that there really is very little logic and flow defined within the configuration. All the minutia for creating native windows, native view, native images, and allocating memory are "hidden" within our framework. This allows the programmer to focus on the domain specific behavior of the program itself. Programmers don't have to twiddle with the idiosyncrasies languages and frameworks introduce into the development process.

That being said, there are cases where logic and flow needs to be supported at the domain level. For these cases, Interface Vision has Parts to deal with logic and flow.

Controls marked as "draggable" can be dragged. To check if a control is draggable requires some logic and flow within our configuration.

In [Configuration-1.1](#id-s1-1-top) we use When to alter flow and IsNotNil as our Logic.

    When (
      condition IsNotNil ( part PropDynamicGet ( nameStr "draggable" part PropScopeGet ( nameStr "controlFocus" ) required false ) )
      action (
        // do the dragging thing
      )        
    )

The above could be read as "When condition is not nil property 'draggable' part scope 'controlFocus'".

The action is run only if the condition is true: in this case, only if there is a control with focus and it has a dynamic property "draggable".

### Configuring Domain Specific Behavior

In Interface Vision, the position of a control is determined using two dynamic properties named 'top' and 'left' (the same as CSS). To move a control, we need to update these two values.

To do this, we will need to get the current position of our mouse/touch within the current event and subtract it from the prior position of the mouse/touch of the prior event. This gives us the distance the mouse/touch moved between the events. We then add that value to the existing top/left properties of the control.

Let's look at the configuration to adjust the top position of the view that has focus:

    PropSet ( nameStr "withFloat"
      part PropDynamicGet ( nameStr "top" part PropScopeGet ( nameStr "viewFocus") )
      source Add (
        left PropDynamicGet ( nameStr "top" part PropScopeGet ( nameStr "viewFocus") )
        right Subtract (
          left PropGet ( nameStr "y" part PropScopeGet ( nameStr "position") )
          right PropGet ( nameStr "y" part PropScopeGet ( nameStr "positionPrior") )
        )
      )
    )

The Add Part has a left and right property which are the left and right side of the addition operator similar to:

    left + right
    
The value of left is the dynamic property named 'top' of the view that has focus.
The value of right is the result of the subtraction of the y property of the current position and the y property of the prior position.

The result of this addition is placed into the top property of the view that has focus.

Let's look at how this might be implemented traditionally by coding out a function:

    function void MoveWindow ( Control viewFocus, Pos position, Pos positionPrior ) {
        viewFocus("top").withFloat = viewFocus("top").withFloat + ( position.y - positionPrior.y );
    }
    
### Running Behavior in Series

The last item of interest is causing behavior to happen in series. To do this, we can use an ArrayList setting callBehavior to true.

    ArrayList ( callBehavior true
      insert PropScopeSet ( nameStr "eventBegin" source PropScopeGet ( nameStr "eventCurrent" ) )
      insert PropScopeSet ( nameStr "eventPrior" source PropScopeGet ( nameStr "eventCurrent" ) )
      insert PropScopeSet ( nameStr "viewFocus" ... )
    )

## Conclusion

Configuring conditions, logic and behavior within Interface Vision is both easy and consistent. The nasty bits that we usually need to worry about within frameworks are taken care of by the Vision Framework (you compose with Vision, you don't code against it).

Configurations are consistent throughout. The syntax to describe the creation of parts, declare variables, define behavior, define logic and define flow is the exact same!

If you find our work interesting, please follow us [@interfaceVision](http://www.twitter.com/interfaceVision) and/or [@erichosick](http://www.twitter.com/erichosick).

## Next Step

The next step will be to allow us to re-use parts of a configurations.

The [prior step]({% post_url 2014-01-04-example-events-basic %}) was to create a configurable event system.

## Footnotes

{#id-1} 1. The goal of Interface Vision is to be a Gui based visual development environment so, in the bigger picture, the number of lines of configuration required is not that important. It is probably more important to keep the number of parts manageable.

{#id-2} 2. We intend on hiding the process of reading from and writing to properties. As such, within the visual development environment, the user will not be aware of parts like PropGet, PropSet, PropScopeGet, PropScopeSet, etc.

