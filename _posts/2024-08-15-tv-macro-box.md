---
layout: post
title:  "A TV macro box"
date:   2024-08-15
categories: automation
tags: automation
---

My grandparents can struggle to navigate around their new TV and occasionally get stuck on the wrong input or in some menu that they don't understand. In the past, that's meant that they get stuck without a working TV until someone visits and fixes it for them. This post is about a project I worked on to try help them out. 


## The idea

At a high level, my idea was to build a box with buttons and an IR emitter. Each of the buttons would send some sequence of IR commands to get my grandparents' devices in the desired states. Because I don't live in the same city as them (and would be emigrating before I visited next) my solution also needed to work first time to avoid shipping it back and forth.

What I settled on was a box with both an IR emitter _and_ an IR receiver. It would use the receiver to record IR commands as macros to be played back later. My dad (who's more comfortable navigating TVs) could then take the box to my grandparents and record the macros directly from their remotes[^7]. 

## The hard(ware) parts

I'm a software engineer, so each of these projects is a fun process of discovering new things I didn't know about electronics. For the macro box, I landed on a design with four key components:

### 1 - A bank of 8 buttons, each with a pull-down resistor [^1]

When you read from an input pin on an Arduino's, you get back either a HIGH or LOW value, based on the pin's voltage. When input pins aren't connected to anything, however, that voltage is undefined and can vary randomly (that's called _floating_). When the buttons connected to input pins aren't pressed, their circuit is open and it's as if the input is entirely disconnected, which leads to the pin floating.

Pulldown resistors prevent floating by creating a (high resistance) path straight from the pin to ground. As a result, when the button circuit is open, the input doesn't float and consistently reads a LOW value.

### 2 - An RGB LED that I used as a status indicator

RGB LEDs work just like three separate LEDs that live in a single housing, so there's nothing super interesting there[^5]. In some prior projects, however, I've just wired LEDs straight to the Arduino's pins, but I've since learned that not including a resistor in the circuit leads to excessive current that can damage both the pin and the LED. To find the approximate resistance you need, take the voltage and divide it by the maximum rated current of the components in the circuit. In the case of an LED that can only handle 20mA (like my RGB LED), we have:

$$
minimum\ resistance = \frac{3.3V}{0.02A} = 165\ohm
$$

Because I'm less worried about making the RGB LEDs as bright as possible and more worried about protecting them and the Arduino, I rounded up to 220$\ohm$, the next largest resistor that I had in my toolbox.

### 3 - An IR LED controlled through a transistor

IR LEDs are really different to LEDs that emit visible light slightly, so the logic around picking resistors is much the same as above. The difference here is in how I controlled it. I needed the IR emitter to be as bright as possible to give the box the greatest possible range. As such, instead of powering it straight from the Arduino's data pins (which the internet tells me would limit its brightness), I powered it directly from the 3.3V pin on the ESP32. 

Connecting straight to 3.3V power would just have the LED emitting constantly. To control it, I used a transistor. Transistors are kinda like little light switches where the switch itself is controlled with electricity. The _base_ pin on a transistor is like the "switch" and controls how much electricity flows from the _collector_ pin to the _emitter_ pin[^2]. 

![Circuit Diagram](/assets/images/2024-08-15-tv-macro-box/Transistor.png)

### 4 - An IR receiver

The last component is the IR receiver, which is used to record new macros. I found a handy pack of IR emitters and receivers online, but didn't realise that the ones I bought had no signal amplifiers built in[^3]. As a result, I had to boost the signal myself. I did so by connecting the output _from_ the IR receiver to the base pin (the "switch" pin) of a second transistor. The trickle of power that then came through the receiver when it was exposed to IR light closed the transistor circuit and allowed a larger amount of power to flow through from 3.3V to an input pin on the Arduino.

For reasons that are unclear to me, this didn't work until I added a pulldown resistor. My best guess is that when the transistor is open, it behaves like an unpressed button (or any other open circuit), so the input pin was floating and unusable.

### Putting it all together

Here's the final circuit diagram with the four key components labelled:

![Circuit Diagram](/assets/images/2024-08-15-tv-macro-box/CircuitDiagram.png)

## Putting the magic electricity in the rocks

Luckily, libraries exist to send and receive IR signals, so I didn't have to figure out the protocols from scratch. The [IRremote](https://www.arduino.cc/reference/en/libraries/irremote/) library, which seems to be the most popular, doesn't work on ESP32s, so I used [IRRemoteESP8266](https://github.com/crankyoldgit/IRremoteESP8266) instead. The library's auto-generated documentation is borked, but it's easy enough to read through the comments directly in the header files. it also has some nice example projects, which make it easier to get going. 

I built a little state machine with three states to control the box:
- `READY_TO_PLAYBACK`: If a button is pressed and that button has a macro saved, the macro will be played through the IR emitter
- `READY_TO_RECORD`: If a button is pressed while in this state, the box will move to the recording state and start recording to a new empty macro
- `RECORDING`: All IR signals received by the box while in this state are appended to the new macro. If the button is pressed again, the macro is saved to the ESP32's flash memory.

The box has a DIP switch that moves it between the `READY_TO_PLAYBACK` and `READY_TO_RECORD` states. I recessed the switch in the body of the box to prevent it from being flipped accidentally. Each of the states also has associated colours on the status indicator LED to make it clear what's happening.

### Saving macros

Given the 4KB of flash memory on my ESP32 (shared with the compiled program itself), saving macros between power cycles was actually a fairly interesting constraint. [^6]. Each IR command (as modelled by the library I was using) consists of three components:
- A protocol: The library supports 255 different manufacturers' IR protocols, so this fits neatly in a u8.
- A size: This is the length of the command. The IR library stores it as a u8, so I did the same!
- The command itself: This is stored in a u64.

The ESP32 Preferences library presents a handy interface for saving key/value pairs, but back-of-the-envelope calculations told me that storing the above data naively would've limited me to fewer than 9 commands per button, which I wasn't too happy with. Instead, I opted to create one record per command, where the components of the command are encoded as single u64 value. 

In my testing, I didn't come across any command values that couldn't fit in a u32, so I threw caution to the wind and used the 16 most significant bits of the u64 to save the size and protocol (8 bits each). That left 48 bits for the command.

```cpp
uint64_t encodeCommand(SavedCommand command) {
  uint64_t size = command.size;
  size = size << 56;
  uint64_t protocol = command.protocol;
  protocol = protocol << 48;
  uint64_t value = command.value;

  uint64_t encoded = value | protocol | size;

  return encoded;
}
```

I'd been aiming for 16 commands per macro and this approach did so with 1KB left over for whatever overhead is introduced by the Preferences library's saved key hashes.

### Playing them back

Once macros were saved, playing them back consisted of shoving them through the IR library's `send` method. Because IR communication is unreliable, I enabled repeats in the IR library (so it idempotently repeats each command once) and replayed each command twice (which simulates someone holding down a button for slightly too long). I didn't take the most scientific approach to picking those numbers, but the result was consistent enough for me.

I also built in a 750ms delay between each command in a macro to give devices time to respond. That's still not enough time for most devices to power on, however, so "power on" can't really be used as part of a larger macro.


## Reasoning about commands

With macro playback working, I put together a guide on how to make "good" macros. Not all sequences of commands work well and there are two key behaviours to aim for when programming new ones:
- **Macros need to be idempotent.** The idea is that these buttons help my grandparents get out of situations that they don't understand. Pressing the button multiple times should land them in the same place every time rather than putting them in some _new_ confusing state because they pressed it one too many times.
- **Macros should work from anywhere.** Because the macros are open-loop (and can't incorporate any feedback from the device that they're controlling), the macros can't have _any_ preconditions.

The best way I've found to achieve those behaviours is to find a sequence of commands that reliably gets the device to some known state. If you can build a sequence of commands to navigate from that known state to a _good_ state, then you can combine the two sequences to get to a good state from anywhere.

As an example, on our Samsung TV, a macro to switch to input two looked like:
```
Exit
Exit <-- The two "Exit" presses always get back to the home screen
Input
Input <-- Pressing input multiple times opens the input select screen and moves the cursor all the way to the first option
Left
Left
Left
Left
Left <-- There are only five possible inputs, so pressing left five times means that we know we're on the first one
Right <-- From the known state on the first input, move one to the right
Select <-- Select the second input
```

## But does it work?

Yes! Here's a video:

<iframe width="560" height="315" src="https://www.youtube.com/embed/EjT0cX3k6I8?si=iDY-MABrLRenPPpI" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" referrerpolicy="strict-origin-when-cross-origin" allowfullscreen></iframe>

I've since couriered it to my family and it seems to be working for my grandparents, so I'd call the project a success.

[![Hits](https://hits.seeyoufarm.com/api/count/incr/badge.svg?url=https%3A%2F%2Fbrydonleonard.github.io%2Fautomation%2F2024%2F08%2F15%2Ftv-macro-box.html&count_bg=%2379C83D&title_bg=%23555555&icon=&icon_color=%23E7E7E7&title=hits&edge_flat=false)](https://hits.seeyoufarm.com)


## Footnotes

[^1]: If I'd remembered that ESP32s have internal pull-down resistors, I could've saved myself some time. I didn't remember, though.
[^2]: If you search for "transistor pinout", make sure that you're actually looking at the right kind. NPN and PNP resistors pins are reversed and that caused me to waste a bunch of time.
[^3]: Initially, I didn't realise I needed to amplify the signal at all
[^4]: "Lived" because I now live even _further_ away
[^5]: If you are new to electronics things, the "blink" project that comes built into the Arduino IDE will help you make a single-colour LED flash.
[^6]: Otherwise load shedding starting again in South Africa would render the box basically useless.
[^7]: Hardcoding the commands might've been a nicer solution, but I didn't have a reliable way to figure out the protocols used by their devices without being there in person or building an IR tester and sending that off first.