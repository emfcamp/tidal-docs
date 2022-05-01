# TiDAL badge App Quickstart

## Intro

Apps are written in [MicroPython](https://docs.micropython.org/en/latest/index.html). TiDAL specific definitions are in [`tidal.py`](modules/tidal.py):

```python
from tidal import *
```

The TiDAL CPU is an ESP32S3. There is a useful page on the general ESP32 capabilities here:

https://docs.micropython.org/en/latest/esp32/quickref.html

Although it does not cover the specifics of the ESP32S3 or the TiDAL badge.

## Badge hardware

### GPIOs and buttons

`tidal.py` includes defintions for the 4 accessible GPIOs (`G0` to `G3`, which are configured as inputs by default) and the joystick and other buttons (`BUTTON_x` and `JOY_UP`, `JOY_DOWN` etc). These are [`Pin`](https://docs.micropython.org/en/latest/library/machine.Pin.html) objects. The buttons are all active-low, meaning their `value()` is `0` when they are pressed, and `1` when they are unpressed.

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
but.on_pressed(JOY_CENTRE, lambda pin: print("Button pressed!"))
```

**TODO instantiate a Buttons in App**

### Display

The `display` object is how you write to the screen. It is a [`ST7789`](st7788/README.md) object. The display is configured in portrait by default, and is 135x240 pixels supporting 16-bit colour. Several bitmap fonts are included. Using the default 8x8 font means you have room for 16x30 characters (or 16x26 if you leave a 1 pixel gap between lines)

```python
from tidal import *
import vga1_8x8 as font
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
```

TextWindow may be instantiated separately, or your `App` subclass may inherit it (see [the torch app](modules/torch/__init__.py) for an example of this).

### Timers

TODO use `App.periodic()` or `App.after()`.

## Structure of an App

To allow for apps to include resources or other modules, each app should be in a separate directory with its init code in an `__init__.py`:

```
- myapp/
|- __init__.py
|- <more files here if needed>
```

Apps should inherit from `app.App` or one of its subclasses. The example below uses `TextApp` and `BG` and `TextWindow.FG` to specify its default colour scheme.

```python
from tidal import *
from app import TextApp

class MyApp(TextApp):
    
    BG = BLACK
    FG = WHITE

    def on_start(self):
        self.window.cls()
        self.window.println("Hello world!")

```

## Testing on badge

The micropython REPL is available by default on the USB port, eg `screen /dev/ttyUSB0 115200` or `minicom -D /dev/tty.usbmodem1234561` (depending on OS etc)

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
