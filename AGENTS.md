# AGENTS.md

Read README.md completely before working in this repository.
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
Architecture and technologies are unapproved. Do not preemptively select
frameworks, interfaces or directory layouts.
