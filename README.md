# Twilio Remote ID
This is a technical proof-of-concept implementation of Remote ID for the Twilio Programmable Asset Tracker using BlueTooth 4 Legacy advertisements.

Code is originally based on [v3.1.2 of Twilio's code](https://github.com/twilio/programmable-asset-tracker/tree/v3.1.2/build). All changes are in the device code. There are no changes to the agent code.

The Remote ID BLE advertisement is based on the Standard Specification for Remote ID and Tracking ([ASTM F3411 − 22a](https://www.astm.org/f3411-22a.html)).

Some of the published values are hard coded either due to lack of hardware support, such as no barometer for pressure altitude, or because the agent hasn't been modified to support changing the values that should be customizable.

Deployed through [impCentral on Electric Imp](https://impcentral.electricimp.com).
