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
