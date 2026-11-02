# Audio Module Application — product capabilities and user journeys

Status: **product discovery / requirements capture**, 2026-11-02.
This is the product-level meaning of the device, not an implementation
specification, DSP algorithm, API contract, database design or approved MVP
backlog. It is based on the owner's stated use cases. Distinguish **required
product direction**, **first-release candidates**, and **optional/future
capabilities** below; the actual MVP scope and acceptance will be agreed
before any engineering breakdown. Existing platform choices are owned by
[FOUNDATION.md](FOUNDATION.md), and cross-repo ownership by Hardware's
[REPOSITORIES.md](https://github.com/KobaProduction/audio-module-hardware/blob/main/docs/REPOSITORIES.md).

## 1. Product purpose and boundaries

A modular, permanently available, configurable smart-audio device and
networked speaker platform. It observes local inputs and sound, responds to
wake/stop/command phrases, plays music and system/assistant audio to available
destinations, and provides one coherent control surface for physical buttons,
voice, web administration, authorized external systems and assistants.
Capabilities depend on actual device hardware and selected services; the
product must degrade gracefully when microphones, outputs, accelerators,
radios or network access are missing.

**Not the same thing:** a device-local Application, the underlying Hardware
HAL/firmware, and any external intelligent/home-automation services. Application
coordinates device-side state and scenarios. Hardware owns drivers, physical
board definitions and NAND/update protection. External systems provide optional
STT, TTS, language models, music catalogs and smart-home orchestration.

## 2. Product functional domains

These are **logical responsibilities**, not processes, classes, build targets
or a mandate to create a daemon for every item.

### A. Continuous listening and acoustic front end

- Capture from available microphones while the device is operating; report
  whether capture is available, active, muted, denied or failed. Support
  differing microphone/channel configurations rather than assuming an array.
- Continuously inspect incoming speech for **wake phrases**, **stop/cancel
  phrases** and optionally simple **local commands** (e.g. volume). Distinguish
  wake detection, speech presence, transcription, intent and cancellation:
  they are different product outcomes.
- Provide a processed voice stream/session to downstream recognition when
  authorized/needed. Desired quality goals: suppress stationary/background
  noise, mitigate reverberation, compensate own playback and support
  appropriate normalization/resampling/format transformation.
- Support lightweight idle listening and more elaborate active speech
  processing. A voice session can start at wake, continue during an utterance,
  end/cancel or time out. Exact buffering and phrase engine are **not chosen**.
- Prefer validated available on-chip/DSP/accelerator capabilities where
  present, with **software providers** when feasible. A provider must expose
  its actual supported operations; software fallback is not a promise of
  equivalent accuracy or performance on all CPUs.
- Acoustic preprocessing, feature extraction (including possible real-input
  FFT), resampling and wake/voice classifiers are **later technical design**
  topics. No algorithm/library is committed here.

### B. Barge-in and playback-aware acoustic compensation

- Speech must remain usable while the device is playing music or system
  sounds (**barge-in**). The microphone can contain the near-end speaker,
  acoustic reflections and independently generated room sound.
- When the platform controls a playback stream, its reference signal may be
  used for acoustic echo cancellation/suppression, with route-specific timing
  and an estimate of what really reaches the room. Real-room behavior is not
  solved by a fixed phase inversion: acoustic paths have delays, reflections,
  nonlinearities and may change over time.
- Playback may reach the room via the internal speaker, **AUX**, **S/PDIF**,
  external Bluetooth device or future outputs. Model each destination's
  capabilities separately, including whether its playback reference is
  accessible and whether its acoustic output can affect local microphones.
  A digital route existing does **not** prove audible acoustic feedback.
- Some external routes introduce independent buffering/DSP/latency and may
  lack usable reference/timing. The user-visible behavior must tolerate
  inaccurate/absent reference and lower voice recognition quality.
- Possible route/environment calibration or audibility probing is an
  **optional research direction**, not a periodic automatic beep. Acoustic
  measurements must be controlled, unobtrusive and explicitly authorized
  where they emit sound; passive evaluation is preferable when sufficient.
- The detailed AEC, delay alignment, multi-reference combination, beamforming
  and algorithm selection require a later measured design and comparison of
  existing solutions.

### C. Audio playback, outputs and routing

- Play **music**, **voice-assistant TTS**, and **system/notification/reminder
  sounds**. Distinguish these as logical playback roles for routing, priority,
  interruption and volume behavior.
- Product output categories: **internal speaker**, **AUX**, **S/PDIF** and
  **external Bluetooth audio device**. Additional adapters are extensible.
  Presence, direction (input/output/bidirectional), connection, supported
  formats, physical audibility and volume control vary by device.
- User/admin can choose an available destination, start/stop playback and
  control master and per-route volume/mute wherever supported. Exact
  simultaneous routing, source mixing, priority, ducking, crossfade and
  independent DSP/equalizer policies are yet to be defined.
- Music processing, adjustable frequency response/EQ, advanced mixing and
  arbitrary routing graph are **future/advanced** items, not assumed MVP.
- Inbound AUX/Bluetooth sources (if actually available) and outbound
  destinations must not be conflated merely because their connector names
  match. Physical capabilities require verification through Hardware.

### D. One coherent device action/control model

- Represent device actions independently of where a command originated.
  **Physical touch/buttons/encoders**, recognized voice, web SPA, authorized
  external API clients, Home Assistant and AI-driven intent should converge
  on the same device semantics and state. Avoid parallel implementations of
  volume, output selection or mute for each channel.
- Initial actions include volume up/down/set, mute/unmute, playback
  play/pause/stop, selecting a capable output and invoking a configured
  interaction. More actions can be registered as the product grows.
- The system must report current state, success/failure and capability
  limitations; interactive clients should observe changes made elsewhere.
- Access and safety policy depends on actor and action. A recognized
  utterance, external LLM output or network connection is not automatically
  permission to run arbitrary hardware operations.

### E. Voice assistant, recognition and intelligence

- **Local fast path:** wake/stop and simple configured commands can be
  recognized/handled locally when the device/provider supports them.
- **Optional external path:** submit an authorized voice segment/stream to
  speech-to-text (STT), turn its text into a command/intent using a selected
  LLM/assistant or rule provider, then validate and execute the resulting
  device or home-automation action. A text response can also be synthesized
  by a TTS provider and played back on a chosen output.
- Treat **STT**, **intent/LLM**, **action execution** and **TTS** as distinct
  product capabilities, each with independently selectable/configurable
  local or remote providers where the platform permits.
- Initial MVP assumes external intelligence/recognition/synthesis as needed;
  **on-device LLM is not an MVP requirement**. Local models/providers remain
  a future optional direction on sufficiently capable devices.
- Provider settings may include service endpoint, credentials, model/voice
  choice, prompts/policies, capabilities and connection health. Exact
  invocation, streaming format and timeouts are deferred.
- Support an extensible external-action surface, potentially MCP, to expose
  reviewed device actions to AI agents. MCP is a **candidate integration**,
  not an implemented transport or a permission bypass.
- User voice, transcript, prompt and provider traffic can be sensitive:
  explicitly govern consent, outgoing audio, retention and credential access.

### F. Media discovery, catalogs and playback services

- Provide a common user experience to search, browse and play music,
  podcasts and other supported media from configurable providers.
- Potential connectors cited by the owner: **Spotify, Apple Music, Yandex
  Music, VK Music, local media storage and YouTube**. These are intended
  integration candidates, **not** claims of available APIs, licensing or
  supported playback today.
- Users should be able to request/select content by voice, app or approved
  external control and route playback to a supported output.
- Service authentication, subscriptions, DRM, redistribution rights, content
  availability and region limitations need provider-specific review **before**
  any provider is promised or integrated. No bypass of protected streams.

### G. First-use onboarding, connectivity and network management

- A new or factory-reset device needs a discoverable **setup mode**, to
  select an available network connection and establish safe administration.
- Product connectivity includes **Wi-Fi** and **Ethernet** where hardware
  exists; optional onboarding channels are Wi-Fi AP/captive portal and/or
  Bluetooth provisioning if actual radio/OS support permits.
- Candidate first-run journey: discover device; pair/authorize setup;
  choose Wi-Fi credentials if needed; attempt connection; show status;
  retry or return to setup after failure; complete admin enrollment.
  Ethernet can skip Wi-Fi credential entry when it supplies connectivity.
- Provide network settings/status and recovery from bad credentials or
  connectivity loss, without assuming a captive portal or AP mode works
  on every device.
- Network and provider credentials are **secrets**, not public profile
  values. A processor serial/identifier alone is typically not a secret
  encryption key. Secure local storage, credential recovery and ownership
  transfer need their own security/storage decision; do not prescribe a
  reversible CPU-ID-derived key.
- Existing Hardware identity/UBI preservation problems are **not solved**
  by keeping Application configuration in SQLite; factory recovery and
  off-device backup remain Hardware-owned gates.

### H. Administration, configuration and permissions

- A web SPA is the primary configurable management surface for onboarding,
  connectivity, capabilities, inputs/outputs/routes, voice triggers, provider
  selection, music integrations, keys/endpoints, assistant prompts/policies,
  notifications and observed system health.
- Settings should be understandable even when a capability is absent, an
  external integration is unauthorized or a subsystem is temporarily down.
- Authentication, roles/permissions, authorized external API credentials,
  pairing, secret revocation and device ownership changes are product
  requirements; no unauthenticated remote administration or LLM-driven raw
  firmware access.
- All actor types should use a consistent control domain; the frontend is
  an API consumer, not a separate privileged control implementation.

### I. Home Assistant and external device ecosystems

- **Home Assistant integration is required** at product level: discover or
  configure an HA connection, expose appropriate column/device status and
  actions, and allow authorized user/assistant intents to act on smart-home
  entities through the approved integration.
- Distinguish commanding **the speaker itself** from commanding
  **third-party home devices**. A remote model should not gain universal
  credentials or arbitrary actions just by returning a text prompt.
- Other controllers/automation systems may integrate through an approved,
  versioned API/adapter later; the exact external protocol and discovery
  model are deferred.

### J. Telemetry and observable behavior

- Show useful device status in the SPA: load, memory/resource pressure,
  microphone/capture availability, output/route status, stream activity,
  subsystem health, provider connectivity, voice-session outcomes and
  relevant processing latency where measurable.
- **OpenTelemetry** is the intended common observability vocabulary for
  traces/spans, metrics and correlated events/logs where useful. Treat
  Prometheus-compatible metric exposure and external Grafana dashboards
  as integration goals/candidates; final exporter shape is not yet chosen.
- Avoid exposing captured speech, transcript content, API secrets or raw
  prompts as default telemetry. Device collection and remote export must
  be controllable, capacity-aware and privacy-preserving.

### K. Light and audio-reactive visualization

- **Color music / audio-reactive lighting** is a desired first-class product
  domain. It consumes an authorized playback/analysis signal to drive
  effects where the actual device exposes suitable lighting outputs.
- Light modes/settings may include brightness, modes, responsiveness,
  scene selection and relation to playback. All physical LED capabilities
  remain subject to Hardware discovery.
- A specific Capsula Mini **ring LED is currently excluded** from its
  hardware capability profile; the product's generalized lighting domain
  does not override that or authorize the historical private binary.
- Advanced spectral effects, synchronization algorithms and hardware
  drivers are future technical design, not already delivered.

## 3. End-to-end user journeys to preserve

| Journey | Product behavior / expected handoff |
| --- | --- |
| First power-on / unconfigured | Device offers an actually supported onboarding path; authorized owner chooses connectivity, provisions credentials and reaches working administration, with retry/recovery |
| Idle listening / quiet room | Detect configured wake phrase; optionally respond to local command; create a voice interaction if needed; do not require internet for every basic action |
| Listening while playing music | Use available playback-reference/echo-mitigation capability where feasible, detect near-end speaker, allow interruption or ducking according to later policy; handle lack of reference gracefully |
| Local simple request | Same change (e.g. lower volume) regardless of whether invoked by voice, touch or SPA; publish updated state to all authorized observers |
| External assistant request | Authorized audio -> STT -> intent/LLM -> permitted action or reply -> optional TTS -> selected playback destination; report failures and offer a safe fallback |
| Switch playback destination | Only list actual available endpoints; select desired route; preserve predictable state and report disconnected/unsupported outputs |
| Choose and play media | Search or receive a choice from an authorized provider; resolve rights/capability; play and control through common routing |
| External integration / HA | Pair/authorize connector; expose only permitted actions/states; apply HA/device actions with consistent ownership and feedback |
| Admin and monitoring | Configure providers/routes/network, inspect device metrics, connectivity and subsystem states, without seeing private audio or secrets by default |
| Network failure / provider outage | Preserve local feasible controls; show useful error and retry/reprovisioning path; do not assume a working cloud or local LLM |
| Audio-reactive lighting | If a supported lighting endpoint exists, reflect selected playback/analysis signal without disrupting audio or implying nonexistent LEDs |

## 4. Product vocabulary (concepts, NOT database tables)

- **Device / profile / capability:** device instance, builder-declared
  hardware features and runtime-verified available features are distinct.
- **Input / capture session / processed stream:** a microphone or external
  source, an observation interval, and a downstream-ready signal are not the
  same thing.
- **Output endpoint / route / playback role / reference:** physical/logical
  output, selected path, music/TTS/system role and the audible echo reference
  are separate; each may be missing or unverified.
- **Wake/stop phrase / speech segment / intent / action:** detected marker,
  possible speech transcription, desired operation and authorized execution
  must not be conflated.
- **Provider / connector / account / secret:** selected STT, LLM, TTS, media
  or HA integration; authentication and permissions are separate from provider
  business data.
- **Device state / setting / event / telemetry:** live observation, persisted
  preference, significant change and operational measurement are separate.
- **User/owner/actor/permission:** physical touch, browser, an external
  program or LLM may initiate the same action but must not inherit the same
  authorization automatically.

These terms form an initial shared product language, **not** a commitment
to a specific number of services, processes, tables or API methods.

## 5. Release-scope hypotheses — not final MVP commitments

**Strong first-slice candidates:** initial setup on a Windows development
host; one microphone/capture path; selectable wake/stop trigger and local
volume command; one working output and playback of a system/assistant sample;
one coherent control state reflected in SPA and physical simulation; a
configured external STT/assistant/TTS round trip; basic SQLite configuration
and health display. The exact inclusion of network enrollment and a real
speaker deployment remains to be agreed.

**Foundational but separately measured:** near-end speech while the product
plays music, AEC with available reference, per-route delay variability,
software/hardware provider switching and real A113X peripheral acceptance.
These are core *product quality goals*; do not label them solved or invent
an acoustic quality threshold. Some may be milestones after the first slice
depending on hardware measurements.

**Requested broader platform scope:** internal speaker/AUX/S/PDIF/external
Bluetooth routes, authorized external API and MCP possibility, Home Assistant
integration (**required target**), configurable STT/LLM/TTS and music
providers, first-use provisioning, OTel telemetry, audio-reactive lighting.
Sequence and per-device availability are open.

**Advanced/future:** multiple independent output/mix graphs, per-route
equalization/DSP tuning, local LLMs, advanced lighting scenes, multi-device
synchronization, further media services and multiple-daemon isolation.

Do not turn these hypotheses into a GitHub implementation backlog or claim
an MVP has been approved. Requirements, release priorities and measurable
acceptance should be negotiated with the owner after the product map is
reviewed.

## 6. Critical product decisions still open

1. Which speech actions must work with no internet: wake, stop, volume,
   playback control, local commands and help?
2. What behavior is expected when music is playing: guaranteed barge-in
   for which hardware/routes, which quality and user-visible fallback?
3. Which audio source/output directions actually exist for each platform,
   especially AUX, S/PDIF and external Bluetooth? Which references can be
   observed or controlled?
4. Should a voice utterance be forwarded continuously, only after wake,
   only after local recognition fails, or according to owner-selected modes?
   What are consent/retention requirements?
5. What are the minimum safe setup/admin/ownership-transfer and recovery
   experiences, with AP/Bluetooth/Ethernet optional by capability?
6. Which control actions may outside systems and AI propose, execute or
   require confirmation for? What are actor permissions?
7. What is the initial Home Assistant scope: speaker entity/status,
   home-device intents, bi-directional events or all of them?
8. Which music/media providers are actual first-release priorities and
   have usable documented APIs, licenses and account flows?
9. What does first-release telemetry need to show locally, and which
   exports require opt-in to remote systems?
10. Which lighting hardware is actually present on the first supported
    device? Color music remains a product feature, not proof of ring LEDs.
11. What is the **agreed MVP** versus phased milestones, and what
    observable acceptance criteria establish success?

## 7. Handoff and implementation boundary

This document owns **product meaning and workflows**. The previously
approved [FOUNDATION.md](FOUNDATION.md) owns initial Qt/Vue/SQLite/WebSocket
technology direction. Hardware owns device and driver capability evidence.
[Application Issue #1](https://github.com/KobaProduction/audio-module-application/issues/1)
owns further discussion/approval, decisions and progress.

**No implementation or external library choice has been approved by this
product capture.** Review existing smart-speaker, DSP, AEC, voice assistant,
music and home-integration solutions **after** the product scope is accepted,
then adapt implementations to the approved modular contracts rather than
copying an unrelated platform or promising unsupported capabilities.
