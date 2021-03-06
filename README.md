# Chomp

[![Build Status](https://travis-ci.org/m4rw3r/chomp.svg)](https://travis-ci.org/m4rw3r/chomp)
[![Documentation](https://img.shields.io/badge/rustdoc-documentation-blue.svg)](http://m4rw3r.github.io/chomp)

Chomp is a fast parser combinator designed to work on stable Rust. It was written as the culmination of the experiments detailed in these blog posts:
* [Part 1](http://m4rw3r.github.io/parser-combinator-experiments-rust/)
* [Part 2](http://m4rw3r.github.io/parser-combinator-experiments-errors)
* [Part 3](http://m4rw3r.github.io/parser-combinator-experiments-part-3)

For its current capabilities, you will find that Chomp performs consistently as well, if not better, than optimized C parsers, while being vastly more expressive. For an example that builds a performant HTTP parser out of smaller parsers, see [http_parser.rs](examples/http_parser.rs).

##Installation
As of yet, there is no crate on crates.io. Just clone this repository and hack away.

##Usage
Parsers are functions from a slice over an input type `&'a [I]` to a `ParseResult<'a, I, T, E>`, which may be thought of as either a success resulting in type `T`, an error of type `E`, or a partially completed result which may still consume more input of type `I`.

The input type is almost never manually manipulated. Rather, one uses parsers from Chomp by invoking the `parse!` macro. This macro was designed intentionally to be as close as possible to Haskell's `do`-syntax or F#'s "computation expressions", which are used to sequence monadic computations. At a very high level, usage of this macro allows one to declaratively:
* Sequence parsers, while short circuiting the rest of the parser if any step fails.
* Bind previous successful results to be used later in the computation.
* Return a composite datastructure using the previous results at the end of the computation.

In other words, just as a normal Rust function usually looks something like
```
fn f() -> (u8, u8, u8) {
    let a = 3;
    let b = 3;
    launch_missiles();
    return (a, b, a + b);
}
```

A Chomp parser looks something like

```
fn f(i: Input<u8>) -> U8Result<(u8, u8, u8)> {
    parse!{i;
        let a = 3;
        let b = 3;
        string(b"missiles");
        ret (a, b, a + b);
    }
} 
``` 

For more documentation, see the rust-doc output.


## Example

```rust
#[macro_use]
extern crate chomp;

use chomp::{Input, ParseResult, Error};
use chomp::{take_while1, token};

#[derive(Debug, Eq, PartialEq)]
struct Name<'a> {
    first: &'a [u8],
    last:  &'a [u8],
}

fn main() {
    let i = Input::new("martin wernstål\n".as_bytes());

    let r = parse!{i;
        let first = take_while1(|c| c != b' ');
                    token(b' ');
        let last  = take_while1(|c| c != b'\n');

        ret @ _, Error<_>: Name{
            first: first,
            last:  last,
        }
    };

     assert_eq!(r.unwrap(), Name{first: b"martin", last: "wernstål".as_bytes()});
}
```

##Contact
You can contact the author either through an issue here on GitHub, or you can query him at m4rw3r on mozilla's irc network.
```
