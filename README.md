# SUNRISE_CTRL
A customizable sunrise and sunset using addressable led light strip.
Retro green screen style gui for setting sunrise and sunset parameters.

Will work with a variety of light strips as supported by NeoPixel:
  - When using WS2812 or other 5v strip, ensure separate power feed to
    strip from your 5v power source - don't run high amp current through
    your dev board.
  - When using WS2815 or other 12v strip, tie all grounds together.
  - Calculate your amp draw! WS2815 144 draws nearly 2A at 100% at end of 
    sunrise. WS2812 draws over 8A at 100% at end of sunrise.
  - Size your power supply with these numbers in mind. Don't run your power
    supply at full load.
  - Consider your heat generation. Mounting the strip to an aluminum plate
    can prolong your strip lifespan.

Parts list
Part	          12V build (WS2815)	          5V build (WS2812)
---------------------------------------------------------------
Controller	    NodeMCU v2 (ESP-12E, CP2102)	Same
LED strip	      WS2815, 144 px	              WS2812B, 144 px
Power supply	  12V, 3A or more	              5V, 10A or more
Buck converter	12V to 5V, 1A or more         Not needed
Level shifter	  74AHCT125 (or 74HCT125)	      Same
Series resistor	330 ohm, 1/4 W	              Same
Bulk capacitor	1000 uF, 25V or higher	      1000 uF, 10V or higher
Fuse	          Inline blade fuse, 3A	        Inline blade fuse, 10A
Power wire	    18 AWG	                      16 AWG, inject power at both ends
Optional	      Aluminum channel for heat	    Same

Interconnects
NodeMCU VIN              -->          5V
NodeMCU GND              -->          GND
NodeMCU D4 (GPIO2)       -->          74AHCT125 1A (pin 2)
74AHCT125 1OE (1)        -->          GND
74AHCT125 1Y (3)   --[330 ohm]-->     strip DI
74AHCT125 GND (7)
74AHCT125 VCC (14)       -->          5V
74AHCT125 (4, 10, 13)    -->          5V
74AHCT125 (5, 9, 12)     -->          GND

Why each part is there
  - Level shifter: the ESP8266 drives 3.3V logic. A WS2812 wants about 0.7 x VDD, roughly 3.5V, to read a high, so 3.3V sits right on the edge. It often works on the bench, then glitches when the supply sags or the wire gets longer. The 74AHCT125 reads 3.3V as a clean high and drives a full 5V edge.
  - Level shifter, second reason: GPIO2 is a boot strapping pin and has to be high at power-on. Wired straight to a strip that is unpowered, or powered up later, the strip's input can drag GPIO2 low and the board won't boot. The shifter's input is high impedance, so GPIO2 stays where the board's pullup puts it.
  - 330 ohm resistor: dampens ringing on the data edge and limits current into the first pixel's input if the strip and board power up at different times. Put it close to the strip's DI.
  - 1000 uF capacitor: absorbs the inrush when the supply first connects, which can kill the first pixel. Mount it across V+ and GND at the strip input and observe polarity.
  - Buck converter (12V build): the NodeMCU's onboard regulator is linear. From 12V it burns about 0.7W as heat under WiFi load and runs hot. A small buck to 5V into Vin avoids that and also powers the level shifter.
  - Fuse: a 144 px strip at full white can pull 8A or more. Fuse at the supply, sized just above the strip's real draw.

WS2815 notes
  - WS2815 strips have a backup data line (BI) next to DI. On the first pixel, tie BI to GND. Leaving it floating can cause flicker.
  - At 12V the voltage drop along 144 px is small, so power injection at one end is usually enough. At 5V it is not: feed V+ and GND at both ends of the strip, or the far end will drift toward red at high brightness.

Grounding
  - Every ground ties together: power supply, strip, buck converter, NodeMCU and level shifter. Data has no reference without a shared ground, and a missing ground is the most common reason a strip shows random colors.

Before first power-on
  - Change OTA_PASSWORD in src/main.cpp and --auth= in platformio.ini from changeme to your own password. They must match.
  - Set TZ_STRING in src/main.cpp if you are not in US Central time.
  - Check NUM_LEDS against your strip.
  - If red and green come out swapped, change NeoRgbFeature to NeoGrbFeature.
  - Power the strip from the supply, never through the NodeMCU's USB or 5V pin.
  - Flash over USB first with the nodemcuv2 env. After that, use nodemcuv2_ota.

  Sunrise Light v2
  ESP8266 + WS2815 strip (144 px), green-on-black CRT control page.

  - First boot: no credentials -> opens AP "Sunrise-Setup" with a captive
    portal (WiFiManager). Pick your network there; it's remembered.
  - Page lives at http://sunrise.local (mDNS). The IP is shown on the page too.
  - Time from NTP, US Central with DST.
  - OTA updates via PlatformIO env d1_mini_ota, or in a browser at
    http://sunrise.local/update (user "admin", password OTA_PASSWORD).
    Upload .pio/build/d1_mini/firmware.bin there.

  LED data: GPIO2 (D4). The UART1 method hardware-shifts the bits, so WiFi
  interrupts can't corrupt the timing and the radio never gets blocked.
  Don't use Serial1 anywhere; the strip owns UART1.

Acknowledgments
  - Firmware design, code review and documentation were developed with Maya, an AI agent by Sesame.
