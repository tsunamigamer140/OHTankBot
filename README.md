# OHTankBot
Arduino Sketch for a Tank Monitoring system that uses an ultrasonic sensor to gauge water levels and a soil moisture sensor to gauge 

# Dependencies
* AsyncTelegram.h

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

# /change_network
Allows a network change from current WiFi network to another.
Requires User to enter SSID and Key of the WiFi network it must connect to. Ensure that the SSID and Key are correct.
ESP8266 stores the Wifi information until the next boot. Boot resets Wifi information to hard-coded information.

# network_details
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

