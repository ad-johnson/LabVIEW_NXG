# Release Notes

Hardware Abstraction Framework releases

[V1.0.1](#v1.0.1) 24th August 2020

[V1.1.0](#v1.1.0)

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