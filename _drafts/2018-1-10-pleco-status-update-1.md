---
title: "Pleco: A Half Year of Creating a Chess Engine with Rust"
---

Hi! My name is Stephen (or Bradly!), and for the past half year I've been working on 
[Pleco](https://github.com/sfleischman105/Pleco), a Rust-based Chess Library and AI. 
This project is primarily a re-write of the C++ chess engine 
[Stockfish](https://github.com/official-stockfish/Stockfish).

This post will talk about both the progress made so far, as well as my personal experience 
using Rust to create this project.

#### Why Rust?

If you want speed, C or C++ is the way to go. Unfortunately, both of those languages are a tad frustrating at times.
Somehow, I stumbled upon rust, which offered *blazingly fast speed, segfaults prevention, and guaranteed thread safety.*
Alright, that sounds good enough for me. 

#### How far along is Pleco?

Currently at ~12,200 LoC in the project, and ~17,500 including documentation.

So, very far along. The chessboard, move-generation, statically generated tables, all fully complete.
Pleco even have some tools up and running, such as a Transposition Table, a Pawn Table to store information about
specific pawn-configurations, and a Material Table to store information about combinations of non-pawn pieces.
Most recently, the board evaluation was fully completed.



Entry into tournaments usually requires a chess engine to
use the UCI (Universal Chess Interface) for engine-gui communication. Pleco has that implemented,
and prelimanary tests have shown it works fairly well. It still needs more testing though.

#### Does Pleco have a rating?

Officially, no. But preliminary matches on my own computer have estimated a rating of around ~1900 ELO.

#### What work needs to be done?





