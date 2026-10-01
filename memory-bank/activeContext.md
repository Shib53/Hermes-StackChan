# Active Context

## Snapshot

Last updated: **October 1, 2026**.

Branch: `fix/voice-echo-loop-led-state`, tracking `fork/fix/voice-echo-loop-led-state`.

Most recent observed commit: `66f03b7` dated July 10, 2026 — `Optimize local voice conversation and firmware resilience`.

## Current Implementation State

- `firmware/main/main.cpp` installs Mooncake apps, waits for an explicit Hermes start request, tears Mooncake down under `LvglLockGuard`, cleans handoff UI, prepares the Hermes display, and starts the runtime.
- Launcher autostart remains compile-time guarded by `CONFIG_HERMES_AUTOSTART`; the observed generated config has it disabled.
- Opening the HERMES app manually validates the WebSocket scheme and Wi-Fi/config readiness independently of the autostart setting.
- The bridge exposes WebSocket/media service on the main port (default `8765`) and local robot-control HTTP on loopback (default `127.0.0.1:8766`).
- Reminder tools, power-off, volume/test-tone, movement, LEDs, camera, image preview, screen capture, and background sub-agent delegation are represented in the public MCP layer and README tool lists.
- Voice behavior includes configurable local VAD, cooldown, short-transcript filtering, optional barge-in, fast acknowledgement caching, staged TTS, local audio output, automatic LEDs, and optional Hermes warmup.
- The voice bridge now resets capture state at TTS completion, drops ordinary microphone frames during post-TTS cooldown, and rejects a recent STT transcript that matches StackChan's own last spoken reply. Explicit wake-word detection still overrides cooldown.
- A device WebSocket `hello` now explicitly sends the automatic idle/off LED state before listening starts, preventing a stale green listening indication during Hermes startup.
- Firmware microphone transport is now suppressed for the complete speaking state. Entering speaking atomically enables suppression and clears pending PCM encode, Opus send, and AEC timestamp queues; the encoder checks suppression before publishing in-flight microphone packets; the application send loop also drops queued input if speaking begins mid-loop. Returning to listening clears stale input and releases suppression for both auto-stop and realtime modes.

## Working Tree Warning

Pre-existing user changes were present before Memory Bank initialization:

- modified `firmware/patches/esp-lvgl-port-rgb565-swap-buffer.patch`;
- modified `firmware/patches/esp-ml307-tcp-shutdown.patch`;
- modified `firmware/patches/esp-wifi-connect-station-stability.patch`;
- modified `firmware/patches/xiaozhi-esp32.patch`;
- untracked `firmware/ready-to-flash/`.

Do not overwrite, revert, stage, or otherwise disturb these without explicit task requirements and inspection.

## Current Focus Guidance

The echo-loop and startup LED bridge/firmware fix is implemented and software-validated. For the next task:

1. Read all files in `memory-bank/`.
2. Read `AGENTS.md` and any more-specific nested `AGENTS.md` for the target area.
3. Inspect the current Git diff before editing, especially firmware patches.
4. Treat current implementation and tests as authoritative over unchecked boxes in older `docs/hermes-refactor/07-task-checklist.md`.
5. Build the firmware in an activated ESP-IDF v5.5.4 environment; ESP-IDF was not installed or discoverable in the October 1, 2026 agent shell.
6. Flash and validate on physical StackChan hardware that startup LED is off/idle and that speaker leakage no longer triggers self-replies over 5–10 repeated turns.

## Immediate Verification Baseline

For `ai-server` changes:

```powershell
Set-Location 'c:\Users\Lester\Documents\Codex\stackchan\Hermes-StackChan\ai-server'
npm install
npm run build
npm test
```

For firmware changes, use an ESP-IDF v5.5.4 environment:

```powershell
Set-Location 'c:\Users\Lester\Documents\Codex\stackchan\Hermes-StackChan\firmware'
python .\fetch_repos.py
idf.py build
```

Hardware-facing display, SD, audio, servo, and camera changes require real-device validation in addition to compilation.

## Decisions to Preserve

- Autostart-off must not disable manual HERMES startup.
- LVGL operations require the project’s display/LVGL locks; avoid double locking.
- Build Hermes avatar UI only after Mooncake teardown during runtime handoff.
- Never assume logs such as `SetupUI complete` prove the physical LCD is working.
- Keep MCP definitions, firmware mappings, tests, and both READMEs synchronized.
- Centralize media path/URL behavior in `ai-server/src/media.ts`.
- Keep control HTTP loopback-only by default.
