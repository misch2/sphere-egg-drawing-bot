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


