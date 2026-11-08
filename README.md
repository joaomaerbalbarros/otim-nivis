# Otim Nivis — an AI rival that watches you play

![Otim Nivis cover art](assets/cover.webp)

> **Status: planning.** No code yet. This repository holds the project plan and the character design; implementation starts at Phase 0 of the [plan](docs/PLAN.md).

Otim Nivis is a program that follows a **Minecraft** session in real time and reacts with short, mocking voice lines, with a simple animated avatar on screen. She is designed as the AI rival of my VTuber, [Mito Ignis](https://twitch.tv/mito_ignis), to watch and tease her during streams. It is being built from scratch, with the goal of understanding every layer of the system rather than just making it work.

## MVP

Minecraft Java Edition on PC, game events read from the game log, an LLM API generating the line, text-to-speech playing the audio, and a PNG avatar with an animated mouth captured by OBS. All in Python.

Out of the MVP, as optional later phases: vision from screenshots, local models, Live2D, other games, Twitch chat integration and long-term memory.

## Architecture

Five modules in a line, each built in its own phase and connected to the next by a queue or a simple call. The director is the core: it decides *whether* and *when* to speak.

```text
Minecraft (latest.log)
   │  new log lines
   ▼
Event reader     (Phase 1)   tails the log and turns each line into an Event
   │  Event: type, time, data
   ▼
Director         (Phase 2)   queue, priority and cooldown decide if a line is worth saying
   │  speech request with context
   ▼
Brain            (Phase 3)   persona + recent events go to the LLM, which returns the line
   │  line text
   ▼
Voice            (Phase 4)   TTS generates the audio; a speech queue prevents overlap
   │  audio and volume
   ▼
Avatar           (Phase 5)   the mouth follows the audio volume; OBS captures it
```

Configuration, logging and tests cut across every module. Keeping each module behind a small interface is what makes it possible to swap the LLM or the TTS later without rewriting the rest.

## Requirements

| Code | Requirement | Phase |
|---|---|---|
| FR01 | Detect game events: death, damage, system message, advancement, joining the world | 1 |
| FR02 | Represent each event as an object with type, timestamp and data | 1 |
| FR03 | Decide whether and when to speak, with cooldown, priority and a repetition counter | 2 |
| FR04 | Generate a short line with a defined persona and the context of recent events | 3 |
| FR05 | Convert the line to audio and play it without blocking the rest of the program | 4 |
| FR06 | Show an avatar that opens its mouth while the audio plays | 5 |
| FR07 | Read configuration (keys, voice, persona, cooldowns) from a file, without editing code | 2 |
| FR08 | Log everything that was detected, decided and spoken | 1–5 |
| NFR01 | Latency from event to start of audio under 3 s; final target under 1.5 s | 6 |
| NFR02 | Run for 1 hour of gameplay without freezing or leaking memory | 6 |
| NFR03 | No API keys in the repository | 0 |
| NFR04 | API cost below a defined ceiling per hour of gameplay | 3 |
| NFR05 | Swappable modules: change TTS or LLM without touching the rest | 3–4 |
| NFR06 | Automated tests for the parser and the director | 1–2 |

Commits and the README will reference these codes to track what is done.

## Roadmap

| Phase | Deliverable | Status |
|---|---|---|
| 0 — Foundation | package structure, virtual environment, `.gitignore`, `python -m rival` running from a clean clone | planned |
| 1 — Eyes | real-time log reader that turns lines into event objects, with tests | planned |
| 2 — Director | decides when to speak from an event queue, verified by an event simulator | planned |
| 3 — Brain | LLM client with persona and context, cost measured per hour | planned |
| 4 — Voice | non-blocking TTS playback with a speech queue | planned |
| 5 — Body | avatar whose mouth follows the audio's RMS envelope, captured by OBS | planned |
| 6 — Measure and harden | per-stage latency table and a 1-hour run without failures | planned |
| 7 — Expansions | vision, local LLM, Fabric mod, Twitch chat | optional |

Open technical decisions (concurrency model, event source, LLM, TTS, avatar, config format) and the full phase breakdown are in [docs/PLAN.md](docs/PLAN.md).

## Character

The AI has its own original character: **Otim Nivis**, the digital rival of **[Mito Ignis](https://twitch.tv/mito_ignis)**, my original VTuber, whom I created and stream as on Twitch.

Otim is "Mito" spelled backwards: she is Mito's mirror image. She was born as a program that watched Mito play and, from watching so much, learned to talk like her, react like her and eventually cast magic. That origin is the same premise as the technical project: an AI that observes the game, learns and comments. She is smug, sarcastic and quick to comment on every mistake, but only *she* gets to mock Mito.

|  | Mito Ignis | Otim Nivis |
|---|---|---|
| Nature | half demon, half dragon | artificial intelligence |
| Element | fire (*ignis*) | ice (*nivis*, "of snow"), in the sense of a program freezing |
| Aesthetic | medieval and arcane, with modern clothes | tech, modern, hacker |
| Corruption | draconic, grows red crystals | data, spawns glitch pixels |
| Palette | red, wine, gold | cyan, blue-violet, purple, amber accents |

The designs mirror each other: Otim's cyan crystal horn sits on the opposite side of Mito's golden horn, her eye colors are swapped, and her glitch pixels are the digital version of Mito's red crystals.

<table>
  <tr>
    <td><img src="assets/mito-ignis-sheet.webp" alt="Mito Ignis character sheet"></td>
    <td><img src="assets/character-sheet.webp" alt="Otim Nivis character sheet"></td>
  </tr>
  <tr>
    <td align="center"><b>Mito Ignis</b></td>
    <td align="center"><b>Otim Nivis</b></td>
  </tr>
</table>

Otim's design is still being refined.

## Stack (planned)

Python 3.11+ · pytest · an LLM API (provider to be decided) · a TTS engine (to be decided) · OBS · Minecraft Java Edition. Developed on Windows and tested on Linux.
