---
layout: post
title: "Configurable Event System"
description: "An event system that is fully configurable."
category: Design
tags: [Composition, Interface Vision, Window, Event, Events, Drag and Drop]
author: Eric Hosick
author_twitter: erichosick
---

## Introduction

Most, if not all, frameworks with some kind of User Experience (UX) have some sort of event system to handle events generated from mice, keyboards and other devices. Our goal is to create an event system that can be fully configured.

### Traditional Approaches

[Source-1.1](#id-s1-1-top) shows one way developers handle events in Ux frameworks. In this case, we've inherited from a class NSWindow provided by [MonoMac](http://www.mono-project.com/MonoMac) and override the method MouseDown.

<div id='id-s1-1-top'>&nbsp;</div>
    public partial class MainWindow : NSWindow {
      
      // other code
      
      public override void MouseDown (NSEvent theEvent) {
        base.MouseDown (theEvent);
        // custom code to handle MouseDown event
      }
      
      // other code
  		
    }

###### Source-1.1: Traditional approach to handling events using C# MonoMac. {#id-s1-1}


[Source-1.2](#id-s1-2-top) shows another way developers handle events in Ux frameworks. In this case, we use delegates and "compose" the handling of the event within a UIButton, provided by [MonoTouch](http://xamarin.com/ios), and override the method MouseDown.

<div id='id-s1-2-top'>&nbsp;</div>
    // other code
    
    UIButton button = UIButton.FromType(UIButtonType.RoundedRect);
    button.TouchDown += delegate(object sender, EventArgs e) {
      // custom code to handle Button pressed event
    };

    // other code

###### Source-1.2: Traditional approach to handling events using C# MonoTouch. {#id-s1-2}

[Source-1.3](#id-s1-3-top) shows another way developers handle events in Ux frameworks. In this case, we register a recognizer with a Ux Control, in this case a UIWindow provided by [MonoTouch](http://xamarin.com/ios), that supports Panning with one or two fingers.

<div id='id-s1-3-top'>&nbsp;</div>
    public partial class MainWindow : NSWindow {
      
      // other code

      protected void gestureAction (UIPanGestureRecognizer theAction ) {
        // custom code to handle gesture
      }

      MainWindow() {
        UIPanGestureRecognizer gesture = new UIPanGestureRecognizer (gestureAction);
        gesture.MinimumNumberOfTouches = 1;
        gesture.MaximumNumberOfTouches = 2;
        this.AddGestureRecognizer (gesture);
      }
      
      // other code
  		
    }

###### Source-1.3: Traditional approach to handling events using C# MonoTouch. {#id-s1-3}

The recognizer is "attached" to a protected method gestureAction which contains the custom code to handle the gesture.

## Event Handling in Interface Vision

We want to provide a consistent way to compose the definition of and handling of events. Not only does it need to be consistent, but it needs to be operating system agnostic.

We will need to "wrap" the traditional approach to handling events and forward those events to our event handling system described in detail below.

## Our Design

### Events

We want to be able to handle events at different levels:

* Cloud "level" (Across multiple devices - planned)
* Global to OS (Maybe)
* Active Application
* Active Window
* Active View

An event can occur in one of the following steps:

* EventStepEmpty - The "empty" step. No step has been defined for the event.
* EventStepBegin - The start/beginning of an event (Pan, Press, keyPressDown, etc.)
* EventStepProcessing - The event is still running (Drag/Drop, IsPanning, etc.)
* EventStepFinished - The event finished (keyPressUp, finger released from screen, etc.)
* EventStepCancelled - The event was cancelled during the processing phase.

An even has a source. Sources can be native to the operating system such as (few examples):

* EventSourceWinNatResize - Native window was resized.
* EventSourceWinNatMove - Native window was moved.
* EventSourceWinNatScreenFullExit - Native window exited full screen
* EventSourceWinNatScreenFullEnter - Native window entered full screen

Sources can be gestures such as (few examples):

* EventSourceGesturePan - The source of the even was a pan gesture.
* EventSourceGesturePress - The source of the even was a press gesture.
* EventSourceGestureRotation - The source of the even was a rotation gesture.
* EventSourceGestureSwipe - The source of the even was a swipe gesture.
* EventSourceGestureTap - The source of the even was a tap gesture.

Sources can be from native physical devices such as (few examples):

* EventSourceKeyboard - The source was a keyboard
* EventSourceMouse - The source was a mouse

A specific Event my require more information. For example, a GesturePan event may require the number of touches required for the event to "fire". A GesturePress event may require the number of touches and the touch duration for the event to "fire".

### Unique Events with the Event Part

We will need a way to register events, monitor for events and "fire" those events (run the behavior configured for a given event). Let's first see how we define a unique event to monitor.

Currently, we identify an event by creating a unique hash code for the event.

    public class Event : Part, IEvent {

      public override object keyHashCode {
        get { return this.keyString; }
      }

      public override string keyString {
        get { return eventSource + eventStep; }
      }

      public string eventSource = "SourceEmpty";
      public string eventStep = "StepEmpty";

      }
    }

In Interface Vision, the Part class has a keyHashCode and keyString properties. We override these properties and return "eventSource + eventStep" as the hashcode. An event instance can now be placed in a HashTable part for quick lookup.

A GesturePan event has more information provided, the touchesMin, which is used to generate a unique hash code.

    public class GesturePan : Event {

      public override string keyString {
        get { return base.keyString + "Touch"+touchesMin.ToString(); }
      }

      public override string eventSource {
        get { return "GesturePan"; }
      }
		
      public int touchesMin = 0;

    }

The unique hash code for a GesturePan with 2 for touchesMin with two fingers would be "GesturePanStepEmptyTouch2" [1](#id-1).

### Attaching an Action to an Event with EventMonitor

When an event "fires" we need to attach a behavior to that event which is what the EventMonitor part does.

    public class EventMonitor : PartParent, IEventMonitor {

      public override object keyHashCode {
        get { return keyString; }
      }

      public override string keyString {
        get { return eventToMonitor.keyString; }
      }
		
      public IPart eventToMonitor { get; set; }
      public IPart action { get; set; }

      public override IPart withPart {
        get { return action.withPart; }
      }

    }

The hash code for the EventMonitor part will always be the same as the event it monitors (see the keyString property). The property withPart of our EventMonitor simply calls withPart of the Part located within the action property (see [Our Technology]({% post_url 2013-11-13-design-our-technology %}) to understand the trickery behind the withPart property).

### The Configuration using the EventManager part

Let's define a few events to monitor. Let's build on top of the configuration we had in our [prior step] ({% post_url 2013-12-31-example-window-basic %}) - a program to display a native window.

We add to our Scope an EventManager as follows:

    Scope (
      properties HashTable ( // Contains the 'variables' for this Scope.
        insert CssManager ( ... )
        insert EventManager ( keyString "PropEventManager"
          properties HashTable (
            insert EventMonitor (
              eventToMonitor GesturePan ( eventStep "Begin" touchesMin 1 )
              action ConsoleWriteLine ( text "GesturePan - EventStep Begin - One Finger")
            )
            insert EventMonitor (
              eventToMonitor GesturePan ( eventStep "Processing" touchesMin 1 )
              action ConsoleWriteLine ( text "GesturePan - EventStep Processing - One Finger")
            )
            insert EventMonitor (
              eventToMonitor GesturePan ( eventStep "End" touchesMin 1 )
              action ConsoleWriteLine ( text "GesturePan - EventStep End - One Finger")
            )
          )
        )
      )
      part AppNat (
        ...
      )
    )

We've now configured our application to watch for panning using a single finger for the Begin, Processing and End event steps. In this case, we will write to the console text based on which event is fired.

Note that we can have more than one action taken by plugging in an ArrayList part for the action.

## Hooking Up The "Old" With The New

Hooking up the event systems with the IOS and OSX frameworks is an interesting challenge.

### Standardizing on Gestures and Events

A gesture on a touch device or screen is a bit different from a Gestures created using something like a [TrackPad] (http://www.apple.com/magictrackpad/) or Mouse.

Let's consider Panning. On a gesture device, the first time the user touches the screen and moves their fingers, we know that the beginning of a Gesture has occurred and that we are processing the Gesture. When the user releases their finger, we are at the end of the pan gesture.

However, with a TrackPad or mouse, we should only consider the Gesture to be panning when the user has pressed down on the Trackpad or one of the mouse buttons. We can track the position of a mouse as the user moves it but we can't track where a user is going to put their finger on the screen of a touch screen device until they have done so.

A similar issue comes up with a trackpad where the user can move the cursor on the screen without actually pressing down on the trackpad. This lets the user position the cursor on the screen without "pressing": similar to the mouse.

So, we will need to standardize how we represent gestures between different devices like touchable devices, mice and trackpads [2](#id-2).

### Hooking Up IOS

Let's hook up the Interface Vision Event System with the iOS event system. [Source-1.3](#id-s1-3-top) shows how iOS's event system works. Let's generalize it by creating a part to support Panning.

    public class GesturePanRecognizer : GesturePan {

      protected void gestureAction (UIPanGestureRecognizer theAction ) {
        eventManager.insert = new GesturePanEvent { eventStep = eventStep, theAction = theAction, touchesMinVal = this.touchesMinVal };
      }

      [XmlIgnore] public override IPart withPart {
        get {
          UIPanGestureRecognizer gesture = new UIPanGestureRecognizer (gestureAction);
          gesture.MinimumNumberOfTouches = (uint)this.touchesMinVal;
          IPart window = parentLogical;
          window.AddGestureRecognizer (gesture);
          return base.withPart;
        }
      }
    }

The GesturePanRecognizer inherits from GesturePan which is an Event (if you recall). In this way, we can create a UIPanGestureRecognizer based on properties unique to the GesturePan such as touchesMinVal.

The property withPart of our GesturePanRecognizer creates an instance of the UIPanGestureRecognizer and adds it to the window (see [Our Technology]({% post_url 2013-11-13-design-our-technology %}) to understand the trickery behind the withPart property).

What is interesting to note is that the window is found by calling parentLogical which means we need to configure our Recognizer "under" our MainWindow. Let's see what this looks like in SipCoffee.


    Scope (
      ...
      part AppNat (
        action UxWindowNat (
          uxActions ArrayList (
            insert GesturePanRecognizer ( touchesMin 1 )
          )
          ...
        )
      )
    )

We've configured the GesturePanRecognizerNat under our UxWindowNat Part within the uxActions property. We are now ready to receive any single touch pan events in iOS.


### Hooking Up OSX

OSX is a little more difficult than iOS because OSX does not provide gesture recognizer classes. Instead, we are going to have to "hard code" our idea of a pan gesture into OSX. This is where we have to make that decision on what it means to "pan" in OSX.

We've decided that a single finger press on the TrackPad or the left mouse button on a mouse should generate a pan gesture with a touchesMin of one. Let's look at the code to do this.

The implementation of the GesturePanRecognizer in OSX is an empty class (it does nothing) [3](#id-3). Instead, we have to override methods in NSWindow to get the same results.

    public class MainWindow : NSWindow {

      public IPart eventManager {
        get { return new EventManagerGet{ }.withPart; }
      }

      protected bool p_draggingTouchOne = false;
      public override void MouseDragged (NSEvent theEvent) {
        base.MouseDragged (theEvent);
        if (false == p_draggingTouchOne) {
          p_draggingTouchOne = true;
          eventManager.insert = new GesturePanEvent {
            eventStep = Event.EventStepBegin,
            touchesMinVal = 1,
          };
        }
        eventManager.insert = new GesturePanEvent {
          eventStep = Event.EventStepProcessing,
          touchesMinVal = 1,
        };
      }

      public override void MouseUp (NSEvent theEvent) {
        base.MouseUp (theEvent);
        if (true == p_draggingTouchOne) {
          p_draggingTouchOne = false;
          eventManager.insert = new GesturePanEvent {
            eventStep = Event.EventStepFinished,
            touchesMinVal = 1,
          };
        }
      }

    }

For OSX, we override the MouseUp and MouseDragged events on NSWindow and generate the different GesturePanEvent's from within these methods using a flag letting us know if we were "dragging".

### EventManager and Handling "Triggered" Events

The line of code "eventManager.insert = new GesturePanEvent { ... };" seems to cause the event to trigger. Let's see the code for that in the EventManager.

    public class EventManager : PartProperties, IEventManager {

      public override IPart insert {
        set {
          IPart eventFound = properties [value.keyHashCode];

          if ( !eventFound.isEmpty ) {
            IPart ignore = eventFound.withPart;
          }
        }
      }
    }

Oh wow! There will be a lot more to this later, but for now that is all we need to do to trigger the Event. We use the Hash Code of the event provided (value.keyHashCode) to find the event which is located within the properties of the EventManager.

We then access the withPart property of the eventFound. Remember, from above, that eventFound will contain an EventMonitor part. The withPart property of EventMonitor calls withPart on the action property. This contains, in our case, ConsoleWriteLine. The property withPart of ConsoleWriteLine causes text to be written to the console.

That's it!

Something to note is that the EventManager has no reference to the Event part, the EventMonitor part or even the GesturePanEvent part. The Event handling system is 100% decoupled from the rest of the event system allowing people to build any kind of event and throw it at the event handling system.

## Conclusion

Configuring events within Interface is both easy and consistent. There is no need to code out events nor do you have to deal with all the gory details on how to define events within any given framework or operating system.

If you find our work interesting, please follow us [@interfaceVision](http://www.twitter.com/interfaceVision) and/or [@erichosick](http://www.twitter.com/erichosick).

## Next Step

The [next step]({% post_url 2014-01-13-example-window-move %}) will be to create a configuration to allow us to move around our "non-native" window.
The [prior step] ({% post_url 2013-12-31-example-window-basic %}) was to get a program to display a native window using SipCoffee.

## Footnotes

{#id-1} 1. We will optimize the hash code at a later time so this is not an issue. However, a real draw back to this approach is that we can not define events with ranges. For example, we can't fire on all Pans with 1 to 3 fingers.

{#id-2} 2. At some point, we may make this configurable across OSX, Windows, IOS, Android, etc. However, for the time being, we've "hard coded" it into our framework.

{#id-3} 3. We need to leave the part within our SipCoffee configuration because we use the same configuration between different operating systems: in this case iOS and OSX. That's why, for OSX, we have recognizer parts but they are empty.