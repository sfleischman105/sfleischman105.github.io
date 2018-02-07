---
title: "Using Unsafe Rust to Create a Chess Transposition Table"
---

Hi! My name is Stephen (or Bradly!), I'm the creator and maintainer of  
[Pleco](https://github.com/sfleischman105/Pleco), a Rust-based Chess Library and AI. 

After reading all of the Rust2018 blog-posts, it became clear that the community is missing
material focused for Intermediate Rustaceans. Personally, some of the most helpful Rust readings 
are ones that explain a concept, and then show how to implement it with Rust. A perfect example of 
this is [Learning Rust with Too Many Linked Lists](http://cglab.ca/~abeinges/blah/too-many-lists/book/).

This post will talk about Transposition Tables, an invaluable structure for chess AI. I'd recommend 
skimming [my previous blog post](https://sfleischman105.github.io/2017/10/26/creating-a-chess-engine.html) 
for basic understandings of how a chess AI works.

#### What is a Transposition Table?

Simply put, a Transposition Table is a method of storing information that we encounter upon traversing
the search-tree. If our searching algorithm encounters the same position more than once, any previous 
work we've done to evaluate that position is wasted. A Transposition Table is a type of hash-table,
mapping from a position to an entry. Each position has a semi-unique zobrist key associated with it.
A zobrist key is simply a `u64` in disguise, and is generated for each `Board` from the starting position,
and is modified upon doing and undoing a move. See [this article](https://en.wikipedia.org/wiki/Zobrist_hashing)
for more information on Zobrist Hashing.

### Layout

Our basic layout will look like this:

```rust
pub struct TranspositionTable {
    entries: *mut Entry, // pointer to the heap-allocated array of entries
    cap: usize, // number of entries
}

/// Entry corresponds to a particular position
pub struct Entry {
    pub key: u64,
    pub best_move: BitMove, // What was the best move found here?
    pub score: i16, // What was the Score of this node?
    pub eval: i16,  // What is the evaluation of this node
    pub depth: u8,  // How deep was this Score Found?
}
```

On initialization, our table will allocate an array in the heap of `cap` number of entries.  

### Just use a `HashMap` though...

Technically we could utilize a `HashMap` for this functionality. But we're aiming for the fastest access
possible here, and that means a few things:
1. Data stored in the table must be as compact as possible!
2. Synchronous access between threads. Data stored inside is alot more valuable if it can be retreived by 
other threads searching.
3. Lock-Free probing and mutating. Locks would allow for safe access, but slow down both concurrent and
single-threaded access.


For these reasons, we see a `HashMap` is of little use. A `HashMap` doesn't allow concurrent mutating 
operations without a `Arc<Mutex<...>>` around it. Furthermore, any key used to accessed the `HashMap` 
is re-hashed before access. This is wasted effort, as know that our zobrist keys are going to be
randomly distributed. 

### Why do we need `unsafe` for a TranspositionTable?

Well, the above requirements should scare most Rustaceans. A `Transposition Table` breaks the most
sacred of the rust rules, *__mutating shared data__*. On top of that, we *__we want to do this 
between threads__*. This is akin to practicing black magic at a church, *__while naked__*.

Luckily, confessing your sins is easy with Rust. Simply approach the holy priest and mutter the 
words `unsafe`, and he'll grant the privilege to do whatever sacrilegious deed your mind desires. 
However, he won't save you from whatever Cthulhu-esq monster ends up spawning, and he certainly
won't save you from it eating your family as well. 

So, we need some ideas of how this Lovecraftian monster could be spawned, as well as guidelines
to prevent this, deal with it, or accept it's possibility.

##### Invalid pointers

As we are using raw pointers here, there is a possibility of accessing invalid memory and invoking
a segmentation fault. We will have to guarantee that our accessed `Entry` lies within the bounds of
the accessed memory. 

##### Re-sizing during concurrent access

A problem with an easy fix: don't allow re-sizing when **any** threads are searching.

##### Key Collisions

A zobrist key is stored in a `u64` integer, so it has 2^64 (~18.4 Quintillion) possible values.
The chance of collision is *extremely* low, but it's still a possibility. It's easier to accept 
this small of error while searching than to find a way to deal with it.

##### Entry Overwriting

It's physically impossible to have a table with a capacity to fill all 2^64 possible keys, so
our table will be much smaller. It's likely that entries will hash to the same index during a
search. This will cause some entries to over-write each other, reducing the real capacity. 

##### Concurrent `Entry` Mutation

If we want fast & lock-less access, it's likely that that two threads will access the same 
`Entry` concurrently. Yuck. In the best scenario, both threads are writing with identical positions. 
In worst case, each thread is writing with different positions. This problem is further exasperated 
by the previous issue, being a reduced table size. 

Like the previous collision issues, there is no good way to mitigate this without reducing speed. 
The usage of `Atomic` loading and storing is a possibility, but those entail some (small) overhead.
For now, we're just gonna have to try our best to mitigate this by locally storing all the data
retrieved at once, as well as modifying all the data at once.



