# Koba Audio Module — Application

Device-side runtime and management application for Koba Audio Module devices.

## Responsibilities

- Always-running local device daemon and control backend.
- Web-based control and configuration UI.
- Audio routing, volume, pipelines and output selection (AUX/SPDIF).
- Hardware capability discovery and local event/action handling.
- Integration points for external voice assistants and smart-home services.

The application is hardware-independent at the platform level. Kernel drivers
and Device Tree describe device capabilities without requiring a different
backend for every minor hardware variant.

## Boundaries

- Firmware/Linux/boards/backup/flashing and the internal image Builder live in
  [KobaProduction/audio-module-hardware](https://github.com/KobaProduction/audio-module-hardware).
- Shared conversations, assistants and their orchestration live in external services.

## Start here: repository ownership and HAL contract

This repository already exists; it is the **second and only other current
Audio Module product repository**. Hardware keeps all logical Kernel,
U-Boot, Firmware, Builder and Tools components internally. No separate
Builder or Firmware GitHub repository is planned. Before designing an
Application runtime, follow the [Hardware-owned two-repository map and
handoff](https://github.com/KobaProduction/audio-module-hardware/blob/main/docs/REPOSITORIES.md).
OpenIPC and OpenWrt are architecture/development inspirations only; we are
not replacing Alpine with OpenWrt.

The current [Hardware HAL v1 contract](https://github.com/KobaProduction/audio-module-hardware/blob/main/docs/HAL_CONTRACT.md)
uses a **build-time** descriptor at /usr/share/audio-module/capabilities-v1.json.
Its origin is builder-declared and runtime_verified is false. It is **not** an
implemented HAL daemon, IPC, device probe or guarantee of operating hardware.
The Application will own runtime discovery, audio/control logic and UI; native
Linux HAL provider work is tracked in [Hardware #4](https://github.com/KobaProduction/audio-module-hardware/issues/4).
Do not bind code to assumed ALSA/input indices, invent the HAL transport,
activate SPDIF from an unverified profile or perform raw NAND writes.

An Application release may eventually be selected by Hardware's **existing
Builder** after agreeing on package, version and compatibility contracts.
No such bundling exists now. Proposed daemon/audio/API framework decisions
belong to [Application Issue #1](https://github.com/KobaProduction/audio-module-application/issues/1)
not to an implicit choice by a new agent.

## Product capability map

[docs/PRODUCT_SCOPE.md](docs/PRODUCT_SCOPE.md) captures the owner-described
smart-speaker domains and user journeys: continuous voice capture and wake/
stop detection, playback-aware noise/echo mitigation, modular audio outputs,
shared physical/voice/web/external control, optional STT/LLM/TTS providers,
media discovery, onboarding/networking, Home Assistant, telemetry and
color music. **This is product discovery, not an approved MVP feature list,
DSP algorithm, process topology or implemented service.** Decisions and
priorities remain in Application Issue #1.

## Initial platform direction

The first agreed platform/transport/storage direction is recorded in
[docs/FOUNDATION.md](docs/FOUNDATION.md): C++ / Qt 6 backend and native
service, Vue 3 / TypeScript / Vite / Pinia frontend with considered Shadow DOM
components, binary WebSocket RPC / Protobuf messages, and local SQLite with
versioned schema. Windows is the desktop development target, Linux/AArch64 the
first device target; the speaker does not require Docker.

**Status:** specification only, not an implemented Qt server, SPA, RPC or DB.
Exact daemon topology, full RPC protocol, audio engine, domain modules and
runtime hardware integration remain open in Application Issue #1.

Workflow: https://github.com/ArthurKoba/ai-agent-workflow

## Development status

Project scope, initial stack/interface direction and repository safety checks
are documented. No application code, board support sources or compiled
runtime toolchain have been adopted yet.
The current pre-push and CI verification checks documentation completeness, whitespace,
and accidental secrets; it **rejects source files** until the relevant build, lint and
test toolchain is approved and added to the same gate.

On a fresh checkout, enable the tracked Git hook:

    git config core.hooksPath .githooks

Run the initial verification from the repository root:

    sh scripts/verify.sh
