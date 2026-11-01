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

## Status

Initial repository only. Language/framework, audio pipeline engine, API contracts,
licensing and detailed architecture remain open for design.

Workflow: https://github.com/ArthurKoba/ai-agent-workflow

## Development status

Only project scope and repository-level safety checks are initialized.
No application code, board support sources or runtime toolchain have been adopted.
The current pre-push and CI verification checks documentation completeness, whitespace,
and accidental secrets; it **rejects source files** until the relevant build, lint and
test toolchain is approved and added to the same gate.

On a fresh checkout, enable the tracked Git hook:

    git config core.hooksPath .githooks

Run the initial verification from the repository root:

    sh scripts/verify.sh
