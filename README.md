# Koba Audio Module ? Application

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

- Firmware/Linux/boards/backup/flashing live in KobaProduction/audio-module-hardware.
- Shared conversations, assistants and their orchestration live in external services.

## Status

Initial repository only. Language/framework, audio pipeline engine, API contracts,
licensing and detailed architecture remain open for design.

Workflow: https://github.com/ArthurKoba/ai-agent-workflow
