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
lights, why not combine these things togethor? 

Written in Rust, I've managed to successfully create a way to drive these LEDs from an
microcontroller. More so, it is asychronous, using hardware peripherals to generate
the signal without wasting cpu cycles. I believe this implementation is the fastest 
I've seen, due to some bitbanding techniques used later. This stuff was a technical
joy to see work in action, and this post will be dedicated to documenting the process
of creating these.



## Addressable LEDs Protocol



## Color Library

cichlid!

## Software-Based Writing

talk about cycling

## Asycnhronous Writing

### Capture Compare Events

### DMA to registers

These two generally the same

### Optimzation with Bit Set / Reset

## Buffer Setting Tecnhiqus

## Advanced Buffer BitBanding Techniques

## Outline of requirements

- Fast CPU
- Timer with 2 Compature/Compare Regs
    - DMA Single Transfer per event
    - Timer Interrupt
    - DMA Interupt on Complete
        - Can use Timer interrupt for this, but less concise
