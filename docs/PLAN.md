# Project plan

Written on 2026-10-22. The requirements (FR/NFR codes) and the architecture are in the [README](../README.md).

## Scope

- **Goal:** build, from scratch and writing the code myself, a program that follows a Minecraft session in real time and reacts with short, mocking voice lines, with a simple avatar on screen. The focus is learning each layer of the system, not only getting it to work.
- **MVP:** Minecraft Java on PC, events read from the game log, an LLM via API generating the line, TTS playing the audio and a PNG with an animated mouth in OBS. All in Python.
- **Out of the MVP:** screenshot vision, local models, Live2D, other games, Twitch chat integration and long-term memory. Each becomes an optional phase once the core is stable.
- **Estimated duration:** 3 to 5 months at a semester pace, with phases of 1 to 3 weeks. The deadline is flexible; a phase is finished when its done criterion is met.
- **Assumptions:** Python 3.11 or newer, Minecraft Java Edition, development on Windows with tests on Linux as well.

## Open technical decisions

Each decision will be researched and recorded in `docs/decisions.md` with the reason for the choice (Architecture Decision Records).

| Decision | Options to compare | When |
|---|---|---|
| Concurrency model | threads, `asyncio`, separate processes | Phase 2 |
| Event source | read `latest.log`, custom Fabric mod, Mineflayer bot | Phase 1 (start with the log) |
| LLM | paid API, API with a free tier, local model via Ollama | Phase 3 |
| TTS | Edge TTS, Piper, Kokoro, CosyVoice | Phase 4 |
| Avatar | Pygame window, local web page as an OBS source, VTube Studio | Phase 5 |
| Config format | TOML, YAML, `.env` | Phase 2 |

## Phases

Eight phases. The next one starts only when the current one meets its done criterion. Each phase has an optional extra challenge.

### Phase 0 — Foundation (1 week)

- **Deliverable:** repository with a package structure, virtual environment, `.gitignore` and an initial README.
- **Study:** Git with branches and small commits, `venv` and `pip`, Python package layout (`src/`, `__main__.py`), environment variables for secrets.
- **Done when:** `python -m rival` runs on a clean clone by following only the README.
- **Challenge:** set up `ruff` and `pre-commit` to format and lint the code on every commit.

### Phase 1 — Eyes: event reader (2–3 weeks)

- **Deliverable:** a module that follows `latest.log` in real time and turns lines into event objects.
- **Study:** continuous file reading (the `tail -f` pattern), regular expressions, `dataclasses` and `Enum`, `pytest`. Finding out which messages the game writes to the log, and which it does not, is part of the work.
- **Done when:** dying in three different ways produces three correct events in the terminal, with tests based on sample log lines.
- **Challenge:** handle the game restarting and the log file being recreated.

### Phase 2 — Director: deciding when to speak (2 weeks)

- **Deliverable:** a component that receives events from a queue and decides whether to produce a line, still without calling any AI.
- **Study:** producer/consumer pattern, `queue` with `threading` vs `asyncio`, state machines, priority and cooldown, reading a config file.
- **Done when:** a simulator that injects fake events produces exactly the expected decisions, checked by tests.
- **Challenge:** escalating irritation: the same mistake repeated raises the intensity of the reaction, which cools down over time.

### Phase 3 — Brain: generating the line (2–3 weeks)

- **Deliverable:** a client that sends the persona and context to an LLM and returns a short sentence.
- **Study:** HTTP and REST APIs, JSON, `httpx` or `requests`, timeouts and retries, tokens and cost, system prompts with examples, abstract classes to swap providers.
- **Done when:** a real game event becomes a sentence consistent with the persona, with cost measured per hour of play.
- **Challenge:** make the AI reference earlier events from the same session without overflowing the context.

### Phase 4 — Voice: TTS and audio (2 weeks)

- **Deliverable:** the sentence becomes audio and plays without blocking event detection.
- **Study:** digital audio (sample rate, PCM, WAV vs MP3), asynchronous playback, a speech queue so lines do not overlap. The signals part connects directly to the Electrical Engineering course.
- **Done when:** the game can be played normally while lines play in sequence, without cuts or overlap.
- **Challenge:** streaming: start playing before the whole audio has been generated.

### Phase 5 — Body: avatar (1–2 weeks)

- **Deliverable:** a character image that opens its mouth according to the audio volume, captured by OBS.
- **Study:** envelope and RMS of an audio signal, a simple graphics window or local web page, inter-process communication (WebSocket or local socket).
- **Done when:** the avatar shows up in OBS in sync with the voice.
- **Challenge:** the LLM returns an emotion along with the line and the avatar changes expression.

### Phase 6 — Measure and harden (1–2 weeks)

- **Deliverable:** per-stage latency measurements and a long session without failures.
- **Study:** the `logging` module, timestamps at each pipeline stage, basic profiling, network error handling.
- **Done when:** the README shows the average latency per stage and the program runs for 1 hour straight without freezing.
- **Challenge:** a per-stage latency plot over the session, generated with Matplotlib.

### Phase 7 — Optional expansions

Any of these, in any order:

- **Vision:** capture the screen, downscale it and send it to a vision model to comment on the scene, comparing cost and latency with the event-based approach.
- **Local model:** run the LLM on my own machine and measure what changes in latency, quality and GPU usage.
- **Fabric mod:** write a Java mod that sends richer events (damage, biome, held item) over a socket, replacing the log.
- **Twitch chat:** the rival also reacts to chat during a stream.

## Rules for using AI

AI is a tutor and reviewer, never the author of the code. Every line in this repository is typed and understood by me.

- **Allowed:** explaining a concept, giving hints about an error, reviewing code I already wrote, suggesting exercises and test cases, discussing the pros and cons of a decision.
- **Not allowed:** asking for the implementation of a function or module, pasting generated code into the project.
- **Stuck on an error:** try alone for 30 minutes, then read the official documentation, and only then ask for a hint, not the solution.
- **Docs first:** for every new library, read the official tutorial before asking.
- **Log:** record in `docs/journal.md` what blocked me and how I solved it.

## Risks

| Risk | Mitigation |
|---|---|
| Abandoning mid-semester | small phases with visible deliverables; pause during exams and resume from the current phase |
| Scope growing before the MVP | nothing from Phase 7 before Phase 6 is done |
| Boring, repetitive lines | invest in the director and the prompt; test with friends and note what works |
| API cost getting out of control | spending cap set at the provider and a call counter in the program |
| API key leaked on GitHub | `.env` in `.gitignore` from Phase 0 |
| The game log not containing a desired event | accept the limitation in the MVP; the Fabric mod in Phase 7 solves it |
| Third-party characters and voices | use an original character and a TTS voice, without cloning real people |
