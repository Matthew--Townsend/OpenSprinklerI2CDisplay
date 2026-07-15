# OpenSprinklerI2CDisplay
A driver for I2C displays that integrates with the updated unified firmware for Open Sprinkler

Initial Prompt for AI

You are generating a complete C++ solution targeting a Raspberry Pi 3 B+ running Debian Trixie 64‑bit.
The goal is to implement **two separate selectable architectures** for adding LCD and button support
to an OpenSprinkler system that uses the unified firmware (2025 version) which currently lacks LCD
and button support.

====================================================================
OPTION 1 — External “OpenSprinkler LCD Daemon” (runs alongside firmware)
====================================================================

Implement a standalone C++ daemon that provides LCD and button UI without modifying the unified
OpenSprinkler firmware. The daemon must:

1. Drive a 20x4 HD44780 LCD connected through a PCF8574 I2C backpack at address 0x27.
2. Use /dev/i2c-1 for I2C communication.
3. Implement LCD initialization, clear, setCursor, writeText, and full 20x4 rendering.
4. Poll OpenSprinkler’s HTTP JSON API every second to retrieve:
   - current time
   - active zones
   - next scheduled run
   - controller status
5. Render a multi-screen menu system on the LCD:
   - Main screen
   - Zone status screen
   - Next run screen
   - Manual run screen
   - Settings screen
6. Implement a menu stack with navigation:
   - Up button
   - Down button
   - Back button
   - OK button
7. Read 4 physical buttons from Raspberry Pi GPIO pins:
   - Up: GPIO17
   - Down: GPIO27
   - Back: GPIO22
   - OK: GPIO23
   Configure them as input with internal pull-ups, active-low.
8. Use a clean architecture:
   - LcdDriver class (I2C + HD44780 + PCF8574)
   - ButtonInput class (GPIO + debouncing)
   - MenuState enum
   - MenuController class (state machine + rendering)
   - OpenSprinklerApiClient class (HTTP GET + JSON parsing)
   - Main loop that ties everything together
9. Provide all necessary C++ source files, headers, and a CMakeLists.txt.
10. The daemon must run continuously and be robust against API timeouts or LCD errors.

====================================================================
OPTION 2 — Embedded LCD + Button Subsystem inside unified firmware
====================================================================

Implement a native LCD + button subsystem directly inside the OpenSprinkler unified firmware.
The unified firmware already contains I2C support for the DS3231 RTC (see I2CRTC.cpp and I2CRTC.h).
Use this as a structural reference for adding a second I2C device (the LCD).

Requirements:

1. Add new modules:
   - LCDI2C.cpp / LCDI2C.h
   - ButtonInput.cpp / ButtonInput.h
   - MenuController.cpp / MenuController.h

2. LCD driver must:
   - Use /dev/i2c-1 via Linux file descriptors
   - Use ioctl(I2C_SLAVE) to set address 0x27
   - Implement HD44780 initialization sequence
   - Implement PCF8574 bit mapping for RS, EN, D4–D7, backlight
   - Provide clear(), setCursor(), writeText(), and full rendering

3. Button subsystem must:
   - Use libgpiod (preferred) or sysfs GPIO
   - Support GPIO17 (Up), GPIO27 (Down), GPIO22 (Back), GPIO23 (OK)
   - Implement debouncing and edge detection
   - Provide a clean event interface for the menu system

4. Menu system must:
   - Integrate with unified firmware’s main loop
   - Render:
     - Main screen
     - Zone status
     - Next run
     - Manual run
     - Settings
   - Support navigation via button events
   - Update LCD at appropriate intervals without blocking core firmware tasks

5. Modify unified firmware:
   - Add ENABLE_LCD flag in config.h
   - Add LCD initialization during startup
   - Add LCD update calls in the main loop
   - Add button polling in the main loop or a dedicated thread
   - Ensure no interference with RTC I2C operations

6. Provide:
   - All new source files and headers
   - Patch instructions for integrating into the unified firmware tree
   - Updated build instructions (CMake or Makefile depending on firmware structure)
   - Comments explaining how the new subsystem fits into the firmware architecture

====================================================================
GENERAL REQUIREMENTS FOR BOTH OPTIONS
====================================================================

- Generate full source code, headers, and build files.
- Use real logic for:
  - I2C communication
  - GPIO input
  - JSON parsing
  - HTTP requests
  - LCD rendering
  - Menu navigation
- Include comments explaining each subsystem.
- Provide a complete main() for Option 1.
- Provide integration instructions for Option 2.
- Do NOT generate placeholder code; implement real functionality.

Produce both options in the same output, clearly separated.
