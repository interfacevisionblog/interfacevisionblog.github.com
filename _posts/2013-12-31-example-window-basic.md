---
layout: post
title: "Example - Basic Windows"
description: "The first window with a unusable button in the Vision framework."
category: Design
tags: [Composition, Interface Vision, Window, Button]
author: Eric Hosick
author_twitter: erichosick
---

## Introduction

[Figure-1.1](#id-f1-1-top) shows a native OSX window, a native IOS window and a "non-native" window contained within each of these native windows.

<div id='id-f1-1-top'>&nbsp;</div>
<p class="pagination-centered"><img class="img-polaroid" src="/assets/img/example_window_basic_1.png"><img></p>
###### Figure-1.1: An OSX native window with a "non-native" window. {#id-f1-1}


[Figure-1.2](#id-f1-2-top) shows the native IOS window after the device has been rotated.

<div id='id-f1-2-top'>&nbsp;</div>
<p class="pagination-centered"><img class="img-polaroid" src="/assets/img/example_window_basic_2.png"><img></p>
###### Figure-1.2: An iOS native window with a "non-native" window. {#id-f1-2}

Let's look at the [SipCoffee]({% post_url 2013-12-19-design-composition-based-language %}) 'code' to create this program which runs in both iOS and OSX natively.

## The CSS

In Interface Vision, the layout of native controls is done using CSS syntax.

[Css-1.1](#id-c1-1-top) contains the css required for our first application.

<div id='id-c1-1-top'>&nbsp;</div>
    body {
      background-color: rgba(255, 255, 255, 1); // default background (white)
      color: rgba(0,0,0,1); // default font color (black)
      font-family: 'Helvetica Neue Medium' ;
      font-size: 20px;
      text-align: left;
      vertical-align: top;
      border-color: rgba(0,0,0,1); // default border color (black)
      direction: ltr; // default direction (left)
    }
    
    // Canvas where windows show up.
    .canvas {
      background-color: #F0F0F0;
    }

    // Window frame for non-native window
    .windowFrame {
      position: fixed;
      background-color: #F0F0F0;
      -moz-border-radius: 10px;
    }
    
    // Buttons that show up on title bar of window
    .windowButton {
      position: fixed;
      background-color: #F7F7F7;
      width: 28px;
      height: 28px;
      padding: 4px;
      margin: 4px;
      -moz-border-radius: 6px;
    }
    

###### Css-1.1: The CSS for our window and windowButton. {#id-c1-1}

## The Configuration

Programs in Interface Vision are composed. Instead of coding, developers hook up parts either visually (we are developing the Gui based development environment now: starting with this post) or via C# code (how we do it now) or using the [SipCoffee]({% post_url 2013-12-19-design-composition-based-language %}) language (the source code in this post).

### Scope

#### About Scope In Languages

Traditionally, scope is defined using scope operators such as { and } or ( and ). Scope takes place in modules, methods, functions, classes and so on.

[Source-1.1](#id-s1-1-top) is an example of a function named foo using { and } as the scope operators. Anything between the { and } is considered within the Scope of the function foo. For example, the variable named localVar is only available within the scope of the function foo.

The parameter name is also only available within the function foo. The stuff between { and } is considered the behavior of the function: the behavior contained within the scope.

<div id='id-s1-1-top'>&nbsp;</div>
    void function foo ( string name ) {
      string localVar = "Hello "
      WriteLine (scoped + " " + name);
    }

###### Source-1.1: Example of defining scope using a function. {#id-s1-1}

{#id-scope}
#### How We do Scope

Interface Vision has no methods, functions or modules. Instead, a Part (called Scope) is used to define the scope of behavior as shown in [Source-1.2](#id-s1-2-top). The Scope class contains a properties property which contains a HashTable. The HashTable can contain one or more Parts with a unique key. For Scope, we use this Unique Key as the name of the property: the name of the variable.

<div id='id-s1-2-top'>&nbsp;</div>
    Scope (
      properties HashTable ( // Contains the 'variables' for this Scope.
        insert CssManager ( keyString "PropCssManager" // Adding a 'variable' named PropCssManager
          ruleSet HashTable (
            insert StringKeyString ( keyString = "default.css"
              withString File ( name = "default.css")
            )
          )
        )
      )
    )

###### Source-1.2: Our scope has a property named "PropCssManager" which contains all the CSS for our application. {#id-s1-2}

Within the Scope, we are inserting a Part with a Unique Key of "PropCssManager". In this case, the Part is a CssManager and we load the Css ruleSet from a file named "default.css" which is defined in [Css-1.1](#id-c1-1-top).

Accessing a property within the Scope is done using a ScopePropGet part we'll explain later.

#### Scope and Threads

Note that Scope is thread aware. This means, if you define a Scope within a Thread part, that Scope is only accessible from the Thread it was defined in. This means you can easily configure thread safe programs.

A single Scope can be defined on the main thread and is considered the 'global' scope.

### The Native Application, Native Window and Native Ux Controls

The AppNat part is the native application. It is contained within a Scope part. Since this Scope runs on the main thread, it is considered the 'global' scope making any items located within the Scope's properties global. This also means that AppNat will run on the main thread.

The action property of AppNat is set to a UxWindowNat Part. This Part does all the work to create a Native window including the shadow around the native window: as seen in [Figure-1.1](#id-f1-1-top).

<div id='id-s1-3-top'>&nbsp;</div>
    Scope (
      properties ( ... ) // see Source-1.2
      part AppNat (
        action UxWindowNat (
          frame RectanglefFixed ( x 10 y 10 width 600 height 600 )
          HasShadow true
          uxControls UxViewPrimaryNat (
            uxControls UxViewScrollable ( styling ".canvas"
              scrollVerticalAllowedBool true
              scrollHorizontalAllowedBool true
              controlBounds RectanglefFixed ( x 0 y 0 width 4000 height 4000 )
              uxControls UxViewDrawNat (
                controlBounds RectanglefFixed ( x 0 y 0 width 4000 height 4000 )
                uxControls UxViewDrawNat ( styling ".windowFrame"
                  properties HashTable (
                    insert StringKeyString ( keyString "Css"
                      withString "{ top: 100px; left: 100px; width: 400px; height: 400px }"
                    )
                  )
                  uxControls UxRectRoundNat ( styling ".windowButton"
                      uxControls UxImageNat (
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

###### Source-1.3: The AppNat part is the native application and is defined within a Scope. {#id-s1-3}

All User Experience controls for the native window are located in the uxControls property. In most Ux systems, the native window contains only one control which is a "view". As such, we plug into the uxControl a UxViewPrimaryNat which is the "glue" between the native window and controls that show up within the native window. This is just an industry standard, and we could have implemented it in other ways.

The UxViewPrimaryNat has a single control added to it of type UxViewScrollable. This is a scrollable view and if we had a video demo of our program, you would see that we could scroll the view vertically and horizontally allowing us to move around within our "canvas". Which bring up an interesting point, because you will note that the styling of the UxViewScrollable is set to ".canvas" which is defined in our Css located in [Css-1.1](#id-c1-1-top). Yep. It's that easy to style any Ux control.

Our UxViewScrollable has a rather large bounds of 4000 by 4000 pixels configured in the controlBounds property.

The final configured part of interest is the UxViewDrawNat with a styling of type ".windowFrame". The positioning, border color, border size, etc. is all defined within the Css. However, the coordinates of this "non-native" window could and do change when defining more than one "non-native" windows. As such, we've "inlined" Css by adding a property to our UxViewDrawNat.

Yep! In general, you can add properties to almost any part within Interface Vision: in this case a property named Css.

## The Entire Program

Our entire program fits in just 39 lines of [SipCoffee]({% post_url 2013-12-19-design-composition-based-language %}) code, 24 if you don't include lines of ), and 33 lines of CSS.

That defines everything required to have a native cross platform Gui based application that runs on both OSX and iOS (oh, and that supports rotation of an iOS device too)!

<div id='id-s1-5-top'>&nbsp;</div>
    Scope (
      properties HashTable ( // Contains the 'variables' for this Scope.
        insert CssManager ( keyString "PropCssManager" // Adding a 'variable' named PropCssManager
          ruleSet HashTable (
            insert StringKeyString ( keyString = "default.css"
              withString File ( name = "default.css")
            )
          )
        )
      )
      part AppNat (
        action UxWindowNat (
          frame RectanglefFixed ( x 10 y 10 width 600 height 600 )
          HasShadow true
          uxControls UxViewPrimaryNat (
            uxControls UxViewScrollable ( styling ".canvas"
              scrollVerticalAllowedBool true
              scrollHorizontalAllowedBool true
              controlBounds RectanglefFixed ( x 0 y 0 width 4000 height 4000 )
              uxControls UxViewDrawNat (
                controlBounds RectanglefFixed ( x 0 y 0 width 4000 height 4000 )
                uxControls UxViewDrawNat ( styling ".windowFrame"
                  properties HashTable (
                    insert StringKeyString ( keyString "Css"
                      withString "{ top: 100px; left: 100px; width: 400px; height: 400px }"
                    )
                  )
                  uxControls UxRectRoundNat ( styling ".windowButton"
                    uxControls UxImageNat (
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

###### Source-1.5: The entire program to show a Native Window in iOS and OSX. {#id-s1-5}

## Conclusion

Interface Vision's [SipCoffee]({% post_url 2013-12-19-design-composition-based-language %}) language is able to configure complex cross platform applications using 100% composition. Our next example will extend our existing program by allowing us to move the "non-native" window within our "canvas" (the Scrollable view).

If you find our work interesting, please follow us [@interfaceVision](http://www.twitter.com/interfaceVision) and/or [@erichosick](http://www.twitter.com/erichosick).

## Next Step

The [next step]({% post_url 2014-01-04-example-events-basic %}) is to create a configurable event system. We were planning on creating the configuration to allow us to move around our "non-native" window but we got ahead of ourselves.

The [prior step] ({% post_url 2013-12-19-design-composition-based-language %}) was to create a language based on composition.
