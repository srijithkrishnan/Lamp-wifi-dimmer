# Lamp wifi dimmer

# controlling the brightness of an LED or a lamp using mobile app or using current time. 
* The code contains of two sections, namely setup and loop respectively, The former setup section initiates the wifi configuration between the microcontroller and the hotspot.
It asks for the ssid and the password to connect to the respective wifi hotspot.


* The latter loop section contains two parts, controlling the brightness of the led using a mobile app and also controlling the brightness using the current time respectively.


* In the former part an interface is being created between the mobile app and the microntroller, using which the brightness is being send in bytes to the controller and the controller sets the brightness of the led with the respective bytes recieved from the mobile app.


* In the latter part, time is being synced with the microcontroller using the wifi, the microcontroller is connected with. A request is being send to the NTP servers using UDP and recieves the total seconds from JAN 1 1970 which is then decoded to obtain the current hour, minutes and seconds.

* Using this information the brightness of the led is being set or controlled. 
