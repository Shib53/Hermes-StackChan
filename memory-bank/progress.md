# Progress

## Working Capabilities Observed

### Firmware

- Mooncake Launcher and applications are installed at startup.
- HERMES launcher auto-open is compile-time optional and disabled in the observed generated configuration.
- Manual HERMES open performs readiness checks and can request runtime startup.
- Mooncake teardown and LVGL cleanup occur before Hermes display preparation.
- Avatar display, status/chat bubbles, emotions, image preview, screenshots, camera, LEDs, servo motion, audio, reminders, and power control have implementation paths.
- Shared LCD/SD SPI hazards and safe import/restart behavior are extensively documented.
- Speaking-state microphone isolation now clears pending PCM/Opus/AEC timestamp queues, blocks new and in-flight microphone packets from entering the send queue, and guards the application send loop against state-transition races.

### ai-server

- Device WebSocket server and media HTTP serving exist.
- Loopback robot-control HTTP server exists.
- Hermes Dashboard/session integration exists.
- Incoming/outgoing Opus audio handling, STT/TTS helper integration, staged speech, local VAD, cooldown, optional barge-in, and local audio output exist.
- Automatic conversation-state LEDs and manual override hold behavior exist.
- Background sub-agent follow-up queuing exists.
- Public MCP tool schemas and firmware mappings include status, volume, test tone, head angles, LEDs, power-off, photo, image preview, screen capture, reminders, and sub-agent delegation.
- Unit tests cover audio, Hermes audio/dashboard behavior, VAD, media, session behavior, local audio output, and the MCP server.
- Post-TTS echo protection now has two layers: input/capture reset plus cooldown frame dropping, followed by configurable recent-TTS transcript matching through `STACKCHAN_RECENT_TTS_ECHO_WINDOW_MS`.
- Automatic LED initialization explicitly sets idle/off on WebSocket hello before any listening transition; explicit wake-word activation remains allowed during cooldown.

### Documentation

- English and Japanese READMEs describe architecture, tools, setup, voice tuning, local-only operation, and boot behavior.
- `docs/hermes-refactor/` records startup/display/tool/natural-dialogue design and LCD/SD incidents.
- Repository-wide `AGENTS.md` defines implementation and validation rules.

## Remaining / Ongoing Work

The repository does not present a single active unfinished feature at Memory Bank initialization. Ongoing engineering work should focus on:

- validating changes on physical CoreS3 hardware;
- preserving LCD/SD shared-SPI stability;
- tuning latency, VAD, cooldown, gain, preroll, and local audio routing for the deployment environment;
- keeping public MCP tools synchronized across firmware, bridge, tests, and bilingual docs;
- maintaining compatibility with HermesAgent Dashboard/API changes;
- reconciling historical unchecked refactor checklists with current code when those docs are edited.

## Known Risks

- Logs can indicate successful LVGL/runtime setup while the physical LCD remains black or frozen.
- SD access can break LCD signaling because of shared SPI/GPIO35, even after a failed/no-card probe.
- Incorrect LVGL lock ownership can cause races or deadlocks.
- The M5 microphone may capture its own speaker, making barge-in unreliable.
- Small-speaker clipping and lost initial phonemes require conservative gain and preroll.
- Firmware depends on managed-component patches; partial or incompatible patch application must fail visibly.
- A disconnected StackChan must remain a recoverable tool error, not a process-wide failure.

## Validation Status at Initialization

- Memory Bank documentation: initialized on September 30, 2026.
- No source code was changed as part of initialization.
- `ai-server` build/tests were not run because this task only documented the existing project state.
- Firmware build and hardware validation were not run.
- Pre-existing modified/untracked firmware files remain untouched and are listed in `activeContext.md`.

## Validation Status — October 1, 2026 Echo/LED Fix

- `ai-server`: `npm run build` passed.
- `ai-server`: full `npm test` passed with 61 tests, 0 failures.
- Scoped `git diff --check` passed for implementation, tests, environment example, and both READMEs.
- No firmware source was changed, so no firmware build was required for this bridge-only fix.
- Physical CoreS3 validation remains required for acoustic echo behavior and the top LED state during Hermes startup.

## Validation Status — October 1, 2026 Firmware Echo Isolation

- Regenerated tracked `firmware/patches/xiaozhi-esp32.patch` from the complete fetched dependency diff; the fetched dependency remains ignored and is not the tracked source of truth.
- The patch applies cleanly to a clean checkout of the pinned Xiaozhi dependency.
- Applied-patch content matches all 13 modified dependency files after LF/CRLF normalization.
- `git diff --check` passed inside both the fetched dependency and the clean patched checkout.
- `ai-server`: `npm run build` passed.
- `ai-server`: full `npm test` passed with 61 tests, 0 failures.
- Firmware compile was not run because `idf.py`, `IDF_PATH`, and the usual local ESP-IDF install paths were unavailable in the current shell.
- Physical build/flash validation remains required. Run 5–10 voice turns and confirm that TTS playback produces no retransmitted microphone audio or self-replies.

## Definition of Done for Future Changes

- Relevant implementation is complete with no placeholders.
- Existing conventions and nested instructions are followed.
- `ai-server`: `npm run build` and `npm test` pass when affected.
- Firmware: `idf.py build` passes when affected, or the exact environment limitation is reported.
- Hardware-dependent behavior is checked on-device where required.
- Public behavior is reflected consistently in `README.md` and `README.ja.md`.
- Memory Bank files are updated when architecture, active focus, major behavior, or known risks change.
