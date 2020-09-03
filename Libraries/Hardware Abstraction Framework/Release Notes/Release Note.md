# Release Notes

Hardware Abstraction Framework releases

[V1.2.0](#v1.2.0)    3rd September 2020

[V1.1.0](#v1.1.0)    24th August 2020

[V1.0.1](#v1.0.1)    Older

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