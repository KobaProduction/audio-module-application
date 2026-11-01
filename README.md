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
