---
title: "LED Replica Glasses - Progress Report"
---

As a side project, I've been trying to recreate 
[REZZ's Crazy LED glasses](assets/images/ledglasses/rezz.jpg). 
Seemed like a great and usable introduction to embedded programming, as well
as using fields outside of programming (Electronics, 3D Design, possibly music). 

There are [other replicas](https://www.etsy.com/listing/545785280/arcane-led-goggles-88-full-color-leds?ga_order=most_relevant&ga_search_type=all&ga_view_type=gallery&ga_search_query=led%20goggles&ref=sr_gallery-1-1)
currently made, most of them using various [NeoPixel rings](https://www.adafruit.com/product/1463)
wired together. These are all impressive works, but they are a tad too different from the original
glasses for my liking, I'd rather use through-hole pixels than the SMD versions.

A Teensy3 microcontroller is being used to power the glasses, due to its large memory, cost, 
and slim profile. Through-hole WS2812 LED's ("NeoPixel") were chosen due to being individually
addressable, as well as being the right size. 

Using custom Printed Circuit Boards seemed like the best way to go about this. With such a small
space inside each frame, hand soldering LEDs together would be extremely cumbersome, and take
up too much space as well. NeoPixels also advise the inclusion of a 0.1µF capacitor between each
pixel, to prevent misbehavior. It's unlikely a capacitor would also fit inside the frame.

## PCB and Circuit Layouts

Each NeoPixel has a `data_in`, `data_out`, `5V`, and `GND` pins. The first pixel starts at 9-0'clock
on the board's outer ring, and continues clockwise until connecting to the next inner ring. There 
are a total of 44 pixels; 24 on the outside, 16 in the middle, and 4 on the inside. 

KiCAD was used to design the PCB, with the outline being imported from a AutoDesk Fusion 360 sketch. 
Getting the LED holes and Capacitor plates to the right radial position and angle was achieved by
using KiCad's Python scripting console (so nifty!). Being my first time ordering PCBs, I didn't know
if it was possible to connect the bridges between the rings, so I included pins on the internal rings
for the data/power/ground to be hand wired.

![sketch1](assets/images/ledglasses/PCB_sketch1.jpg)

OSHPark was used to print, turning out very well. _(The purple color is lovely)_

![sketch1](assets/images/ledglasses/PCB1_initial.jpg)


## Soldering
I've never soldered before, so this was absolutely a learning experience

Firstly, SMD 0.1µF capacitors were soldered using a Reflow oven.

(This was a huge pain, I underestimated how small the capacitors would really be)

![Capacitor Soldered](assets/images/ledglasses/PCB1_cap_solder.jpg)

Now for the LEDs...

![Half Soldered Back](assets/images/ledglasses/PCB1_led_solder_b.jpg)

![Half Soldered Front](assets/images/ledglasses/PCB1_led_solder_f.jpg)

And some testing...

![Half Soldered Testing](assets/images/ledglasses/PCB1_test_wiring.jpg)

Finally, the rest.

![One Fully Soldered](assets/images/ledglasses/PCB1_led_solder_final.jpg)

![Both Fully Soldered](assets/images/ledglasses/PCB1_final.jpg)

They turned out great! 

## 3d Printed Front Panel

I printed out a dummy front panel for just one side. The LEDs fit fairly snugly inside - 
no need for any screws holding it in

![With 3d Printed mask](assets/images/ledglasses/3Dprint_front.jpg)

However, wiring turned out to be **_ugly_**.

![Ugly wiring](assets/images/ledglasses/PCB1_ugly_wires.jpg)

But hey, they work great!

## Second attempt

So some problem points with the first incarnation:
- Wiring between rings took too much space, looked bad, and were infuriatingly hard to attach wire to.
- Pin holes barely fit 22AWG wire.
- Pin hole lips are too small to solder wire and guarantee a good connection.
- `5V`/`GND` tracks do not need to come from one direction.
- `5V` tracks should be wider for accommodation of bigger currents.

So, I redesigned and ordered another sets of PCBs to accommodate this.

Changes:
- Added Bridges between the rings, with tracks between for Data/Power/Ground.
- Increased width of `5V` tracks.
- `5V` and `GND` tracks branch at multiple spots to/from the inner rings, for better voltage distribution
and flow.
- Larger pin holes for 20AWG wires coming in, and 202WG wires between the two eyepieces.
- Wider pin hole lips for better soldering connection.
- Track Cleanup. 


![sketch2](assets/images/ledglasses/PCB_sketch2.jpg)


Future steps:
1) Model, 3D print, and assemble the glasses with the new frames.
2) Create a box to hold the Circuitry (Teensy, batteries, etc)
3) Attach teensy to a bluetooth controller
4) Create app to communicate with bluetooth trinket.


