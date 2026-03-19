# Seven years unimplemented. Three weeks after I published the solution, it appeared in Redox OS with AGPLv3 violation

## Intro. Who am I?
I am keepitupkitty, 22 year old amateur programmer who writes in C, C++ and Rust most of the time. I love doing it so much and so I decided to write my own libc in Rust for my own GNU/Linux distribution. Why? I want a secure solution that implements wide support for compiler (and not only) mitigation like LLVM SafeStack, cross-DSO CFI and much more, additionally using Rust essentially **forces** programmer to write correct and memory safe code which also benefits overall safety and reliablity of the libc.

## Issues implementing libc in pure Rust
As someone who programs in Rust might now that Rust supports C FFI and thus C variadic arguments, although it is considered to be an unstable feature, also someone might know too, that C has a special type called `long double` which differs across ABIs. One of such ABI was System V ABI for both 32-bit adn 64-bit x86 processors. The key difference from the rest is that both ABIs use 80-bit Intel x87 floating point numbers for `long double` types which has better precision compared to binary 64-bit IEEE 754 float type, known in C as `double`. This float type is **NOT** supported by Rust and as of March 19 of 2026 there is no RFCs for supporting such type leaving developers who build libcs either to change the compiler (to make `long double` to be `double`) or leave support for %L{f,F,g,G,e,E,a,A} behind or worse (in case of `strtold`), do bad casts for routines that return `long double`.
Another issue is that Rust has set boundaries for the types that can be fetched from variadics, especially it is enforced by `unsafe trait VaArgSafe`. Since I do not want to hassle with compilers in hope nothing will break, I had to come up with an elegant solution.

## A brilliant idea that has came to me while I was reading Rust's `core` library source
