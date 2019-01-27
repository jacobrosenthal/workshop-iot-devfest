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
So lets play. [Crates.io](https://crates.io) is where all the fun libraries live. Lets make some noise with [rodio](https://crates.io/crates/rodio) (Note to linux users: You may need to install libasound2-dev with something like `apt install libasound2-dev`

So rodio says it wants us to add it to our Cargo.toml under dependencies so lets do that
```
[dependencies]
rodio = "0.8.1"
```

You can take a look at the [rodio documentation](https://docs.rs/rodio/0.8.1/rodio/), but heres some simple code to make a sin wave.
```
use rodio::default_output_device;
use rodio::source::{SineWave, Source};
use std::time::Duration;

fn main() {
    let device = default_output_device().unwrap();
    let sink = rodio::Sink::new(&device);

    //make a little sine wave
    let source = SineWave::new(440);
    let short = source.take_duration(Duration::from_millis(200));
    sink.append(short.buffered());

    //actually play
    sink.sleep_until_end();
}
```
and just like last time lets `cargo run`. Make sure you have your volume turned up!

So neat, but whats unwrap? If you take that out, the compiler will tell you `expected cpal::Device, found enum std::option::Option` Its saying, getting default_output_device might not exist and come back empty as an [Optional](https://doc.rust-lang.org/std/option/index.html) and you need to handle that. Here we handle it by just crashing if it goes poorly, well have to fix that later. I hope you're seeing a pattern, all the answers to our questions are in the Rust book and Language Reference.


* Play around here a bunch and get comfortable with the language. 
* You're going to start running into the borrow checker here. You''ll see `use of moved value` which means you're using some variable a second time, but the first you used it, it was 'consumed'. One solution to [ownernship](https://doc.rust-lang.org/book/ch04-00-understanding-ownership.html) problems especially when you're not in a performance sensitive environment is to use the `clone()` function to make a copy of the thing, so try that. 
* Read the [rodio documentation](https://docs.rs/rodio/0.8.1/rodio/) or [source code and examples](https://github.com/tomaka/rodio) for more options to try, like fading in and out, playing multiple tones at the same time, loading audio files from the filesystem.

## lets make text
Lets get another package, a morse code package called [light-morse](https://crates.io/crates/light-morse) 
*See if you can install this one yourself.
*Have it print out Hello world but in morse code this time.


## lets make traits
Anybody can write a wall of code. The way we organize code in Rust is in modules, and by making new types extending existing types. How did morse make a `to_morse()` method on a regular old String? Thats definately not in the standard library! Lets look at how morse was implemented. Code for libraries is generally in a file called [lib.rs](https://github.com/luki/light-morse/blob/master/src/lib.rs) We see they defined a trait called MorseSubstitution which is why they can call `to_morse()` on a String that normally doesnt have a to_morse method. This is [trait based inheritence](https://doc.rust-lang.org/book/ch10-02-traits.html) (called mixins in other languages) is an important concept in Rust/
* Lets make a new trait in a new file called text.rs that extends Morse (which itself is just a String). For every Morse char they translate, lets print to screen with delays:
```
use light_morse::Morse;
use std::io::stdout;
use std::io::Write;

pub trait Display {
    fn display(&self);
}

impl Display for Morse {
    fn display(&self) {
        for item in self.chars() {
            print!("{}", item);
        }
        println!();
    }
}
```
Here were using [iterators](https://doc.rust-lang.org/book/ch13-00-functional-features.html) another important concept in Rust you need to get to know. We can call the [.chars() function](https://doc.rust-lang.org/std/str/struct.Chars.html) on a String (Morse is just a String behind the scenes here) and do a for each over that iterator which is safer than a for loop where a common error you might have made is is off by one your index variable.

Finally we need to make this new text.rs file available to our main.rs by importing it with `mod text;` and using your new trait with `use crate::text::Display`
```
mod text;

use crate::text::Display;
use light_morse::MorseSubstitution;
use light_morse::MorseType;

fn main() {
    "Morse".to_string().to_morse(MorseType::Gerke).display();
}
```
Note were not printing anything out here anymore, weve hidden all that code in the display trait. Also note this hooking many functions together (builder pattern) as well as use of iterators above is one of the more declaritive (or functional) styles seen in Rust. Both syntaxes lead to rather pleasant and easy to reason about code.

* Can you use of [thread::sleep](https://doc.rust-lang.org/std/thread/fn.sleep.html) to sleep between characters so it looks stuttery like text coming over the wire? You'll also needed to `stdout().flush()` between characters or they get buffered by the operating system and all come out at the same time.


## put it all together
Can you edit the Display trait to make a long beep for dashes and a short beep for dots instead of/in addition to printing the character? You're probably thinking of doing this with an if statement, but a more Rusty solution would be to use [Match](https://doc.rust-lang.org/1.5.0/book/match.html). Note here, the morse characters are not a regular dash, but rather these these ascii characters `−` `·` 

### So why is Rust different than C++? 
Safey can be acheived in C, for time and money. But its not baked into the language and its not easy to teach. Further, Rust's lack of legacy is a benefit more than a hinderance. Rust only has one compiler, not competing vendors with multiple compilers. Further Rustaceans are designing the language in github issues and nightly code releases instead of by committes that only Facebook and Google can afford to fly to attend (and thus only their needs are ever met) Rust has modularized code and a single package manager, (Cargo, like npm for packages) which means code sharing is far easier and comes more naturally to Rustaceans. C++ hopes to ship module support in their 2020 release, but many people are still running C++ 2014 today... Most of you here today have never and may never reach for C or C++ to play a bunch sounds and


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

## tomu
[Tomu](https://tomu.im) is a fun little arm microcontroller that fits in your USB-A port. I'm not affiliated in any way, I just hoped you'd always have a microcontroller with you would and no excuses to goof around with it. My thanks to [Crowd Supply](https://www.crowdsupply.com) for sponsoring these today. The Tomu has 2 leds (red and green), capacitive touch buttons and USB. With their [easy to use C code examples](https://github.com/im-tomu/tomu-quickstart) you could (in any language of your choosing) watch the USB for 'button presses' or have a program on your laptop trigger leds on the device like a notification, and many more things.

But I'm developing the Rust capabilities for this board. GPIO blinky stuff is done, but USB and capacitive touch aren't yet so stay tuned.
 
## setup and blink
* Run `rustup target add thumbv6m-none-eabi` to add the compiler bits for this board
* Run `rustup component add llvm-tools-preview`
* Install [dfu-util](https://tomu.im/samples) from directions on this page

Download [my tomu hal fork](https://github.com/jacobrosenthal/imtomu-rs/tree/iot-dev-fest) and specifically the [blink example](https://github.com/jacobrosenthal/imtomu-rs/blob/iot-dev-fest/examples/blink.rs)

And then when we run well need to run in 'release mode' and since this is normally a library, we can tell cargo to run its example file with `cargo run --release --example blink` Ive hooked up scripts so that upon building, it also uploads the code to your tomu via dfu.  

So what embedded rust attempts to do is remove 'global state' that is the registers by turning them into structs, extending functionality on them with traits, and restricting usage of them via the same ownership models weve seen.

References:
* http://blog.japaric.io/brave-new-io/
* https://doc.rust-lang.org/book/ch01-03-hello-cargo.html
* https://docs.rust-embedded.org/book/start/registers.html
* https://docs.rust-embedded.org/embedonomicon/main.html






