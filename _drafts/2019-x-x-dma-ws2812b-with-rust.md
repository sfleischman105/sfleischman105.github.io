---
title: "Embedded Rust Adventures and Insanity"
---

Hi, my name is Stephen (or Bradly) and I've been dabbling into using Rust in 
embedded systems. 

One of my favorite things in the world is fancy light-displays. Thousands of LEDs 
synchronized to create a beautiful display. And even better, we can create similar
arrangements at home, using addressable LED's like the APA106 and WS2812B, also known as
NeoPixels. Generally these are driven through through hobbyist MicroController systems
alike Arduino. Since I like Rust, I like embedded system, and I love these colorful
lights, why not combine these things together? 

Written in Rust, I've managed to successfully create a way to drive these LEDs from an
microcontroller. More so, it is asynchronous, using hardware peripherals to generate
the signal without wasting cpu cycles. I believe this implementation is the fastest 
I've seen, due to some bitbanding techniques used later. This stuff was a technical
joy to see work in action, and this post will be dedicated to documenting the process
of creating these.

## Addressable LEDs Protocol

Addressable LED's are red-blue-green LEDs controlled through a single data wire. 
More so, they can be chained together and individually addressed (hence the 
addressable part), allowing for control of nearly unlimited pixels. 

Each color component takes up a single byte, so each LED uses 3 bytes of data.
Writing `N` LEDs is done by first sending a reset code, and then writing the LEDS
to the data wire starting at LED 1, and ending at LED N.

Each protocol has a specific color order each LED must be sent in. In this post,
the [WS2812B/Neopixel](https://cdn-shop.adafruit.com/datasheets/WS2812B.pdf) 
protocol will be used, which has a sending order of G->R->B. The diagram below 
shows this nicely.

PICTURE

Sending a single bit through the wire is done through pulling the data line to
high voltage for a set amount of time, and then pulling the data line low. 
Writing a `1` vs a `0` is differentiated by the amount of time spent in 
the voltage high vs voltage low. 

PICTURE 2

It's important to note that writing a `1` vs `0` takes the same amount of time,
as we'll use this information later. Also notably, the reset code is simply
pulling the line low for at least 50 micro-seconds. 


## Color/RGB Library

Before we start creating drivers, it'd be great to have a way to represent
and manipulate RGB's in Rust. We want something along the lines of:

```rust
pub struct ColorRGB {
    pub r: u8,
    pub g: u8,
    pub b: u8
}
```

For this reason, I've written a library to help with color RGB manipulation, 
[cichlid](https://github.com/sfleischman105/cichlid)!

It allows for easy and fast manipulation of RGBs, as well as many other useful 
functions for dealing with addressable LEDs. It's based primarily on the 
arduino library FastLED, but only implements the portions dealing with color 
and other device-agnostic features.

We'll be using cichlid to abstract away dealing with colors.

Onwards!

## Devices / Software used

The board I'm using is the STM32F3DISCOVERY. The `cortex-m`, `cortex-m-rt`, 
and `stm32f3` crates are used as well to easily interface with the
board's peripherals. 

 
## Software-Based Writing

In order to show the utility of hardware-based displaying of WS2812's, we're
going to implement the driver manually in software first. This'll also
help with understanding the protocol for later sections.

```rust
#[entry]
fn main() -> ! {

}
```

## Asycnhronous Writing

### Capture Compare Events

### DMA to registers

These two generally the same

### Optimzation with Bit Set / Reset

## Buffer Setting Tecnhiqus

## Advanced Buffer BitBanding Techniques

### Possible Future

Talk about presetting 2 + (1 * NUM STRANDS) LEDS in advance
Doesn't allow for less CPU (perhaps more) but 

## Outline of requirements

- Fast CPU
- Timer with 2 Compature/Compare Regs
    - DMA Single Transfer per event
    - Timer Interrupt
    - DMA Interupt on Complete
        - Can use Timer interrupt for this, but less concise
