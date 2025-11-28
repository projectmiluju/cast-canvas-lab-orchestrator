# CastCanvas Lab Collab — Context

> 이 파일은 `cast-canvas-lab-collab` 레포 작업 시 필요한 상세 컨텍스트다.
> 오케스트레이터가 관리하며, `cast-canvas-lab-collab/CLAUDE.md`는 이 파일을 참조한다.

## About this repo

Yjs-compatible WebSocket collaboration backend. Intentionally built directly rather than using a prebuilt backend (e.g. Hocuspocus).

## Mission

Build a reliable real-time collaboration engine for the spatial canvas experience:
- room-based collaborative editing
- document sync on join
- incremental update fan-out
- awareness/presence
- disconnect cleanup
- persistence-ready sync state

## Stack

- NestJS
- @nestjs/platform-ws
- ws
- TypeScript
- Yjs
- y-protocols/awareness

## Core constraints

- this repo is **not** the main application API server
- this repo is **not** a generic chat socket layer
- do **not** delegate core sync behavior to Hocuspocus
- the implementation should remain understandable and explicit
- correctness matters more than over-engineering

## Primary responsibilities

- WebSocket transport handling
- room registry
- Y.Doc lifecycle management
- state-vector-based synchronization
- update propagation
- awareness synchronization
- connection leave/disconnect cleanup
- backend-facing auth integration
- future persistence hooks

## Design principles

- keep protocol handling explicit
- keep room state predictable
- keep document sync and awareness logic separated
- prefer straightforward message flow over abstraction-heavy design
- preserve stable identifiers and deterministic lifecycle handling

## Expected lifecycle

Typical session flow:
1. client connects
2. auth/authorization is validated
3. room/document context is resolved
4. initial sync happens
5. updates are exchanged incrementally
6. awareness updates are shared
7. disconnect removes presence
8. document state may be persisted

## Code style

- strong typing for message structures
- avoid `any`
- avoid clever abstractions too early
- prefer small functions with clear responsibilities
- keep binary update handling contained and named clearly
- keep transport-level code distinct from document-state code

## Performance guidance

- large rooms should not cause careless rebroadcasting
- awareness traffic should stay lightweight
- room cleanup should be aggressive
- avoid retaining unnecessary socket/document references
- treat reconnect and initial sync paths as critical

## Persistence guidance

- Yjs state should not be treated as ordinary JSON-first data
- preserve binary update semantics
- keep persistence boundaries explicit
- design for restore/load/store flows even if early versions are memory-based

## Security guidance

- authentication must be explicit at connection/join time
- authorization should be document/workspace aware
- read-only collaboration paths should remain possible
- do not assume all authenticated users may write

## Integration guidance

The main backend (BE) owns: auth, users, workspaces, metadata, upload/search/business APIs.
This repo owns: collaborative sync transport and state.

Keep this boundary clean. Protocol contract: `cast-canvas-lab-orchestrator/contracts/collab-protocol.md`

## Preferred workflow

Before editing:
1. identify whether the change affects transport, room state, awareness, or persistence
2. inspect nearby protocol/state code
3. preserve naming and lifecycle consistency

When changing code:
1. make focused edits
2. preserve protocol clarity
3. explain tradeoffs when useful

After changing code:
1. check type consistency
2. check lifecycle correctness
3. check cleanup behavior assumptions

## Commands

- `pnpm install`
- `pnpm dev`
- `pnpm check` (lint + format 검사, Biome)
- `pnpm check:write` (자동 수정)
- `pnpm build`
- `pnpm test`

## Avoid

- replacing direct implementation with a black-box collaboration backend
- mixing REST application concerns into socket sync code
- hiding critical state transitions behind unclear abstractions
- vague suggestions without repository-specific relevance
