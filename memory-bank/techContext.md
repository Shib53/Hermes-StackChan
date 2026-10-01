# Technical Context

## Repository Components

| Path | Technology | Role |
|---|---|---|
| `firmware/` | C/C++, ESP-IDF, CMake, LVGL, Mooncake, Xiaozhi-derived runtime | CoreS3 device firmware |
| `ai-server/` | Node.js, TypeScript | WebSocket/audio/Hermes bridge and MCP/control services |
| `hermes-agent/` | Git submodule, Python-based external project | Agent backend |
| `app/` | Flutter/Dart | Optional provisioning/setup client |
| `remote/` | Embedded firmware | ESP-NOW remote |
| `server/` | Go | Existing product backend; not required for the local Hermes voice loop |

## ai-server

- TypeScript target: ES2022.
- Module output: CommonJS with `.js` import paths in source.
- Strict TypeScript enabled.
- Runtime dependencies: `dotenv`, `opusscript`, `ws`.
- Tooling: TypeScript 5.7, `tsx`, Node type definitions.
- Tests: Node.js built-in test runner through `tsx --test test/*.test.ts`; do not introduce Jest assumptions.

Commands:

```powershell
Set-Location 'c:\Users\Lester\Documents\Codex\stackchan\Hermes-StackChan\ai-server'
npm install
npm run build
npm test
npm run dev
npm start
npm run probe:voice -- --preflight
```

Key files:

- `src/index.ts` — process startup and optional Hermes warmup.
- `src/server.ts` — device WebSocket and media HTTP server.
- `src/session.ts` — conversational state, VAD/STT/Hermes/TTS pipeline, follow-ups, LEDs.
- `src/hermes.ts` — Hermes Dashboard/session client.
- `src/hermes_audio.ts` — STT/TTS helper integration.
- `src/device_control.ts` — local HTTP control endpoint and public-to-firmware tool mapping.
- `src/stackchan_mcp_server.ts` — stdio MCP JSON-RPC server.
- `src/media.ts` — media resolution and serving.
- `test/*.test.ts` — unit/integration-style tests with mocks.

## Firmware

- Target: ESP32-S3 / M5Stack CoreS3 StackChan.
- Required toolchain family: ESP-IDF v5.5.4.
- Firmware version observed in root CMake: `1.4.1`.
- Build system applies guarded patches to managed components and fails if a patch is neither applicable nor already applied.
- `sdkconfig.defaults.local` is an optional git-ignored local overlay automatically included when present.

Commands:

```powershell
Set-Location 'c:\Users\Lester\Documents\Codex\stackchan\Hermes-StackChan\firmware'
python .\fetch_repos.py
idf.py build
idf.py flash
idf.py monitor
```

Important firmware areas:

- `main/main.cpp` — startup and Mooncake/Hermes transition.
- `main/apps/app_launcher/` — Launcher and optional compile-time HERMES auto-open.
- `main/apps/app_ai_agent/` — manual HERMES readiness checks/start request.
- `main/apps/app_setup/` — explicit configuration flows, including SD import.
- `main/hal/hal.cpp`, `hal.h` — board/runtime lifecycle.
- `main/hal/board/stackchan_display.*` — avatar and display behavior.
- `main/hal/hal_mcp.cpp` — firmware-side MCP tools.
- `main/stackchan/` — avatar, modifiers, motion, servo, and related robot behavior.

## Configuration and Services

- Device WebSocket default: `ws://<server>:8765/ws`.
- Media server shares port `8765` under `/media/...`.
- Control HTTP default: `http://127.0.0.1:8766`.
- Hermes Dashboard commonly uses `http://127.0.0.1:9119` and `/api/ws`.
- Hermes MCP configuration launches `ai-server/dist/stackchan_mcp_server.js` as a stdio process.
- Numerous `STACKCHAN_*` environment variables tune VAD, recording limits, cooldown, barge-in, reply length, segmentation, preroll, gain, local audio output, auto LEDs, warmup, and local-only mode. Use `.env.example` and README as the configuration references.

## Platform Notes

- Current agent environment: Windows 32-bit platform identifier (`win32`) with PowerShell and VS Code.
- ESP-IDF may require its activated environment and connected hardware; document inability to build/flash rather than claiming success.
- The desktop UI simulator is documented as primarily maintained for macOS, so do not assume it is a Windows validation substitute.

## Security and Privacy

- Never commit Wi-Fi passwords, tokens, credentials, or personal absolute paths.
- Do not log full WebSocket URLs if they may carry sensitive data; expose only safe scheme/host-level status.
- Keep robot control bound to loopback unless a deliberate, secured design change is requested.
