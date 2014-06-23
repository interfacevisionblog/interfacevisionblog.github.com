---
layout: post
title: "A Simple Webserver That Could Some Day Challenge the Big Names"
description: "How to confiure "
category: Design
tags: [Interface Vision, CSS, resize, Compose-able, Web Server, nginx, message-oriented Programming]
author: Eric Hosick
author_twitter: erichosick
---

## Introduction

One question we've been asked is how fast is the Vision framework? To answer that, we've created parts (aka messages) that can be used to configure a web server from scratch. The results are shown below.

### The Results

We ran siege on a single 15" macbook pro (C# mono):

    sudo siege -b -c300 -r50 http://localhost:3050

with the following results:

    Transactions:             15000 hits
    Availability:             100.00 %
    Elapsed time:             9.55 secs
    Data transferred:         12.47 MB
    Response time:            0.13 secs
    Transaction rate:         1570.68 trans/sec
    Throughput:               1.31 MB/sec
    Concurrency:              199.18
    Successful transactions:  15000
    Failed transactions:      0
    Longest transaction:      4.39
    Shortest transaction:     0.00

the same siege test was ran by [Centmin Mod](http://centminmod.com/siegebenchmark_nginx_test1.html) with the following results:

    Transactions:             15000 hits
    Availability:             100.00 %
    Elapsed time:             20.20 secs
    Data transferred:         43.89 MB
    Response time:            0.27 secs
    Transaction rate:         742.57 trans/sec
    Throughput:               2.17 MB/sec
    Concurrency:              199.77
    Successful transactions:  15000
    Failed transactions:      0
    Longest transaction:      7.43
    Shortest transaction:     0.00

> We realize the comparison is **not** apples to apples but we wanted to have something we can start comparing to.

Let's look at the [SipCoffee]({% post_url 2013-12-19-design-composition-based-language %}) message-configuration that gave us these results.

## Quick Introduction to SipCoffee

SipCoffee is a message-oriented programming language. By that, we mean that all parts within the language are considered messages. A message contains some specific behavior and an aggregate of named-message (a class (message) with computed properties (named-messages)).

A program is created by composing messages. This can also be viewed as creating a data structure which contains both data and behavior (behavior and data-structures are inseparable: as they should be).

When a message is activated, the behavior composed within the message is executed. This may cause named-message (those computed properties of the message) to also active.

We call SipCoffee programs message-configurations because we've completely separated mechanisms from the business domain. Any changes you make in a program are directly related to the business domain.

In the message-configuration below, you can twiddle with the Web Server at any level. You can change queue sizes, time out periods, the server port and uri, the number of parts in a part pool, and even the cache size of the socket reader.

## Building A Web Server From Scratch

The following is the message-configuration of a very very simple, but complete, WebServer in only **50 lines** of SipCoffee.

    Scope ( scopeId -2
      ins (
        File ( named "Http200.Body.Html" fileUriStr "./Html/200.html" )
        ArrayByteBuilder ( named "Http200.Header"
          replace Byte ( byte 120 )
          part ArrayByte ( withArrayByte byte ( 72 84 84 80 47 49 46 49 32 50 48 48 32 79 75 13 10 68 97 116 101 58 32 120 120 120 120 120 120 120 120 120 120 120 120 120 120 120 120 120 120 120 120 120 120 120 120 120 120 120 120 120 120 120 120 120 120 120 13 10 83 101 114 118 101 114 58 32 86 105 115 105 111 110 13 10 67 111 110 116 101 110 116 45 84 121 112 101 58 32 116 101 120 116 47 104 116 109 108 59 32 99 104 97 114 115 101 116 61 85 84 70 45 56 13 10 67 111 110 110 101 99 116 105 111 110 58 32 107 101 101 112 45 97 108 105 118 101 13 10 67 111 110 116 101 110 116 45 76 101 110 103 116 104 58 32 120 120 120 120 120 120 120 120 120 120 120 120 13 10 13 10 ) )
          items ArrayList (
            ins (
              DateTimeNow ()
              WithLength ( part ScopeGet ( withName "Http200.Body.Html" scopeId -2 ) )
            ) ) ) )
      part SocketRouter ( named "restServ" 
        streamSource SocketListener ( backlogQueueSize 2000 
          ipEndPoint IPEndPoint ( port 3050 
            ipAddress DNSLookup ( hostEntryStr "localhost" )
          )
        )
        socketFactory PartPool ( initialSize 500
          item FactoryInstance (
            part PartPoolDecorator (
              part RunWorkerWithTimeOut ( waitTime 60000
                part Scope (
                  ins (
                    Socket ( named "socket" )
                    ArrayByteBuffered ( named "requestBody" sizeGrowBy 1024 sizeInitial 1024 )
                    ScopeGet ( named "respHead" scopeId -2 withName "Http200.Header" )
                    ScopeGet ( named "respBody" scopeId -2 withName "Http200.Body.Html" )
                    HttpHeader ( named "httpHeader" )
                  )
                  part Array ( run true
                    ins (
                      SocketReaderHeaderBody ( autoOpen true
                        streamSource ScopeGet ( withName "socket" )
                        data ArrayByteBuffered ( sizeGrowBy 1024 sizeInitial 1024 )
                        body ScopeGet ( withName "requestBody" )
                        headerEncoder ScopeGet ( withName "httpHeader" )
                        buffer ArrayByteBuffered ( sizeInitial 1024 sizeGrowBy 1024 )
                        terminator ArrayByte ( withArrayByte byte ( 13 10 13 10 ) )
                      )
                      SocketWrite ( autoOpen false 
                        streamSource ScopeGet ( withName "socket" )
                        data ScopeGet ( withName "respHead" )
                      )
                      SocketWrite ( autoOpen false 
                        streamSource ScopeGet ( withName "socket" )
                        data ScopeGet ( withName "respBody" )
                      )
                      NamedMessageGet ( withName "close" 
                        part ScopeGet ( withName "socket" callBehavior false )
                      ) ) ) ) ) ) ) ) ) )

Example messages are **Scope**, **File**, **ArrayByteBuilder**, **ArrayByte** and **Socket** (all implemented as classes). Example named-messages are *ins*, *named*, *sizeGrowBy* and *withName* (all implemented as computed properties).

Let's break the message-configuration down into smaller parts we can easily explain.

### The SocketRouter Message

The **SocketRouter** message, when activated, listens to a data stream on a given port. Any new connections on that data stream are forwarded to another data stream for handling.

The **SocketRouter** has two named-messages:

* streamSource - The source of all data streams.
* socketFactory - The message-configuration that handles reading from and writing to of any incoming data streams.

The message-configuration for the *streamSource* named-message is very simple. 

    streamSource SocketListener ( backlogQueueSize 2000 
      ipEndPoint IPEndPoint ( port 3050 
        ipAddress DNSLookup ( hostEntryStr "localhost" )
      )
    )

The *streamSource* named-message is a **SocketListener** message with a *backlogQueueSize* named-message of a **2000** message (Yep! 2000 is also a message) and a configured *ipEndPoint* which is easy to figure out.

The *socketFactory* named-message is a bit more complicated. We could configure our Web Server to handle one request at a time. However, that isn't very practical and scales worth shit. We need a way to allow messages to run on multiple threads at the same time.

To do this, we can use factories and part pools.

### The FactoryInstance, PartPool and PartPoolDecorator Messages

A **FactoryInstance** message, when activated, will return a copy of the message-configuration located in the *part* named-message.

* part - The message-configuration we'll make a copy of.

Our message-configuration is as follows:

    FactoryInstance (
      part PartPoolDecorator (
        ...
      )
    )

In the above message-configuration, the **FactoryInstance** message creates a copy of a **PartPoolDecorator** and all messages configured in the **PartPoolDecorator** message. This is one of the advantages of programming using data structures as opposed to parameterized functions. Want to create a copy of a program? Just duplicate it (Functions are great at manipulating data structures but don't make for very good data-structures in and of themselves).

We could implement our Web Server using only the **FactoryInstance** message-configuration. Each time a request comes in, a copy of the message-configuration to handle the request is made and ran in it's own thread (more on that below). However, why keep making copies of our message-configuration for every request by a client? Allocating and freeing memory are expensive operations. We should be able to re-use our copied message-configurations. This is where the **PartPool** and **PartPoolDecorator** messages come into play.

The messages **PartPool** and **PartPoolDecorator**, together, allow any message-configuration to be re-used multiple times throughout the execution of a program.

The **PartPool** message is simple:

* initialSize - The initial number of parts in the pool. If all the parts in the pool are in use, a new item is created and added to the part pool.
* item - The message-configuration that is pooled by the **PartPool** message. This is always configured with some kind of factory. The item's message-configuration is activated initialSize number of times and the result pooled inside the **PartPool** message.

The **PartPoolDecorator** message, when activated, activates the message contained in the *part* named-message. When the configured-message in *part* finishes, the **PartPoolDecorator** message automatically returns itself to it's part pool.

* part - The configured message to activate. When the message is finished, the **PartPoolDecorator** places itself back into the **PartPool**.

Using these two messages in conjunction with a factory, we can take **any** message-configuration and make it re-usable. This allows us to configure very scalable message-configurations.

Our specific message-configuration is as follows:

    PartPool ( initialSize 500
      item FactoryInstance (
        part PartPoolDecorator (
          part RunWorkerWithTimeOut (
            ... 
          ) ) ) )

The **PartPool** message starts out with an initialSize named-message of **500**. When the PartPool is first activated, the **FactoryInstance** message is activated 500 times (on as many threads as we can) and the results are placed in the **PartPool** message. The **PartPool** pulls a single message from it's pool and activates it.

In this case, the **PartPoolDecorator** is configured to contain a **RunWorkerWithTimeOut** message. This means the **PartPoolDecorator** is actually activated in a new thread meaning the **PartPool** returns control, almost immediately, to it's parent message: the **SocketRouter** message. The **SocketRouter** message is now ready to handle the next incoming request.

### The RunWorkerWithTimeOut Message

Here are where things get even cooler. The **RunWorkerWithTimeOut** message is able to run **any** message-configuration in a thread. Yep. Multi-threaded programming in SipCoffee is just that simple.

* waitTime - The time in milliseconds before the worker thread times out.
* part - The message-configuration to run within the worker thread.

### The Scope and ScopeGet Messages

Take a quick look at the original message-configuration. What you will notice is that there are no functions, parameters or variables. In message-oriented programming, **everything is a message**. Even the context/scope that a message is activated in, usually defined with a language using { and } (Defining context in a language isn't very data-structure friendly), is a message.

To support context, we have a **Scope** message to define the context a message-configuration runs in.

* part - The configured-message to activate within the context.
* ins - One or more configured-messages contained within the context of the scope. These can also be viewed as Variables.
* scopeId - When configured with a negative numeric message, the scope is "global" in nature. When not configured, the scopeId is based on the managed thread id a message-configuration is ran in (this is really powerful and explained below in detail)

In our message-configuration we have the following **Scope** message configured:

    part Scope (
      ins (
        Socket ( named "socket" )
        ArrayByteBuffered ( named "requestBody" sizeGrowBy 1024 sizeInitial 1024 )
        ScopeGet ( named "respHead" scopeId -2 withName "Http200.Header" )
        ScopeGet ( named "respBody" scopeId -2 withName "Http200.Body.Html" )
        HttpHeader ( named "httpHeader" )
      )
      part Array ( ... )
    )

The *part* named-message contains an **Array** message (more on that later). The *ins* named-message contains 5 messages, something like variables, each with a unique name within that Scope's context:

* A Socket message named "socket" - The socket we use to read and write to the stream within the current thread.
* An ArrayByteBuffered message named "requestBody" - The buffer where the body of the request from the client is placed.
* A ScopeGet message named "respHead" - The header that will be sent to the client.
* Another ScopeGet message named "respBody" - The message body that will be sent to the client.
* An HttpHeader message named "httpHeader" - The http header received from the client.

Here comes some programming power! A "variable" can be a single message like **Socket** or it can be another configured message! Yep! Just think of the power you have as a programmer. Variables are actually configured-messages!

Notice that, for this **Scope** message, the *scopeId* named-message is not defined. This means, the context of the Scope will be based on the current named thread. Yep! Even more **POWER**! We can run this message-configuration in as many threads as we want and there is no chance of the "variables" being accessed from multiple threads. You no longer need to worry about re-entrance or things being thread safe within this **Scope** (this isn't the case when a **Scope** message is configured to be global).

In message-oriented programming, accessing a "Variable" is also done using messages. SipCoffee has no equals (=) operator. To access a "Variable" within the current scope, you use the **ScopeGet** message.

* withName - The name of the "Variable".
* named - The unique name of the ScopeGet message (so we can access a ScopeGet message from another ScopeGet message if we like).
* scopeId - When configured with a negative numeric message, the scope is "global" in nature. When not configured, the scopeId is based on the managed thread id a message-configuration is ran in (this is really powerful and explained below in detail).

In this example:

    ScopeGet ( named "respHead" scopeId -2 withName "Http200.Body.Html" )
    
The **ScopeGet** message locates and activates a message *named* "respHead" in a *scopeId* of -2. This is configured as follows:

    Scope ( scopeId -2
      ins (
        File ( named "Http200.Body.Html" fileUriStr "./Html/200.html" )
      )
    )

Awesome! It looks like we have a "global variable" named 'Http200.Body.Html' that, when activated, reads from a file named './Html/200.html'. 

In this example:

    SocketReaderHeaderBody ( autoOpen true
      streamSource ScopeGet ( withName "socket" )
    )  

the *streamSource* named-message of the **SocketReaderHeaderBody** is a **ScopeGet** message. That message is configured to locate a message named "socket" configured as follows:

    part Scope (
      ins (
        Socket ( named "socket" )
      )
    )

Even more Awesome! It looks like we have not configured the scopeId meaning that this Scope is based on the managed thread id. When the **SocketReaderHeaderBody** is activated within a managed thread, the Scope is also contained within that managed thread.


## Reading from and Writing to the Stream

To activate messages in a given order, we can use an *Array* message setting the *run* named-message to a **true** message (Yep, even true is a message).

    Array ( run true
      ins ( ... )
    )

Any messages in the *ins* named-message of the *Array* message are activated in the order they were inserted.

Let's look at how the message-configuration works.

First, we activate a **SocketReaderHeaderBody** message:

    SocketReaderHeaderBody ( autoOpen true
      streamSource ScopeGet ( withName "socket" )
      data ArrayByteBuffered ( sizeGrowBy 1024 sizeInitial 1024 )
      body ScopeGet ( withName "requestBody" )
      headerEncoder ScopeGet ( withName "httpHeader" )
      buffer ArrayByteBuffered ( sizeInitial 1024 sizeGrowBy 1024 )
      terminator ArrayByte ( withArrayByte byte ( 13 10 13 10 ) )
    )

* autoOpen true - Automatically open the stream source.
* streamSource ScopeGet ( withName "socket" ) - the source of the stream is located in a Scoped "Variable" named 'socket'
* data ArrayByteBuffered ( sizeGrowBy 1024 sizeInitial 1024 ) - The header data of the message is an ArrayByteBuffered with an initial size of 1024 bytes and grows by 1024 bytes to place the header data in.
* body ScopeGet ( withName "requestBody" ) - The body buffer is located in a Scoped "Variable" named 'requestBody'
* headerEncoder ScopeGet ( withName "httpHeader" ) - The message to encode and decode the header is located in a Scoped "Variable" named "httpHeader"
* buffer ArrayByteBuffered ( sizeInitial 1024 sizeGrowBy 1024 ) - The buffer used directly by the socket is an ArrayByteBuffered with an initial size of 1024 bytes and grows by 1024 bytes to place the header data in.
* terminator ArrayByte ( withArrayByte byte ( 13 10 13 10 ) ) - The terminator between a message header and body is this ArraByte of 13 10 13 10 which happens to be the terminator between the header and body of html.

Second, we write to the socket the response header:

    SocketWrite ( autoOpen false 
      streamSource ScopeGet ( withName "socket" )
      data ScopeGet ( withName "respHead" )
    )

* autoOpen false - Don't auto open the data stream because we did that with the read above.
* streamSource ScopeGet ( withName "socket" ) - the source of the stream is located in a Scoped "Variable" named 'socket'. Yep. The same streamSource as above.
* data ScopeGet ( withName "respHead" ) - Oh man! Awesome. We are accessing a Variable named "respHead" which is a ScopeGet message which accesses a "Variable" named "Http200.Header". Of course, in future message-configurations, the response header to write could change.


Third, we write to the socket the response body:

    SocketWrite ( autoOpen false 
      streamSource ScopeGet ( withName "socket" )
      data ScopeGet ( withName "respBody" )
    )

* autoOpen false - Yep! Still don't auto open the data stream because we did that with the read above.
* streamSource ScopeGet ( withName "socket" ) - the source of the stream is located in a Scoped "Variable" named 'socket'. Yep. The same streamSource as above.
* data ScopeGet ( withName "respBody" ) - Oh man! Awesome. We are accessing a Variable named "respBody" which is a ScopeGet message which accesses a "Variable" named "Http200.Body.Html". We are writing to our socket the information located in a file.

Finally, we close the socket:

    NamedMessageGet ( withName "close" 
      part ScopeGet ( withName "socket" callBehavior false )
    )

The **NamedMessageGet** message causes a named message to activate. In this case, the named-message *close* (implementation is a calculated property accessed using reflection) of that same socket is activated.

And, once all of that runs we "fall-up" the message-configuration and the **PartPoolDecorator** message returns us to the PartPool to be used again!

## Conclusion

Message-oriented programming kicks ass. In 50 lines of code, we are able to create a scalable web server that is able to keep up with the big names. We haven't even started to optimize our underlying technology and it was written in C#. Imagine if it was implemented in something like [Rust](http://www.rust-lang.org/) (aww please add calculated properties rust people!).

We have a data-structure of messages that is also the behavior of the program. That makes the message-configuration very easily re-used in a multi-threaded environment. We aren't allocating memory all the time and we are able to activate message-configurations without the overhead of locking and worrying about re-entrance. We get the speed of "single threaded" programming in a multi-threaded environment. Woo hoo!

