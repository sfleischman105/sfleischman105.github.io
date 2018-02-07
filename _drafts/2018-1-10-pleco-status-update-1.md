---
title: "Pleco: A Half Year of Creating a Chess Engine with Rust"
---

Hi! My name is Stephen (or Bradly!), and for the past half year I've been working on 
[Pleco](https://github.com/sfleischman105/Pleco), a Rust-based Chess Library and AI. 

This post will talk about both the progress made so far, as well as my personal experience 
using Rust to create this project.

I've found that the easiest way to effectively communicate something is to _pretend to 
be answering a question._ So, this'll be a series of responses to my own questions. woo.

#### Why a Chess Engine?

In one of my computer science classes, we were tasked with creating a couple chess-bots. 
One of which, rather than implementing a specific searching algorithm, had simply the goal of
beating other bots created by the instructor. This consisted of taking the bots previously made,
and modifying them to be faster and stronger. 

Many unnecessary all-nighters were pulled in the search for speed. It felt good. So good, It scratched that
some part of my mind. The part that yearns for raw speed and efficiency. The part craving the hypnotic cycle of
tinker-run-measure.

Unfortunately, the assignment had to end eventually. But it left me with this new craving of building something
*fast*. I figured creating a new engine from the ground up might scratch this itch as well.

#### Why Rust?

If you want speed, C or C++ is the way to go. Unfortunately, both of those languages are a tad frustrating at times.
Somehow, I stumbled upon rust, which offered *blazingly fast speed, segfaults prevention, and guaranteed thread safety.*
Alright, that sounds good enough for me. 

#### How far along is Pleco?

Currently at ~11,342 LoC in the project so far, and ~16,835 including documentation.

But as of now, very far along. Entry into tournaments usually requires a chess engine to
use the UCI (Universal Chess Interface) for engine-gui communication. Pleco has that implemented,
and prelimanary tests have shown it works fairly well. It still needs more testing though.

#### Does Pleco have a rating?

Officially, no. But tournaments on my own computer have estimated a rating of around ~1900 ELO.


