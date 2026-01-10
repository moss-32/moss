# moss

Retro Fantasy Console

Thanks to [`@catnipped`](https://bsky.app/profile/ossianboren.bsky.social) for the **moss** name, creating the Enias font (built into the console), and choosing the VDP color palette!

## Install

### Brew

on macOs and Linux please use [brew](https://brew.sh/) to get **moss**:

```sh
brew tap moss-32/tap
brew install moss
```

you probably need the [Swamp (beta)](https://swamp-lang.org/) compiler as well:

```sh
brew tap swamp/tap
brew install swamp-beta
```

(tap only needs to be added once)

### Scoop

on Windows use [scoop](https://scoop.sh/) to get **moss**:

```powershell
scoop bucket add moss https://github.com/moss-32/scoop-bucket
scoop install moss
```

you probably need the [Swamp (beta)](https://swamp-lang.org/) compiler as well:

```PowerShell
scoop bucket add swamp https://github.com/swamp/scoop-bucket
scoop install swamp
```

(bucket only needs to be added once)

### Manual install

- download the moss executable from [releases](https://github.com/moss-32/moss/releases).

- download swamp cli from [swamp/swamp](https://github.com/swamp/swamp/releases).

## Specification

### Video Display Processor (VDP)

- Fixed 240×136 resolution
- 32-color palette (5-bit indexed color) ([DawnBringer 32](https://lospec.com/palette-list/dawnbringer-32))
- Built-in *Enias* font (8×8 characters)
- Sprite rendering with transparency bit
- Hardware primitives: lines, rectangles, circles (filled and unfilled)

### Audio Processing Unit (APU)

- 8-voice mixer
- 64 sounds with ADSR envelopes
- Output: 22,050 Hz (22.05 kHz), 8-bit unsigned PCM, stereo.

Each sound:

- ADSR envelope generator (attack/decay/sustain/release in milliseconds),
- Root key (MIDI note)
- 4 Loop segments
- Sample playback (mono or stereo, 22.05 kHz)

Each voice:

- volume: 0.0 to 1.0 (Q15.16 fixed-point)
- pan: -1.0 (left) to +1.0 (right) (Q15.16 fixed-point)

### Controller Interface (CI)

Dual digital game controller support.

- **D-Pad**: Arrow Keys (↑, ↓, ←, →) or `E`, `D`, `S`, `F`
- **A Button**: `Z`, `C`,
- **B Button**: `X`, `V`,
- **START**: `enter` and `ESC`

#### Controller Behavior

- All inputs are digital (pressed or released).
- Input state is sampled once per frame.

upcoming revision: second player with gamepad

## Install

### Windows

#### Scoop



- Moss

```PowerShell
scoop bucket add moss https://github.com/moss-32/scoop-bucket
scoop install moss
```




## Getting Started

- create your project directory

- initialize with the demo project

```sh
moss --init
```

the demo project should look like this:

```sh
packages/moss/lib.sw
swamp.yini
main.sw
```

## Usage

Compile with:

```sh
swamp build
```

build places the .moss file at `out/main.moss`

then you can run it with:

```sh
moss
```

(`out/main.moss` is default)

...and you should see:

![spinning](images/spinning_cube.gif)

## API

### Constants

```rust
const WIDTH = 240
const HEIGHT = 136
```

### Display

```rust
fn wait_vsync()
```

Waits for vertical retrace (typically 60 Hz).

```rust
fn clear(palette_index: Int)
```

Clears the screen to a single color.

```rust
fn set(x: Int, y: Int, palette_index: Int)
```

Sets a single pixel.

```rust
fn get(x: Int, y: Int) -> Int
```

Reads the color of a pixel.

### Sprites

```rust
fn sprite(x: Int, y: Int, width: Int, colors: [U8])
```

Draws a sprite. Color 0 is transparent.

```rust
fn sprite_flip(x: Int, y: Int, width: Int, colors: [U8], flip_h: Bool, flip_v: Bool)
```

Draws a sprite with horizontal/vertical flipping.

### Shapes

```rust
fn line(x0: Int, y0: Int, x1: Int, y1: Int, palette_index: Int)
```

Draws a line.

```rust
fn box(x: Int, y: Int, width: Int, height: Int, palette_index: Int)
```

Draws a filled rectangle.

```rust
fn box_outline(x: Int, y: Int, width: Int, height: Int, palette_index: Int)
```

Draws a rectangle outline.

```rust
fn circle(x: Int, y: Int, radius: Int, palette_index: Int)
```

Draws a circle outline.

```rust
fn circle_fill(x: Int, y: Int, radius: Int, palette_index: Int)
```

Draws a filled circle.

### Text

```rust
fn char(x: Int, y: Int, ch: U8, palette_index: Int)
```

Draws a single character.

```rust
fn text(x: Int, y: Int, text: String, palette_index: Int)
```

Draws a text string.

### Input

```rust
struct Gamepad {
    up: Bool,
    down: Bool,
    left: Bool,
    right: Bool,
    a: Bool,
    b: Bool,
    start: Bool,
}

fn gamepad(player: Int) -> Gamepad
```

Reads gamepad state for player 0 or 1.

### Audio

```rust
struct Adsr {
    attack: Int,    // Attack time in milliseconds
    decay: Int,     // Decay time in milliseconds
    sustain: Float, // Sustain level (0.0 to 1.0)
    release: Int,   // Release time in milliseconds
}

struct SoundDefinition {
    adsr: Adsr,
    root_note: Int, // MIDI note number
}

fn sound_mono(id: Int, raw: [U8], sound: SoundDefinition)
```

Defines a mono sound (id: 0-63). The raw samples are unsigned 8-bit values.

```rust
fn sound_stereo(id: Int, raw: [U8], sound: SoundDefinition)
```

Defines a stereo sound (id: 0-63). The raw samples are unsigned 8-bit values interleaved (L,R,L,R,...).

```rust
fn note_on(voice: Int, note: Int, sound_id: Int, volume: Float)
```

Starts playing a note on a voice (0-7).

```rust
fn note_off(voice: Int)
```

Stops playing on a voice.

```rust
fn pan(voice: Int, pan: Float)
```

Sets voice panning (-1.0 = left, 0.0 = center, 1.0 = right).

## Examples

### Plasma

```rust
use moss::*

mut frame = 0

while true {

    wait_vsync()

    for y in 0..HEIGHT {
        for x in 0..WIDTH {
            v1 := (x + frame) / 11
            v2 := (y + frame*2) / 13
            v3 := (x + y + frame) / 17
            v4 := (x - y + frame*3) / 19

            palette_index := (v1 + v2 + v3 + v4) % 32

            set(x, y, palette_index)
        }
    }

    frame += 1
}
```

### Starfield

```rust
use moss::*

const CENTER_X = WIDTH/2
const CENTER_Y = HEIGHT/2
const FLY_SPEED = 3

mut frame = 0

while true {

    wait_vsync()

    clear(0)

    for i in 0..128 {
        star_x := (i * 39 % 160) - 80
        star_y := (i * 13 % 100) - 50

        z := (i * 11 - frame * FLY_SPEED) % 400

        scale := 400 - z

        screen_x := CENTER_X + (star_x * scale / 400)
        screen_y := CENTER_Y + (star_y * scale / 400)

        brightness := 10 + (scale / 11)

        if brightness > 1  {
            palette_index := if brightness >= 32 {
                31
            } else {
                brightness
            }
            set(screen_x, screen_y, palette_index)
        }
    }

    frame += 1
}
```

*Copyright (c) 2026 Peter Bjorklund. All rights reserved.*
