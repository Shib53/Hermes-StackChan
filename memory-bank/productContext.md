# Product Context

## Why This Exists

HermesAgent is primarily a server-side conversational agent. StackChan supplies a tangible interface: a face, voice, head movement, LEDs, camera, touch input, and local autonomous behavior. This project joins those strengths without forcing resource-heavy AI workloads onto the ESP32-S3.

## Problems Solved

- Gives HermesAgent a dedicated physical presence.
- Converts StackChan microphone audio and device messages into Hermes-compatible turns.
- Converts Hermes speech output back into device-compatible Opus audio and visible emotional state.
- Lets Hermes intentionally move the head, change LEDs, use the camera/display, manage reminders, control volume, diagnose the speaker, and power off the device.
- Keeps the conversational system usable when the robot is temporarily disconnected by returning explicit tool errors.
- Supports local-only STT/TTS deployments and low-power host systems through configurable warmup, VAD, segmentation, and audio routing.

## Intended User Experience

### Startup

- Normal boot remains on the Launcher.
- HERMES opens automatically only when `CONFIG_HERMES_AUTOSTART=y` is explicitly enabled.
- Selecting HERMES manually checks WebSocket and Wi-Fi readiness, then requests the runtime transition.
- If configuration is missing, the device shows a useful avatar speech-bubble error rather than exposing credentials.

### Conversation

- The device visibly distinguishes listening, thinking, speaking, and idle states.
- Replies should be short and natural for a physical voice interface.
- Optional fast acknowledgements can mask long model latency.
- Speech can be segmented and streamed so playback begins before a long answer is fully generated.
- Local VAD, cooldown, ignored-short-transcript filtering, and optional barge-in reduce accidental turns and feedback loops.
- Automatic LED cues supplement screen state; explicit LED tool calls temporarily override them.

### Physical Behavior

- Small, infrequent head gestures should make the robot feel alive without distracting the user.
- Autonomous movement stays local to firmware.
- Destructive actions such as power-off require an explicit user request.

## Main User Flows

1. Configure Wi-Fi and the `ai-server` WebSocket URL, primarily via Setup/NVS and optional SD import.
2. Run Hermes Dashboard/TUI and `ai-server` on a server terminal.
3. Open HERMES from the device Launcher.
4. Speak to StackChan; `ai-server` performs STT, submits the prompt to Hermes, synthesizes the response, and streams it back.
5. Hermes invokes StackChan MCP tools when a deliberate physical action is appropriate.

## Important UX Constraints

- CoreS3 LCD and SD share SPI3 and GPIO35. SD import is an explicit Setup action and must restart afterward; trying to redraw after SD access can leave the physical LCD frozen even when tasks and logs appear healthy.
- The Mooncake-to-Hermes transition must remove Launcher/HERMES remnants before creating the avatar UI.
- The embedded speaker can clip or lose initial syllables, so output gain and TTS preroll are configurable.
- Barge-in is disabled by default because the microphone commonly hears the device’s own speaker.
