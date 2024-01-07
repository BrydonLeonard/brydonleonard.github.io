---
layout: post
title:  "The MIDI file standard"
date:   2024-01-07
categories: learning
---

# The MIDI file standard

MIDI files are audio files that your PC can use to play a song, but MIDI was created as wire protocol for _live_ music performance. It's real-time because when the musician presses a key on their keyboard, it immediately sends a MIDI message to a synthesizer to play a note. Synthesizers can play a whole range of sounds, from piano or violin to the sound of tweeting birds. I was interested in learning a little more about MIDI and wrote this post to give a high-level overview for anyone else interested in learning about it.

## A bird's eye view of the standard

Arguably the most important piece of the standard is the MIDI **_message_**; they're what MIDI instruments send to synthesizers to tell them to play a note. Each message is sent on one of 16 MIDI **_channels_**, which can be set up to play the sound of a different instrument. In MIDI, the instruments are called **_programs_**. If it supports it, a single synthesizer could also listen on multiple channels, but have each configured to play a different program.

To give you an example a MIDI message, two types are "Note On" and "Note Off" messages, which cause the synthesizer to start or stop playing a note. Each message also comes with additional **_message data_**. For Note on/off messages, that data is:
- the _pitch_ of the note
- the _velocity_ of the note. It's called that because playing a piano key _faster_ makes it louder.

MIDI messages don't contain any information about when they should be played (they're always played immediately when they're received). If you want a MIDI synthesizer to play a whole song, you need something to send all the messages at the right time; for that, you use a MIDI **_sequencer_**. A second type of message called **_meta messages_** control the sequencer itself and set things like the tempo or time signature.

MIDI **_files_** are binary files containing the timing information used by sequencers. The files contain multiple **_tracks_**, each of which usually\* corresponds to a single instrument. Each track contains a list of MIDI **_events_**, which are a MIDI message paired with a **_delta time_** (the time between MIDI messages).

> \* I say _usually_ because it's also perfectly valid to have a single track that contains multiple instruments, each on a different channel.

## A slightly lower-flying bird's eye view of the standard

I thought it could be fun to dive a little further into the MIDI file binary format, so I wrote [a parser](https://github.com/BrydonLeonard/MidiParser) that describes the contents of a MIDI file. Before we take a look at its output, here's a rough outline of the files' structure:

```
Header:
    HeaderPrefix            # Always 4d 54 68 64 00 00 00 06
    SequenceType            # Whether tracks should be played simultaneously or synchronously
    TrackCount              # The number of tracks in the file
    DeltaTicksPerCrotchet   # The ratio of delta ticks to crotchets. This and the time signature determine the length of a delta tick
Tracks:
    - TrackHeader:
        TrackHeaderPrefix   # Always 4D 54 72 6B
        TrackLength         # Number of bytes in the track
      Events:
        - DeltaTime         # This is a variable-length encoded
          Message           # The actual MIDI message to be sent
        - DeltaTime
          Message
        ...
    - TrackHeader:
    ...
```

## A MIDI file case study

Now that we know what to expect, let's look at a specific MIDI file. [This one is a bassline that I found on the MIDI wikipedia page](https://upload.wikimedia.org/wikipedia/commons/a/a0/Bass_sample.mid). The right-hand side contains the full description, so don't worry about the left if you're unfamiliar with hexadecimal numbers:

```
                 4d 54 68 64 00 00 00 06 |> MIDI file standard header
                                   00 01 |> 1 (multiple synchronous tracks)
                                   00 02 |> There are 2 tracks
                                   00 f0 |> There are 240 ticks per crotchet
                             4d 54 72 6b |> Standard track header. Track 1 starting
                             00 00 00 1c |> The track is 28 bytes long
                 00 ff 58 04 04 02 18 08 |>   Delta time =     0. [Meta] Set time signature to 4/4. Metronome will tick every 1 beat(s). There are 8 demisemiquavers per crotchet
                    00 ff 51 03 07 a1 20 |>   Delta time =     0. [Meta] Set tempo to 120 bpm
              00 ff 54 05 40 00 00 00 00 |>   Delta time =     0. [Meta] This track's SMPTE offset is [64, 0, 0, 0, 0]
                             00 ff 2f 00 |>   Delta time =     0. [Meta] END OF TRACK
                             4d 54 72 6b |> Standard track header. Track 2 starting
                             00 00 00 87 |> The track is 135 bytes long
                                00 c0 21 |>   Delta time =     0. [Channel  0] Switch to program 33
                             00 b0 07 7f |>   Delta time =     0. [Channel  0] Adjust controller 7 to value 127
                             00 90 2d 4e |>   Delta time =     0. [Channel  0] Stop A3 with velocity 78
                          81 01 80 2d 40 |>   Delta time =   129. [Channel  0] Start A3 with velocity 64
                          81 67 90 30 51 |>   Delta time =   231. [Channel  0] Stop C4 with velocity 81
                             78 90 32 4f |>   Delta time =   120. [Channel  0] Stop D4 with velocity 79
                             ... Snip
                             05 80 2d 40 |>   Delta time =     5. [Channel  0] Start A3 with velocity 64
                          81 61 80 30 40 |>   Delta time =   225. [Channel  0] Start C4 with velocity 64
                             0a 90 2d 4e |>   Delta time =    10. [Channel  0] Stop A3 with velocity 78
                             5a 80 2d 40 |>   Delta time =    90. [Channel  0] Start A3 with velocity 64
                          83 05 ff 2f 00 |>   Delta time =   389. [Meta] END OF TRACK
```

I found it interesting to see the MIDI file standard in action, so here are some notes to give you a little more insight:
- This file has two tracks, though the first one was just used to configure the sequencer with meta-messages.
  - Because the meta messages are never sent to the synthesizer, they have no associated channel.
- To give you a sneak peek into MIDI's binary encoding (see the references for more), `80` means "start a note on channel 0" while `90` means "stop a note on channel 0". You'll see those repeated several times on the left side above. 
- The `Switch to program 33` message sets up channel 0 to use the bass guitar program (33). If we switched that to 126, the song would be played [by a helicopter instead!](https://en.wikipedia.org/wiki/General_MIDI)

## References

I've tried to keep the technical details light in this post. If you want any more information on the MIDI standard, these are all great resources to learn a little more:
- [recordingblogs.com](https://www.recordingblogs.com/wiki/musical-instrument-digital-interface-midi)
- [faydoc.tripod.com](https://faydoc.tripod.com/formats/mid.htm)
- [The Java MIDI package docs](https://docs.oracle.com/javase/tutorial/sound/overview-MIDI.html)
- [Wikipedia's handy list of the MIDI program numbers](https://en.wikipedia.org/wiki/General_MIDI)