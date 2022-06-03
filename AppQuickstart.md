# TiDAL badge App Quickstart

## Intro

Apps are written in [MicroPython](https://docs.micropython.org/en/latest/index.html). TiDAL specific definitions are in [`tidal.py`](https://github.com/emfcamp/TiDAL-Firmware/blob/main/modules/tidal.py):

```python
from tidal import *
```

The TiDAL CPU is an ESP32S3. There is a useful page on the general ESP32 capabilities here:

https://docs.micropython.org/en/latest/esp32/quickref.html

Although it does not cover the specifics of the ESP32S3 or the TiDAL badge. There is more info about the badge hardware in [`boarddescription.md`](boarddescription.md).

## Badge hardware

### GPIOs and buttons

`tidal.py` includes definitions for the 4 accessible GPIOs (`G0` to `G3`, which are configured as inputs by default) and the joystick and other buttons (`BUTTON_x` and `JOY_UP`, `JOY_DOWN` etc). These are [`Pin`](https://docs.micropython.org/en/latest/library/machine.Pin.html) objects. The buttons are all active-low, meaning their `value()` is `0` when they are pressed, and `1` when they are unpressed.

```python
from tidal import *
if JOY_CENTRE.value() == 0:
    print("Joystick is pressed down!")
```

There is a higher-level abstraction in `buttons.py` to allow you to register callbacks for when buttons are pressed:

```python
from tidal import *
from buttons import Buttons
but = Buttons()
but.on_pressed(JOY_CENTRE, lambda: print("Button pressed!"))
```

A `Buttons` instance is automatically created for you by `App` subclasses - call `self.buttons` (or `self.window.buttons`) in your `App` to access it. See [`Torch.on_start()`](https://github.com/emfcamp/TiDAL-Firmware/blob/main/modules/torch/__init__.py) for an example.

`buttons.py` also exposes the a static API compatible with [BADGE.TEAM](https://badge.team/docs/esp32-platform-firmware/esp32-app-development/api-reference/buttons/), but since this is not app-aware it is recommended to use a `Buttons` instance instead.

### I2C

[I2C Docs here](i2c.md)

### Display

The `display` object is how you write to the screen. It is a [`ST7789`](https://github.com/russhughes/st7789_mpy/blob/master/README.md) object. The display is configured in portrait by default, and is 135x240 pixels supporting 16-bit colour. Several bitmap fonts are included. Using the default 8x8 font means you have room for 16x30 characters (or 16x26 if you leave a 1 pixel gap between lines, which is the default used by `TextWindow`).

```python
from tidal import *
import vga2_8x8 as font
display.fill(WHITE)
x = font.WIDTH * 2 # Indent 2 chars
y = (font.HEIGHT+1) * 5 # And 5 lines down
display.text(font, "Hello world!", x, y, RED, WHITE)
```

There is a higher-level abstraction available in `textwindow.py`:

```python
from tidal import *
from textwindow import TextWindow

win = TextWindow(bg=BLACK, fg=WHITE)
win.cls()
win.println("Hello world!")
# use win.display if you need to access the ST7789 directly
```

TextWindow may be instantiated separately, or your `App` subclass may construct one for you (which is what `TextApp` does).

`ST7789` and `TextWindow` etc define colours in 16-bit 565 format - you can translate from RGB using the `color565(r, g, b)` helper function, which for convenience is imported into the `tidal` module along with some common colours like `BLACK`, `BLUE` etc.

The native character set used by `ST7789` is [CP437](https://en.wikipedia.org/wiki/Code_page_437) so only a limited set of non-ASCII characters can be displayed. `TextWindow` takes care of converting Python strings to the appropriate format, so you normally shouldn't have to worry about it.

### Timers

Use `App.after(time_ms, callback)` to queue a timer callback after a certain amount of milliseconds. The result is an object you can call `cancel()` on if you need to cancel the timer. These timers will wake the badge up from light sleep if necessary. Use `App.periodic()` if you want the timer to fire repeatedly. There are no restrictions on how many timers you can queue, unlike if you use `machine.Timer` which is not recommended because it's a much more complicated and nuanced API to use correctly.

```python
self.timer = self.after(1000, do_stuff)
# ...
self.timer.cancel()
```

You can use `Scheduler.after()` instead if you don't have an `App` instance to hand:

```python
from scheduler import get_scheduler

my_timer = get_scheduler().after(1000, do_stuff)
```

### Wi-Fi

Wi-Fi connections can be established using the `wifi` module:

```python
import wifi

print(f"Currently-configured SSID is {wifi.get_ssid()}")
# If not already connected...
if not wifi.status():
    # ... connect to the default-configured AP...
    wifi.connect()
    # ... and synchronously wait for it to connect or time out...
    wifi.wait()
if wifi.status():
    print("Connected!")
else:
    print("Connection failed!")
```

This module is compatible with the [BADGE.TEAM API](https://badge.team/docs/esp32-platform-firmware/esp32-app-development/api-reference/wifi/)

Having Wi-Fi connected has a significant impact on battery life. Do not connect unless your app really needs network access, and disconnect as soon as is practical.

The badge firmware is pre-configured with the correct credentials to use during EMF 2022. The "Wi-Fi Config" app can be used to connect to other networks after the event (or call `wifi.connect(ssid, password)` directly). 

**Note:** Do not configure your badge as an access point during EMF 2022. This is [prohibited by the NOC team](https://wiki.emfcamp.org/wiki/Network/Rogue_Access_Points).

## Structure of an App

All apps should be placed in the `/apps` directory. You may need to create this first. To allow for apps to include resources or other modules, each app should be in a separate directory with its init code in an `__init__.py`:

```
- myapp/
|- __init__.py
|- <more files here if needed>
```

Apps should inherit from `app.App` or one of its subclasses. The example below uses `TextApp` and `BG` and `FG` to specify its default colour scheme.

```python
from tidal import *
from app import TextApp

class MyApp(TextApp):
    
    BG = BLACK
    FG = WHITE

    def on_activate(self):
        super().on_activate()
        self.window.println("Hello world!")


# Set the entrypoint for the app launher
main = MyApp
```


### App life cycle

Your `App` object is instantiated by the scheduler when your app is launched (init is called with no arguments). Apps are expected to construct at least one window (generally, a subclass of `TextWindow`) and an associated `Buttons` instance.

Prior to being displayed for the first time, `on_start()` is called. If overriding, you must call `super().on_start()`. Any initial `window` or `buttons` should normally be constructed by the time of the `App.on_start()` call, therefore a common pattern is to construct the initial window (but not draw to it) in the constructor (for example `TextApp` does this, see [app.py](https://github.com/emfcamp/TiDAL-Firmware/blob/main/modules/app.py)) or immediately prior to the `super().on_start()` call.

Because `on_start()` is called exactly once, is a good place to declare any buttons callbacks you want to have - the app switching logic takes care of switching the active `Buttons` instance, so you don't have to worry about unregistering callbacks when your app deactivates or exits.

`on_activate()` is responsible for actually displaying your app on screen. The default implementation activates the `Buttons` associated with `self.window` and calls `self.window.redraw()` to display the initial contents. Override and call `super().on_activate()` to take additional actions when activated (for example, to start any animation timers). Override and don't call super to completely customise how your app displays.

You should not call anything that attempts to draw to the screen until the `on_activate()` call. Likewise you should not draw anything after `on_deactivate()` has been called - therefore it is a common pattern to cancel any queued timers in `on_deactivate()` if the timer callback results in anything being drawn to the screen.

**Note:** forgetting to call `super().on_start()` or `super().on_activate()` is likely to cause your app to not display correctly or not react to button presses.

`on_deactivate()` is called when your app is no longer being displayed on the screen. Usually this will be because the user has pressed the 'back' button to switch away from it, but it can also be because the inactivity timer has expired and switched off the display.

If/when the firmware supports shutting down apps, `on_stop()` will be called immediately prior.

### TextApp

The simplest way to make an app is to subclass `TextApp`. There are several declarative parameters you can define in your subclass to reduce the amount of code you have to write. Doing so means the only function you have to define is `on_activate()`, to actually determine what to display:

```python
from tidal import *
from app import TextApp

class MyApp(TextApp):
    TITLE = "My App" # if defined, adds a title bar to the window. Edit later with self.window.set_title()
    BG = GREEN # The background colour
    FG = color565(0xF0, 0xF0, 0) # A non-standard colour to use as the default text colour

    def on_activate(self):
        super().on_activate() # This will clear the screen by calling TextWindow.redraw()
        self.window.println("Hello world!")
```

### MenuApp

This is similar to TextApp but instantiates a `Menu` window instead of a plain `TextWindow`. It has one additional declarative parameter you can set, called `CHOICES`, which define the initial items shown in the menu and what should be called when one of them is selected by the user:

```python
from tidal import *
from app import MenuApp

class MyApp(MenuApp):
    # TITLE, BG and FG can also be specified
    FOCUS_FG = BLACK
    FOCUS_BG = CYAN
    CHOICES = (
        ("Item 1", lambda: print("Selected item 1!")),
        ("Item 2", lambda: print("Selected item 2!")),
    )

main = MyApp
```

`MenuApp` takes care of registering the appropriate `Button` callbacks such that the joystick can be used to navigate and select the menu items.

If desired you can omit defining `CHOICES` and instead populate the choices dynamically by calling `self.window.set_choices(...)` in `on_start()` or `on_activate()`. Remember to pass `redraw=False` if calling `set_choices()` from on_start, as drawing should not be done before `on_activate()`.

### micro-gui

In addition to `TextWindow` based Apps, [micropython-micro-gui](https://github.com/peterhinch/micropython-micro-gui) (aka 'ugui') is also available. This is a higher level, widget-based UI framework. To use it, declare a class inheriting from `UguiApp` and set the `ROOT_SCREEN` member to the initial `Screen` to be displayed. `UgiApp` takes care of most of the integration of micro-gui with the TiDAL app framework, including button handling, app switching and power management. See the link above for more info on how to use micro-gui.

```python
from uguiapp import UguiApp, Screen, ssd
from gui.widgets import Label
from gui.core.writer import CWriter
import gui.fonts.arial10 as arial10
from gui.core.colors import *

class MyScreen(Screen):
    def __init__(self):
        super().__init__()
        wri = CWriter(ssd, arial10, GREEN, BLACK, verbose=True)
        Label(wri, 0, 0, "Hello world")

class uGUIDemo(UguiApp):
    ROOT_SCREEN = MyScreen


main = uGUIDemo
```

Up/down/left/right/select and the back button are automatically configured, although micro-gui has no built-in support for the A or B buttons. Call `App.on_press()` to use them.

## Testing on badge

### WebSerial
WebSerial Editor can be found at
http://editor.badge.emfcamp.org/


### REPL
The MicroPython REPL is available by default on the USB port, eg `screen /dev/ttyUSB0 115200` or `minicom -D /dev/tty.usbmodem1234561` (depending on OS etc)

Prior to deploying apps for real, you can upload files to the board using `pyboard.py` and then test them via the REPL:

```
$ cd Mk6-micropython-board
$ python3 micropython/tools/pyboard.py --no-soft-reset -d /dev/tty.usbmodem1234561 -f cp /path/to/myapp.py :/myapp.py
$ python3 micropython/tools/pyboard.py --no-soft-reset -d /dev/tty.usbmodem1234561 -f ls /
ls :/
         139 boot.py
         920 myapp.py

$ minicom -D /dev/tty.usbmodem1234561
>>> import myapp
>>> myapp.main()
```

## Publishing to the Hatchery

http://2022.badge.emfcamp.org/

See [Hatchery](hatchery.md)
