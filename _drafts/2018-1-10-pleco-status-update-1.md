---
title: "Pleco: A Half Year of Creating a Chess Engine with Rust"
---

Hi! My name is Stephen (or Bradly!), and for the past half year I've been working on 
[Pleco](https://github.com/sfleischman105/Pleco), a Rust-based Chess Library and AI. 

This post will talk about both the progress made so far, as well as my personal experience 
using Rust to create this project.

I've found that the easiest way to effectively communicate something is to _pretend to 
be answering a question._ So, this'll be a series of responses to my own questions. woo.

#### Why did you use Rust to make a chess engine?

Not entirely sure.

I'm currently an undergraduate CS student, and I've been versed in languages commonly taught 
in academia. Java was the start, and soon that led to Python, C, and C++. All of these languages I hold dear, and 
I doubt I'd be able to make the jump to Rust without the previous knowledge I had of these language. 

The luxury of being a student is in the ability to pursue side projects while also furthering your studies.
Rust seemed like a foreign language, but with the added bonus of _real_ practical functionality. Being a language
that allowed operation on the bare-metal side of the computer piqued my curiosity. The phrase "It's turtles all 
the way down" has become more of a prophecy than a idiom the further I've delved into Computer Science. Learning
about computers is an act of falling from one turtle to another, hoping for a bottom somewhere. I've learned there
is, in fact, an end to the turtles. But that's simply the distraction. The thing that should be noticed is rather
the fact the turtles are getting bigger, and far __*uglier*__ as you reach the bottom. 

The journey I've experienced in learning computer science has exemplified a belief that there is an inherit
tradeoff in delving down the turtles: The more you know, the uglier it is. Low-level languages have taught me
quiet a large amount. In my first year at the undergraduate, I couldn't fathom why it was necessary to have both a
`double` and `Double` type in java. Why can my jave class contain a `double` variable, but once I needed to place that
variable in an `ArrayList`, it needs to be wrapped in a `Double`? Doesn't make any sense.

...Until I learnt the programming language C. 

C teaches you that the CPU, rather than being a magical meteorite sent from heaven, is rather a really dumb rock capable of 
doing simple things very fast. It all makes sense now, a `double` is stored in the stack, while a `Double` is a wrapper around
a heap-allocated `double` value. 

Unfortunately, learning C is the equivalent of getting near the bottom of the turtles. Yes, you can see feet somewhere down there. 
And yes, upon learning C *you've achieved enlightenment in the ways of the programming gods*.
But upon turning around you see that you're on the back of an *__ALLIGATOR SNAPPING TURTLE__* that requires the *__sacrifice of your 
first-born son__* in order to prevent `SIGSEGV` from being branded yet again into another part of your skin. 

Personally, I'm happy to have quiet a good amount of skin deteriorations to be able to see the feet of the bottom-most turtle.
It's fascinating, isn't it? Knowledge is power, and knowledge of the bare-metal computer allows for some further understanding of the
turtles above it. However, it's a very ugly and larg turtle 


