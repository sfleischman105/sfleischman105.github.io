---
title: "Porting a C++ Chess Engine to Rust"
---

Hi! My name is Stephen (or Bradly!), I'm the creator and maintainer of 
[Pleco](https://github.com/sfleischman105/Pleco), a Rust-based Chess Library and AI. 

First and foremost, Pleco is a Rust port of the Stockfish chess engine.


#### Enums

The fact that `Enums` are called by the same name in C++ and Rust is misleading. Yes, they
both "enumerate" things, and yes, the both provide a provide a way of making named subsets 
of types. But besides that, I see more similarities between potatoes and uranium.

Take this C++ enum, describing the type of pieces on a board:

```C++
enum PieceType {
  NO_PIECE_TYPE, PAWN, KNIGHT, BISHOP, ROOK, QUEEN, KING,
  ALL_PIECES = 0,
  PIECE_TYPE_NB = 8
};
```

Not only does this enum describe the pieces on the board, it also details a few
non-pieces as well. `NO_PIECE_TYPE` is for the absence of a piece, similar to null.
Okay, I guess the absence of a piece is a type of piece. But now we run into `ALL_PIECES`,
describing the superset of all pieces. Huh? _"A smoothie is not a type of fruit, therefore
all pieces isn't a type of piece."_ Adding to this exotic blend of definitions, there's also a 
defined array size, `PIECE_TYPE_NB`, for arrays needing an index per piece. 

Oh, and the values aren't discriminate, as both `ALL_PIECES` and `NO_PIECE_TYPE` have the same underlying
integer representation of `0`. 

Anyways, Rust takes a slightly different path for enumerations. Rather than being a series of named constants backed
by an integer, rust allows for a enumerations which contain data as well. This is nice, but doesn't exactly translate
one-to-one with the C++ representation. 

If we were to construct a `PieceType` Rust enum naively (as in, disregarding the want to translate from this C++ code),
it would look something like this:

```rust 
enum PieceType {
    Pawn,
    Knight,
    Bishop,
    Rook,
    Queen,
    King
}
```



- _"What about the `NO_PIECE_TYPE` discriminant?"_

Easy, we use a `Option<PieceType>` to describe this.

- _"And the `PIECE_TYPE_NB` discriminant?"_

We can define a constant which expresses the number of `PieceType` discriminants: `const PIECE_TYPE_NB: usize = 6;`

- _"You're forgetting the `ALL_PIECES`."_

Let's throw it out the window and work around it.

This was my original way of thinking about representing a `PieceType` in Pleco. However, I've found that the specific
use cases of a C++ enum do not directly translate to rust so nicely, but come with benefits in performance and an
underlying integer representation. 




#### Global State


#### Threading