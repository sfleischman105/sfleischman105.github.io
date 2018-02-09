---
title: "Plucking Hairs & Jumping Down the Turtles: Why did I learn Rust?"
---

As an undergraduate CS student, I was plunged into the spinning wheel of industry standard languages. Java to start,
followed by Python, C, and C++. Each language flung me into a new world of programming idioms, each filled with 
their own insights and alternate ways to *pull out your hair*. Java is like pulling hairs out with needle-node pliers:
a tad excessive _(can't I just use tweezers?)_, but it usually works. Python is like waving a magic wand over your hair 
*and somehow it disappears*. Attempting to use this magic over any large portion of the body will leave you with 
thoughts of _"Where did my arm go?"_ The equivalent to using a knife to pull out your hair would be C. It's *fast*,
and you can target any hair desired. But every other cut **_somehow takes off a chunk of your skin with it._**
C++ is like C, but with a chainsaw. 

Eventually, It becomes boring pulling your hair out the same way every time. Maybe it takes too long to get out
the surgical equipment. Or maybe you're sick of *missing with the knife* and having a **_hole in your arm_**. 

In the search for a new way to pull hair from my skin, I stumbled upon Rust. It offered promises of an 
industrial-strength language that allowed for both operation on the bare-metal side of the computer, as well as safety.
This piqued my curiosity, for I didn't believe it... _"Where is the little asterisk at the end of that sentence?"_
 
The phrase "It's turtles all the way down" has become more of a prophecy than a idiom the further 
I've delved into Computer Science. Learning about computers is an act of falling from one turtle to another, hoping for
a bottom somewhere. There is, in fact, an end to the turtles. But that's simply the distraction. 
Rather, it should be noticed that the turtles are getting bigger, and far __*uglier*__ as you reach the bottom. 

I'm of the belief that there is an inherit trade-off in delving down the turtles: You know more, you can do more, but
it is _ugly_. Low-level languages have taught me quite a large amount. In my first year at the undergraduate, I couldn't
fathom why it  was necessary to have both a `double` and `Double` type in java. _THEY BOTH HAVE THE SAME NAME AND DO THE 
SAME THING._ Why can my java class contain a `double` variable, but once I need to place that variable in an `ArrayList`,
it needs to be wrapped in a `Double`? 
I'm just trying to pluck multiple hairs at the same time, _WHY DO I NEED DIFFERENT PLIERS FROM THE SAME MANUFACTURER?_ 
What type of sorcery is this?

Learning C helped pull back the curtain. 

C teaches you that the CPU, rather than being a magical meteorite sent from heaven, is rather a really dumb rock capable of 
doing simple things very fast. _Ah-ha!_ It all makes sense now, a `double` is stored in the stack, while a `Double` is a
wrapper around a heap-allocated `double` value. 

Unfortunately, learning C is the equivalent of getting near the bottom of the turtles. Yes, you can see feet somewhere 
down there. And yes, upon learning C *you've achieved enlightenment in the ways of the programming gods*. But if you
glance around you'll notice that you're on the back of an *__ALLIGATOR SNAPPING TURTLE__* that requires the
*__sacrifice of your  first-born son__* in order to prevent `SIGSEGV` from being branded yet again into another
part of your skin. 

Personally, I'm happy to have quiet a good amount of skin deteriorations to be able to see the feet of the bottom-most turtle.
It's fascinating, isn't it? Knowledge is power, and knowledge of the bare-metal computer allows for some further understanding of the
turtles above it. And *my goodness!* This turtle is **HUGE**, there's so much I can do down here! And this ancient 
reptile *doesn't bat an eye* if you want to play with fire, unlike the ones above it. 

The mute alligator snapping turtle, _unfortunately_, has a back filled with multiple poisonous swamps,
and no danger signs to mark them. It's uncomfortably easy to light a match down here and
discover *__THE SWAMP WAS FILLED WITH METHANE.__*  That wood cabin you've spent hours building? _It's on fire now._ 
And the fire-department is a couple turtles up, 
introducing the `Optional` type to the people of Java-land. It's a long trek back to this methane swamp, and a lonely
one. _No one can hear you scream down here._ Equipped with a screwdriver and two coconuts, it's now on you to discover
the boundary of the sparky-boom-boom gas. Hey, freedom comes with a price, and that price is being able to discover 
pyromancy at inopportune times.

Through a half year of Rust, I've come to re-evaluate my previous thoughts concerning this tradeoff. _It's dangerous 
wandering the swamps, but it allows for freedom seldom granted by the turtles above._ Rust lives up to its claims, 
as there's no longer a choice between freedom and safety. _"Why not both?"_ says Rust, handing me a crumpled up piece of
paper detailing its unique system of borrowing and mutating. Now, I have the same sense of freedom while programming, 
alongside a knowledge of the overhead and costs of my actions. But it also includes safety measures to prevent me from
doing anything terrible. Equipped with my piece of paper, I can now focus my efforts on building a log cabin with the
right material, and spend my time worring about whether or not the window curtains should be colored burgundy or
space-grey. No more accidentally removing the front wall while the roof is being built. 

With this praise in mind, Rust still offers it's own ways of pulling out your hair. The Rust turtle (Or should I
say crustacean?) *requires* that when pulling out a hair, *though shall not touch any hairs in the vicinity*. Attempts
to break this causes the crustacean to grab you with it's pincers, swinging you around to it's meaty face, right before
belching out a lecture about the **terrible things** you are trying to do. Learning how to pluck my hairs with Rust has
been the worst part so far. It's a learning curve too steep to bike up. On top of this, the crustacean is more picky 
than even the topmost turtles. It's annoying, but the crustacean is doing this to protect you...
*it's probably not a good idea to take out a wall while the roof is on anyways*.

Despite the possibility of never using Rust in an industrial project, I wouldn't regret learning it. Alike C, Rust
has shown me a new way to look at the world of programming. The concepts of immutability and borrowing were seldom
taught explicitly in my computer science classes. Yet, they are concepts applicable to any programming language.
These concepts have changed the way I program, allowing me to write code that is safer, and more
logically organized. 

For myself, there is no direct answer to the question, *"Why did I learn Rust?"*, but I'm thankful for the lessons the
crustacean has taught me, and still teaches me to this day.


