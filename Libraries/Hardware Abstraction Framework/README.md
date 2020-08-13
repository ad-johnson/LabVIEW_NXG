# Hardware Abstraction Framework
This framework provides a means of driving test instruments through the use of abstract commands (non-instrument specific) in an automated way.

Currently, the framework is very SCPI focussed but can be extended to support other test syntaxes. In the current version (1.0.0) available commands are extremely limited but the fundamental framework is in place.

## Dependencies
Depending on how you wish to deploy, there are no dependencies for the framework.  If you want to use the framework's Unit Testing then you must download and deploy the Unit Testing Framework as well.

## Installation
Download directory `Hardware Abstraction Framework` to wherever you want it and you are good to go.  

If you wish to use the Unit Tests in this framework, download the `Unit Testing Framework` into the same root directory and it should be picked up as a link.  If not, or you deploy the UTF to a different directory, you will need to re-import it.

## Overview
The basic approach to instrument testing is to send one or more commands to one or more instruments and process the results.  Some tests will be simple, one-off commands; others will be more complex involving multiple commands, over multiple instruments, over an extended period of time.

The framework attempts to meet these needs and has some basic concepts which a user should be aware of:
* **Commands:** a command is an instruction sent to an instrument that it can understand, run and, if necessary, provide a result.  The framework seperates out *instrument specific* commands from *instrument independent* commands which means that you can build tests up without a specific understanding of instrument-specific syntax, e.g. you could use a command such as `identity` rather than `*IDN?`.  The framework calls these `Executables` and there are two types: a `Write Executable` - sends a command only - and `Read Executable` - sends a command then reads a result.
* **Instrument:** a representation of an acutal test device to be used.  This can be anything that can be driven by LabVIEW NXG and the framework doesn't require instrument-specific drivers.  Instruments are abstracted so that you can use them 'generically' leaving the framework to do any necessary translation into 'specifics'.  Obviously, to use your instruments you will need to provide some of this translation but the benefit will be that your overarching tests can be generic and reusable across different instruments, e.g. substitution of one manufacturer's DMM with another manufacturer's DMM without change to the test.  The framework calls these instrument abstractions `Drivers`.  Drivers provide the interface between the framework and an instrument.
* **Processors:** these actually run the Executables against a Driver in a controlled manner.  They control the overall execution flow utilising events and messages to interact with the test application itself.  Complex processing can be controlled by using specific types of processors: single-shot, repeating, continuous, timed and so on.  The framework calls these `Executable Processor` and `Executable Director`.

The basic approach to running a test is to follow these steps:
1. Instantiate an `Executable` that represents the command you wish to use.
2. Associate a `Driver` to that Executable which will represent the instrument to send the Executable (command) to.
3. Repeat this for each command you want to run in your test.
4. Associate these Executables to one or more `Executable Directors`, selected on the basis of how you wish the commands to run.  For example, you may select an `Executable Director` to run once a `Reset` Executable and a `Repeating Director` to run 10 times a `Measure` Executable.
5. Associate these Executable Directors to a `Executable Processor` and run the test.  The Processor will process each Executable Director until it they have finished; results are sent back to the testing application as notifications and the Processor will listen for events such as `Pause`, `Resume`, `Stop` and modify the processing accordingly.
6. Your test app can listen out for a `Done` event to know when processing has finished; it can also look for `result` messages (notifications) to handle and display those.  How complex you make these is up to you but check out the examples for ideas.

Many Executables in the framework can be used as-is in test applications but there may be Executables that are missing for your purpose.  You can create your own by following the pattern established for the existing ones.

Similarly, the Executable Directors are likely to be sufficient for most test applications but again, you can create your own by following the pattern established for the existing ones.

Unless you use the same instruments that are wrapped by Drivers in the framework, it is most likely that you will have create your own for your instruments.  Again you can follow the pattern established for the existing ones; you may want to consider uploading these to the repository for inclusion in a future release.

The rest of this document describes the framework in more detail and how you can use it.
### Interacting with the Framework
Your test application will interact with the framework through other means than just asking it to process Executables.  The Framework is set up to respond to `Events` your test application can raise:
* **Pause:** Processing will be paused at the end of the current processing of Executable Director(s) - remember these could be long-running.  It does need to finish any currently running processing so as to ensure instruments are not left in an unknown state.
* **Resume:** Processing will be resumed if paused.
* **Stop:** Processing will stop immediately after the current processing of Executable Director(s) - again, so that instruments are not left in an unknown state. Any available results that have been notified but not processed by your application will be lost.

The Framework will interact with your test application through `Events` as well:
* **Done:** Raised by the Framework when all processing has completed. You could perhaps, on hearing this event, unblock the UI, display any messages, quit the test app, write results to a database and so on.

It also interacts with your test application via messages on a Notification Queue:
* **Result:** When a `Read Executable` runs and receives a result, it stores that and posts itself on to the Notification Queue as message data for the `Result` message.  Your test application should be waiting to receive these messages so you can do something with the result.  The Executable knows how to process a raw result (from the instrument) into something useable and of a specific data type; any results for parameters can also be processed into useable information.  Each Executable has a unique ID and you can tie a result on the Notification queue to a specific Executable instance so your test application knows how to handle the result. 

Events, Messages and Notification Queues are all established by the Framework and are available for your test application to use.

Examine the example applications to see how to use Events and Notifications.  These example applications form a great template for building your own applications on.

## Framework Structure
The framework has been mostly written using LabVIEW NXG's object oriented features so the majority of interaction between your test application and the framework will involve the use of classes; extensions to the framework will likewise be done through extending existing classes and overriding methods.

As the source is provided, you are encouraged to use one of the examples and single-step through it observing how the framework is called and how interactions play back-and-forth.

In terms of processing, the framework follows a Queue Message Handler pattern which is standardised within LabVIEW NXG.  The example, although simple in itself, also uses this approach to provide a basic pattern for you to follow in your own test applications.

***
**A note on LabVIEW NXG's approach to object oriented programming**

NI decided that a by-value approach to objects will be used, rather than by-reference.  The implications of this is that all data flows work on *copies* of data rather than references.  You need to bear this in mind as it is easy to get caught out.  In many cases, changing some object's data member will be on that copy and not an originating instance - the data will need writing back to that instance.  This can be confusing when changes don't work!

There is very little in the way of reflection available in the language so some actions are not as independent of implementation as they could be otherwise.
***
### Core Classes

![UML class diagram of framework core classes](./images/image00.png)

The class diagram above shows the classes and relationships that form the core element of the framework.  I'll break it down into smaller elements.
#### Executable
![UML class diagram of framework Executable classes](./images/image01.png)
An Executable represents a command that can be sent to an Driver for running on an instrument.  As the framework evolves over time, these will become much more than just commands for Drivers, and will represent other execution constructions such as optional pathways, or directives to help with processing.

There are two types of Executable:
* **WriteExecutable:** represents a command that can be sent to an instrument but which requires no response.
* **ReadExecutable:** represents a command that can be sent to an instrument and for which a response or result will be returned.  You will note it inherits some of its functionality from `WriteExecutable`: you don't need to create a Write Executable and a Read Executable as the latter incorporates the functionality for you.

An Executable is associated with a `Driver` whose responsibility it is to undertake any instrument specific translation of a command.  The framework will arrange for the Driver to run the command at the appropriate moment.

There are two ways a `Driver` can run an Executable:
* Immediately: as it says - the Driver sends the command straight to the instrument.
* When able: in this case, the Driver will wait for the instrument to finish processing any existing commands before sending this one.
An Executable controls this with its `canOverlap` property: if true, the Executable will be run immediately; if false, when able.

A `ReadExecutable` will hold a result returned from the instrument.  It does so in a raw form, `rawResult` - that is, it provides no interpretation of that result until asked to do so by the test application calling `buildResult`.  The test application will be notified of an available result via a `result` message raised on the `Notification` queue.  

Some Executables can take parameters to send along with the base command.  These are represented by the `Parameter` class.  The framework will automatically add any available parameters to the command before it is sent to the instrument: you just need to add the ones you want to the Executable when you create an instance of it.  A Parameter will hold a result, if available, and is capable of parsing that into a useable value for the test application.

`Executable Settings` are a way of provisioning configuration settings for Drivers, independently of how they may be applied to an instrument.  For example, you might use an Executable Setting called 'Settling Time'; on one DMM it might equate to 'Settle Time' whilst on another it might be 'Settling Duration'.  You won't care - the driver will map these to instrument specific configurations.

You'll see from the class diagram that Executable Settings provides no specific behaviour so is really just a placeholder.  There is more information in the [Settings](#settings) section below, but the functionality for settings hasn't needed any specific behaviour for Executables in the current version.

So what do you need to do?  Basically, just use one of the existing commands and add it to an Executable Director.  If none of the existing commands are of any use, then create your own by following the pattern established by an existing command.

Here's an overview of two commands:
![UML class diagram of concrete Executable classes](./images/image02.png)

You can see that all is needed is a concrete implementation of `getBaseCommand` (the command as it appears without any parameters or end of line characters) and, for Read Executables, a concrete implementation of `buildResult` which parses a raw result into a useable format for the test application.

#### Driver
![UML class diagram of framework Driver classes](./images/image03.png)

`Driver` is responsible for interfacing with actual test instruments providing the necessary mappings between a generic test language and instrument-specific language.  By doing this, a user of the framework can standardise on names for things like commands, parameter names, setting names and so on and leave the Driver to translate on an instrument-by-instrument basis.

It is envisaged that future versions of the framework will be able to load mappings between the generic and specific from an external source to allow tests to be more configurable.

A Driver will execute a query for an Executable when directed by the framework, obtaining a response as necessary.

`Driver Settings` are configurable items for an instrument and are mapped between a generic `Executable Setting` and the instrument setting.  More information is available in the [Settings](#settings) section.

So what do you need to do?  When you instantiate an Executable you need to give it a Driver to be used to process it.  It's likely you will need to create your own Driver instances but examples are available to draw from:

![UML class diagram of concrete Driver classes](./images/image04.png)

You can see here that all you need to provide is a concrete implementation for `buildDriverSettingsMappings` and `getInstrumentAlias` as the framework handles everything else.

#### Setting
![UML class diagram of Setting classes](./images/image05.png)


#### Processing
