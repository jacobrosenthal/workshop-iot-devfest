# workshop-iot-devfest

Rust is an incredibly promising new (~10 year old) systems language whose offers of safety, speed, and concurrency are badly needed in the IOT space right now. We all know a great engineer can develop safe C code, but clearly we need more, faster, and cheaper lines of code than C has been able to deliver in recent decades.

In the first part well just generally learn the language and how to talk to the compiler by developing a simple CLI application on our laptops. With what time we have left well see what embedded code looks like and try some blink examples on the Tomu USB device.

Prerequisites
* [Rust installed](https://www.rust-lang.org/tools/install)
* Basic programming skills - though feel free to come and work at your own pace.
* Laptop with a common USB-A port or dongle.
* [Tomu](https://tomu.im)
* vscode for code completion. (vim, sublime, nothing else completes decently)


## anotomy of rust program and some workflow

lets open a terminal and create a new package with `cargo new rust-iot-devfest` and go to that directory with `cd rust-iot-devfest`

Now we have a Cargo.toml which defines our project, not unlike a package.json if you're familiar with Node.js, it defines dependencies we're using and other project information:
```
[package]
name = "rust-iot-devfest"
version = "0.1.0"
authors = ["Jacob Rosenthal <jacobrosenthal@gmail.com>"]
edition = "2018"

[dependencies]
```
In src folder we have main.rs, a rust file. In this case it generated a simple hello world. main is a function denoted by fn. The exclamation after println! means that is a macro. When you see something you dont understand, the place to look is the [Rust Book](https://doc.rust-lang.org/book/ch19-06-macros.html). For now though, well just think of macros as a fancy function.
```
fn main() {
    println!("Hello, world!");
}
```

We can both build and run our program with `cargo run`
```
$ cargo run
   Compiling rust-iot-devfest v0.1.0 (/Users/jacobrosenthal/rust-iot-devfest)
    Finished dev [unoptimized + debuginfo] target(s) in 0.50s
     Running `/Users/jacobrosenthal/.cache/target/debug/rust-iot-devfest`
Hello, world!
$ 
```

## define your terms

Rust is a safe, typed, compiled, general programming language.
* Safe is a very overloaded term in Rust, but by default Rust uses static analysis at compile time to enforces rules about memory usage. Traditonally, managing your own memory in languages like C++ and Objective-C has been very tedious and still error prone. Rust's solution to this, The borrow checker, is probably the defining feature of the Rust language. You have to spend a little more time annotating your code to say who owns a variable at any given time with the borrow checker keeping you honest the whole time. Its like pair programming with a friend.
* Typed - As opposed to languages like javascript, Rust forces you to state the types of variables going into and out of functions. This helps you organize your intentions and acts much like a set of tests to make sure your code makes sense.
* Compiled - Rust has to do all its work up front at compile time and turns into a binary immediately. Compiles can be slow sometimes, but our code runs fast, and anywhere, as a result. [obligatory xkcd](https://xkcd.com/303/) 
* General. Much like most modern laguages these days its not strictly functional or object oriented (OO). Futher it can be deployed almost anywhere. We can write backend server code, embedded microcontroller applications, and with WASM, even front end web applications, cloud functions and blockchains.

## lets make sound
So lets play. [Crates.io](https://crates.io) is where all the fun libraries live. Lets make some noise with [rodio](https://crates.io/crates/rodio)

It says it wants us to add this to our Cargo.toml under dependencies
```
[dependencies]
rodio = "0.8.1"
```

You can take a look at the documentation, but lets make a sin wave. 
We need to import our package rodio and anything else we want to use from the package. Then well make a device and play a sound. 
```
use rodio::{
    default_output_device, play_raw,
    source::{SineWave, Source},
};
use std::time::Duration;

fn main() {
    let device = default_output_device().unwrap();
    let source = SineWave::new(440);
    let short = source.take_duration(Duration::from_millis(1000));
    play_raw(&device, short.convert_samples());

    loop {}
}

```
and same thing as last time `cargo run`. 

So neat, but whats unwrap? If you take that out, the compiler will tell you `expected cpal::Device, found enum std::option::Option` Its saying, getting default_output_device might not exist and come back empty as an [Optional](https://doc.rust-lang.org/std/option/index.html) and you need to handle that. Here we handle it by just crashing if it goes poorly, well have to fix that later. I hope you're seeing a pattern, all the answers to our questions are in the Rust book.

Also notice the loop. Our program will just exit before our sound can even start we just busy loop in there. Try to take it out.


* Play around here a bunch and get comfortable with the language. You're going to start running into the borrow checker here. If you see `use of moved value` try to use the `clone()` function to make a copy in this case. Thats the easy way out, when you're on a laptop sized machine its perfectly fine.
* Try to download a wave file and play it instead of our generated sin wav

## lets make text
Lets get another package, a morse code package called [light-morse](https://crates.io/crates/light-morse) 
*See if you can install this one yourself.
*Have it print out Hello world but in morse code this time.


## lets make traits
Anybody can write a wall of code. The way we organize code in Rust is in modules, and by making new types extending existing types. How did morse make a to_morse method on a String? Thats definately not in the standard library. Lets look at how morse was implemented. Code for libraries is generally in a file called [lib.rs](https://github.com/luki/light-morse/blob/master/src/lib.rs) They defined a trait called MorseSubstitution which is why they can call to_morse on a String that normally doesnt have a to_morse method. This is trait based inheritence is an important concept in rust.
* Lets make a new trait in a new file called text.rs that extends Morse. For every Morse char they translate, lets print to screen with delays
```
use light_morse::*;
use std::io::stdout;
use std::io::Write;
use std::thread;
use std::time::Duration;

pub trait Display {
    fn display(&self);
}

impl Display for Morse {
    fn display(&self) {
        for item in self.chars() {
            print!("{}", item);
            stdout().flush().unwrap();

            thread::sleep(Duration::from_millis(200));
        }
        println!();
    }
}
```
and in main.rs
```
mod text;

use crate::text::Display;
use light_morse::*;

fn main() {
    "Morse".to_string().to_morse(MorseType::Gerke).display();
}

```

Alright this is the big one. Can you do the same thing with audio beeps?

### So why is Rust different than C++? 
Safey can be acheived in C, for time and money. But its not baked into the language and its not easy to teach. Further, Rust's lack of legacy is a benefit more than a hinderance. Rust only has one compiler, not competing vendors with multiple compilers. Further Rustaceans are designing the language in github issues and nightly code releases instead of by committes that only Facebook and Google can afford to fly to attend (and thus only their needs are ever met) Rust has modularized code and a single package manager, (Cargo, like npm for packages) which means code sharing is far easier and comes more naturally to Rustaceans. C++ hopes to ship module support in their 2020 release, but many people are still running C++ 2014 today...


## My bold claim
Rust's guarantees means it allows more, better code, to be written faster, and it unifies all of development finally. We can write driver code for microcontrollers and kernels, libraries on top of that for laptops and servers, and take logic from that and deploy it to the cloud and browser. Programming is difficult enough without switching languages and one programmer not necessarily able to mentor another. So no more this guy writes linux kernel C but cant write anything for the web (and therefor shittalks js devs), or this guy is a web developer it thinks it would take an engineering degree to be able to write some non toy code for a microcontroller.

And lets face it we need sea changes in software development. Were 10 years into hackerspaces and bootcamps are we making enough strong developers to guard the internet of things as we move to a billion devices in the years to come?




# Embedded Rust

## anotomy of no_std rust program
Mostly the same, but a little more boiler plate
```
#![no_std]
#![no_main]

extern crate panic_halt;
use cortex_m_rt::entry;

#[entry]
fn main() -> ! {
    loop {
    }
}
```
So no_std is the big deal here. We don't get anything that starts with std:: including String! There are make do packages, and ways around, but for now lets assume we don't use what we call things that 'allocate'

## lets go
So lets play with [Tomu](https://tomu.im) Tomu only has 2 leds (red and green) capacitive touch buttons and USB (though Rust drivers dont exist for those yet). So were just going to show off gpio blinking. First You're going to need to download dfu-util for your platform in order to upload code. Follow the directions there.

Download [my tomu hal fork](https://github.com/jacobrosenthal/imtomu-rs/tree/iot-dev-fest) and specifically the [blink example](https://github.com/jacobrosenthal/imtomu-rs/blob/iot-dev-fest/examples/blink.rs)

From the directions there youll see were going to need a few more commands to set up embedded: `rustup default nightly` `rustup target add thumbv6m-none-eabi` `rustup component add llvm-tools-preview`

And then when we run well need to run in 'release mode' and since this is normally a library, we can tell cargo to run its example file with `cargo run --release --example blink` Ive hooked up scripts so that upon building, it also uploads the code to your tomu via dfu. Youll need dfu-util installed

So what embedded rust attempts to do is remove 'global state' that is the registers by turning them into structs, extending functionality on them with traits, and restricting usage of them via the same ownership models weve seen.


References:
* http://blog.japaric.io/brave-new-io/
* https://doc.rust-lang.org/book/ch01-03-hello-cargo.html
* https://docs.rust-embedded.org/book/start/registers.html
* https://docs.rust-embedded.org/embedonomicon/main.html






