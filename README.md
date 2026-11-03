# catprint

A single-file command-line tool that prints text, images, emoji and Braille art from macOS on GB01/GB02/GB03 ("cat printer") Bluetooth thermal printers.

These printers do not pair in the macOS Bluetooth settings. They only speak BLE (Bluetooth Low Energy). `catprint` finds the printer by name, turns the content into a 384 pixel wide black and white image, and sends it directly.

## Requirements

- macOS (font paths are macOS specific)
- [uv](https://docs.astral.sh/uv/). The script is a `uv` script and installs its Python dependencies (`bleak`, `pillow`, `fonttools`, `prompt_toolkit`) on first run.
- Optionally `fribidi` (`brew install fribidi`). When present, skin tone, flag and ZWJ emoji sequences print as a single glyph.
- On first run macOS asks for Bluetooth permission for your terminal app.

## Installation

```sh
curl -fsSL https://raw.githubusercontent.com/cihanyakar/catprint/main/catprint -o ~/.local/bin/catprint
chmod +x ~/.local/bin/catprint
```

Any folder on your `PATH` works instead of `~/.local/bin`.

## Usage

```sh
catprint "Hello world"              # text
printf -- '- milk\n- bread\n' | catprint   # stdin
catprint photo.jpg                  # image (scaled to 384 px and dithered)
catprint -c -b -s 40 "Title"        # centered, bold, large
catprint -p preview.png "test"      # save a preview instead of printing
catprint                            # open the editor
```

### Editor

Running `catprint` in a terminal without any text opens a full-screen editor. It is handy for pasting multi-line art.

| Key | Action |
|---|---|
| Ctrl+Enter | Print and exit |
| Ctrl+S | Print and exit (for terminals that cannot report Ctrl+Enter) |
| Esc | Clear the buffer |
| Esc (when empty) | Exit |
| Ctrl+C | Exit without printing |

Ctrl+Enter is detected through xterm `modifyOtherKeys` or CSI u key reporting. The screen is cleared on exit.

### Options

| Option | Description |
|---|---|
| `-s N` | Font size (default 26) |
| `-b` | Bold |
| `-c` | Center |
| `-e N` | Print darkness (default 12000) |
| `-f N` | Paper feed lines at the end (default 112) |
| `-r`, `--no-rotate` | Rotate the image, or never auto-rotate wide images |
| `--no-dither` | Threshold images instead of dithering |
| `--no-braille` | Render Braille art with a font instead of dot mode |
| `-p [PNG]` | Save a preview instead of printing |

The printer name defaults to `GB03`. Set the `CATPRINT_NAME` environment variable to use another one.

## Text rendering

- Text is set in Arial.
- Symbols missing from Arial (such as ✓ ★ →) fall back to Arial Unicode and Apple Symbols.
- Emoji come from Apple Color Emoji and are printed as dithered grayscale.
- Combining marks stay with their base character, so kaomoji render correctly.
- Text made only of Braille characters (U+2800 to U+28FF) is detected automatically. Each dot is drawn as a pixel block scaled to the paper width.

## Protocol

A short summary for anyone who wants to modify the script.

- GATT service `AE30`. Write characteristic `AE01` (write without response), notify characteristic `AE02`.
- Packet format is `51 78 cmd 00 len 00 data crc8 FF`. CRC8 uses polynomial `0x07` over the data bytes only.
- Sequence is `A3` (status), `A4 33` (quality), `A6` (lattice start), `AF` (energy), `BE 01`, `A9 00`, one `A2` per row (48 bytes, LSB first), `A1` (feed), `A6` (lattice end), `A3`.
- The printer does flow control on `AE02`. Byte 6 of a notification is `0x10` to pause and `0x00` to resume.

## Credits

Developed with [Claude Code](https://claude.com/claude-code) using Claude Opus 5.5.

## License

MIT
