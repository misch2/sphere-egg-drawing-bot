# TODO notes for the sphere/egg drawing bot

I was very eager to build an eggbot/spherebot and I thought that HW will be the most challenging part of this. But it was a bit more complicated.

Here are my notes of what to be aware of and what to use:

Hardware:
- Arduino Uno - any cheap clone with suitable USB controller is fine
- Arduino CNC shield
- DRV8825 drivers
- NEMA 17 stepper motors
- SG90 servo

Firmware: grbl-servo - https://github.com/cprezzi/grbl-servo. 

Yes, the last commit is 6 years old. No, it's not a problem because the CNC shield doesn't need any fancy driver, this firmware is perfectly suitable for eggbot in 2018 as well as in 2025.

This firmware allows us to use G-code for controlling the eggbot. This is IMO much better than a specialized eggbot firmware because any G-code producing software can now be used. We can also use some GRBL GUI like [GRBL-Panel](https://github.com/gerritv/Grbl-Panel/releases) etc. to verify that hardware is working.

Servo needs to be connected to:
 - GND (black/brown) - any GND pin on the shield
 - +5V (red) - any 5V pin on the shield. Just be aware that the servo needs some power so the USB power source should provide at least 0.5 A
 - PWM (orange) - "Z+" pin in the "end stops" section on the shield (top right).

Servo can be controlled by commands `M3` (start), `M5` (stop) and ideally `M3 S<0-255>` to set the pen height more precisely. 
Use `$30=255` to set the top limit for servo to 255 (originally it's set to 10000 because the FW expected spindle RPM here). The $ variables are saved to EEPROM IMHO so it doesn't need to be explicitly initialized on every restart.

I checked the PWM using oscilloscope and it looks like S70 produces 1ms pulses while S200 produces 2ms pulses so any value between these is suitable for the SG90 servo.


### Coords system:

- X = rotate egg (=long side of the virtual rectangle)
- Y = rotate pen (=short side of the virtual rectangle)
- Point \[0; ???/2\] is at the center of the egg

Z axis is not used, instead the "M3 S70" and "M3 S100" is used to raise or lower the pen (adjust the height as necessary).

The coordinate system is expected be positive to the top right with a zero at the bottom left.

### Units

My stepper motor uses 200 steps per full revolution:

```
steps_per_revolution = 200
```

The DRV8825 drivers can be set up to 32 microsteps per step:

```
microsteps = 32
```

The X axis motor has full 360 degrees range while the Y axis can swing approx 90 degrees:

```
x_rotation_degrees = 360
y_rotation_degrees = 90
```

Therefore it has 200 * 16 = 3200 steps per revolution, i.e. 3200 virtual "pixels" around the egg axis.

```
x_size = steps_per_revolution * microsteps * x_rotation_degrees/360   # 6400
y_size = steps_per_revolution * microsteps * y_rotation_degrees/360   # 1600
```

Let's convert this to physical units because that's what GRBL expects. It's also a lot easier to specify velocity or acceleration in physical units. An average egg may have diameter around 45 mm so we will use this for an approximate conversion:

```
diameter_mm = 45
```

```
circumference_mm = diameter_mm * 3.14   # 141
steps_per_mm = x_size / circumference_mm   # 45
```



GRBL uses physical (metric or imperial) distances so let's say we want 100 mm to be a full circle (3200 steps). In that case:
$100 (=$101) = 3200/100 = 32 steps/mm

Speed should also be adjusted. For a full rotation in let's say 5 seconds it would be 100 mm steps per 5 seconds i.e. 100 * (60/5) = 1200 mm per minute:
$110 (=$111) = 100mm * (60 sec / 5 seconds) = 1200 mm/min

Similarly for the acceleration:
$120 (=$121) = for example 20 mm / 1sec^2
