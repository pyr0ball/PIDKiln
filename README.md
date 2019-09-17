# pidkiln
Ceramic kiln PID controller based on Arduino/Weemos board.

This is work in progress - but advanced enough to start uploading it.

## Key features:
- interface accessible both from LCD screen and WWW webpage
- unlimited (only by storage) kiln programs number, program file size limited to 10KiB (but this is artificial limit - can be extended)
- internal ESP storage for programs, data, logs (perhaps later SD - but I'm not sure yet)
- local preferences on disk, editable with Web interface
- online monitoring, program management, editing, graphs and kiln control
- build in clock synchronised with NTP servers (if Internet connected)
- safety features build in (temperature run out protection, probe failure, SSR failure, kiln insulation failure)
- simply cool and cheap (comparing to commercially available products) all in one solution

![LCD menu sample](https://raw.githubusercontent.com/Saur0o0n/pidkiln/master/Documentation/images/PIDKiln_LCD_sample.png)

## Required components:
- ESP32 board
- MAX31855 breakout board
- K-type thermocouple
- DC->AC solid state relay

Kind of optional, but recommended:
- 128x65 dot matrix LCD 12864B v2
- Rotary encoder with button

Optional:
- DC/AC secondary relay - like SLA-05VDC-SL-C (240V/30A) mechanical relay
- Thermistors
- Perhaps a kiln :)

## BOM and expenses

Total expenses for this set should be around 30-40$
- ESP32 board: 14-15$ if exactly like mine, but other ESP32 you can bought for 6$+
- MAX31855 board: 2$
- K-type thermocouple: 1$-10$ - depending on max temperature
- LCD 12864B: 5$
- Encoder: 1$
- SSR: 4$ + 4$ for radiator

- Mechanical relay SLA-05VDC-SL-C (Songle): 3$
- Thermistors: 1$

## Why this configuration?

I've already used "kind of" controller made on Linux server and cron procedures, and it was ok.. for a while. Then I bought almost cheap Chinese controller PC410, just to find out, it is missing most of required stuff (that original PC410 should have), and I already was accustom to be able to see everything over Internet.
So I've made few attempts with Arduino - it was fine, but since I need remote access - ESP will be much better choice. I've started to work on ESP8266, but being afraid that I will be lacking some GPIOs - I've moved to ESP32. And since price difference is negligible, this is the platform of choice for this project.
It is also beneficial, because it has build in flash memory, that I can use for all required data - without need of connecting additional SD cards.

MAX31855 comparing to more available MAX6675, is better choice since it allow us to work up to 1350C and it has 3,3V logic.

LC12864B is perhaps not the best choice, but I simply had this one already (I've used if before for my 3D printer). Perhaps later I'll change it. Problem with this LCD is that it has 5V logic. Sometimes it works on 3,3V (depending on version), in my case it does not work correctly. But since it's one way communication (only MISO) hooking it up to 5V for both logic and back light works and does not crash my board. Clean solution would be to use logic voltage translator (there is plenty of them for 1$) - I'm planing that, as soon as I get my ordered board.

Relays - Main relay is SSR (Solid State Relay) type. It's because we need to switch it fast and often - SSR can do it, but it will get hot, so make sure you have good radiator. Also if you are going to use cheap Chinese knock offs, make sure it rated twice the output current of your heater.
All relays may fail, and they may fail in closed (conductive) state. Because of it, I've also implemented second stage mechanical relay in case of SSR failure. It's mechanical, so it won't get hot. This will allow to turn off the kiln in case of SSR failure with mechanical one (and other way around too). Additional relay (SLA-05VDC-SL-C) is optional.

Thermistors are to measure outside kiln temperature. In case of insulation failure - we can shut it off. This are extremely cheap (used in 3d printing) thermistors.

## GPIO/PINs connection

**LCD**

Connected to one of three SPI on ESP32 - called VSPI (MOSI-23, MISO-19, CLK-18, CS-5)

ESP32	| LCD
--------|---------
+3.3V	| BLA (this can be also +5V if you wish)
GND	| BLK
4	| RST
GND	| PSB
+5V	| VCC (This should be - for ESP sake - 3,3V, but my LCD doesn't work with lower voltage. Try first with 3,3V)
GND	| GND
5	| RS
18	| E
23	| R/W

**Encoder**

ESP32	| Encoder
--------|---------
+3.3V	| 5V/VCC
GND	| GND
32	| Key
34	| S2
35	| S1

**MAX31855**

Connected to one of three SPI on ESP32 - called HSPI (MOSI-13, MISO-12, CLK-14, CS-15)

EPS32	| MAX31855
--------|---------
+3.3V	| VCC
GND	| GND
12	| SO/DO (slave output/data output)
15	| CS (chip select)
14	| SCK (clock)

**Relays**

ESP32	| SSR
--------|-------
GND	| GND
5V      | VIN
21      | IN

ESP32   | SLA-05VDC-SL-C
--------|----------------
GND     | GND
5V      | VIN
19      | IN


**Thermistors**

ESP32	to thermistor

## Installation

- Assembly hardware, as specified above.
- Clone git into the Arduino user programs directory (on Linux "/home/username/Arduino/").
- You have to already have installed ESP32 framework - if don't, do it now (https://github.com/espressif/arduino-esp32/blob/master/docs/arduino-ide/boards_manager.md).
- Don't forget about ESP32FS plugin (drop it to "/home/username/Arduino/tools")
- Install required additional libraries (all can be installed from Arduino IDE Library Manager): [ESPAsyncWebServer](https://github.com/me-no-dev/ESPAsyncWeb.Server),[AsyncTCP](https://github.com/me-no-dev/AsyncTCP), [PID Library](https://github.com/br3ttb/Arduino-PID-Library/)
- Install also [my clone of Adafruit-MAX31855-library](https://github.com/Saur0o0n/Adafruit-MAX31855-library) - this implements HW SPI
- Update (there is no other way to do it) libraries/ESPAsyncWebServer/src/WebResponseImpl.h variable TEMPLATE_PLACEHOLDER to '~'.
- Compile and upload.
- Open data/etc/pidkiln.conf and edit your WiFi credentials (if you want to use) and, if you want, some additional parameters.
- Upload sketch data (from data directory) to ESP32 SPIFFS with help of ESP32FS plugin (Menu->Tools->ESP32 Sketch Data Upload).

## Documentation

- Most of the documentation you can find on [PIDKiln github page](https://github.com/Saur0o0n/pidkiln) - so please use it
- Some less formal updates information and step by step instructions will be on [my webpage](https://adrian.siemieniak.net/portal/tag/PIDKiln/)

