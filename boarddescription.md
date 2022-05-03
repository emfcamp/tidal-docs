# TiDAL description and features

TiDAL is a handheld device in USB stick form factor. It has a USB-C plug at the front, and consists of a sandwich of two PCBs with a battery in between.

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

# The TiDAL device
![The TiDAL device](/images/tidal-full-annotated.png)

When not plugged into a computer, TiDAL is meant to be used in the vertical orientation, with the USB plug pointing away from the user. In this configuration, the on/off switch is to the left of the screen.
Sliding the switch towards the front enables power to the device.

There are two side buttons either side of the device - the one on the right of the screen is **BUTTON_A** and the one on the left is **BUTTON_B**. BUTTON_A is used to wake the device from sleep mode.
The button next to the corner of the screen is **BUTTON_FRONT**. By default, that's the button used to go back to the previous screen, or back to the launcher from inside an app.

**To start your badge for the first time, slide the switch to the on (forward) position, and press BUTTON_A.**

There is a joystick below BUTTON_FRONT. It can register clicks in four directions (JOY_LEFT, JOY_RIGHT, JOY_UP, JOY_DOWN) and can also be pressed in the middle (JOY_CENTRE).

Below that, there are two buttons. The one at the corner of the board is a **RESET** button, which restarts the firmware on the badge. The one next to it is the **BL** button.
The BL button has two functions - it force-activates the backlight, and if held down while the board is reset, it activates the USB bootloader, which is used to replace the entire firmware on the board.

Above the BL and RESET buttons are two flat flexible cable (FFC) connectors. They let you attach expansion boards to the badge.
You can use the generic expansion board that comes with the device, and attach things to that, or you can design your own board that connects to this one with an FFC cable.
The two connectors are identical and connected to the same signals. The pinout is (looking into the connector, left to right):

3V3, GND, SCL, SDA, G0, G1, G2, G3

The same signals, in the same order, are exposed on solder pads at the bottom of the badge.

![The TiDAL device](/images/tidal-edgeview.png)

