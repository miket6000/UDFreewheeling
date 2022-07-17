# UDFreewheeling
A Freewheeling DRSSTC controller based loosely on the UD2.7

Follow the development history at https://highvoltageforum.net/index.php?topic=2054.0

The driver behind building a new Tesla Coil Controller was to create something as simple to build and use as the UD2.7 (https://loneoceans.com/labs/ud27/) but with the below improvements:
* Freewheeling over current protection, this limits the maximum current without terminating the burst
* Over current and current phase detected with a single current transformer (CT)
* Removed the need for difficult to obtain variable inductors
* Self oscillating for improved startup at lower feedback currents

Huge thanks to the assistance of all those at highvoltageforum.net and especially David Knierim for his extensive contributions.
