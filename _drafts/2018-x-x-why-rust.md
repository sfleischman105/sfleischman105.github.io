---
title: "Plucking hairs & jumping down the turtles: Why did I learn Rust?"
---

Not entirely sure.

As undergraduate CS student, I was plunged into the spinning wheel of industry standard languages. Java to start,
followed by Python, C, and C++. Each language flung me into a new world of programming idioms, each filled with 
their own insights and alternate ways to *pull out your hair*. Java is like pulling hairs out with a needle-node plier:
verbose, but it usually works. Python is like waving a magic wand over your hair *and somehow it disappears*. The 
equivilent to using a knife to pull out your hair would be C. It's *fast*, and my god you can cut any hair you want,
but every other cut **_somehow takes off a chunk of your skin with it._**. C++ is like C, but with a chainsaw. 

Eventually, It becomes boring pulling your hair out the same way every time. Maybe it takes too long to get out
the surgical equipment, or you're sick of missing with the knife and having a **hole in your arm**.

In the search for a new way to pull hair from my skin, I stumbled upon Rust. It offered promises of an industrial
strength language that allowed for both operation on the bare-metal side of the computer, as well as safety. This 
piqued my curiosity. The phrase "It's turtles all the way down" has become more of a prophecy than a idiom the further 
I've delved into Computer Science. Learning about computers is an act of falling from one turtle to another, hoping for
a bottom somewhere. I've learned there is, in fact, an end to the turtles. But that's simply the distraction. 
The thing that should be noticed is rather the fact the turtles are getting bigger, and far __*uglier*__ as you reach the bottom. 

I'm off the belief that there is an inherit trade-off in delving down the turtles: You know more, you can do more, but
it is _ugly_. Low-level languages have taught me quiet a large amount. In my first year at the undergraduate, I couldn't fathom why it 
was necessary to have both a `double` and `Double` type in java. Why can my java class contain a `double` variable, but once
I needed to place that variable in an `ArrayList`, it needs to be wrapped in a `Double`? Why do I need to grab a different tool
to pluck multiple hairs at once?

Upon learning C, I gained some insight.

C teaches you that the CPU, rather than being a magical meteorite sent from heaven, is rather a really dumb rock capable of 
doing simple things very fast. It all makes sense now, a `double` is stored in the stack, while a `Double` is a wrapper around
a heap-allocated `double` value. Ah-ha!

Unfortunately, learning C is the equivalent of getting near the bottom of the turtles. Yes, you can see feet somewhere down there. 
And yes, upon learning C *you've achieved enlightenment in the ways of the programming gods*.
But upon turning around you see that you're on the back of an *__ALLIGATOR SNAPPING TURTLE__* that requires the *__sacrifice of your 
first-born son__* in order to prevent `SIGSEGV` from being branded yet again into another part of your skin. 

Personally, I'm happy to have quiet a good amount of skin deteriorations to be able to see the feet of the bottom-most turtle.
It's fascinating, isn't it? Knowledge is power, and knowledge of the bare-metal computer allows for some further understanding of the
turtles above it. And *my goodness!* This turtle is **HUGE**, there's so much I can do down here! And this turtles *doesn't bat an eye*
if you want to play with fire, unlike the ones above it. 

This mute alligator snapping turtle is unfortunately filled with multiple poisonous swamps and no *danger* signs to mark them. It's 
uncomfortably easy to light a match down here and discover *__A SWAMP FILLED WITH METHANE__*, evaporating the wood cabin you've spent
hours building. It's a long trek back to this methane swamp, and it's even harder to find exactly where to place the danger sign.

Through a half year of Rust, I've come to re-evaluate my previous thoughts concerning this tradeoff. It's no fun wandering the swamps,
but it allows for freedom the turtles above lacked in. Through it's unique system of borrowing and mutating, Rust has given me a sense
of freedom while programming, alongside a knowledge of the overhead and costs of my actions, *AND* safety measures to prevent me from
doing anything terrible. Now, I can focus my efforts on building a log cabin with the right material, and worry about whether or not
the window curtains should be colored *burgendy* or *space grey*. No more accidentally removing the front wall while the roof is being
built. 


With this praise in mind, Rust still offers it's own ways of pulling out your hair. The Rust turtle (Or should I say Crustacean?) 
*requires* that when pulling out a hair, *none of hairs around it can be touched*. Any attempts to break this causes the crustacean
to grab you with it's pincers, bring you to it's meaty face, and then lectures you about the terrible things you are trying to do.

Also, the crustacean is currently missing a leg or two, but they're growing slowly. 



