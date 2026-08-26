# RS485 Sensor Interface Board

A unit to interface various RS485 sensors with a simple serial interface via an ATMega328-based circuit. The unit also provides averaged data, controlled stepped-up supply voltage and a field-adjustable ID.

"Why do you need that?", you ask.... 

I had been wanting to use an industrial soil mositure sensor for a simple project. This device was only available as an RS3485 device. So I ordered that - it then took days of head scratching to sort basic communication with the device. This unit is meant to simplify reading of avaeraged data from RS485 sensors and make it easy to get that data into your project via a simple serial request/reply set of commands.

The other issue with a lot of the RS485 sensor is that they need higher voltages supplied (such as 12V or even 24V DC). So on this interface board I added a DC-DC step up converter to power the sensor. 

I was also concerned with energy consumption (my project was battery based). If I was to leave the DC-DC converter running all the time then my battery would run down very quickly. So I added control of the power to the sensor, so that I can power up, take the readings then power down. This saves a huge ammount of energy (well, proportional to my battery size).

## Overview

The problem with measuring sensors with values that change relatively quickly is that they constantly need to be checked. This requires a bit of microcontroller time and processing. This unit is designed to solve that.

Wire up your RS485 sensor. Power the unit up. Then it will save the averaged data for you. You can then get hold of the data through serial requests and process as you need.

![Overview](https://github.com/curiouselectric/SensorInterfaceBoard/blob/1cc6fdd7eef303f40e9d5f870216e7cde911cf6a/RS485%20Interface%20Board%20Instructions/Images/Sensor%20Interface%20overview.png)

I wrote this to interface to an ESP32 data logger, which sleeps most of the time. It wakes up, talks to the RS485 sensor, gets the data it needs, then goes back to sleep, knowing the RS485 Interface Board is always monitoring.

It was designed as a relatively simple interface to remove the need for monitoring pulses/dealing with RS485 requests/getting sensor data and averaging the sensor data.

The brilliant [ESPHome project](https://esphome.io/components/#environmental) has loads of sensor types you can interafce easily with a low cost ESP microcontrollers. For systems that have WiFi and good power connections then that project is a good resource to use. This project is designed for very low power systems (battery based datalogging) and for situations where easy access to Wi-Fi is not available.

# RS485 Sensor Interface Firmware

The firmware for this project has been moved to: 

[https://github.com/curiouselectric/SensorInterfaceBoard/tree/main/Sensor%20Interface%20Board%20Firmware](https://github.com/curiouselectric/SensorInterfaceBoard/tree/main/Sensor%20Interface%20Board%20Firmware)

This is within the Sensor Interface Board repository: [https://github.com/curiouselectric/SensorInterfaceBoard](https://github.com/curiouselectric/SensorInterfaceBoard)

I have done this to keep all the sensor boards up to date with just one set of firmware for all the sensor boards.

# Hardware

A number of PCBs have been designed in KiCAD. This repository holds the RS485 sensor and is available here. A small PCB has been designed.

There is one reset switch, one user input switch and one LED output. 

There is a step-up DC-DC converter with power control (to power higher voltage sensors). 

There is a TTL to RS485 converter (to connect to the RS485 sensors)

There is a 4 pin 'Grove'-type connector for direct connection I2C (code for this is not yet implemented) 

## Board ID Number

Each unit can have a unique ID (using a push link 6 pin pad for 0-7 values), so multiple units can be added to a serial bus, if needed. The defalt is 0 (no links used).

## PCB User Switch and User LED

There is one user switch and one user LED on the unit.

The LED will show a regular flash every 5 seconds. This will briefly flash once every 5 seconds if the unit is in 'Response' mode. The LED will briefly flash twice every 5 seconds if the unit is in 'Broadcast' mode. Data will be sent at the broadcast rate.

The LED will also flash whenever data is sent of the serial port. The LED will go on before data sent and then off after data is sent.

Pressing the user switch for >0.5 seconds and then releasing will result in a switch press.

A switch press will increment the mode from 0-1-2-3-4-5 then back to 0.

The unit will flash after a button press to indicate the broadcast mode (so 0 flashes if the value is 0, 1 flash if the value is 1 etc).

If this is set to 5 then the unit works in 'Response' mode.

If this is set to 0-4 then the unit is in 'Broadcast' mode and will send the data at the relevant interval (0 = 1s, 1 = 10s, 2 = 1 min, 3 = 10 min and 4 = 1 hour).

The mode can also be set with a serial request, using the "Set the unit to broadcast:" method (see below).


# Serial Data and Commands

It returns the average values and information when requested on serial port.

If the 8-bit CRC (Cyclic Redundancy Check) is used then each request must have a 2 char CRC added between the ? and # within these commands (labelled ^^ here).
For the responses, if no CRC enabled then the ?^^ within these commands is NOT returned. You must use the ? for a command, but the response will not contain it.

At all other times then the unit is asleep.

## Set the unit to broadcast:

Request: "aaI0SEND*?^^#" where * is an int (0)= 1s data, (1)= 10s data, (2)= 60s/1 min data, (3)= 600s/10 min data, (4)= 3600s/1hr data, (5)= NO data

Returns: "aaI0SEND*?^^#" + CRC if requested where * is an int (0)= 1s data, (1)= 10s data, (2)= 60s/1 min data, (3)= 600s/10 min data, (4)= 3600s/1hr data, (5)= NO data

You can also set the unit to broadcast using the user switch. Press the button for around 0.5s or more then release. This will go through the boradcast modes from 0-1-2-3-4-5 then back round to 0. The LED will flash the number of times for the setting (so send = 0 the unit will not flash, but data will appear within 1 second!).

If the unit is in broadcast mode then the minimum and maximum wind speeds and the wind vane data are all reset each time period.

## What is baud rate?:

Request: "aaI0STBD?#" ("aaI0BD?dc#" with CRC)

Returns: "aI0CHBD9600?^^#"  // Where 9600 is the baud rate 

## Set Baud Rate:

Request: "aaI0STBD\*?^^#"  Where \* is (0)1200, (1)2400, (2)9600, (3)57600, (4)115200

Returns: "aaI0BD9600?^^#"   // Where 9600 is the baud rate + CRC if requested

## What is ID?:

Mentioned at start up of unit - it is hardware set... and cannot be changed in code.

ID selection is by using a shorting link on the pads labelled 1, 2 and 4. The default is for no pads to be connected and the ID is 0. This means the unit will respond to "I0" as the ID.

To change the ID to another number from 0-7 then use a shorting link on the pins to create a binay number. The connections are:

1    |2    |4     | ID
-------|-------|-------|----
NC     |NC     |NC     | 0
CONN |NC     |NC     | 1
NC     |CONN |NC     | 2
CONN |CONN |NC     | 3
NC     |NC     |CONN | 4
CONN |NC     |CONN | 5
NC     |CONN |CONN | 6
CONN |CONN |CONN | 7

You can request the ID:

Request: "aaI*ID?^^#"  The * value can be anything.

Returns: "aaIXID:X?^^#"   // Where X is the device ID + CRC if requested

## Request Data from ONE channel

Request: “aaI0R00A*?^^#” where 00 is the channel number (00, 01 etc) and * is the averaging period (* = 0 for 1 sec data, 1 for 10 sec data, 2 for 60 sec (1 min) data, 3 for 600 sec (10 min) data and 4 for 3600 sec (1 hr) data 

Returns:  "aaI0R00A*:123.40:123.40:123.40?^^#"  // This wil return the average data then : then the minimum value then : then the maxximum value.

## Request Data from ALL channels

Request: “aaI0RAAA3?^^#”

Returns: "aaI0RAAA1:123.40:567.80?^^#" where the first item of data is channel 0, the next is channel 1. This will depend upon the number of channels.

## Request ALL Minimum data

Request: “aaI0RMNA4?^^#”  - does not matter what averaging period. min/max are just the min/max seen at max data rate.

Returns: "aaI0RMN:123.40:567.80?^^#"  where the first item of data is channel 0, the next is channel 1. This will depend upon the number of channels.

## Request ALL Maximum data

Request: “aaI0RMXA0?^^#”  - does not matter what averaging period. min/max are just the min/max seen at max data rate.

Returns: "aaI0RMX:123.40:567.80?^^#"  where the first item of data is channel 0, the next is channel 1. This will depend upon the number of channels.

## Reset the Min and Max value:

The min and max of each channel are logged. These are reset if the data is sent in broadcast mode. If the unit is in Response mode then you need to reset the min/max when you have taken the data.

Request: "aaI0RESET?#" ("aaI0RESET?d9#" with CRC)

Returns: "aaI0RSTOK#"

## Request Temperature, Humidity and Pressure Data

This command can only be used if the I2C AHT20/BMP280 sensor is attached and has been defined in the code. Otherwise it will return an error.

You can call this at any time and the unit will request from the sensor. It is NOT averaged or requested at any other point.

The Temp and Humidity board is an AHT20 (with device ID 0x38) and a BMP280 is used to measure the air pressure (with device ID 0x77).

Request: "aaI0TEMP?^^#"  (^^ for the CRC)

Returns: "aaI0T$$.$$H$$.$P$$$$$$?^^#" (^^ for the CRC) and $$$ are the values (floats) for Temeprature in C and Reltive Humidity in % and Air Pressure in Pa.
Returns: "aaI0T-H-P-#" if the board is not connected or there is an issue wth one of the sensors.

## Serial 'Button' press

You can simulate a button press with a serial command. This might be useful for some logging applications

The command "aaI0SWA?^^#"  (Where ^^ is the CRC of the data before the ?) will act just like a button press. This is for control via a data logger serial port without access to the physical switch.

Request: "aaI0SWA?^^#" 

Returns: "aaSEND*?^^#", where * is the time interval to send data (0->1->2->3->4->5->0 etc, with 5 never sending)

## What is the software version?:

Request: "aaI0SWV?^^#"

Returns: "aaI0SWV:1.0?^^#" (where 1.0 is the software version)

## What is the device type?:

Request: "aaI0DT?^^#""

Returns: "aaI0DT:SM?^^#" where SM is the device type (SM = Soil Moisture) - check the sensor list for these 2 char codes.

## Add CRC check:

Within the config of the firmware a CRC (Cyclic Redundancy Check) can be added to the data (or not!).

Set this true using the flag in the config.h file:

  #define ADD\_CRC\_CHECK     true    // Use this to add CRC check to incomming and outgoing messages
  
This uses the CRC routines from Rob Tillaart, available here: https://github.com/RobTillaart/CRC

A 'CRC-8/SMBUS' is perfromed on the data and a 2 byte CRC code is added to all replys (and expected on all enquiries). This is added between a ? and # symbol.

If no CRC then the end char is a #.

For example: aaI0RESET?d9# has the CRC check d9 added to the reset request.

Remember: Capitalisation will affect the results: D is not the same as d!

You can use this online calculator to check your CRC: https://crccalc.com/ The type of CRC is CRC-8/SMBUS.

## Failure codes:

If data is not that length or does not have 'aa' and '#' at start/end then return with send "aaFAIL\*\*#" error code. All will have CRC on these codes, if requested.

+  "aaFAILCRC?^^#": CRC check fail
+  "aaFAILTL?^^#": String too long
+  "aaFAILID?^^#": Problem with ID
+  "aaFAILIDX?^^#": ID not correct to device
+  "aaFAILPE?^^#": No aa and # on the string
+  "aaFAILBD?^^#": Baud rate change fail
+  "aaFAILCN?^^#": Channel number requested is greater than channels existing
+  "aaFAILCMD?^^#": Command not recognised 

# Sensor Specific Commands
For each sensor type there are additional commands. These are only available if the unit is in the correct mode.
They are listed here.

## 'SM' RS485 Soil Moisture Sensor

There are no extra commands.
Soil Moisture (% (m3 water in m3 soil)) and Soil Temperature (C) are monitored.

## 'PY' RS485 Pyranometer Irradiance Sensor

There are no extra commands.
Irradiance (in W/m2) is monitored.

## 'DC' RS485 DC Power Sensor

There are no extra commands.
Voltage (V), Current (A), Power (W) and cumulative Energy (Wh) are monitored.

