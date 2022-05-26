
Notes on sensors:


Step counter will be active all the time so make sure accelerometer clock is slow (and clock divider is low).
For example, with 50kHz clock and divider 1935 there is an output data rate of 26Hz, which should be fine for steps.
This adds some 8uA to power draw, which is fine.
To set this combination, write 0xe2 to register 0x10 and 0xc4 to register 0x11

To activate step counter, write 0x94 or 0x8c to register 0x12
Read out steps from registers 0x7 and 0x8: steps=(reg_0x8<<8)|reg_0x7;

Sensor orientation is different for accelerometer and magnetometer.
Accelerometer positive X is to left, positive Y is away from USB.
Magnetometer positive X is away from USB, positive Y is to the right.

To convert magnetometer coordinates to accelerometer coordinates:

def convert_to_accel_coords(mag_x, mag_y, mag_z):
	return(-mag_y, mag_x, mag_z)

