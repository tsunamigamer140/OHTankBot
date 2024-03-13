# OHTankBot
Arduino Sketch for a Tank Monitoring system that uses an ultrasonic sensor to gauge water levels and a soil moisture sensor to gauge 

# Dependencies
* ESP8266WiFi.h
* UniversalTelegramBot.h
* TimeAlarms.h
* ArduinoOTA.h

# Features
* Monitors level of a water tank.
* Query current level of water in tank - returns distance of surface of water from sensor, NOT HEIGHT OF WATER IN TANK
* Querry current motor status - ON or OFF
* Get SSID and RSSI of currently connected WiFi network
* Scan for local WiFi networks and returns their SSID, RSSI and Network Encryption Status (Open or Closed)
* Make ESP8266 go into deep sleep for X seconds
* Returns current battery level. 
* Change connected Wifi network remotely.
* Checks for a new message every 1 second
* Alerts user when it detects that the motor has been turned on. Also sends distance of water surface from the ultrasonic sensor along with the percentage of water present in tank.
* Alerts user when it detects that the motor has been turned off. Also sends a mini-report detailing the change of the water level over time between Motor ON and Motor OFF.
* Mini-report included change of level of water over time - Increasing, Decreasing and Constant.
* Mini-report also includes Level of Water in the tank for intervals of 5cm, starting with the initial level of water when the Motor has been turned on.
* Added OTA functionality to enable flashing sketches over the same WiFi network as the ESP. [If OTA does not work on Arduino IDE 2.X.X, try OTA uploading on Arduino IDE 1.8.X

# Commands
* /check_level
* /check_motor
* /change_network
* /network_details
* /network_scan
* /deep_sleep
* /battery_lvl

# /check_level
returns distance of water surface from the ultrasonic sensor.

# /check_motor
returns the status of the motor - ON or OFF

# /get_time
returns local time on ESP in 25 hour format

# /set_time
Allows user to manually set time in 24 hour format as HH MM SS
Asks user for confirmation after entering the time.

# /calibrate_time
Autoamtically calibrates local time according to NTP Time Servers
Change the Time Zone delay during TimeClient creation according to your Time Zone.
Eg. GMT +5:30 -> 330 Minutes -> 19800 seconds
Eg. GMT -2:00 -> 120 Minutes -> 7200 seconds

# /change_network
Allows a network change from current WiFi network to another.
Requires User to enter SSID and Key of the WiFi network it must connect to. Ensure that the SSID and Key are correct.
ESP8266 stores the Wifi information until the next boot. Boot resets Wifi information to hard-coded information.

# /network_details
returns SSID, RSSI of currently connected network

# /network_scan
 Scans for local WiFi networks and returns their SSIDs, RSSIs and Network Encryption Type - OPEN (O) or CLOSE (*)

# /deep_sleep
Makes the ESP go to sleep for the specified number of SECONDS. 
ESP becomes unresponsive for this duration.

# /battery_lvl
Returns the current battery level.
Hardware must use a voltage divider to reduce battery voltage to between 0 and 3.3V (Rated Voltage of the ESP8266).
Battery voltage is mapped between 0 and 100 and value is returned



