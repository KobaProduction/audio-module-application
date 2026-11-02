# AGENTS.md

Read README.md completely before working in this repository.
For Application stack and transport direction, read docs/FOUNDATION.md before planning implementation.
For cross-repository ownership and Hardware/HAL integration, use the authoritative
Hardware docs/REPOSITORIES.md and docs/HAL_CONTRACT.md (linked in README).
For current Application architecture decisions and blockers, inspect GitHub Issue #1.

Project: Koba Audio Module — Application.
Owner: device-side daemon, backend, frontend, audio pipelines, local hardware
integration and event/action handling.
Do not embed Linux firmware, NAND/bootloader tools or centralized assistant services.
Keep runtime audio/UI/control logic here; physical board profiles and Builder
remain in audio-module-hardware. A HAL JSON declaration is not a live API.

Use relevant task skills from https://github.com/ArthurKoba/ai-agent-workflow.
The initial Qt 6/C++, Vue 3/TypeScript/Vite/Pinia, WebSocket + Protobuf and SQLite
direction is selected in docs/FOUNDATION.md. Daemon topology, concrete RPC/IPC
protocol, audio engine and further module design remain open in Issue #1.
Do not implement or silently lock undecided details ahead of user input.
