---
title: "Pleco: Creating A Chess Engine With Rust"
---

Over the past couple months, I've been working on [Pleco](https://github.com/sfleischman105/Pleco),
a dual-purpose **chess AI and Library** written in Rust. This has been a personal project of mine the
past several months, and one that is still in active development.

**Pleco aims to one day compete amongst the highest rated chess-engines.** This blog post will be
providing a brief overview of Pleco's representation of a chessboard, the reasoning behind these
choices, and how Pleco uses Rust's *zero-cost abstractions* for easy and fast parallelism.

### Mile-High Chess Board Design
The goal of any chess AI is to find the best move available to play from the current position. The basic
algorithm is very simple, and boils down to looping through each move, applying that move,
and then searching/evaluating that new position. It's a recursive algorithm that generates a "move-tree",
with the "nodes" of the tree being the positions visited, and the "edges" being the moves available to play
for each position.

It looks like this, in Rust pseudocode:

```rust
let chessboard = Board::new();
let moves: Vec<Moves> = chessboard.generate_moves(); // All moves for the current position

for move in moves {
    apply_move();
    evaluate_search_move_strength(); // Should we play this move?
    undo_move();
}
```

With current processors trending towards adding additional cores, rather than increasing clock-speed, any
application designed for running fast must be designed for taking advantage of every core available. For a
chess engine, that means being able to split the work of searching the game-tree to all available cores is essential.

In more Rust pseudocode, a parallel search of a position would be something akin to:
```rust
...

for move in moves {
     search_move_in_parallel();
}
```

This is a great start! But it begs the question, __*why should we think about our search algorithm
before our board representation is even designed?*__

Good question! And it's a great point as well, searching the move-tree can't be done without first defining
out board, figuring out how to generate moves, evaluation positions, etc. Significant work must be done
before a search algorithm is even implemented! Laying out how a search algorithm should work gives us a chance
to define *what we need*, and more so gives us a glimpse at *where* we need to focus our optimization work on.
This is especially important for eventual parallelism, where resources must be sent and shared between threads.

Let's clarify the search algorithm a tad more by defining it as a function. As this is a recursive algorithm,
we need a base-case to stop and actually return a move (unless we want to search every possible position
forever). The algorithm should reach the base-case once a certain number of moves are applied, and then
evaluate that position.

```rust
fn search_board(chessboard: Board, max_depth: u16) -> BitMove {
    if chessboard.depth == max_depth() {
        return evaluate();
    }

    let legal_moves = chessboard.generate_moves();
    in parallel {
        let new_board = chessboard.clone();
        for move in legal_moves {
            new_board.apply_move();
            search_board(new_board, max_depth);
            new_board.undo_move();
        }
    }
    return best_move;
}
```

Obviously, there are quite a few things skimmed over in this implementation, such as determining the best move to
return, evaluating a position and returning that as a move, etc. These specifics can be solidified later. But for now,
our rough prototype for determining the best move of a position hints at the requirements for implementing a `Board`:

* If we want to search the same position with multiple threads, Our `Board` should be cloneable, and those clones must be
have the ability to be sent across threads. When a board is cloned, any modification of the clone's state (such as through a
 `Board::apply_move()` or `Board::undo_move()` must not modify the state of the original copy.
* `Board::apply_move()` and `Board::undo_move()` change the board state to reflect a move being applied and subsequently
un-applied. Un-doing a move must return the state of the board to the **exact** previous state. If we are going to be
returning to the previous state often, we should consider storing it to save some time.

Now that we know what we need, let's get to work!

### The Basics of a Board

Throwing all the basic information needed for a chessboard (alongside our earlier requirements) leaves us with a good prototype:

```rust
struct Board {
    turn: Player,                 // Who's turn is it?
    pieces: [Option<Piece>; 64],  // Where are the pieces?
    half_moves: u16,              // How many moves have been played total?
    depth: u16,                   // Current depth since we started searching
    check: bool,                  // Are we in check?

    prev_state: Option<Board>,    // The previous board, if any
}
```

Doesn't that look gorgeously expressive? I'd like to think so myself. Inside the board, we are defining all the data it needs
to operate correctly. We also are including a reference to the previous state of the board with a `Option<Board>`, so we can
easily access it for a `Board::undo_move()`. The `Option<..>` is necessary for cases where we don't know the previous state
of the board, such as the opening position.

Unfortunately, the compiler doesn't like this:

```
 --> src/main.rs:2:1
  |
2 | struct Board {
  | ^^^^^^^^^^^^ recursive type has infinite size
...
7 |     prev_state: Option<Board>, // The previous board
  |     ----------------------------- recursive without indirection
  |
  = help: insert indirection (e.g., a `Box`, `Rc`, or `&`) at some point to make `Board` representable
```

:|

I've found the compiler to be a very strict parent, yelling at you for breaking *any* rule. Lovely. But the compiler is also
very loving as well, and tells you exactly how to improve. Let's try listening to its suggestion, and wrapping the
`previous_state` inside an `Arc`, an Asynchronous Reference Counter . We use that rather than an `Rc`, as we intend
to share this between threads.

Modified, our `Board` looks like:

```rust
struct Board {
    ...
    prev_state: Option<Arc<Board>>, // The previous board, if any
}
```

It is extremely important for a `Board` to have access to it's previous state. This is because anytime a move is applied,
the board needs to do some calculations regarding the current state of the board. **Computing if the board is in check is
a resource-intensive task**, unlike simply adding or removing pieces from an array. The ability to undo a move and not
recompute if the board is in check will allow a much faster search algorithm.

Unfortunately, this is still a very inefficient solution. If we implement a copy, apply_move and undo_move function, we begin
to see some slowness:

```rust
impl Board {
    pub fn copy_board(&self) -> Board {
        Board {
            turn: self.clone(),
            pieces: self.pieces.clone(),
            half_moves: self.half_moves,
            depth: self.depth,
            check: self.check,
            prev_state: Arc::clone(self.prev_state.unwrap())
        }
    }

    pub fn apply_move(&mut self) {
            let prev_state = self.copy_board(); // !!!!!
            self.prev_state = Arc::new(prev_state);

            ...
    }

    pub fn undo_move(&mut self) {
        let prev: Board = self.prev_state.unwrap();
        self.turn = prev.turn;
        self.pieces = prev.pieces.clone();
        self.half_moves: prev.half_moves - 1;
        self.depth = prev.depth - 1;
        self.check = prev.check;
        self.prev_state = Arc::clone(prev.prev_state);
    }
}
```

While we get access to any previous computations, there is **A LOT** of cloning & moving the board around. Both applying a move and undoing
a move require us to clone a board! Yuck! Ideally, a board should be able to modify itself with very little copying. This is the
fault of the `Arc`, as it allows for shared references, but no mutability. We can fix this mutability problem by replacing the
`Arc<Board>` with a `Arc<Mutex<Board>>`. But, this leads to a whole basket of other problems, such as letting the Board modify state
 that another thread might be referencing, and *we still* would need to copy the previous state if we want to undo a move.

However, this is not a dead-end. We are on the right track for fulfilling all the requirements we outlined earlier.

### A Better Board

The way Pleco represents a `Board` is *extremely* similar to the attempts above, yet fixes all the problems encountered, allowing
for a board with __*non-cloning access to previous & current states*__  and __*thread safe cloning*__.

We do this by separating the data a `Board` encases into two categories:

1. Data relating to the current & future positions (`Board`)
2. Data that is either computed upon making a move, or only relevant to the current position (`BoardState`)

```rust
struct BoardState {
    check: bool                           // Are we in check?
    move_played: Option<BitMove>          // Move played to get into this position
    prev_state: Option<Arc<BoardState>>,  // The previous board state, if any
}

struct Board {
    turn: Player,                 // Who's turn is it?
    pieces: [Option<Piece>; 64],  // Where are the pieces?
    half_moves: u16,              // How many moves have been played total?
    depth: u16,                   // Current depth since we started searching
    state: Arc<BoardState>,       // State of the board
}
```

Undoing moves is easily done through probing the `Board.state.prev_state`, and then modifying our
current board respectively by switching the `turn`, decrementing `half_moves`, etc. Cloning a `Board` now consists
of cloning everything inside of the board, alongside a cloned reference to the current state. The `BoardState` **doesn't
need to be mutable**, as it only contains information that is exclusively for the current position, hence leaving
the `Arc`. Now, If we pass a clone to another thread, that clone can mutate itself, access it's state, and operate
without worrying about ruining another thread's `Board`.

If you know your data structures very well, you'll also recognize the links between `BoardState`s forming a
**Persistent Singly-Linked Stack**. The idea for this was spawned from the
[guide for creating various LinkedLists in Rust](http://cglab.ca/~abeinges/blah/too-many-lists/book/third.html).
Even if you're not interested in data structures, give it a read.

**Preventing excessive copying is an invaluable feature for the board to have**. There is a huge amount of information
that can be computed for each position, for later use in evaluation or move generation. As mentioned earlier,
it is *very expensive* to determine if the board is in check. If we're going to determine if the
board is in check, why not compute and store the pieces that are checking the king? Why not store the location of
those pieces? Why not make it easier to determine checks by storing pieces that are blocking check or pinned to the
king?

This rabbit hole is very deep, and quickly we realize that the amount of information we need to immutably store is *large*.
*Very large*. For example, here is what Pleco's current `BoardState` looks like:
```rust
pub struct BoardState {
    castling: Castling,     // Castling Rights
    pub rule_50: i16,       // Tracking the 50 move rule
    pub ply: u16,           // How deep is the searcher ?
    pub ep_square: SQ,      // Is there a square where En-Passant is available?
    pub zobrast: u64,       // Hash Key of the current position

    pub captured_piece: Option<Piece>,          // Previously captured Piece (If any)
    pub checkers_bb: BitBoard,                  // What squares is the current player receiving check from?
    pub blockers_king: [BitBoard; PLAYER_CNT],  // What squares are blocking a check on the king (for each player)?
    pub pinners_king: [BitBoard; PLAYER_CNT],   // What squares are pinning another piece to the king (of each player)?
    pub check_sqs: [BitBoard; PIECE_CNT],       // For each piece, where is there a check?
    pub prev_move: BitMove,                     // What was the previous move to get to this position?
    pub prev: Option<Arc<BoardState>>,          // Reference to the previous position (If any)
}
```

The `BoardState` is around ~100 bytes of information. This seems small, but a basic [Minimax search](https://chessprogramming.wikispaces.com/Minimax)
to a depth of 4 will iterate through **millions** of different positions. Preventing the copying of this 100 bytes
**significantly** improves the performance of Pleco due to reduced memory usage (less cache misses) and fewer
clock-cycles wasted on copying.

### Conclusion and After Thoughts

With very few lines of code, we are able to express something *immensely* powerful. Pleco utilizes many of
Rust's **zero-cost abstractions** to define a chessboard that is **fast to clone**, **thread safe**, and
**easy to reason about** with the distinction of immutable vs. mutable data. Rust has proven to be a joy to
work in, allowing me to focus on the overarching design and logic of the program, all while providing
a near guarantee of safety.

I mentioned previously that Pleco aims to compete amongst the fastest chess engines in the world. It's a stretch goal
for sure, but it's absolutely doable. Right now, any chess engine that aims to be competitive is created with
C++, and only C++. Pleco isn't trying to prove that Rust can be faster than C++, but rather that Rust is a worthy competitor
in high-performance applications.

Pleco has been developing very well so far. The core of the Engine (`Board`, `MoveGen`) is 99% done,
and there are already a handful of parallel searchers implemented. While future posts will delve a little
deeper into Pleco and the world of Chess Engines, please don't hesitate to [open an issue on github](https://github.com/sfleischman105/Pleco/issues)
for any questions, concerns, or anything of that sort. New Contributors are always welcome as well!
