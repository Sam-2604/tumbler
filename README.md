# TUMBLER

*A story-driven lockpicking game. Ten locks, one drowned city, and the name they tried to erase.*

You are Kess, a thief in the sinking city of Harrowmere. Every chapter is a single lock: poor-boxes, warded cabinets, safe-doors, and finally the great two-keyed door of the Vault of Names, deep in the drowned city your family died protecting. People talk to you while you work. Some of them are lying.

## Play

The game runs in any modern browser, no install. Play it at: **[samarthgoradia.com/personal/tools/tumbler](https://samarthgoradia.com/personal/tools/tumbler/)**

## What's in it

- **Ten chapters**, each one lock, each lock a story beat
- **Three mechanics** that combine and escalate:
  - **Pin locks** - hold to lift a pin, release inside the band
  - **Warded rings** - spinning rings; tap to freeze each gap on the bolt-line
  - **Dials** - sweep a needle, release inside the arc; direction reverses each ring
- Late-game modifiers: decoy pins, locks picked blind by feel, tide-surged wards, a rival fighting you through the same mechanism
- A complete story with dialogue during play, illustrated chapter pages, and a plain-words retelling ("The Story") that unlocks when you finish
- Generative music: every chapter has its own motif, voice, and pulse, synthesized live in WebAudio - no audio files
- Saves your progress and settings locally; works on phones and desktops

## Controls

Everything works by touch or mouse: hold/release the big button (or the lock itself), tap pins to select them. Keyboard: **Space** hold/release, **← →** select pin, **Enter** advance text, **M** mute, **Esc** settings.

## Run it locally

```bash
git clone https://github.com/Sam-2604/tumbler.git
cd tumbler
python3 -m http.server 8000
# open http://localhost:8000/tumbler.html
```

Note: the illustrated backdrops (`assets/`) are not in the repository. Without them the game still runs, falling back to canvas-painted scenes. To get the full visuals, place 9:16 portrait images in `assets/` named `lv01.jpg` ... `lv10.jpg`, `title.jpg`, `epilogue.jpg`.

## Tech

One self-contained HTML file. No framework, no build step, no dependencies beyond Google Fonts. Canvas rendering, WebAudio synthesis, CSS container queries for the aspect-locked stage. See [TECHNICAL.md](TECHNICAL.md) for the architecture and the reasoning behind it.

---
<em>Built by Samarth Goradia</em>