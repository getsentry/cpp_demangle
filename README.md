# `cpp_demangle`: a C++ linker symbol demangler

[![](https://docs.rs/cpp_demangle/badge.svg)](https://docs.rs/cpp_demangle/) [![](http://meritbadge.herokuapp.com/cpp_demangle) ![](https://img.shields.io/crates/d/cpp_demangle.png)](https://crates.io/crates/cpp_demangle) [![Build Status](https://travis-ci.org/gimli-rs/cpp_demangle.png?branch=master)](https://travis-ci.org/gimli-rs/cpp_demangle)

This crate can parse a C++ “mangled” linker symbol name into a Rust value
describing what the name refers to: a variable, a function, a virtual table,
etc. The description type implements `Display`, producing human-readable text
describing the mangled name. Debuggers and profilers can use this crate to
provide more meaningful output.

C++ requires the compiler to choose names for linker symbols consistently across
compilation units, so that two compilation units that have seen the same
declarations can pair up definitions in one unit with references in another.
Almost all platforms other than Microsoft Windows follow the
[Itanium C++ ABI][itanium]'s rules for this.

[itanium]: http://itanium-cxx-abi.github.io/cxx-abi/abi.html#mangle

For example, suppose a C++ compilation unit has the definition:

    namespace space {
      int foo(int x, int y) { return x+y; }
    }

The Itanium C++ ABI specifies that the linker symbol for that function must be
named `_ZN5space3fooEii`. This crate can parse that name into a Rust value
representing its structure. Formatting the value with the `format!` macro or the
`std::string::ToString::to_string` trait method yields the string
`space::foo(int, int)`, which is more meaningful to the C++ developer.

## Usage

Add `cpp_demangle` to your crate's `Cargo.toml`:

```toml
[dependencies]
cpp_demangle = "0.2.8"
```

And then demangle some C++ symbols!

```rust
extern crate cpp_demangle;
use cpp_demangle::Symbol;
use std::string::ToString;

let mangled = b"_ZN5space3fooEibc";

let sym = Symbol::new(&mangled[..])
    .expect("Could not parse mangled symbol!");

let demangled = sym.to_string();
assert_eq!(demangled, "space::foo(int, bool, char)");
```

## Documentation

[Documentation on docs.rs](https://docs.rs/cpp_demangle)

Example programs:

* [A `c++filt` clone](./src/bin/cppfilt.rs)

## Implementation Status

Work is ongoing. While `cpp_demangle` can parse every mangled symbol in
`libiberty`'s demangler's test suite (the canonical Itanium C++ symbol demangler
used by GNU tools such as `c++filt`), it does not format all of them
character-for-character identically. I'm working on fixing that ;)

Despite that, I believe `cpp_demangle` is fairly robust. I've been
running [AFL][] on `cpp_demangle` overnight and it hasn't found any panics for a
long time now (and never found any crashes -- thanks Rust!).

[AFL]: https://github.com/rust-fuzz/afl.rs

## License

Licensed under either of

  * Apache License, Version 2.0 ([`LICENSE-APACHE`](./LICENSE-APACHE) or http://www.apache.org/licenses/LICENSE-2.0)
  * MIT license ([`LICENSE-MIT`](./LICENSE-MIT) or http://opensource.org/licenses/MIT)

at your option.

## Contribution

See [CONTRIBUTING.md](./CONTRIBUTING.md) for hacking.

Unless you explicitly state otherwise, any contribution intentionally submitted
for inclusion in the work by you, as defined in the Apache-2.0 license, shall be
dual licensed as above, without any additional terms or conditions.
