# UDFreewheeling
A Freewheeling DRSSTC controller based loosely on the UD2.7. See it running my 35cm coil here: (https://youtu.be/EUlrq9CbajI)

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

After some amount of time the increasing voltage on the non-inverting input and the decreasing voltage in the inverting input will be equal and the pass each other. At this moment the comparator will toggle states at the output will go high. This will reverse our initial state and everything happens again, but with the voltages reversed, i.e. it oscillates. The oscillation frequency can be set by adjusting R25. This should be set as close as possible to the frequency of the primary. To set the correct frequency ensure there is no feedback from the CT (i.e. disconnect the output stage or the CT) and measure the frequency on an oscilloscope. Adjust R25 until this matches your expected resonant frequency.

When there is a signal from the CT it swamps the ability of the negative feedback from R21 + R25 to control the inverting input pin so is effectively overriden with the natural resonant frequency of the system.

### Over current detection
The current through the CT creates a voltage across R1 (5R1). This is clamped between the 0V and 5V rails and feed into the inverting input of U5A. If the voltage exceeds the voltage set by R20 then the output of the comparator will go low signaling an over current event to the logic controlling the gate drivers. The over current threshold is determined by taking your intended over current peak, dividing it by the CT ratio and multiplying by 5.1V. R20 is then adjusted until this voltage is measured on TP6. E.G. I have a 300:1 CT and want the over current limit to be set to 200A, so I adjust R20 until I see 200A / 300 * 5.1V = 3.4V.

### Interrupt input
A signal present on the optical input of RX1 (active low) is inverted by IC4F so that the output is high when the signal is present (LED on). A signal present on the optical input of RX2 (active high) is first inverted and the pulls down the same node as RX1 which is then inverted again so that the output is again high when the signal is present. The two RX inputs are effectively OR'd together so it is not recommended to use both inputs at the same time. If RX2 is not placed it is necessary to either tie the input of IC4B low (by shorting RX2-RX1) or remove D12. Note, if you do decide to remove D12 it is still necessary to include R11 to prevent oscillation caused by a floating input to IC4B.

### Undervoltage Lockout (UVLO)
The under voltage lockout compares the 24V rail (divided by 6 and applied to the non-inverting input of U5B) to the reference voltage set by R27 applied to the inverting input of U5B. If the voltage drops below the reference voltage the output goes low signaling to the gate drive logic that an undervoltage has occured. The reference is set by dividing your desired UVLO voltage by 6 and adjusting R27 until you see the desired voltage on TP4. E.G. I want UVLO to trigger at 18V, 18V / 6 = 3V so I adjust R27 until I measure 3V on TP4.

### Gate drivers
This is borrowed directly from UD2.7C, the two UCC27423 dual inverting gate drivers are used to drive two FDD8424H complementry MOSFETs to run the gate drive transformers. The only difference is that there are seperate signals for the two GDTs. This means it is not possible to parallel the twou outputs, each needs to run its own GDT, one for each half of the full bridge. 

### Gate drive logic
This is the complex part. We'll start at the gate drivers and move backwards...
IC12 is a multiplexer. It has two sets of 4 inputs and will take one of the two sets of inputs and connect them to the outputs. The schematic uses a slightly different nomenclature than the datasheet but for each pair of input pins ('I0a' & 'I1a' are one pair) if the 'S' pin is low then the voltage at input 'I0a' is connected to output 'Za', if the 'S' pin is high then the voltage at 'I1a' is connected to 'Za'. All the pairs are switched at the same time. This is all assuming the Enable (E) pin is low, otherwise the outputs are all low.

We connect CLK to our 'S' pin. Looking at the 'a' and 'b' inputs which connect to GDT1 we can see that 'Za' and 'Zb' have complementy outputs, if 'Za' is high, 'Zb' is low and vice-verse. CLK is derived from our phase signal so always drives GDT1 in phase with the phase signal. Inputs 'c' and 'd' are more complex. They are driven by the putpus of IC6B so depending on the state of IC6B could be either driven in phase or out of phase with CLK. If they're driven in phase with CLK (i.e. 'Q' is high) then both sides of the H-bridge will be in phase with each other and there will be no voltage across the primary tank circuit, i.e. we're freewheeling. The currents in the LC circuit are free to slosh backwards and forwards without significant resistance from the bridge. If however 'Q' is low then the output to GDT2 is out of phase and the HVDC bus voltage will occure across the primary tank causing it to ring up.

IC6B samples IC6A's 'Qn' output and latches it for the next clock cycle on every rising 'CLK' edge. Under "normal" circumstances IC6A clocks in 5V on the 'D' pin so 'Qn' is 0V. 0V on the 'D' pin of IC6B means the drive is out of phase and we ring up the current. If however OCD is asserted low by the over current detect comparator at any stage during the clock cycle the output of IC6B's 'Qn' output will go high. It will stay high until the next clock edge samples the 'D' input and drives 'Qn' low again, but going low takes time, before that happens the state has already been samples by IC6B and locked in for the next cycle. The INTERRUPT signal on the 'set' pin just ensures that we're in the correct state (freewheeling off) in between on signals from the interrupter, otherwise we wouldn't be able to start.

IC5A will disable the output of IC12 whenever INTERRUPT is low and there is a rising edge on the CLK signal. It can also disable IC12 if its 'R' line is asserted low by IC5B which is used to latch the UVLO signal. This means that if the UVLO is triggered then the output will remain locked out until the next rising edge of INTERRUPT.


