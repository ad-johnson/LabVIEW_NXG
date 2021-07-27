# Release Notes

Hardware Abstraction Framework releases

[V1.4.1](#v1.4.1)    27th July 2021

[V1.4.0](#v1.4.0)    17th July 2021

[V1.3.0](#v1.3.0)    14th October 2020

[V1.2.0](#v1.2.0)    3rd September 2020

[V1.1.0](#v1.1.0)    24th August 2020

[V1.0.1](#v1.0.1)    Older

## V1.4.1
Minor bug fixes and added an example for running an ADC on the Instrument Control Board.

## V1.4.0
Minor bug fixes and added an Arduino Mega 2560 device

* Timestamp (used in Parameters) now parses days and months correctly
* Possible to use an Arduino Mega 2560 as a serial connected device
* Added a new example using the DMM6500 and something called an Instrument Control Board (a serially connected device that interfaces to an Arduino created and built by the framework author.)  Whilst you won't have such a device, the example shows how these two devices can cooperate to create a chart and allow the export of correlated Voltage readings against DAC output (from the ICB.)

## V1.3.0
Extended the functionality of the framework to handle Serial connected devices and provide a more complex example showing iteratively stepping a voltage.

* Updated the Voltage Executable to take an increment value.  After processing the Executable, the VI 'Post-Exec Callout' increments the Volts value by the Increment value.  This is best used with an Executable Director that runs multiple times, e.g. Repeating Director.  The default value for Increment is 0 (zero) so no increment is applied.
* Created an example to show Stepping Voltage to demonstrate the increment change. 
* Refactored the examples to order them by complexity, with the simplest first.
* Created a new class Serial Driver to encapsulate functionality for interacting with an instrument connected over a COM port.  Basically, this just ensures that the Driver is initialised in the correct manner and allows the provision of specific values for driving the Serial connection.
* Created a new data type Serial Port Attributes that is used to configure a serial connection, e.g. Baud rate.  It can be used as-is with reasonable defaults but can be changed as necessary.
* Created an implementation of an Arduino Uno R3 Driver to make use of the Serial Driver.
* Updated the Instrument Identification example to show a sample Arduino sketch that can be used with the Arduino Driver to return an Identity.
* Updated the ReadME to reflect changes to the framework.
* Fixed a bug with Repeating Director not repeating executions the correct number of times.

## V1.2.0
Extended the functionality of the framework to provide a Voltage generate executable.  This also required the provision of an Instrument exectuable and an Output executable which are used to control generation output channels and channel on/off switching.

* Added members to Parameter so that they can return a typed response.
* Added a member to the Measure Executable so that it can return a typed response (Double.)
* "Reset Framework" VI changed to "Setup Framework" and "Teardown Framework" to properly configure the framework.
* Created a Notification approach to posting results to a Test App.  If a Notifier is passed to the framework, it will use that instead of sending a Notification Message.  When the Test App calls the framework to process Executables, then the app is effectively blocked until processing completes, and then results can be processed.  This is fine for simple commands that being processed, but for long-running commands, e.g. via a Continuous Executable Director, then notifications will allow results to be processed as they are returned from the Instrument.  Examples show the use of both approaches.
* Created a Waveform Measurement example which uses a Continous Director to generate a result which can be displayed on a chart.  This uses the Notification method of result processing.  It also demonstrates the use of Pause/Resume events to interrupt and resume the generation of the waveform.
* Changed existing examples to use Setup Framework and Teardown Framework.
* Created Executables to generate a Voltage.
* Created a Generate and Measure example which allows an instrument to output a voltage and an instrument to measure it for verification.
 
## V1.1.0
Extended the functionality of the framework to use Parameters and provided an example to show their use.

* Changed Read Executable member Build Result to return the Raw Result as a default, rather than raise an error.
* Updated Executable IDN to use the parent Build Result member rather than override it.  This Executable returns a String value, thus Raw Result is a suitable response.
* Created Measure Executable to obtain a result from an Instrument that supports it.  This expects a function to measure as part of its setup.  It can also use a separate 'buffer' or 'Store Name' if desired: this must exist when the Executable is processed.
* Created an example test application for Measure: Take Measurement.  This allows the use of additonal parameters (Buffer Elements for the DMM6500 that I use.)  The example follows the standard implementation pattern for test applications.
* Refactored example Instrument Identification to share sub-VIs.
* Added a new generic Executable: User Defined Write.  This has no inbuilt command to process, the test application must provide it.  No result is expected or processed so this should not be used when result(s) are required.
* Added a new generic Executable: User Defined Read.  This has no inbuilt command to process, the test application must provide it.  A result is awaited and processed.  This Executable is used in the Take Measurement example.
* Results are parsed into seperate Exectuable result and Parameter result(s) on the following basis:

    * If no parameters are used, the result is solely for the Executable.
    * The result is tokenised on comma (,); if the number of tokens matches the number of parameters the results are solely for the parameters.
    * If the number of tokens matches the number of parameters + 1, the first result is for the Executable and the remaining results are for the parameters.
    * Otherwise, the results are mapped to the Parameters from the first parameter; some parameters may not have any result in this scenario.
    * Subclasses can override this default behaviour.
* Updated the HAF Readme.

## V1.0.1
* Added readme for framework to describe what it is and how to use it.