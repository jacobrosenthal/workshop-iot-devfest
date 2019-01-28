# workshop-iot-devfest
Rust is an incredibly promising new (~10 year old) systems language whose offers of safety, speed, and concurrency are badly needed in the IOT space right now. We all know a great engineer can develop safe C code, but clearly we need more, faster, and cheaper lines of code than C has been able to deliver in recent decades.

In the first part of this workshop we'll just generally learn the language and how to listen to the compiler by developing a simple CLI application on our laptops. With what time we have left well see what embedded code looks like and try some blink examples on the Tomu USB device.

Prerequisites
* [Rust installed](https://www.rust-lang.org/tools/install)
* Basic programming skills - though feel free to come and work at your own pace.
* Laptop with a common USB-A port or dongle.
* [Tomu](https://tomu.im)
* (optional) VScode or IntelliJ for code completion. (neither vim, sublime, or anything else completes decently sadly)

## me
I'm a freelance embedded engineer. I've specialized in embedded bluetooth consulting recently which has kept me all up and down the stack from Chrome and Web Bluetooth down to drivers and RTOS. I was recently diagnosed with Rust and am told I have been insufferable since. I've recently taken over the [monthly Phoenix Rust meetup](https://www.meetup.com/Desert-Rustaceans/) and we'd love to have you there to continue what we start today. You'll also find a weekly study group called booze.rs held both offline and online in #rust on http://az-webdevs.slack.com so please join us there to keep up with the community and ask your code questions.

## anatomy of Rust program and some workflow

Open a terminal and create a new package with `cargo new rust-iot-devfest` and go to that directory with `cd rust-iot-devfest`

Now we have a Cargo.toml which defines our project, not unlike a package.json if you're familiar with Node.js, it defines dependencies we're using and other project information:
```
[package]
name = "rust-iot-devfest"
version = "0.1.0"
authors = ["Jacob Rosenthal <jacobrosenthal@gmail.com>"]
edition = "2018"

[dependencies]
```
In src folder we have main.rs, a Rust file. In this case it generated a simple hello world. main is a function denoted by fn. The exclamation after println! means that is a macro. When you see something you don't understand, the place to look is the [Rust Book](https://doc.rust-lang.org/book/ch19-06-macros.html). For now though, well just think of macros as a fancy function.
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
* Safe is a very overloaded term in Rust, but by default Rust uses static analysis at compile time to enforces rules about memory usage. Traditionally, managing your own memory in languages like C++ and Objective-C has been very tedious and still error prone. Rust's solution to this, The borrow checker, is probably the defining feature of the Rust language. You have to spend a little more time annotating your code to say who owns a variable at any given time with the borrow checker keeping you honest the whole time. It's like pair programming with a friend.
* Typed - As opposed to languages like javascript, Rust forces you to state the types of variables going into and out of functions. This helps you organize your intentions and acts much like a set of tests to make sure your code makes sense.
* Compiled - Rust has to do all its work up front at compile time and turns into a binary immediately. Compiles can be slow sometimes, but our code runs fast, and anywhere, as a result. [obligatory xkcd](https://xkcd.com/303/) 
* General. Much like most modern languages these days it's not strictly functional or object oriented (OO). Further it can be deployed almost anywhere. We can write backend server code, embedded microcontroller applications, and with WASM, even front end web applications, cloud functions and blockchains.

## let's make sound
So lets play. [Crates.io](https://crates.io) is where all the fun libraries live. Lets make some noise with [rodio](https://crates.io/crates/rodio) (Note to linux users: You may need to install libasound2-dev with something like `apt install libasound2-dev`

So rodio says it wants us to add it to our Cargo.toml under dependencies so lets do that
```
[dependencies]
rodio = "0.8.1"
```

You can take a look at the [rodio documentation](https://docs.rs/rodio/0.8.1/rodio/), but here's some simple code to make a sine wave.
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

So neat, but what's unwrap? If you take that out, the compiler will tell you `expected cpal::Device, found enum std::option::Option` It's saying, getting default_output_device might not exist and come back empty as an [Optional](https://doc.rust-lang.org/std/option/index.html) and you need to handle that. Here we handle it by just crashing if it goes poorly, well have to fix that later. I hope you're seeing a pattern, all the answers to our questions are in the Rust book and Language Reference.


* Play around here a bunch and get comfortable with the language. 
* You're going to start running into the borrow checker here. You''ll see `use of moved value` which means you're using some variable a second time, but the first you used it, it was 'consumed'. One solution to [ownership](https://doc.rust-lang.org/book/ch04-00-understanding-ownership.html) problems especially when you're not in a performance sensitive environment is to use the `clone()` function to make a copy of the thing, so try that. 
* Read the [rodio documentation](https://docs.rs/rodio/0.8.1/rodio/) or [source code and examples](https://github.com/tomaka/rodio) for more options to try, like fading in and out, playing multiple tones at the same time, loading audio files from the filesystem.

## let's make text
Lets get another package, a morse code package called [light-morse](https://crates.io/crates/light-morse) 
* See if you can install this one yourself.
* Have it print out `Hello World` but in morse code this time.


## let's make traits
Anybody can write a wall of code. The way we organize code in Rust is in modules and by making new types extending existing types. How did the morse author make a regular old String have a method called `to_morse()` Code for libraries is generally in a file called lib.rs and [light morse](https://github.com/luki/light-morse/blob/master/src/lib.rs) is no different. We see they defined a trait called MorseSubstitution which implements `to_morse()` on a Plaintext type (which is defined above as an alias type of String). This is [trait based inheritance](https://doc.rust-lang.org/book/ch10-02-traits.html) (called mixins in other languages) is an important concept in Rust.
* Lets make our own new trait in a new file called text.rs that extends Morse (the type to_morse() returns).
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
Instead of just `println!()` the whole String, here we're using [iterators](https://doc.rust-lang.org/book/ch13-00-functional-features.html) another important concept in Rust you need to get to know. We call the [.chars() function](https://doc.rust-lang.org/std/str/struct.Chars.html) on a String and for each over that iterator. It replaces the for loop over an index variable which removing the possibility of the common 'off by one' error most of us have made.

Now we need to make this new text.rs file available to our main.rs by importing it with `mod text;` and using your new trait with `use crate::text::Display` and then instead of println! in your main, you can just call `.display()`. This is what mine looks like
```
mod text;
use crate::text::Display;
use light_morse::MorseSubstitution;
use light_morse::MorseType;

fn main() {
    "Hello World"
        .to_string()
        .to_morse(MorseType::Gerke)
        .display();
}
```
Now you can `cargo run` to see it print
```
······−···−···−··· ·−−·−····−··−··−··
```
Note, this hooking many functions together is called the builder pattern, and along with iterators are specific examples of some of the more declarative (or functional) styles seen in Rust. Both syntaxes lead to rather pleasant and easy to reason about code.

* Can you use [thread::sleep](https://doc.rust-lang.org/std/thread/fn.sleep.html) to sleep between characters so it looks stuttery like text coming over the wire? You'll also needed to `stdout().flush()` between character writes or they'll get buffered by the operating system and all come out at the same time.


## put it all together
Can you edit the Display trait to make a long beep for dashes and a short beep for dots instead of/in addition to printing the character? You're probably thinking of doing this with an if statement, but a more Rusty solution would be to use [Match](https://doc.rust-lang.org/1.5.0/book/match.html). Note here, the morse characters are not a regular dash, but rather these these ascii characters `−` `·` 

### So why is Rust different than C++? 
Safety can be achieved in C, for time and money. But it's not baked into the language and it's not easy to teach. Further, Rust's lack of legacy is a benefit more than a hindrance. Rust only has one compiler, not competing vendors with multiple compilers. Further Rustaceans are designing the language in Github issues and nightly code releases instead of by committees that only Facebook and Google can afford to fly to and attend and thus our needs can never be met. Rust has modularized code and a single package manager which means code sharing is far easier and comes more naturally to Rustaceans. C++ hopes to ship module support in their 2020 release, but many people are still running C++ 2014 today... 


## My bold claim
Most of you here today have never and may never reach for C or C++ to script some simple CLI, filesystem or networking stuff, but I think you could and maybe even should start doing just that with Rust. Those skills will build and transfer to webassembly, microcontrollers, blockchains and the many other 'unhosted' environments in the future.

And let's face it we need sea changes in software development. We're 10 years into hackerspaces and bootcamps and are we making enough strong developers to guard the internet of things as we move to a billion devices in the years to come?

Rust's guarantees means more, better, code to be written faster. 



# Embedded Rust

Embedded Rust is a single example of 'unhosted' or 'freestanding' code. There is no operating system under the hood to lean on. In Rust this is generally referred to as no_std. We don't get to use anything that starts with std::. Including std::String! There are make-do packages and ways around, but for now let's assume we don't use what we call things that 'allocate'.

Just like the web design world, personified by React, is hell bent on removing global state, so too are the embedded Rust community driven. We do this by turning all microcontroller registers into singleton structs, extending functionality on them with traits, and restricting usage of them via the same ownership models we've seen in other Rust. This with the existing Rust safety guarantees vastly cleans up microcontroller code hygiene and is proving to crate far safer code.


## anatomy of no_std Rust program
Mostly the same, but a bit more boilerplate to start. 
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
Note the main function could be called anything, but is generally still called main for posterity. Also note that it returns `!` ie it doesn't return, as you can see the loop can't ever exit. The panic handler is what gets called if we something goes wrong. Here we import a package to handle that for us which will just halt the microcontroller.


## tomu
[Tomu](https://tomu.im) is a fun little arm microcontroller that fits in your USB-A port. I'm not affiliated in any way, I just hoped you'd always have a microcontroller in your pocket (or port). My thanks to [Crowd Supply](https://www.crowdsupply.com) for sponsoring these today. The Tomu has two leds (red and green), capacitive touch buttons and USB. With their [easy to use C code examples](https://github.com/im-tomu/tomu-quickstart) you could (in any language of your choosing) watch the USB for 'button presses' or have a program on your laptop trigger leds on the device like a notification led, and many more things.

But I'm part of a group developing the Rust capabilities for this board. GPIO blinky stuff is done, but USB and capacitive touch aren't yet so stay tuned.
 
## setup and blink
* Run `rustup target add thumbv6m-none-eabi` to add the compiler bits for this board
* Run `rustup component add llvm-tools-preview`
* Install [dfu-util](https://tomu.im/samples) from directions on this page
* Download [tomu rust](https://github.com/fudanchii/imtomu-rs)

Finally, this time when we run, we need to run in 'release mode' and since this package is a lib.rs there nothing to run, we can tell cargo to run its example file instead with `cargo run --release --example blink` I've hooked up scripts so that upon building successfully, it also uploads the code to your Tomu via dfu. If you want to reprogram your Tomu you'll need to pull it out of the port to reset it back to the bootloader.  

## what happened
There are ~three layers in the embedded Rust world. 

**Peripheral Access Crates (PAC)**

These are generally generated from hardware vendor xml files (called SVDs) of all the available registers and functionality available. We take those and generate the peripherals object which usually has something akin to a GPIO object. To write to a register to enable or disable an led you might write:
```
    p.GPIO.pb_doutset.write(|w| unsafe { w.bits(1 << 7) });
```
There's quite a bit happening here we need to talk about. The write function here takes a [closure](https://doc.rust-lang.org/book/ch13-01-closures.html), an anonymous function, is much like a passing a callback function. `write()` is going to call our closure with a variable of `w`. Also note the use of [unsafe](https://doc.rust-lang.org/book/ch19-01-unsafe-rust.html) The `bits()` function deals with memory directly and puts means the compiler can't guarantee our safety during this operation. That's OK and even common especially at the lowest levels of Rust, but it is designed to jump out at you and make you manually reason about the safety of this operation. Finally, notice there is no semicolon after the `bits()` function? You might have seen this in code above but we haven't addressed it yet. Rust has [implicit return of expressions](https://doc.rust-lang.org/book/ch03-03-how-functions-work.html#function-bodies-contain-statements-and-expressions) so on the last line of a function its common to leave off the semicolon in order to return the result of the line. Here `bits()` actually returns an integer (u8) and were implicitly returning that value. So what we've done is return 1<<7 or 0x80, or 128. I hope you'll agree this is a bit low level and ugly and while less error prone than C, still rather error prone. We further use traits as we've seen before to clean this up
 
**Hardware Abstraction Layer (HAL)**

```
pub trait OutputPin {
    fn set_low(&mut self);
}

impl OutputPin for B7<WiredAnd> {
    fn set_low(&mut self) {
        (*efm32::GPIO::ptr()).pb_doutset.write(|w| unsafe { w.bits(1 << 7) );
    }
}
```
and now we can simply
```
b7.set_low();
```
Now that's starting to look like high level code, but on a microcontroller! We can often do one better. 

**Board Support Package (BSP)**

For a specific revision of our board, like the Tomu, we could further clean up the api and offer drivers and things that only make sense with this specific set of hardware and features. We can also rename pins based on what's printed on the board. For electrical reason, LEDs often aren't necessarily on when they're high, we can clean this up here too so maybe now it looks like
```
red.enable();
```

Just like today, all this is generally often going to be done for you by the PAC, HAL, and BSP authors. You can find a list of things like this in crates.io in general and [awesome-embedded-rust](https://github.com/rust-embedded/awesome-embedded-rust) a curated list of crates specifically for hardware. Hopefully your favorite bit of hardware is already supported and there's some goodies to try on top of it.


References:
* http://blog.japaric.io/brave-new-io/
* https://doc.rust-lang.org/book
* https://docs.rust-embedded.org/book/start/registers.html
* https://docs.rust-embedded.org/embedonomicon/main.html


