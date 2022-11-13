# What's the `cfg` stuff?

Lots of the CosmWasm contracts I've seen start with this.

```rust
#[cfg(not(feature = "library"))]
```

Fairly guessable that `cfg` means "configure" or "configuration", but beyond that this is pretty opaque!

Rust By Example [says](https://doc.rust-lang.org/rust-by-example/attribute/cfg.html) that it is for a "configuration conditional check".

The Rust reference [clarifies](https://doc.rust-lang.org/reference/conditional-compilation.html#the-cfg-attribute) that it "conditionally includes the thing it is attached to based on a configuration predicate."

> If the predicate is true, the thing is rewritten to not have the `cfg` attribute on it. If the predicate is false, the thing is removed from the source code.

My editor helps clarify things a bit—it shows the next line, the import of `cosmwasm_std::entry_point`, in gray.

```rust
#[cfg(not(feature = "library"))]
use cosmwasm_std::entry_point;
```

Throughout the rest of the file, we can see matching lines:

```rust
#[cfg_attr(not(feature = "library"), entry_point)]
pub fn instantiate(...)

...

#[cfg_attr(not(feature = "library"), entry_point)]
pub fn execute(...)
```

etc.

So the `cfg`/`cfg_attr` macros are disabling all the `entry_point` stuff if ... something? ... is ... not a library?

Since this has something to do with building, let's see if `Cargo.toml` reveals anything.

```toml
[features]
backtraces = ["cosmwasm-std/backtraces"]
# use library feature to disable all instantiate/execute/query exports
library = []
```

Yep!

But I still have a couple questions.

# Does `library = []` mean that `(not(feature = "library"))` currently evaluates to true or false? Is empty array considered true or false, in the context of Cargo/toml?

Reading [The Cargo Book's chapter on Features](https://doc.rust-lang.org/cargo/reference/features.html) tells me that [features are disabled by default](https://doc.rust-lang.org/cargo/reference/features.html#the-default-feature). Which means that all of the `cfg` macros throughout these codebases evaluate to `true` by default, since the condition is inverted (_not_ feature = "library").

We can test this by using the `cfg!` function-like macro that I learned about while reading these docs. First a sanity check: a `println!` in a test:

```rust
#[test]
fn add_message() {
    println!("LOL!");
    ...
}
```

This doesn't work at first, and the tickle at the back of my brain reminds me that Rust (or maybe Cargo?) hides output in tests by default. Find the StackOverflow post about it, try again  with `cargo test -- --nocapture`, and hooray, there's my LOL!

Ok, so now we can wrap it in a `cfg!` macro:

```rust
#[test]
fn add_message() {
    if cfg!(library) {
        println!("LOL!");
    }
    ...
}
```

Run with `cargo test -- --nocapture`, and...

Nothing. Can't figure out what syntax to pass to `cfg!`. How about this instead?

```rust
#[test]
fn add_message() {
    println!("LOL! {}", cfg!(library);
    ...
}
```

Eh, at least now I see it says "false".

Ok, try something else.

```rust
#[cfg(test)]
#[cfg(not(feature = "library"))]
mod tests {
    ...
```

Now can I disable all tests? Yes!

```bash
cargo test --features library -- --nocapture
```

No tests!

While getting to this point, I first tried to add both `cfg` settings to the one declaration.

```rust
#[cfg(test, not(feature = "library"))]
```

Rust-analyzer complained.

    multiple `cfg` predicates are specified

Ah! So these are called "predicates". And the `cfg!` macro [takes in](https://doc.rust-lang.org/reference/conditional-compilation.html#the-cfg-macro) "a single configuration predicate". So... maybe this?

```rust
#[test]
fn add_message() {
    println!("LOL! {}", cfg!(not(feature = "library")));
    ...
```

Run it:

```bash
$ cargo test -- --nocapture
...
LOL! true
```

Hooray! We can now see that by default, this does _not_ have the "library" feature enabled.

A little playing around, and we can figure out how to enable it:


```bash
$ cargo test --features library -- --nocapture
...
LOL! false
```

Great, so building with `--features library` means that `execute`, `query`, `instantiate`, and `migrate` are all excluded from the build. Which brings me to my last remaining question:


# What is the point of this?

It seems like maybe it allows me to write a contract that doubles as a library. But is that a good idea? Why not let contracts be contracts and libraries be libraries? Maybe there's something I'm missing?

It's hard to find good results searching for "cosmwasm feature library". Lots of noise about the great features of the CosmWasm library.

And the couple of contracts I've `git blame`d so far seem to have started out with this `[features]` setting in `Cargo.toml`.

And since they all get initialized from... what was it?

[cw-template](https://github.com/CosmWasm/cw-template)

...let's see when this was added to `cw-template`...

[Aug 4, 2021](https://github.com/CosmWasm/cw-template/pull/76/commits/efd5fba4662da292f7123ae6408d305f21162d02).

Not much of a commit message, though. I still don't know why this was thought to be useful enough to clutter every `cw-template`-started project with it.

Maybe I'll find out, over the remainder of this course.
