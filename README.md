# Inland-Raspberry-Pi-Kit
A repository for learning more about raspberry pi and the micropython language. 

![IMG_1369](https://user-images.githubusercontent.com/66803124/184785475-19ed5705-e2c6-48d0-ae1a-3ffc1aacfbf6.png)

First we will make an LED light up using a breadboard, raspberry pi, and transitor, 2 cables, and an LED light. 

Stay tuned for more updates and step by steps. 


Installing MicroPython
See the corresponding section of tutorial: Getting started with MicroPython on the ESP32. It also includes a troubleshooting subsection.

General board control
The MicroPython REPL is on UART0 (GPIO1=TX, GPIO3=RX) at baudrate 115200. Tab-completion is useful to find out what methods an object has. Paste mode (ctrl-E) is useful to paste a large slab of Python code into the REPL.

The machine module:

import machine

machine.freq()          # get the current frequency of the CPU
machine.freq(240000000) # set the CPU frequency to 240 MHz
The esp module:
```
import esp

esp.osdebug(None)       # turn off vendor O/S debugging messages
esp.osdebug(0)          # redirect vendor O/S debugging messages to UART(0)

# low level methods to interact with flash storage
esp.flash_size()
esp.flash_user_start()
esp.flash_erase(sector_no)
esp.flash_write(byte_offset, buffer)
esp.flash_read(byte_offset, buffer)
The esp32 module:

import esp32

esp32.hall_sensor()     # read the internal hall sensor
esp32.raw_temperature() # read the internal temperature of the MCU, in Fahrenheit
esp32.ULP()             # access to the Ultra-Low-Power Co-processor
Note that the temperature sensor in the ESP32 will typically read higher than ambient due to the IC getting warm while it runs. This effect can be minimised by reading the temperature sensor immediately after waking up from sleep.
```
Networking
The network module:
```
import network

wlan = network.WLAN(network.STA_IF) # create station interface
wlan.active(True)       # activate the interface
wlan.scan()             # scan for access points
wlan.isconnected()      # check if the station is connected to an AP
wlan.connect('ssid', 'key') # connect to an AP
wlan.config('mac')      # get the interface's MAC address
wlan.ifconfig()         # get the interface's IP/netmask/gw/DNS addresses

ap = network.WLAN(network.AP_IF) # create access-point interface
ap.config(ssid='ESP-AP') # set the SSID of the access point
ap.config(max_clients=10) # set how many clients can connect to the network
ap.active(True)         # activate the interface
A useful function for connecting to your local WiFi network is:

def do_connect():
    import network
    wlan = network.WLAN(network.STA_IF)
    wlan.active(True)
    if not wlan.isconnected():
        print('connecting to network...')
        wlan.connect('ssid', 'key')
        while not wlan.isconnected():
            pass
    print('network config:', wlan.ifconfig())
Once the network is established the socket module can be used to create and use TCP/UDP sockets as usual, and the urequests module for convenient HTTP requests.
```
After a call to wlan.connect(), the device will by default retry to connect forever, even when the authentication failed or no AP is in range. wlan.status() will return network.STAT_CONNECTING in this state until a connection succeeds or the interface gets disabled. This can be changed by calling wlan.config(reconnects=n), where n are the number of desired reconnect attempts (0 means it won’t retry, -1 will restore the default behaviour of trying to reconnect forever).

Delay and timing
Use the time module:
```
import time

time.sleep(1)           # sleep for 1 second
time.sleep_ms(500)      # sleep for 500 milliseconds
time.sleep_us(10)       # sleep for 10 microseconds
start = time.ticks_ms() # get millisecond counter
delta = time.ticks_diff(time.ticks_ms(), start) # compute time difference
Timers
The ESP32 port has four hardware timers. Use the machine.Timer class with a timer ID from 0 to 3 (inclusive):

from machine import Timer

tim0 = Timer(0)
tim0.init(period=5000, mode=Timer.ONE_SHOT, callback=lambda t:print(0))

tim1 = Timer(1)
tim1.init(period=2000, mode=Timer.PERIODIC, callback=lambda t:print(1))
The period is in milliseconds.
```
Virtual timers are not currently supported on this port.

Pins and GPIO
Use the machine.Pin class:
```
from machine import Pin

p0 = Pin(0, Pin.OUT)    # create output pin on GPIO0
p0.on()                 # set pin to "on" (high) level
p0.off()                # set pin to "off" (low) level
p0.value(1)             # set pin to on/high

p2 = Pin(2, Pin.IN)     # create input pin on GPIO2
print(p2.value())       # get value, 0 or 1

p4 = Pin(4, Pin.IN, Pin.PULL_UP) # enable internal pull-up resistor
p5 = Pin(5, Pin.OUT, value=1) # set pin high on creation
p6 = Pin(6, Pin.OUT, drive=Pin.DRIVE_3) # set maximum drive strength
Available Pins are from the following ranges (inclusive): 0-19, 21-23, 25-27, 32-39. These correspond to the actual GPIO pin numbers of ESP32 chip. Note that many end-user boards use their own adhoc pin numbering (marked e.g. D0, D1, …). For mapping between board logical pins and physical chip pins consult your board documentation.
```
Four drive strengths are supported, using the drive keyword argument to the Pin() constructor or Pin.init() method, with different corresponding safe maximum source/sink currents and approximate internal driver resistances:
```
Pin.DRIVE_0: 5mA / 130 ohm

Pin.DRIVE_1: 10mA / 60 ohm

Pin.DRIVE_2: 20mA / 30 ohm (default strength if not configured)

Pin.DRIVE_3: 40mA / 15 ohm
```
The hold= keyword argument to Pin() and Pin.init() will enable the ESP32 “pad hold” feature. When set to True, the pin configuration (direction, pull resistors and output value) will be held and any further changes (including changing the output level) will not be applied. Setting hold=False will immediately apply any outstanding pin configuration changes and release the pin. Using hold=True while a pin is already held will apply any configuration changes and then immediately reapply the hold.
