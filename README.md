# Twilio Remote ID
This is a technical proof-of-concept implementation of broadcast Remote ID for the Twilio Programmable Asset Tracker using BlueTooth 4 Legacy advertisements.

Code is originally based on [v3.1.2 of Twilio's code](https://github.com/twilio/programmable-asset-tracker/tree/v3.1.2/build). All changes are in the device code. There are no changes to the agent code.

The Remote ID BLE advertisement is based on the Standard Specification for Remote ID and Tracking ([ASTM F3411-22a](https://www.astm.org/f3411-22a.html)).

Some of the published values are hard coded either due to lack of hardware support, such as no barometer for pressure altitude, or because the agent hasn't been modified to support changing the values that should be customizable.

Receiver used for testing: [OpenDroneID for Android](https://github.com/opendroneid/receiver-android)


### Architecture
The Asset Tracker uses the Squirrel language. It is separated into agent and device code.

The agent code runs in the cloud and communicates with the device, providing a frontend for the user. Twilio's original code uses it to display tracking information and allow the user to configure the tracker. In this project, the agent is currently not used, as phase 1 was just to show that it was technically possible to use the tracker to broadcast Remote ID. The minimum viable product would have it modified to allow the user to configure the device and set the customizable Remote ID values.

The device code runs on the Asset Tracker. Twilio's original code uses it to gather a variety of tracking telemetry and periodically sends it via cell networks to the agent.

The agent and device code are deployed through [impCentral on Electric Imp](https://impcentral.electricimp.com).


### Summary of changes
- Removed all code related to the photoresistor (shipping mode and tampering detection)
- Removed repossession mode
- Removed location based on cell network, wi-fi, and BLE. Only using GNSS (GPS)
- Removed sleep, ensuring Remote ID broadcasts run continuously
- Added more telemetry captured from GNSS
- Added BLE advertising based on the Remote ID spec for BlueTooth 4 Legacy


### Detailed changes by class and function
For all changes in phase 1, see the [diff](https://github.com/riis/remoteid/compare/4c41e02..a4f598b#diff-b0c11272ea9b5050fb9a591d5c2464e8e89665d6dfb813db955fc790dbc29186).

All classes are from upstream.

Modified classes:
- `ProductionManager`
  - Removed sleep/shipping mode
- `ESP32Driver`
  - Notable changes:
    - `_init`: Removed scanning and added BLE advertising
  - Added functions:
    - `advStaticData` - Creates and broadcasts the BLE advertisement for data that rarely changes. Called every 3 seconds according to spec. Most of the data would ultimately be set by the user.
    - `advDynamicData` - Creates and broadcasts the BLE advertisement for telemetry data.
    - `getPrefixHeader` - Creates the BLE advertisement data that is prefixed to all messages.
    - `generateBasicIdMsg` - Converts the given parameters into hex data for the Basic ID message according to spec.
    - `generateLocationVectorMsg` - Converts the given parameters into hex data for the Location Vector message according to spec.
    - `generateAuthMsgPg0` - Converts the given parameters into hex data for the first page of the Auth message according to spec.
    - `generateAuthMsg` - Converts the given parameters into hex data for additional pages of the Auth message according to spec.
    - `generateSelfIdMsg` - Converts the given parameters into hex data for the Self ID message according to spec.
    - `generateSystemMsg` - Converts the given parameters into hex data for System message according to spec.
    - `generateOperatorIdMsg` - Converts the given parameters into hex data for the Operator ID message according to spec.
    - `getMsgCounter` - Returns the counter for the given message type.
    - `incMsgCounter` - Increments the counter for the given message type.
    - `incAs8Bit` - Increments an integer as an 8-bit value, looping 255 back to 0.
    - `latLonToHex` - Converts a float value to a 32-bit hex string.
    - `altToHex` - Converts a float value to a 16-bit hex string.
    - `getRemoteIdTimeHex` - Converts a Unix timestamp to a 32-bit hex string.
    - `getTenthsSecAfterHourHex` - Gets the number of tenths of a second as a 16-bit hex string.
    - `getHorizAcc` - Converts horizontal accuracy in meters to an enum ordinal hex value according to spec. 
    - `getVertAcc` - Converts vertical accuracy in meters to an enum ordinal hex value according to spec.
    - `getSpeedAcc` - Converts speed accuracy in meters/second to an enum ordinal hex value according to spec.
    - `integerToHexString` - Converts a decimal integer into a hex string. Based on function in utilities lib, removing "0x" prefix and setting uppecase.
    - `int16ToLeHexString` - Converts an integer to a 16-bit little-endian hex string.
    - `int32ToLeHexString` - Converts an integer to a 32-bit little-endian hex string.
    - `stringToHexString` - Converts text to a hex string.
    - `blobToHexString` - Convert a blob (array of bytes) to a hex string. Based on function in utilities lib, removing "0x" prefix and setting uppecase.
    - `updateAdv` - Sends the assembled BLE advertisement data (AT+BLEADVDATA) to the ESP32.
  - Removed functions:
    - `scanWiFiNetworks`
    - `scanBLEAdverts`
    - `_parseWifiNetworks`
    - `_logVersionInfo`
    - `_removeColon`
    - `_parseBLEAdverts`
- LocationMonitor
  - Notable changes:
    - `getStatus`: Default location object - Added `altitude`, `groundSpeed`, `velocity`
  - Removed functions:
    - `setRepossessionEventCb`
- DataProcessor
  - Removed all code related to photoresistor, repossession, and tampering
- LocationDriver
  - Notable changes:
    - `_onUBloxNavMsgFunc` - Added Telemetry values: `altitude`, `groundSpeed`, `velocity`, `velocityVert`, `altMsl`, `headingMotion`, `headingVehicle`, `accHoriz`, `accVert`, `accTime`, `accSpeed`, `accHeading`
  - Added functions:
    - `_getLocationContinuous` - Initializes the U-Blox module and starts polling GNSS for location.
  - Removed functions:
    - `configureBLEDevices`
    - `_getLocationCellTowersAndWiFi`
    - `_getLocationBLEDevices`
- Application
  - Notable changes:
    - `_initConnectionManager` - Config values removed: `autoDisconnectDelay` and `maxConnectedTime`, added: `stayConnected`

Removed classes:
- `BG9xCellInfo`
- `Photoresistor`

Unmodified classes:
- `UartOutputStream`
- `LedIndication`
- `PowerSafeI2C`
- `FlipFlop`
- `CfgManager`
- `CustomConnectionManager`
- `CustomReplayMessenger`
- `FloatVector`
- `AccelerometerDriver`
- `BatteryMonitor`
- `MotionMonitor`
- `SimUpdater`
