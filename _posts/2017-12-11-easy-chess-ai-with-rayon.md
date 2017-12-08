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

**M<sub>b</sub> = max(eval(M<sub>0</sub>)...eval(M<sub>i</sub>))**

Where `M` is the list of moves for the current position, `b` being the index of the best move,
and `i` being the length of our moves.

Unfortunately, simply evaluating the position after a move wouldn't lead to a very smart chess AI. 
As of now, we're failing to consider future moves! It'd be wise to consider what our opponent may do
in response to our move, as well as considering our response to that move as well! 

### MiniMax & NegaMax

The basis of our algorithm stems from [MiniMax](https://en.wikipedia.org/wiki/Minimax), a recursive algorithm
that determines the best possible move by evaluating the opponent's next possible moves. 

More specifically, most chess AI is based on [NegaMax](https://en.wikipedia.org/wiki/Negamax), a variation of MiniMax
that takes into account that chess is symmetric. e.g., For any position, the score for white is the opposite of the
score of black.

Negamax selects the best move to play by considering the opponent's best response.
__*The worse your opponent's best reply is, the better your move.*__

Modifying our equation from earlier, the best move for any position (and therefore the value of that position as well)
 is determined by:

**max(M<sub>i</sub>) = -min(value(M<sub>0</sub>)...value(M<sub>i</sub>))**

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

But, if we attempt to use this, we'll notice notice two problems:

1) It runs forever! As a recursive algorithm, we forgot to add a base case.
2) The value of a stalemate (where there are no legal moves to play) returns a score of negative infinity.

So, we'll define the base-case as when the algorithm reaches a certain __*depth*__. Depth is defined as the 
number of moves applied from the starting position. When the algorithm reaches it's maximum depth, instead of
determining the value of that position by evaluating the moves available, we'll determine the value through
evaluation of the board itself. On the most basic level, evaluation is simply summing up the value of
each piece on the board.

If there are no moves available to play, we can determine that there is either a stalemate or checkmate.
Checkmate is obviously the worst position available, so we'll give it an extremely low value. Stalemate leads
to a draw, so the value of that is zero.

With these two improvements, we are left with the following algorithm:

```rust
...

pub const STALEMATE: i16 = -25000;
pub const MATE: i16 = 0;

pub fn negamax(board: &mut Board, max_depth) -> BestMove {
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
        let returned_move: BestMove = negamax(board, max_depth).negate();
        board.undo_move();
        if returned_move.score > best_value {
            best_move.score = returned_move.score;
            best_movemov = Some(mov);
        }
    }
    best_move
}
```

### Parallizing NegaMax



