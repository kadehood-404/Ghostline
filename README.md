# GHOSTLINE

A terminal hacking game in a single HTML file. Break into a procedurally generated network, steal data, and jack out before the trace lands.

No backend, no build step, no dependencies, no API keys. Open `ghostline.html` in a browser and play.

## The pitch

You are a netrunner working out of a shed. Every run drops you into a fresh network of hosts guarded by gate codes. Crack the gates, grab the valuable files, and `jackout` to bank them. Get greedy and the trace hits 100%, you lose your haul, and the run is over.

Somewhere at the bottom of the network is the BLACKSITE, and a grimoire worth more than everything else combined.

## Quick start

1. Open `ghostline.html`.
2. Type `help`.
3. Type `start` for a random network, or `start daily` for today's shared network.
4. `scan`, `connect <host>`, crack the gate, `ls`, `download <file>`, `jackout`.

On a phone, use the button row under the terminal. Buttons marked with a trailing space (connect, cat, download) fill the input so you can finish typing the argument.

## Commands

Between runs:

| Command | What it does |
|---|---|
| `start [daily]` | Begin a run. `daily` uses the same network for everyone today |
| `shop` | Show the black market and your gear |
| `buy <item>` | `proxy`, `cracker`, or `wipe` |
| `stats` | Credits, best haul, runs, grimoires stolen |
| `clear` | Clear the screen |
| `reset confirm` | Erase your save |

During a run:

| Command | What it does |
|---|---|
| `scan` | List hosts linked from your current host |
| `connect <host>` | Connect by name or IP. Locked hosts start the gate minigame |
| `ls` | List files on the current host |
| `cat <file>` | Read a text file |
| `download <file>` (or `dl`) | Steal a `.dat` file. Loud |
| `map` | Hosts you have discovered |
| `status` | Trace, location, path, and haul |
| `back` | Step back one host |
| `wipe` | Scrub logs: -30% trace. Uses one charge |
| `jackout` (or `quit`) | Bank your haul and end the run |

Tab autocompletes commands, host names, and file names. Up/Down arrows cycle command history.

## How it works

### The network

Each run generates four tiers of hosts: 3, then 3, then 2, then the BLACKSITE. Hosts only link forward, and every tier-3 host links to the BLACKSITE. You can only connect to hosts linked from where you are standing, and only after you have scanned for them.

Each host has an ICE rating from 1 to 5. Higher ICE means longer codes, tougher gates, and a faster-moving trace.

### Gates

Locked hosts are cracked with a Mastermind-style puzzle:

- The code uses the symbols **A to F**.
- ICE 1-2 hosts use 3-symbol codes. ICE 3-5 hosts use 4-symbol codes.
- After each guess you get `` for a right symbol in the wrong spot.
- Run out of attempts and the gate **locks out**: it rotates to a new code, resets your attempts, and costs a big trace spike.
- `abort` leaves the gate without penalty.

Digits 1 to 6 also work as symbols (1=A, 2=B ... 6=F), handy on phone keyboards. Malformed guesses (wrong length, bad symbols) are free, and the error tells you which rule you broke.

### Intel

Some hosts hold an `ops_notes.txt` that leaks symbols from a neighboring host's gate code. Read it with `cat` and the game remembers. The notes on the two hosts guarding the BLACKSITE always leak two symbols of its gate, so exploring on the way down pays off.

### Trace

Trace is the clock. It rises with every action you take inside a host, and slowly with real time while you sit in one (it does not tick while you are on your own rig).

| Action | Base trace |
|---|---|
| `scan` | 2 |
| `ls`, `cat` | 1 each |
| Guess | 2 per guess |
| Entering a host | 4 |
| Lockout | 15 |
| `download` | 4 + file value / 120 |
| Stealing the grimoire | 35 |

All costs are scaled by the ICE of the host you are in, and reduced by your proxy level. At 100% you are caught. You lose everything in your haul, but banked credits and gear stay.

### Credits and gear

Credits only count once you `jackout` alive. They persist between runs and buy:

| Item | Effect | Price |
|---|---|---|
| Proxy (max 3) | -20% trace gain per level | 150 / 300 / 500 |
| Cracker (max 2) | Pre-reveals 1 / 2 gate symbols (never the whole code) | 250 / 500 |
| Wipe (carry up to 5) | `wipe` for -30% trace, consumable | 100 each |

### Loot

| Tier | Typical file value |
|---|---|
| 1 | 60 to 120 |
| 2 | 150 to 300 |
| 3 | 400 to 700 |
| BLACKSITE grimoire | 5000 |

### Difficulty

Don't expect to steal the grimoire on your first run. The alarm trips a huge trace spike, and without a proxy you will not survive it. Plan on a few runs of smaller scores to buy gear first. The shape of the game is greed versus safety: every extra file is more credits and more trace.

### Daily seed

`start daily` seeds the network from your local date, so everyone playing on the same day gets the same hosts, codes, and loot. Compare hauls. The end-of-run summary prints a one-line result you can paste anywhere.

## Hosting

It's one static file. Put it anywhere that serves HTML:

- A folder on your own web server or a subdomain like `hack.yoursite.com`
- GitHub Pages, Cloudflare Pages, Netlify, or similar free static hosts
- Open it straight from disk

There is nothing to configure server-side.

## Tweaking the game

All balance numbers live in the `CFG` object at the top of the script block in `ghostline.html`:

- `tickMs`, `tickBase`, `tickPerSec`: how fast trace creeps with time
- `cost`: base trace cost of each action
- `wipeAmt`, `maxWipes`, `proxyReduce`: wipe and proxy strength
- `shop`: prices

Host names, loot names, and flavor text files are in the `TIERS`, `LOOT`, and `LORE` arrays just below it. Adding to those is the easiest way to give the game more personality.

## Save data

Credits, gear, and stats are stored in your browser's `localStorage` under the key `ghostline`. Clearing site data or typing `reset confirm` wipes your progress. Progress is per browser and per device.

## Testing

The engine can be run headless for testing: set `window.__GHOST_TEST = true` before the script loads and it exposes `window.__ghost` with the world generator, a command executor, and the live run state. Worth using if you change the generator, since the main invariants are that every host is reachable from the rig and the BLACKSITE is always reachable.