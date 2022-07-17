# UDFreewheeling
A Freewheeling DRSSTC controller based loosely on the UD2.7

Follow the development history at https://highvoltageforum.net/index.php?topic=2054.0

The driver behind building a new Tesla Coil Controller was to create something as simple to build and use as the UD2.7 (https://loneoceans.com/labs/ud27/) but with the below improvements:
* Freewheeling over current protection, this limits the maximum current without terminating the burst
* Over current and current phase detected with a single current transformer (CT)
* Removed the need for difficult to obtain variable inductors
* Self oscillating for improved startup at lower feedback currents

There are a couple of breaking changes that are worth pointing out:
* While the board outline is approximately the same as UD2.7 I can't guarantee the mounting holes are in the exact same locations
* There is no longer the ability to disable phase lead, it wasn't obvious why this would be required and can be done by using a short instead of L1
* The location of the HFBR is approximately the same as the UD2.7 but the location of the IF-D95 has changed slightly

Huge thanks to the assistance of all those at highvoltageforum.net and especially David Knierim for his extensive contributions.

## How it works
### Power supply
There's nothing much here to talk about, there are three regulators, 24V, 9V and 5V. 24V and 5V are linear, the 9V should be a DC/DC converter such as  RECOM Power R-78E9.0-0.5 or similar. It is recommended for the input supply that you use a 1A 24V SMPS and the DC input. Do not use the DC and AC inputs at the same time.

### Phase detect
The CT should be designed to provide a current of ~500mA to 1A peak at maximum primary current.
C32 provides HF bypass for noise. The current from the CT passes through R3 || R8 L1 and R1 generating a voltage ton pin 1 of R8 hat leads the current by theta = arctan(Xl / R). Xl depends on the operating frequency of the primary but R just depends on the position of the wiper on R8 and can be set from 5.1R up to R3 || R9 which is approximately 22R. This voltage is then clamped to +/- 0.65V by the back to back diodes and capacitvely coupled into the inverting input of the comparator IC8.

IC8 is set up with both positive and negative feedback with the non-inverting input held at nominally 2.5V. If we initially ignore any input from the CT we can see that the comparator will attempt to pull the inverting input low. There will be a time constant set by C5 and (R21 + R25) which will mean this takes some amount of time but the voltage at the input will be reducing. Meanwhile when the input transitioned low C33, R26 and R7 immediately pulled the non-inverting input low but C33 will slowly charge so the voltage at the non-inverting input will be increasing.

After some amount of time the increasing voltage on the non-inverting input and the decreasing voltage in the inverting input will be equal and the pass each other. At this moment the comparator will toggle states at the output will go high. This will reverse our initial state and everything happens again, but with the voltages reversed, i.e. it oscillates. The oscillation frequency can be set by adjusting R25. This should be set as close as possible to the frequency of the primary.

When there is a signal from the CT it swamps the ability of the negative feedback from R21 + R25 to control the inverting input pin so is effectively overriden with the natural resonant frequency of the system.

