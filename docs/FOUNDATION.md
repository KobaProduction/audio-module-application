# Application foundation — agreed direction, not yet implemented

This document records the **initial platform and interface choices** for
Koba Audio Module Application (2026-11-02). It is an architecture baseline,
not a completed daemon/API specification, technology benchmark, or permission
to implement audio features before their contracts are discussed.

## Product and platform boundary

- Application stays in `KobaProduction/audio-module-application`; Hardware's
  Kernel/U-Boot/Firmware/Builder/Tools, device profiles, recovery and NAND
  safety stay in `KobaProduction/audio-module-hardware`.
- A **modular device-side service application** with C++ and **Qt 6** as the
  chosen initial implementation platform. A service/daemon may contain several
  independent functional modules; whether each module must be a separate
  OS process is **not decided**. Do not create microservices prematurely.
- The goal is to compile and run the same product code on **Windows for
  desktop development** and on **Linux/AArch64 for speakers**, with further
  Qt-supported platforms considered via target-specific build/package gates.
  Cross-platform support is a design requirement, **not** a claim that all
  ports or peripherals already build and work.
- The speaker is deployed **natively**, with no Docker Engine dependency.
  Docker/container tooling may be added later for optional host development
  and CI, but must not become a prerequisite for a device runtime.
- Keep domain operations behind typed service boundaries; Qt/network/storage
  are adapters. This preserves an eventual move of selected modules to other
  typed languages (e.g. Rust) without making such a rewrite an active task.

## Interaction map

```text
Browser / Vue 3 SPA (TypeScript, Vite, Pinia)
    | HTTP(S): load compiled frontend assets, not a REST API
    | WebSocket (binary): RPC calls, responses, events, subscriptions
    v
Qt 6 C++ application / network gateway
    | protocol decoding, routing, auth/access control (contract TBD)
    | internal typed interfaces
    +-- independent functional modules (module list TBD)
    +-- Qt SQL/QSQLITE -> local SQLite database
    +-- approved Hardware HAL/native Linux providers (runtime pending)

Windows development: same application and SPA, with simulated or limited
hardware adapters where physical drivers do not exist.
Linux speaker: native service using available platform adapters.
```

The chart shows logical responsibilities, **not** mandatory distinct daemon
processes or a promise of a working transport. Use the simplest service
process topology that can satisfy later reliability and isolation needs.

## Backend and web delivery

- **C++ / Qt 6** is the initial backend platform; CMake is the anticipated
  cross-platform build system, to be set up during implementation.
- Prefer Qt HTTP Server (`QHttpServer`) for delivery of the compiled SPA
  and required web assets. The HTTP listener's purpose is distribution of
  client assets and necessary HTTP/WebSocket negotiation, **not** a full
  REST/CRUD application API. Exact port, HTTPS and WebSocket upgrade setup
  depend on the selected Qt baseline and will be validated in a real build.
- Use Qt WebSockets (`QWebSocketServer` / `QWebSocket`) for an interactive,
  persistent browser-to-backend channel, with server-originated notifications.
- The frontend and backend are independently buildable but delivered as one
  coherent product: Vite builds static assets, which the Qt application
  serves or includes at packaging time. Hot-reload/dev proxy may be used
  on Windows without affecting the speaker's native release architecture.

## Web frontend

- **Vue 3, TypeScript, Vite and Pinia** are the selected SPA foundation.
  Own frontend code under a dedicated `frontend/` directory when bootstrap
  begins; do not inject the frontend into the Hardware repository.
- **Shadow DOM** is part of the desired UI/component approach. Prefer
  deliberate use for encapsulated Web Components where isolation is useful;
  its exact scope (selected components vs entire root application) is still
  open. Vue supports `defineCustomElement` and `.ce.vue` custom elements;
  this does not require forcing all SPA components into Shadow DOM.
- Pinia owns shared client state; backend operations and push events use a
  dedicated typed WebSocket/RPC client layer, not per-component protocol
  handling. Local/transient component state need not be globally stored.

## Binary WebSocket RPC — design boundaries

- **RPC over WebSocket with Protocol Buffers** is the proposed default for
  frontend/backend application communication. Protobuf defines typed,
  evolvable **message schemas**; WebSocket carries binary messages.
- This is **not gRPC over HTTP/2** and must not be described as ready-made
  gRPC transport. Qt GRPC's current documented module is client-focused;
  it is not the backend WebSocket RPC server implementation.
- Required interaction classes: *request -> response*, server-pushed
  *events/subscriptions*, and errors. Support structured protobuf messages
  and optional opaque byte payloads for unstructured/binary data. Avoid
  forcing large PCM/media streams through a generic control RPC until
  the separate audio-transport decision is made.
- Protocol schemas (future `.proto` files) should be owned centrally by
  the Application, and code generation must support C++ and TypeScript.
  Evaluate existing code generators/libraries before adding a custom RPC
  framework. Do not implement an ad-hoc packet format yet.
- Still to decide and version explicitly: message envelope, method IDs/names,
  correlation IDs, framing for opaque/chunked data, size and backpressure,
  cancellation/timeouts, reconnect/re-subscribe, version negotiation,
  authentication/authorization, trusted origin and TLS, and code generator.
  A binary protocol **does not** provide authorization or encryption on
  its own. No unauthenticated network writes or raw NAND access.

## Persistence

- **SQLite** is the chosen initial embedded local database: no separate DB
  server on the speaker. Qt SQL (`QSqlDatabase`, `QSqlQuery` with QSQLITE)
  is a supported access path on Windows and Linux, subject to the platform
  having/deploying its SQLite driver plugin.
- Define one Application-owned database schema and schema version. Include
  a minimal **first migration** when implementation starts (for example
  `0001_initial.sql`); use an explicit user schema version such as SQLite's
  `PRAGMA user_version` or a migration ledger, and apply changes within a
  defined transaction. No speculative table inventory before domains exist.
- A full ORM is **not selected**. Start with Qt SQL and a small repository/
  migration adapter, and compare an ORM only if concrete models/queries
  justify added weight and dependencies. Keep domain interfaces separate
  from database queries. Multiple daemon processes must not assume
  unrestricted concurrent SQLite writers.
- Database placement, encryption, WAL policy, cross-process ownership,
  backup/restore and persistent storage paths are undecided. On the
  current Capsula Mini Hardware, whole-UBI replacement does **not** retain
  user data; do not treat SQLite as persistent across a firmware update
  until Hardware's storage/recovery contract is verified.

## Compatibility and non-goals for this phase

- Windows must exercise the same core services, RPC and database schemas as
  the device build, with platform adapters only where needed. It is **not**
  proof that ALSA, GPIO, SPDIF or other Linux peripherals work on Windows.
- The first real vertical slice should establish **real** Qt server startup,
  SPA asset serving, one WebSocket RPC roundtrip, event push and creation /
  opening of a versioned SQLite DB on Windows; then compile/run the same
  slice on Linux/AArch64 without Docker. This is a suggested acceptance
  checkpoint, **not authorized implementation work yet**.
- No daemon split, precise audio graph/pipeline, HAL runtime adapter, API
  method catalog, table catalog, ORM, WebRTC, gRPC cross-server bridge,
  additional frontend framework or raw-binary media transport is selected.
  The user will define further modules and behavior in subsequent design.
- Hardware HAL v1 at `/usr/share/audio-module/capabilities-v1.json` is
  **build-time only** (`runtime_verified: false`). Do not invent a working
  HAL endpoint. See the authoritative Hardware HAL contract and
  [Application Issue #1](https://github.com/KobaProduction/audio-module-application/issues/1).

## Existing implementations and references to evaluate before coding

| Need | Existing solution to evaluate | Status/limitation |
| --- | --- | --- |
| Qt asset serving | Qt HTTP Server (`QHttpServer`) | Existing Qt 6 module; verify serving/static routing and security in target Qt version |
| Native WebSocket | Qt WebSockets (`QWebSocketServer`) | Existing Qt 6 module; RPC semantics must be layered above WebSocket |
| Binary typed messages | Qt Protobuf with generated TypeScript Protobuf client | Existing Qt message generator; choose browser generator and interoperability during proof-of-concept |
| Alternative service RPC | Qt GRPC | Qt's documented support is client-side; ordinary gRPC/HTTP2 is not WebSocket RPC |
| Embedded DB | Qt SQL/QSQLITE and SQLite `PRAGMA user_version` | Built-in driver family with single-writer constraints; no ORM required initially |
| Isolated UI elements | Vue `defineCustomElement`, `.ce.vue` | Shadow DOM suitable where component encapsulation is necessary |

Primary external documentation:
[Qt HTTP Server](https://doc.qt.io/qt-6/qhttpserver.html),
[Qt WebSockets](https://doc.qt.io/qt-6/qwebsocketserver.html),
[Qt Protobuf](https://doc.qt.io/qt-6/qtprotobuf-index.html),
[Qt GRPC](https://doc.qt.io/qt-6/qtgrpc-index.html),
[Qt SQLite](https://doc.qt.io/qt-6/sql-driver.html),
[Vue Web Components](https://vuejs.org/guide/extras/web-components.html),
[SQLite schema version](https://www.sqlite.org/pragma.html#pragma_user_version).

**Authority:** this file owns the first-phase stack and interface *direction*;
Application Issue #1 owns unresolved design choices. It does not replace
Hardware's cross-repository ownership and HAL contract. No source/runtime
was created by recording these decisions.
