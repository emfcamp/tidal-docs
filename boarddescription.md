# TiDAL description and features

TiDAL is a handheld device in USB stick form factor. It has a USB-C plug at the front, and consists of a sandwich of two PCBs.

Features include:
- Extremely tiny - 60x25mm footprint
- 135x240pixel 24-bit colour TFT screen
    - Portrait orientation
    - ST7789 controller
    - FAST!
- 5-directional joystick button
- 1 button on the top
- 2 buttons on the sides
- backlight/bootloader and reset buttons on top
- 2 0.5mm pitch FFC connectors for expansions
    - Pinout: 3V3, GND, SCL, SDA, 4xGPIO (all ADC capable)
- Same signals as FFC connectors brought out on solder pads on bottom of board with 1.27mm pitch
- 1 addressable LED with switchable power, programmable colour and unbearable brightness so you can find your tent at night
- 3-axis accelerometer (QMA7981)
- 3-axis magnetometer (QMC7983)
- ATECC108A cryptographic accelerator
- on/off switch
- very small battery
- Native USB with goodies


![The TiDAL device](/images/tidal-full-annotated.png)


![The TiDAL device](/images/tidal-edgeview.png)
