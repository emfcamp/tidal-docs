# Badge API reference

All these assume the tidal module has already been imported with
```
>>> import tidal
```

## LED/torch


To turn LED power on
```
>>> tidal.led_power_on()
```

To turn LED power off

```
>>> tidal.led_power_off()
```

To set the LED colour (values are red, green, blue):

```
>>> tidal.led[0]=(0,0,255) #set LED to blue, full brightness
>>> tidal.led.write()
```

Colours are a combination of red, green, and blue. Each colour channel goes from 0 to 255.

To get white light at half brightness, use (127,127,127).

To get red light at full brightness, use (255,0,0)


## Display power

The LCD power enable and backlight enable are controlled separately.
The backlight can be enabled without powering the LCD module on.

to power on the display (this will also power on the backlight, but you can then turn it off again if you like):
```
>>> tidal.lcd_power_on();
```

to power off the display (this will also power off the backlight, but you can then turn it on again if you like):
```
>>> tidal.lcd_power_off();
```

When the display is off, it will retain the image stored, so when you switch it on again the same image that was there last time will be displayed.

to power on the backlight:
```
>>> tidal.lcd_backlight_on();
```

to power off the backlight:
```
>>> tidal.lcd_backlight_off();
```

When the display is powered on but the backlight is powered off, users can press the BL button to view the screen contents. The backlight will
illuminate while the button is pressed. For especially low power operation, you can power off the display, and then power it back on when the BL button is pressed.



## General GPIO (G0, G1, G2, G3)

These are micropython Pin objects connected to the pins on the three expansion connectors. By default they are configured as inputs with pullups.
If you reconfigure these pins, make sure that they are not left floating - either configure them as outputs, or inputs with pullup.

```
>>> tidal.G0.value()
1
```

## Charge detect

the charge detect line comes from the battery charger and is 0 for charging and 1 for not.

```
>>> tidal.CHARGE_DET.value()
0
```


