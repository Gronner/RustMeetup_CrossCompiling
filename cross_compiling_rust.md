---
title: Cross-compiling Rust
author: Felix Bräunling
patat:
    wrap: true
    slideLevel: 2
    margins:
        left: 10
        right: 10
        top: 20
        bottom: 20
---


```text
     _____                            _____                      _    __   _                      ___              __ 
    / ___/  ____ ___   ___  ___ ____ / ___/ ___   __ _    ___   (_)  / /  (_)  ___   ___ _       / _ \ __ __  ___ / /_
   / /__   / __// _ \ (_-< (_-</___// /__  / _ \ /  ' \  / _ \ / /  / /  / /  / _ \ / _ `/      / , _// // / (_-</ __/
   \___/  /_/   \___//___//___/     \___/  \___//_/_/_/ / .__//_/  /_/  /_/  /_//_/ \_, /      /_/|_| \_,_/ /___/\__/ 
                                                       /_/                         /___/                              
                                                     
                                                     _~^~^~_
                                                 \) /  o o  \ (/
                                                   '_   -   _'
                                                   / '-----' \
```

## What is Rust?

```rust
fn main() {
    let what_is_rust = "Rust is a systems language pursuing the trifecta: safety, concurrency, and speed!";
    println!("{}", what_is_rust);
}
```

```text
$ cargo build
. . .
$ cargo run
. . .
Rust is a systems language pursuing the trifecta: safety, concurrency, and speed!
```

## What is cross-compiling?

Compiling stuff on your computer for your computer (e.g., x86_64-unkown-linux-gnu) is easy!

```text
         _________
        / ======= \
       / __________\
      |  _________  |
      | |>_       | |
      | |         | |
      | |_________| |
      \=____________/
      / """"""""""" \
     / ::::::::::::: \
    (_________________)    

        Your PC (x86)
```

```
$ cargo build
$ ./hello_world
Rust is a systems language pursuing the trifecta: safety, concurrency, and speed!
```

```text
$ gcc -dumpmachine
x86_64-linux-gnu
$ uname -o -m
x86_64 GNU/Linux
```

## What is cross-compiling?

What happens if you want to build software for another computer?
```text
    .~~.   .~~.
   '. \ ' ' / .'
   *.~ .~~~..~.*
  *: .~.'~'.~. :*
 *~ (   ) (   ) ~*
*( : '~'.~.'~' : )*
 *~ .~ (   ) ~. ~*
  *(  : '~' :  )*
   *'~ .~~~. ~'*
       *'~'*
       
 Raspberry Pi (ARM)
```

```text
> ./hello_world
bash: /home/pi/hello_world: cannot execute binary file: Exec format error
```

## And now?

We need to compile for another target architecture!

```
$ rustup target add armv7-unknown-linux-gnueabihf
$ cargo build --target=armv7-unknown-linux-gnueabihf
error: linking with `cc` failed: exit code: 1
```

Cargo just uses `cc` and `ld` for this. But these don't know `ARM`, just `x86`.


## And now?

We need to add tools for `ARM` compiling and linking!

```
$ apt install gcc-arm-linux-gnueabihf
$ cargo build --target=armv7-unknown-linux-gnueabihf
error: linking with `cc` failed: exit code: 1
```

Didn't we just install the tools and this should be working?

## And now?

Well, we have to tell cargo about this as well!

```
$ vim ./cargo/config
```
```toml
[target.armv7-unknown-linux-gnueabihf]
linker = "arm-linux-gnueabihf-gcc"
```
Now on our `x86` computer
```
$ cargo build --target=armv7-unknown-linux-gnueabihf"
$ ./rust_cc
Failed to execute process './rust_cc'. Reason:
exec: Exec format error
```
and on our `ARM` computer
```
> ./rust_cc
Rust is a systems language pursuing the trifecta: safety, concurrency, and speed!
```

## Why would you cross-compile?

But on a Raspberry Pi I can just do this:
```
> cargo build
> ./rust_cc
Rust is a systems language pursuing the trifecta: safety, concurrency, and speed!
```

why would I ever go through all that trouble?

This is why:

| Machine                 | Time    |
| ----------------------- | ------- |
| AMD Ryzen3900X 12 cores | 18.73s  |
| RaspberryPi4 4 cores    | 7m 50s  |
| RaspberryPi0 1 core     | 158m 4s |

## What happens if we go smaller?

Let's build some LED blinking for a Cortex-M0 (STM32F0) by STMicroelectronics (`thumbv6m-none-eabi`).

```
$ cargo build
error[E0463]: can't find crate for `core`
```

Well we know what to do!

```
rustup target add thumbv6m-none-eabi
```

what does that actually do?

-> It downloads the standard library as a precompiled binary for that target!

## What if there is no standard library binary?

But sometimes there is no precompiled version :-( Cargo to the rescue!
```
rustup component add rust-src
cargo build -Z build-std=core
```

Deprecated alternatives are `xargo` and `cargo-xbuild`

## Well this all seems tedious

What is the one thing that always solves our problems?

. . .

```text
           *--------------*
          /      /       /|
         /      /       / |
        *--------------*  |
        |              |  |
        |              |  |
        |              |  |
        |              |  *
        |              | /
        |              |/
        *--------------*
```
We put them in a box ... and if a box is not big enough?

. . .

```text
                    ##        .            
              ## ## ##       ==            
           ## ## ## ##      ===            
       /""""""""""""""""\___/ ===        
  ~~~ {~~ ~~~~ ~~~ ~~~~ ~~ ~ /  ===- ~~~   
       \______ o          __/            
         \    \        __/             
          \____\______/                
```

## Cross

`cross` put's all the cross-compiling in a docker container and delivers us just the binary.
It's a drop-in replacement for the `cargo` command.

```text
$ cross build --target=armv7-unknown-linux-gnueabihf
```

Supported targes can be found here: https://github.com/rust-embedded/cross#supported-targets

!! By default this uses `xargo` to build missing standard library binaries !!

This can be changed though.

Cross.toml
```
[build]
xargo = false
```

## Extending cross

Sometimes it is necessary to extend the `cross`-base image.

This is easily done by creating a custom docker image or specifing one.

Cross.toml
```
[target.armv7-unknown-linux-gnueabihf]
image = "rustmeetup"
```

## Running without deploying

`cross` supports `QEMU` for selected targets, so you just can do

```
$ cross run
$ cross test
```

to execute the application and its tests locally!

## Questions?

```
  ___  
 |__ \ 
    ) |
   / / 
  |_|  
  (_)  
```

## Sources

* Rust Supported Platforms: https://doc.rust-lang.org/nightly/rustc/platform-support.html
* Rust Embedded Book: https://docs.rust-embedded.org/book/
* xargo: https://github.com/japaric/xargo
* cargo-xbuild: https://github.com/rust-osdev/cargo-xbuild
* cross: https://github.com/rust-embedded/cross
* Docker Hub of the cross project: https://hub.docker.com/r/rustembedded/cross
