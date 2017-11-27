---
title: "Pleco: Using Rayon.rs for a Ridiculously Easy Chess AI"
---

My name is Stephen, and for the past half year I've been working on 
[Pleco](https://github.com/sfleischman105/Pleco), a Rust-based Chess Library and AI. 

I want to give a brief overview of how a Chess AI / Searcher actually works, and how
incredibly easy it's been to add parallel searching using [rayon.rs](https://github.com/rayon-rs/rayon).

If you haven't read my previous blog post about Pleco, 
[please give it a read!](https://sfleischman105.github.io/2017/10/26/creating-a-chess-engine.html)
It provides a nice high-level overview to the inner workings of Pleco, and how we're able
to use rust's many Zero-Cost abstrations to create a chessboard that can be sent cloned and 
sent between threads easily.

### How does a Chess AI Work?

The goal of a chess AI is to find the best move available for the current position. We do this
by looking through each available move, and evaluating the strength of that move. The Rust pseudo-code
of this looks like:

```rust
pub fn search(board: Board) -> BitMove {
    let all_moves = board.generate_moves();
    
    for mov in all_moves {
        evaluate(mov);
    }
    return best_scoring_move;
}
```

So far so good! But how do we evaluate a move? What makes one move better than another?

The answer lies not with *evaluating the best move*, but *evaluating the best position that move leads to*. 
Rather than attempting to evaluate the move, we need to evaluate the strength of the board after that move is
applied. 

```rust
pub fn search(board: mut Board) -> BitMove {
    let all_moves = board.generate_moves();
    
    for mov in all_moves {
        board.apply_move(mov);
        evaluate(board);
        board.undo_move(mov);
    }
    return best_scoring_move;
}
```

Unfortunately, simply evaluating the position after a move wouldn't lead to a very smart chess AI. 
As of now, we're failing to consider future moves! It'd be wise to consider what our opponent may do
in response to our move, as well as considering our response to that move as well! 

