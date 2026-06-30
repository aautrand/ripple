# Project Brief: Slimmed-Down TidalCycles (Python Proof of Concept)

## Goal
Build a minimal, proof-of-concept clone of **TidalCycles** in Python. TidalCycles is a
live-coding music tool where you write terse pattern strings (its "mini-notation," e.g.
`"bd sn hh sn"`) that get turned into timed musical events and played as sound. This POC
reproduces the core idea — **patterns → timed events → audio** — but in a stripped-down form.

**Explicitly out of scope:** live editing / hot-reloading of sound. This is a batch tool,
not an interactive REPL.

## How it's used
The user writes a song in a separate document, then runs:

```
python tidal.py song.txt
```

The program parses the song file, renders it to audio, and plays it (looping a configurable
number of cycles, then exits).

## Key design decisions (already settled)

### Sound generation: synthesize everything in code
Do **not** use sample `.wav` files, MIDI, or external synths (SuperDirt/SuperCollider).
Generate all audio mathematically with `numpy`. Rationale: no files to source/bundle, and
pitched notes are trivial (a sine wave at a frequency).

- **Kick (`bd`):** short low sine (~50–60 Hz) with a fast downward pitch sweep and a quick
  amplitude decay envelope.
- **Snare (`sn`):** short burst of filtered/white noise + a mid tone, fast decay.
- **Hi-hat (`hh`):** very short high-frequency noise burst, very fast decay.
- **Clap (`cp`):** noise burst (similar to snare, optional).
- **Pitched tone (synth):** sine (or simple square) wave at the note's frequency, with a
  short attack/decay envelope.
- Every generated sound must have a short envelope (a few ms attack, smooth decay) so there
  are no clicks/pops.

### Rendering: pre-render the whole song into one buffer (no real-time scheduling)
Because live editing is out of scope, do **not** schedule sounds in real time. Instead:

1. Compute the total song length in samples (cycle duration × number of cycles).
2. Allocate one silent `numpy` float buffer.
3. For each event, generate its sound array and **mix** (add) it into the buffer at the
   correct sample offset.
4. After mixing all events, clip/normalize to prevent overflow.
5. Play the finished buffer once.

This avoids all real-time timing jitter and is the most robust approach for a POC.

### Audio playback: `sounddevice`
Use `sounddevice.play(buffer, samplerate)` followed by `sounddevice.wait()`. It's one
`pip install` and bundles PortAudio on macOS.

- **Fallback option (mention in code/README):** render the buffer to a `.wav` using the
  stdlib `wave` module and play it via macOS `afplay` — a zero-third-party-dependency path.

### Dependencies
- `numpy` (required)
- `sounddevice` (required, with the `afplay`/`wave` fallback noted)

## The song document format (the slimmed DSL)

A plain text file. Keep the mini-notation **as slim as possible** — just enough for a simple
beat. Example:

```
tempo 120          # BPM; one cycle = one bar (4 beats)
cycles 4           # how many times to loop the whole thing before stopping

d1 drum  "bd sn bd sn"     # a drum track
d2 drum  "hh hh hh hh"     # layered on top of d1
d3 synth "c e g ~"         # a pitched melody; ~ = rest
```

### Format rules
- **Comments:** anything after `#` is ignored.
- **`tempo <BPM>`:** sets beats per minute. One **cycle = one bar = 4 beats**, so cycle
  duration in seconds = `(60 / BPM) * 4`.
- **`cycles <N>`:** how many times the whole pattern loops before the program stops.
- **Track lines:** `d<number> <instrument> "<pattern>"`
  - `d1`, `d2`, … are track slots (like Tidal's `d1`–`d9`). All tracks play
    **simultaneously / layered** (stacked).
  - `<instrument>` is either `drum` or `synth`.
  - The pattern string is **space-separated steps**, divided **evenly** across one cycle.
    If there are N tokens, each occupies `1/N` of the cycle.
  - `~` = a rest (silence for that step).
  - **`drum` tracks:** each token is a drum name — `bd` (kick), `sn` (snare), `hh` (hi-hat),
    `cp` (clap). Unknown names can warn or be treated as rests.
  - **`synth` tracks:** each token is a note — letter names `c d e f g a b` (map to
    frequencies, e.g. middle-octave; optionally support `c4`, sharps, or numbers `0–7` as
    scale degrees). Convert note → frequency → tone.

### Slimming notes (intentionally left out for the POC — keep extensibility in mind)
- No nested sub-sequences `[bd sn]`.
- No polymeter, no euclidean rhythms, no `*n` repeat operator (can be added later — design
  the parser so this is easy).

## Suggested architecture (single file `tidal.py`, or a few small modules)
1. **Parser** — read the file line by line; extract `tempo`, `cycles`, and a list of tracks
   (each: slot id, instrument, list of step tokens).
2. **Pattern → events** — for each track, convert N tokens into N events. Each event has:
   instrument, token (drum name or note), and a start time as a fraction of the cycle
   (`step_index / N`). Combine with cycle duration + cycle index to get absolute start times
   across all loops.
3. **Synth module** — functions `kick()`, `snare()`, `hat()`, `clap()`, `tone(freq, duration)`,
   each returning a `numpy` array (with envelope).
4. **Renderer** — allocate the full silent buffer; for each event across all cycles, generate
   its sound and mix it in at its sample offset; clip/normalize.
5. **Playback** — `sounddevice.play` + `wait` (with the `afplay` fallback).

Use a sample rate of 44100 Hz. Mono is fine for the POC.

## Recommended build order (get something audible ASAP)
1. **Synth + render + play a hardcoded `bd sn bd sn`** — verify audio works on the machine first.
2. Add the **parser for a single track**.
3. Add **multiple layered tracks + tempo/cycles**.
4. Add the **`synth` instrument** with note-name → frequency conversion for melodies.

## Features the POC must demonstrate (all four)
- **Rhythmic sequencing** — multiple steps per cycle, looping over time.
- **Multiple layered tracks** — several patterns playing at once (e.g. drums + bass).
- **Tempo control** — `tempo` sets BPM.
- **Pitched melodies** — `synth` tracks with real pitches, not just drum hits.

## Environment
- Target machine: macOS (Darwin), Python 3, shell `zsh`.
- Deliverables: `tidal.py` (the program), an example `song.txt`, and a short README with
  install (`pip install numpy sounddevice`) and run instructions.