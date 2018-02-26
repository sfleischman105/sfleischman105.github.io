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
to use rust's many Zero-Cost abstractions to create a chessboard that can be cloned and 
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

**max(M<sub>0..i</sub>) = -min(-value(M<sub>0</sub>), ..., -value(M<sub>i</sub>))**

We use `-value(...)` for each reply rather than `value(...)` as we evaluate that position in respect to the player who
is currently moving in the algorithm.

Now, we apply this algorithm recursively to determine the best move for any position!

In rust code, here is the result:

```rust
pub const NEG_INFINITY: i16 = -30001;

pub struct ScoringMove {
    pub bit_move: BitMove,
    pub score: i16
}

impl BestMove {
    #[inline(always)]
    pub fn blank(score: i16) -> Self {
        ScoringMove {
            bit_move: BitMove::null(),
            score,
        }
    }
}

impl Ord for BestMove {
    fn cmp(&self, other: &BestMove) -> Ordering {
        self.score.cmp(&other.score)
    }
}

pub fn minimax(board: &mut Board) -> BestMove {
     board.generate_scoring_moves()
        .into_iter()
        .map(|mut m: ScoringMove| {
            board.apply_move(m.bit_move);
            m.score = -minimax(board, depth - 1).score;
            board.undo_move();
            m
        }).max()
        .unwrap()
}
```

As you can see, this is a pretty simple algorithm. To calculate the value of the first player's move, we need
to calculate the opponent's best possible reply to that move. And to determine the best possible reply, we need 
to calculate the best reply to those moves, and so on.

But, if we attempt to use this, we'll notice notice two problems:

1. It runs forever! As a recursive algorithm, we forgot to add a base case.
2. The value of a stalemate (where there are no legal moves to play) returns a score of negative infinity.

So, we'll define the base-case as when the algorithm reaches a certain __*depth*__. Depth is defined as the 
number of moves applied from the starting position. When the algorithm reaches it's maximum depth, instead of
determining the value of that position by evaluating the moves available, we'll determine the value through
evaluation of the board itself. On the most basic level, evaluation is simply summing up the value of
each piece on the board. This fits in with the symmetric nature of our algorithm. If white has the advantage of
one knight, then black is at a disadvantage by one knight.

If there are no moves available to play, we can determine that there is either a stalemate or checkmate.
Checkmate is obviously the worst position available, so we'll give it an extremely low value. Stalemate leads
to a draw, so the value of that is the same for both sides: zero.

With these two improvements, we are left with the following algorithm:

```rust
...

pub const MATE_V: i16 = 25000;
pub const DRAW_V: i16 = 0;

pub fn minimax(board: &mut Board, depth: u16) -> ScoringMove {
    if depth == 0 {
        return eval_board(board);
    }

    let moves = board.generate_scoring_moves();
    if moves.is_empty() {
        if board.in_check() {
            return ScoringMove::blank(-MATE_V);
        } else {
            return ScoringMove::blank(DRAW_V);
        }
    }

    moves.into_iter()
        .map(|mut m: ScoringMove| {
            board.apply_move(m.bit_move);
            m.score = -minimax(board, depth - 1).score;
            board.undo_move();
            m
        }).max()
        .unwrap()
}

```

### Parallizing NegaMax

This algorithm works pretty well! But can we make it faster?
 
Currently, Negamax runs sequentially. If we're attempting to make a lightning-fast 
chess searcher, __*utilizing all available processors will prove essential*__. 

Luckily, Negamax is very easily parallizable. Given a position, the value of any move
can be determined independently of the other moves. This works by sending a subtree of 
the search space to each processor to work on independently. When both the tasks are
complete, we simply return the better value between them. 

This is an example of a [Fork Join Algorithm](https://en.wikipedia.org/wiki/Fork%E2%80%93join_model), 
one of my favorite types of parallel algorithms due to it's simplicity. 

To implement this, we will use [Rayon](https://github.com/rayon-rs/rayon), a data-parallelism library
for Rust. The function `rayon::join` provides exactly the functionality we want; allowing concurrent 
processing of independent data.

```rust
use rayon::prelude::*;
use mucow::MuCow;

pub fn parallel_minimax(board: &mut Board, depth: u16) -> ScoringMove {
    if depth <= 2 {
        return minimax(board, depth);
    }

    let mut moves = board.generate_scoring_moves();
    if moves.is_empty() {
        if board.in_check() {
            return ScoringMove::blank(-MATE_V);
        } else {
            return ScoringMove::blank(DRAW_V);
        }
    }
    
    let board_wr: MuCow<Board> = MuCow::Borrowed(board);
    
    moves.as_mut_slice()
        .par_iter_mut()
        .map_with(board_wr, |b: &mut MuCow<Board>, m: &mut ScoringMove | {
            b.apply_move(m.bit_move);
            m.score = -parallel_minimax(&mut *b, depth - 1).score;
            b.undo_move();
            m
        }).max()
        .unwrap()
        .clone()
}
```

You'll notice two interesting things about this implementation:
1. We use a `MuCow` to wrap the `Board`
2. We do the search sequentially once we're two plys away from the base depth.

This is because using a Fork-Join algorithm isn't an algorithm without costs. A call to a parallel iterator 
will spawn several tasks to be completed, and each queueing and de-queuing of a task has a cost associated with
it. We're trying to maximize processor utilization, while also minimizing the cost of using all processors at once.

Now, why are we using a `MuCow`? Well, because we want to use `rayon::map_with`, as it only clones the `Board` if 
and only if it needs to. Ideally, rayon will split the array into chunks for each task, and then have each processor
iterate over it's chunk sequentially. `MuCow` is a clone-on-consume smart pointer, only cloning when consumed. 
As `map_with` is a consuming operator, we prefer to only clone the `Board` (an expensive operation!) only when
necessary.

### Benchmark Comparisons


