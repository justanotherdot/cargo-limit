# cargo-limit
[![Crates.io](https://img.shields.io/crates/v/cargo-limit.svg)](https://crates.io/crates/cargo-limit)

Cargo wrapper which limits compiler messages number. Error messages come first.

[Discussion](https://www.reddit.com/r/rust/comments/is9o7x/cargo_with_less_noise/) on reddit.

## Installation
```
cargo install cargo-limit
```

## Usage
Run any of these in your project:
```
cargo lbench [--limit-messages=N]
cargo lbuild [--limit-messages=N]
cargo lcheck [--limit-messages=N]
cargo lclippy [--limit-messages=N]
cargo lrun [--limit-messages=N]
cargo ltest [--limit-messages=N]
```

## Why?
It's a workaround for [this issue](https://github.com/rust-lang/rust/issues/27189). Consider a program:
```rust
fn f() -> Result<(), ()> {
    Ok(())
}

fn main() {
    let mut i: u32 = 0;
    i -= 1;
    f();
    println!("Hello world");
}
```

It's counterproductive to read this kind of compiler noise in attempt to run the program:
```
$ cargo run
   Compiling hello v0.1.0 (/tmp/hello)
warning: variable `i` is assigned to, but never used
 --> src/main.rs:6:9
  |
6 |     let mut i: u32 = 0;
  |         ^^^^^
  |
  = note: `#[warn(unused_variables)]` on by default
  = note: consider using `_i` instead

warning: value assigned to `i` is never read
 --> src/main.rs:7:5
  |
7 |     i -= 1;
  |     ^
  |
  = note: `#[warn(unused_assignments)]` on by default
  = help: maybe it is overwritten before being read?

warning: unused `std::result::Result` that must be used
 --> src/main.rs:8:5
  |
8 |     f();
  |     ^^^^
  |
  = note: `#[warn(unused_must_use)]` on by default
  = note: this `Result` may be an `Err` variant, which should be handled

error: this arithmetic operation will overflow
 --> src/main.rs:7:5
  |
7 |     i -= 1;
  |     ^^^^^^ attempt to compute `0_u32 - 1_u32` which would overflow
  |
  = note: `#[deny(arithmetic_overflow)]` on by default

error: aborting due to previous error; 3 warnings emitted

error: could not compile `hello`.

To learn more, run the command again with --verbose.
```

All we want on this development iteration is to focus on this error:
```
$ cargo lrun
   Compiling hello v0.1.0 (/tmp/hello)
error: this arithmetic operation will overflow
 --> src/main.rs:7:5
  |
7 |     i -= 1;
  |     ^^^^^^ attempt to compute `0_u32 - 1_u32` which would overflow
  |
  = note: `#[deny(arithmetic_overflow)]` on by default

error: could not compile `hello`.

To learn more, run the command again with --verbose.
```

After fixing it we probably want to see the first warning(s):
```
$ sed -i '/.*i -= 1;/d' src/main.rs
$ cargo lrun
    Finished dev [unoptimized + debuginfo] target(s) in 0.00s
warning: unused variable: `i`
 --> src/main.rs:6:9
  |
6 |     let mut i: u32 = 0;
  |         ^^^^^ help: if this is intentional, prefix it with an underscore: `_i`
  |
  = note: `#[warn(unused_variables)]` on by default

     Running `target/debug/hello`
Hello world
```

## License
MIT/Apache-2.0
