---
layout: post
title: "Why We Don't Need Overloading"
description: "Overloading is not longer required because we use parts that can operate on all primitives."
category: Design
tags: [Overloading, Method Overloading, Primitive, subroutines]
author: Eric Hosick
author_twitter: erichosick
---

# Why We Don't Use Overloading

## Introduction

We don't use overloading because our framework has no subroutines to overload. This is because the rules of Simple Interface Programming (SIP) don't allow subroutines with explicit parameters.

Instead of overloading, we use Parts that are able to operate on all primitives. Currently, we have chosen Part (aka object), string, boolean, integer, long, float, double, byte array, int array, long array and float array as the primitives we support.

## Example of Traditional Overloading

The following is how, traditionally, the function add would be defined:

    ...
    int add (int left, int right ) { return left + right }
    long add (long left, int right) { return left = right }
    long add (long left, long right) { return left + right }
    ...

and so on. The left and right parameters for add vary based on the primitives we need to add.

## The "With" Properties

Our Part part use "with" Properties: each "with" property being of a different primitive data type. Here is the pseudo-code for our Part part (note we are only showing withPart, withLong and withFloat).

    [Serializable] public class Part : IPart {
      ...
      [XmlIgnore] public virtual IPart   withPart { get; set; }
      [XmlIgnore] public virtual int    withLong { get; set; }
      [XmlIgnore] public virtual float   withFloat { get; set; }
      ...
    }

By default, these properties do nothing for the set scope and return a "default value" for the get scope: usually 0, false, empty string, empty part and 0 length arrays.

Every Part that inherits from the Part class can optionally implement a "with" property.

Note: The Part class is our framework's "object" from which everything inherits. At some point, we will update the C# object class to contain "with" properties.

## Example Primitive Parts

Let's look at pseudo-code for the Float and Integer part. Our Float and Integer parts implements the "with" properties as follows:

### Float Part

    namespace Vision.Core {
    
      [Serializable] public class Float : Part {
        protected float p_value = 0.0f;
        [XmlElement("value")] public override float withFloat {
          get { return p_value; }
          set { p_value = value; }
        }
    
        [XmlIgnore] public override int withInt {
          get { return (int)p_value; }
          set { p_value = value; }
        }
      }
    }

Our Float Part is able to convert to different primitive types automatically (note this is pseudo-code and more validation occurs in the actual source code).

Note: At some point we will update object of C# to contain with properties. The example Float part would be removed and the float type would be updated. This allows us to take advantage of boxing, etc.

### Integer Part

The Integer Part looks almost similar to the Float part.

    namespace Vision.Core {
      [Serializable] public class Integer : Part {
    
        protected int p_value = 0;
        [XmlElement("value")] public override int withInt {
          get { return p_value; }
          set { p_value = value; }
        }
    
        [XmlIgnore] public override float withFloat {
          get { return (float)withInt; }
          set { withInt = (int)value; }
        }
    
      }
    }

## Operations

So, we have our primitives defined. Let's look at how we bypass the need for Overloading by implementing an Add Part.

### Add Part

    namespace Vision.Core {
      [Serializable] public class Add : OpArgDual {
    
        [XmlElement("argLeft")] public Part argLeft { get; set; }
        [XmlElement("argRight")] public Part argRight { get; set; }
    
        [XmlIgnore] public override int withInt {
          get { return argLeft.withInt + argRight.withInt; }
        }
    
        [XmlIgnore] public override float withFloat {
          get {
            return argLeft.withFloat + argRight.withFloat;
          }
        }

      }
    }

What is most interesting to note is that each "with" Property calls the associated "with" property of argLeft and argRight and then adds them. For example, withFloat is implemented as:

    return argLeft.withFloat + argRight.withFloat;

## Putting it All Together

Let's see how we are able to use operations on different primitive types without Overloading. First, we will setup a few variables:

    ...
    IPart integerA = new Integer { withInt = 3 };
    IPart integerB = new Integer { withInt = 4 };
    IPart floatA = new Float { withFloat = 3.0f };
    IPart floatB = new Float { withFloat = 3.0f };
    
    IPart addTwoInts = new Add { argLeft = integerA, argRight = integerB };
    IPart addTwoFloats = new Add { argLeft = floatA, argRight = floatB };

		IPart addTwoThings = new Add { argLeft = floatA, argRight = integerA };
    ...

Adding two floats:

    float floatResult = addTwoFloats.withFloat;

Adding two integers:

    int integerResult = addTwoInts.withInt;

Adding a float and an integer and getting a float:

    float floatResult = addTwoThings.withFloat;

Adding two integers and getting a float:

    float floatResult = addTowInts.withFloat;

We are now able to support any combination of addition between the primitive data types of floats, strings, integers, arrays, and so on.

## Why We Don't Use Overloading

Since SIP is supposed to be used with Interface Vision, we want to be able to visually represent operations without focusing on the primitives we are operating on.

### Visual Composition

Let's see how addition looks visually.

<p class="pagination-centered"><img class="featurette-image img-polaroid" src="/assets/img/doc-overloading-sip-visual-example.png"> </img></p>

What is important to note is that the inputs and outputs to our addition configuration do not care about what primitive types we will be adding. By hooking up our Add, we are able to support **all** combinations of addition of primitive types. The actual primitive we operate in depends on which "with" property is accessed on the Add part.

## Why Not Use Conversion Inside Operations?

We could have had a single withPart property and then provided conversion within operations.

However, this would require our code to continually validate that the correct type of data was returned before applying an operation. For example, let's say we wanted to implement adding integers and we only have withPart. We would need to do something as follows:

    namespace Vision.Core {
      [Serializable] public class AddInteger : OpArgDual {
    
        [XmlElement("argLeft")] public object argLeft { get; set; }
        [XmlElement("argRight")] public object argRight { get; set; }
    
        [XmlIgnore] public override object withPart {
          get {
            int result = 0;
            if (( argLeft is int ) && ( argRight is int )) {
              result = (int)argLeft + (int)argRight;
            } else {
              // Throw an Exception? Just return 0?
            }
            return result
          }
        }
    
      }
    }

We felt this would greatly slow down execution time: especially in math libraries.

Note: In this pseudo-code, we use object.

### How About Conversion Parts

We could have also created parts to do conversion such as a FloatToInt part. However, we would end up, again, with Part explosion. Also, with a visual integration environment, you would have to use a lot of conversion parts making the program look, visually, messy.

### Subroutine Explosion

In both examples, conversion within operations and conversion parts, we end up with part explosion: AddLong, AddFloat, AddString, AddFloatLong and so on. Really, if you think about it, overloading is not really the best way to define operations between primitive data types because of this subroutine explosion issue: a real problem in the programming industry.

People do solve this problem to some extent by using things like Generics.

