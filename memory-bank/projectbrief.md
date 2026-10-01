# Project Brief

## Project

**Hermes-StackChan** turns an M5Stack CoreS3 / StackChan robot into the physical voice-and-body interface for HermesAgent.

Repository root: `c:\Users\Lester\Documents\Codex\stackchan\Hermes-StackChan`

## Core Objective

Provide a local-first conversational robot in which:

- the ESP32-S3 firmware owns hardware I/O, display, touch, camera, audio transport, LEDs, servos, and autonomous motion;
- `ai-server` bridges the device WebSocket/Opus protocol to HermesAgent and exposes robot controls;
- HermesAgent owns STT, LLM, TTS, memory, skills, providers, and MCP decisions;
- deliberate physical actions are available as MCP tools, while natural blinking, idle motion, speaking motion, and reminder notification remain firmware responsibilities.

## Main Deliverables

1. **Firmware** for M5Stack CoreS3 / StackChan.
2. **TypeScript bridge** between the device and HermesAgent.
3. **stdio MCP server** exposing safe robot actions to HermesAgent.
4. **Setup surfaces**, including the firmware Setup app and an optional Flutter provisioning app.
5. Supporting remote-control and legacy product-server code.

## Product Boundaries

- Do not move STT, LLM, TTS, memory, or skill execution onto the M5Stack.
- Do not make large HermesAgent changes for normal StackChan integration work; `hermes-agent/` is a submodule/external project.
- Avoid major protocol replacement when the existing WebSocket, Hermes Dashboard, local HTTP, and MCP paths suffice.
- Keep secrets out of logs, status payloads, examples, and tool results.

## Success Criteria

- A configured device boots reliably to Launcher by default.
- The user can explicitly open HERMES and transition cleanly to the avatar runtime.
- Voice turns travel between firmware, `ai-server`, and HermesAgent with usable latency and audio quality.
- Hermes can perform intentional robot actions through documented MCP tools.
- Device disconnection and helper failures produce clear errors rather than crashing the conversation.
- Firmware changes build under the pinned ESP-IDF workflow, and `ai-server` build/tests pass.
- Display and shared-SPI behavior are verified on real hardware, not inferred only from logs.

## Source-of-Truth Documents

- `AGENTS.md` — repository-wide engineering and hardware safety rules.
- `README.md` and `README.ja.md` — current user-facing behavior and setup.
- `docs/hermes-refactor/` — historical design rationale, incident analysis, and verification guidance; check current code before treating old checklists as incomplete work.
