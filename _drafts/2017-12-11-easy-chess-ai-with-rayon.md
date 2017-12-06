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

The formulaic equivalent of this is:

M<sub>b</sub> = max(eval(M<sub>0</sub>)...eval(M<sub>i</sub>))

Where `M` is the list of moves for the current position, `b` being the index of the best move,
and `i` being the length of our moves.

Unfortunately, simply evaluating the position after a move wouldn't lead to a very smart chess AI. 
As of now, we're failing to consider future moves! It'd be wise to consider what our opponent may do
in response to our move, as well as considering our response to that move as well! 

### MiniMax

The algorithm we need to use is [MiniMax](https://en.wikipedia.org/wiki/Minimax), a recursive algorithm
that determines the best possible move by evaluation the opponent's next possible moves.

As an example of this, let's take a random move from our list of moves, labelled as M<sub>r</sub>. Now, to score 
this move, we have to consider the moves our opponent must play. We are most concerned about our opponent's best
response to this move. 

After applying this move, the opponent has a list of moves `N` of length `i`. Their best move is given by:

N<sub>b</sub> = max(value(N<sub>0</sub>)...value(N<sub>i</sub>))

Now, since we are evaluating from our opponent's perspective, we need to negate the value of N<sub>b</sub> to get
the final score of M<sub>r</sub>:

M<sub>r</sub> = -N<sub>b</sub> = -max(value(N<sub>0</sub>)...value(N<sub>i</sub>))

Now, we apply this algorithm recursively to determine the best move for any position!

In rust code, here is the result:

```rust
pub const NEG_INFINITY: i16 = -30001;

pub struct BestMove {
    pub mov: Option<BitMove>,
    pub score: i16,
}

impl BestMove {
    pub fn negate(mut self) -> Self {
        self.score = self.score.wrapping_neg();
        self
    }
    
    pub fn new(value: i16) -> Self {
        BestMove {
            mov: None,
            score: value
           }
        }
    }
}

pub fn minimax(board: &mut Board) -> BestMove {
    let mut best_move = BestMove::new(NEG_INFINITY);
    
    let moves = board.generate_moves();
    
    for mov in moves {
        board.apply_move(mov);
        let returned_move: BestMove = minimax(board, max_depth).negate();
        board.undo_move();
        if returned_move.score > best_move.score {
            best_move.score = returned_move.score;
            best_movemov = Some(mov);
        }
    }
    best_move
}
```

But, if we attempt to use this, we'll notice this algorithm runs forever. We forgot to add a base case!

Furthermore, we're not dealing with the case of their being no possible moves available, such as when the board
is in check, or their is a stalemate.


```rust
...

pub const STALEMATE: i16 = -25000;
pub const MATE: i16 = 0;

pub fn minimax(board: &mut Board, max_depth) -> BestMove {
    if board.depth() >= max_depth {
        return eval_board(board);
    }
    
    let moves = board.generate_moves();
    
    if moves.is_empty() {
        if board.in_check() {
            return BestMove::new(STALEMATE);
        } else {
            return BestMove::new(MATE);
        }
    }
    
    let mut best_move = BestMove::new(NEG_INFINITY);
    
    for mov in moves {
        board.apply_move(mov);
        let returned_move: BestMove = minimax(board, max_depth).negate();
        board.undo_move();
        if returned_move.score > best_value {
            best_move.score = returned_move.score;
            best_movemov = Some(mov);
        }
    }
    best_move
}
```

A++

### Parallizing MiniMax



