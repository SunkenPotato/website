+++
date = '2026-07-20T21:55:53+02:00'
draft = true
title = 'Playa: Introduction'
+++
Anybody who has played an instrument has or will consider at some point forming a band to create music with people.
It's normal, you get bored of playing with the backing track, or you feel lonely practicing the same C Blues scale again.

I've also had this experience. Only being able to play with yourself and the pre-programmed computer does get a bit tedious at some point,
especially if you only play one instrument. Imagine if you played the guitar and the drums, for example! You could record yourself 
on the drums and then improvise on the guitar! As a matter of fact, this is my exact situation – except that I don't play the drums.

Since I'm not going to learn the drums (or buy a drumset for that matter), I set out to find an alternative. The first thing that 
comes to mind is synthesisers. You could use one on your computer, or buy a full-on keyboard. There is, to my surprise though, a far 
more ergonomic solution for synthesising drums: 
> A drum machine!

So, the only logical next step for me was to build one myself, because I can't stand using something without knowing how it works. 

In the following articles with the [#playa](/tags/playa) tag, I'll be attempting to build my own drum machine using the 
[Pocket Operator](https://teenage.engineering/store/po-12) series by Teenage Engineering as inspiration, called **Playa**
(from "player", which I'm sure somebody has though of before).

This is not really a guide on how to build a drum machine, though it can definitely be used as one. You can try to follow along,
but I definitely will make breaking changes and swap out parts throughout the series, and one should always keep in mind that this is 
one of my first electrical engineering projects. But if I do finish it, I will upload a final schematic & source to my GitHub or here.

## Parts & Software
- The MCU for a Playa – that's the **M**icro**c**ontroller **U**nit – is going to be an ESP32 (specifcally the ESP32-WROOM-32E). This is the one part I'm fairly confident
about, since it has sufficient GPIOs (for now), WiFi (for over-the-air updates or something else), Bluetooth (maybe for wireless
playback) and is relatively cheap, going for anywhere between 10-20€.

- I'm also adding a `128x64` OLED display to give the user information about the current state of the device 
([this one](https://adafru.it/938)).

- The speaker itself is pretty much up to you, but the amplifier and DAC (digital to analog converter) I'll use is the `MAX98357A`,
  since it's fairly capable and cheap.

- You can use any switches you'd like for this, and make sure you have the "basic" components such as wire, resistors, etc.

I'm going to give this project a go in Rust, with some help from AI of course, since the ESP32-Rust ecosystem has matured quite a bit
in the last few years.

## First Experiments

The first thing you see when you generate an ESP32-Rust project is the `main.rs` file. It's full of boilerplate and looks something
like this:
```rs
#![no_std]
#![no_main]
#![deny(
    clippy::mem_forget,
    reason = "mem::forget is generally not safe to do with esp_hal types, especially those \
    holding buffers for the duration of a data transfer."
)]
#![deny(clippy::large_stack_frames)]

use esp_hal::clock::CpuClock;

extern crate alloc;

// This creates a default app-descriptor required by the esp-idf bootloader.
esp_bootloader_esp_idf::esp_app_desc!();

#[allow(
    clippy::large_stack_frames,
    reason = "it's not unusual to allocate larger buffers etc. in main"
)]
#[main]
fn main() -> ! {
    esp_println::logger::init_logger_from_env();

    let config = esp_hal::Config::default().with_cpu_clock(CpuClock::max());
    let peripherals = esp_hal::init(config);

    // These GPIO pins are in use by some feature of the module and should not be used.
    let _ = peripherals.GPIO6;
    let _ = peripherals.GPIO7;
    let _ = peripherals.GPIO8;
    let _ = peripherals.GPIO9;
    let _ = peripherals.GPIO10;
    let _ = peripherals.GPIO11;
    let _ = peripherals.GPIO16;
    let _ = peripherals.GPIO20;

    esp_alloc::heap_allocator!(#[esp_hal::ram(reclaimed)] size: 98768);

    loop {}
}
```

There's quite a bit going on here, and it's best to break it down to understand what.

1. **Declarations**: We have a bunch of compiler directives like `#![no_std]` & `#![no_main]` as well as `#![deny(...)]`. These are
   important because on the ESP32 – unlike on a normal computer – we don't have access to the standard library, only `core` & `alloc` 
   (technically, we do, but you have to use a different framework). Also, embedded projects don't have a normal `fn main()`. You 
   have to disable the regular main function and use the special `#[main]` attribute used further down.

2. **Main function**: The first odd thing about the main function is that it returns the special type `!` (the "never" type). 
   This means that the program will never end; in fact, the only way this program *can* end is by shutting the ESP off.

   Inside the main function, we setup the config and the access to "peripherals". This is stuff like GPIOs & serial interfaces.

3. **`heap_allocator!`**: Remember how I said we have access to `alloc`? This is the reason why. Technically, the ESP doesn't have a 
   real "heap", so we have to use a pre-allocated buffer.

4. **Loop**: Currently, this is empty. This is what causes the program to never return.

### Testing the speaker
Since I ordered a speaker + amp, the best thing thing to do is probably ensure that it functions correctly, and the most obvious way 
to do that is to play the most simple sound that exists, a sine wave – specifically at 440Hz, since that's the default tuning for 
most instruments.

The way we communicate with an audio device on peripherals such as this is through a protocol called **I2S** (Inter-IC Sound).

Very simply put, all we have to do is supply the level to the `MAX98357A` per sample that it's supposed to play over and over again.
This is somewhere between `-32768` and `+32767`. So to generate sound, all we have to do is write a sine function that generates
a sample at 440Hz between those two values:

$$
f(t) = 32767 \cdot sin(2 \pi \cdot 440 \cdot \frac{t}{f_s})
$$

Wait, what's \\(f_s\\)? \\(f_s\\) is the sample rate we use. In our case, this is going to be 44.1kHz – so the speaker will play
44100 samples per second.

So let's go ahead and implement this, then. 

First, we'll need to connect our DAC/amp to the ESP32 and setup an audio buffer:

```rs
// Setup the buffer
let (_, _, tx_buf, tx_desc) = esp_hal::dma_buffers!(0, 4092 * 4);
// Create the I2S interface
let i2s =
    esp_hal::i2s::master::I2s::new(
        peripherals.I2S0, 
        peripherals.DMA_I2S0, 
        Default::default()
    )
    .unwrap();

// Connect the sender channel to the given pins.
let mut i2s_tx = i2s
    .i2s_tx
    .with_dout(peripherals.GPIO32) // DIN pin on the MAX98357A
    .with_bclk(peripherals.GPIO33) // BCLK
    .with_ws(peripherals.GPIO25)   // LRC
    .build(tx_desc);
```

And then we need to continuously send audio through the I2S channel, but you could of course gate this behind a button interrupt
or such.

```rs
// This tracks the current frame, since the closure within push_with doesn't always 
// call every 44100 samples.
let mut frame_ctr = 0usize;
// "read from tx_buf this forever"
let mut transfer = i2s_tx.write_dma_circular(tx_buf).unwrap();

// [0.0; 1.0]
const LOUDNESS: f32 = 0.5;
const FREQUENCY: f32 = 440.0; // Standard A, adjustable to anything

loop {
    _ = transfer.push_with(|buf| {
        // We divide by four because there are two channels (left & right) and each sample is two bytes.
        // Despite the channels not being used individually, we need to supply them anyway
        for frame in 0..buf.len() / 4 {
            let base = frame * 4;
            let sample = (
                LOUDNESS * 
                i16::MAX as f32 * 
                libm::sinf(
                    2.0 * core::f32::consts::PI * FREQUENCY * 
                    // how far along are we within the cycle?
                    (frame_ctr as f32 / 44100.0),
                )) as i16;

            let bytes = sample.to_le_bytes();

            buf[base] = bytes[0];
            buf[base + 1] = bytes[1];
            buf[base + 2] = bytes[0];
            buf[base + 3] = bytes[1];

            // wrapping add ensures no overflow and resets it to 0
            frame_ctr = frame_ctr.wrapping_add(1);
        }
        buf.len()
    });
}
```

That's pretty much all I have for this post, next time I'd like to try implementing a triggerable sound, such as a kick-drum or 
a decaying sine wave.