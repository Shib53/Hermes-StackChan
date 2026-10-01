# System Patterns

## High-Level Architecture

```text
StackChan firmware
  │ WebSocket: JSON + Opus audio
  ▼
ai-server :8765
  ├─ device session / turn state machine
  ├─ STT and TTS helpers
  ├─ Hermes Dashboard WebSocket client
  ├─ media HTTP endpoint
  └─ local control HTTP :8766
          ▲
          │ tools/call
stdio StackChan MCP server
          ▲
          │ MCP JSON-RPC
HermesAgent
```

## Firmware Patterns

### Ownership Split

- Mooncake owns Launcher and setup/application UI.
- Hermes/Xiaozhi runtime owns the active conversational avatar after handoff.
- `firmware/main/main.cpp` is the ownership transition point.

### Handoff Sequence

1. HERMES app calls `GetHAL().requestHermesStart()` only when readiness checks pass.
2. Main loop exits.
3. Under `LvglLockGuard`, uninstall all Mooncake apps, destroy Mooncake, and clean boot logo/home indicator/status bar/handoff display.
4. `GetHAL().prepareHermesDisplay()` constructs the avatar surface.
5. `GetHAL().startHermes()` starts the non-returning runtime.

### Display Safety

- Every LVGL object operation must occur under the established lock API.
- Confirm object ownership before clean/delete operations.
- `StackChanAvatarDisplay::SetupUI()` must remain idempotent or safely skip duplicates.
- Do not call `esp_lcd_panel_draw_bitmap()` from application code while LVGL owns the panel.
- Preserve the validated LCD reset order: `esp_lcd_panel_reset`, AW9523 ILI9342 reset, then `esp_lcd_panel_init`.

### LCD/SD Shared-SPI Rule

- LCD and SD share SPI3; GPIO35 is LCD DC and SD MISO.
- Keep SD_CS GPIO4 and LCD_CS GPIO3 inactive-high before SPI3 initialization.
- Never probe/import SD during normal display startup or Hermes handoff.
- SD configuration import is explicit Setup UI only and must restart on success or failure.
- Use NVS for normal startup/runtime configuration.

### Firmware MCP

- `firmware/main/hal/hal_mcp.cpp` registers firmware-side tools.
- Use `cJSON` or equivalent safe construction for JSON results; never concatenate unescaped user/reminder text into JSON.
- Device status must redact full URLs and credentials.

### Microphone/TTS Isolation

- Treat speaking as a hard microphone transport boundary, not only a UI state.
- `AudioService::SetInputSuppressed(true)` clears pending encode/send/timestamp queues and prevents both newly processed PCM and already in-flight encoder work from reaching the network send queue.
- The application send loop rechecks device state while draining packets to close asynchronous speaking-transition races.
- Release suppression on every transition to listening, outside processor-start conditionals, because realtime mode can keep the audio processor running while the device speaks.
- Keep server cooldown and recent-TTS transcript rejection enabled as defense in depth; they do not replace firmware queue isolation.

## ai-server Patterns

### Process Startup

- `src/index.ts` starts loopback control HTTP first, optionally warms Hermes, then opens the device WebSocket/media server.
- Defaults: device/media port `8765`; control port `8766`; control host `127.0.0.1`.

### Session State Machine

- `Session` coordinates `idle`, `listening`, and `processing` states plus TTS streaming and queued follow-ups.
- Each device connection gets a session and registers itself as the current control target.
- Device loss cleans up the session rather than terminating the server.
- Environment readers clamp numeric values and use safe defaults.

### Audio Path

- Incoming device Opus is decoded for VAD/STT.
- Hermes STT/TTS helpers or configured local endpoints perform speech conversion.
- TTS is segmented, encoded to device Opus framing, and streamed back.
- Decoder failures are isolated/recreated; one bad input frame must not corrupt the TTS encoder path.

### Tool Bridge

- `src/stackchan_mcp_server.ts` implements line-delimited stdio JSON-RPC and publishes tool schemas.
- It calls `src/device_control.ts` over local HTTP.
- `device_control.ts` maps public names such as `stackchan_set_head_angles` to firmware names such as `self.robot.set_head_angles`.
- Firmware may return stringified JSON; preserve `normalizeFirmwareResult` and MCP-content normalization compatibility.
- If no device is connected, return a clear service/tool error without crashing Hermes.

### Media

- Put file-path, URL, hosting, and image-source resolution logic in `src/media.ts`.
- Avoid duplicating path conversion in the MCP server.

## Documentation and Testing Coupling

When a public tool, startup behavior, or user-visible configuration changes:

- update implementation mappings and schemas;
- add/update Node built-in runner tests;
- update both `README.md` and `README.ja.md` with equivalent meaning;
- compile firmware if touched;
- perform real-device checks for hardware behavior.
