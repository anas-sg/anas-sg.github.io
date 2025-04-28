---
title: MUIS prayer times diplay
date: 2025-04-28 22:00:00 +0800
categories: [embedded]
tags: [esp32, cyd, embedded, Arduino, c]     # TAG names should always be lowercase
description: A desktop clock and MUIS prayer times diplay using an ESP32 CYD
---
{% include embed/youtube.html id='yT5R_Q9c7Bg' title='MUIS Prayer Times Display on ESP32 CYD'%}

*Welcome to the first post of my GitHub Page! I finally released the code for my prayer times display as open source.*

## Background
As my grandma is getting older and more forgetful, she sometimes has trouble figuring out the date, day and prayer times from the monthly calendar. So I decided to create a digital display that shows the current time and the MUIS prayer times for the current day. The first version of that, which is in her room, has a ESP8266 (Wemos D1 R1) that drives 6 7-segment displays (current time and 5 prayer times); all of these are plugged into a PCB breakout board fabricated and assembled by JLCPCB. While that solution works, it is not elegant because of the need to drive multiple displays. It also lacks displays for the Gregorian and Hijri dates. I created 2 more versions using a similar WiFi and display logic, but with different LCDs: [Nokia 5110](https://lastminuteengineers.com/nokia-5110-lcd-arduino-tutorial/) and [ST7789](https://done.land/components/humaninterface/display/tft/st7789/)

## Present version
When I stumbled upon the [ESP32 CYD](https://github.com/witnessmenow/ESP32-Cheap-Yellow-Display/blob/main/SETUP.md), I realised that this would be perfect for my project. As the display and ESP32 SoC are well integrated into the same PCB, it greatly simplifies the setup and development process. It can even be placed into a 3D printed case and sold as a consumer product with custom (possibly open source) firmware! You can find the code [**here**](https://github.com/anas-sg/MUIS_prayer_times_CYD).

## How it works
There are 3 main aspects to the functionality:
1. time
2. prayer times
3. display

**time**<br>
Network time protocol (NTP) is used to periodically get the current epoch seconds from the Singapore NTP server pool and convert that into the current date and time using C standard library functions.

**prayer times**<br>
MUIS previously had an API that accepted an epoch seconds as a request parameter and returned the prayer times for the corresponding day as a JSON object. But about 2 months ago, they updated their backend to provide a JSON file containing the full year's worth of prayer times. Since each day has a fixed number of lines, the code now skips all the previous days' data and processes only the current day's prayer times. Following is a snippet from the JSON file [`https://isomer-user-content.by.gov.sg/muis_prayers_timetable_2025.json`](https://isomer-user-content.by.gov.sg/muis_prayers_timetable_2025.json):

```json
{
  "2025-01-01": {
    "hijri_date": "1 Rejab 1446H",
    "subuh": "5:43am",
    "syuruk": "7:07am",
    "zohor": "1:09pm",
    "asar": "4:33pm",
    "maghrib": "7:10pm",
    "isyak": "8:25pm"
  },
  "2025-01-02": {
    "hijri_date": "2 Rejab 1446H",
    "subuh": "5:44am",
    "syuruk": "7:07am",
    "zohor": "1:10pm",
    "asar": "4:34pm",
    "maghrib": "7:10pm",
    "isyak": "8:25pm"
  },
  ...
  "2025-12-31": {
    "hijri_date": "10 Rejab 1447H",
    "subuh": "5:43am",
    "syuruk": "7:06am",
    "zohor": "1:09pm",
    "asar": "4:33pm",
    "maghrib": "7:09pm",
    "isyak": "8:24pm"
  }
}
```
The prayer times are converted to 24-hour time integers and the code periodically checks if the current time falls within a prayer time. 

**display**<br>
[TFT_eSPI](https://github.com/Bodmer/TFT_eSPI) is the de-facto display library used for CYD as the ST7789 is well supported, the library is very easy to use and it comes with very nice fonts and many other awesome features like [sprites](https://en.wikipedia.org/wiki/Sprite_(computer_graphics)). Since the ST7789 has a GPIO pin for backlight control, PWM dimming is used to reduced the perceived brightness of the display. A Cartesian coordinate system is used to specify the locations of pixels, with the origin at the top left corner. Library functions are used to display text with a black background fill to overwrite the previous contents. This approach is, however, imperfect; as the fonts for dates and prayer times are not monospace, the same number of letters occupy different amounts of space depending on the letters being displayed. So the previous content may not be fully erased and some stray artefacts might sometimes remain. I hope to fix this in future versions. Different fonts are used for different items on the display and if the current time falls within a prayer time, that prayer time is displayed in yellow. Else, the prayer times are displayed in white. Displaying the dhuhr time had a gotcha; unlike the other 4 prayers, the 12-hour representation of the dhuhr time may have 1 or 2 digits for the hour: `12:5x pm` or `1:xx pm`. Hence, all prayer times are printed as `hh:mm`, with the first character usually a space (unless dhuhr is at `12:5x pm`).

## Future work
### WiFi provisioning
I hope to improve the way WiFi credentials are loaded into the program. Currently, they are hardcoded into the header file and compiled into the program. This is fine if I am running this on my own. But it may not be suitable for less tech savvy users. It definitely is not appropriate if this were to be sold as a product. There should be an easy way to load the WiFi credentials dynamically without re-compiling the program. The credentials should also be stored into non-volatile memory, eg. flash and automatically loaded upon a power on reset.

### Display updating
The updating of the display should be made more efficient by updating sprites first and then pushing the sprites to display. This should also solve the problem of erasing the old content.

### Build and flash scripts
I prefer to have the option of running a build script to automate the process of compilation without using the Arduino IDE. While the IDE is alright, having a scripted approach to building the program offers more flexibility. For example, I can use my favourite text editor [Sublime Text](https://www.sublimetext.com/) to write and build programs using Arduino libraries without having to touch the IDE

## Conclusion
This project initially started off as a way to help my grandma but evolved into a passion/hobby project. This is also my first time using GitHub pages and publishing open source code for a ESP32-based project. If you have any feedback or if you would like to contribute, please contact me :)
