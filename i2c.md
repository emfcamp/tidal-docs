# I2C examples

Some examples of reading and writing to devices on the tidal-mk6 badge in micropython. Note, some of these examples may leave the chips in a different state to their original and therefore the chips may need to be reset before any other examples are attempted.

## Getting started


```
import tidal
from tidal import i2c
```

if you want to use the peripheral i2c (for example on the expansion pads on the bottom or on the two ribbon cable connectors) you will need to activate it:

```
tidal.enable_peripheral_I2C()
from tidal import i2cp
```


scan the i2c bus for devices:

```
>>> i2c.scan()                                                                   
[18, 44, 100]                                                                     
```
There should be three devices show, 18 is the QMA7981 accelerometer, 44 is the QMC7983 magnetic sensor and 100 is the cryptography chip


or, for the peripheral i2c:

```
>>> i2cp.scan()                                                                   
```



## QMA7981

Read the identity byte from register 0x00:

```
>>> i2c.readfrom_mem(18,0,1)
b'\xe7'                                                                          
```

interrupt reading (the values assume the chip has an interrupt asserted, if not the values will be oposite):
```
>>> from machine import Pin
>>> irq = Pin(40, Pin.IN, Pin.PULL_UP)
>>> irq.value()
0
```

set interrupt pin to active high:
```
>>> i2c.writeto_mem(18, 0x20, 4);
>>> irq.value();
5
```

set interrupt pin back to active low:
```
>>> i2c.writeto_mem(18, 0x20, b'5')                                              
>>> irq.value()                                                                  
0                                                                                
```

to do a power-on reset...
```
>>> i2c.writeto_mem(18, 0x36, b'\xb6')
```

read the power-state:
>>> bytes = i2c.readfrom_mem(18, 0x11, 1)                                        
>>> print("{}".format(bytes[0]))                                                 

read the data int status:
```
>>> bytes = i2c.readfrom_mem(18, 0x0b, 1)
>>> print("{}".format(bytes[0]))
```

initialse settings to 2G, 1024Hz bandwidth
```
>>> i2c.writeto_mem(18, 0x11, b'\xc0')
>>> i2c.writeto_mem(18, 0xf,  b'\x01')
>>> i2c.writeto_mem(18, 0x10,  b'\x05')
```

do a simple read of the x/y/z acceleration values:

```
>>> i2c.writeto_mem(18, 0x11, b'\xc0')
>>> bytes = i2c.readfrom_mem(18, 1, 6)
>>> x = (bytes[0] >> 2) | (bytes[1] << 6)
>>> y = (bytes[2] >> 2) | (bytes[3] << 6)
>>> z = (bytes[4] >> 2) | (bytes[5] << 6)
>>> print("{} {} {}".format(x, y, z))
```

## QMC7983


Read the chip identification register:
```
>>> i2c.readfrom_mem(44, 0xD, 1)
b'2'                                                                             
```

do a basic initialisation, as per the datasheet

```
>>> i2c.writeto_mem(44, 0xb, b'\x0f')
>>> i2c.writeto_mem(44, 0x9, b'\x3d')
```

read all the data registers

```
>>> bytes = i2c.readfrom_mem(44, 0x0, 8)
>>> print("{}".format(bytes))
b'\xd8\x07\x19\n\x8e_\x00\xfe'                                                   
```




# References

- https://github.com/espressif/esp-who/blob/master/components/modules/imu/qma7981.c
- https://datasheet.lcsc.com/lcsc/2004281102_QST-QMA7981_C457290.pdf
- http://www.siitek.com.cn/Upfiles/down/QMC7983%20Datasheet%20-NCP-Rev1.0.pdf
