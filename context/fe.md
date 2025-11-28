# CastCanvas Lab FE — Context

> 이 파일은 `cast-canvas-lab-fe` 레포 작업 시 필요한 상세 컨텍스트다.
> 오케스트레이터가 관리하며, `cast-canvas-lab-fe/CLAUDE.md`는 이 파일을 참조한다.

## About this repo

워크스페이스 프론트엔드. 캔버스 UI, 문서/이미지 렌더링, 노드/엣지 인터랙션, 협업 클라이언트를 담당한다.

## Tech stack

- React 19
- TypeScript
- Vite
- pnpm
- Zustand
- TanStack Query
- React Flow (`@xyflow/react` v12)
- react-pdf

## Code quality tooling

- ESLint (strict, with react-hooks and react-refresh plugins)
- Prettier
- Husky (pre-commit, commit-msg, pre-push hooks)
- commitlint (conventional commits enforced)
- lint-staged

## Repo priorities

1. Canvas usability
2. Clear information architecture
3. Stable and predictable interactions
4. Maintainable feature boundaries
5. Performance-conscious rendering

## Important product constraints

- The workspace must remain canvas-first
- Documents should be readable, but not at the cost of turning the whole app into a linear editor
- Images and documents should coexist naturally on the canvas
- Connections between nodes are a first-class feature, not decoration
- Future collaboration support must remain possible

## Code style

- TypeScript strict mode; avoid `any` unless absolutely necessary, and explain why if used
- Functional React components with arrow functions
- Destructure props in component parameters
- Prefer named exports
- Prefer clear variable names over abbreviations
- Do not leave dead code or commented-out code behind
- Keep files cohesive; split when a file starts carrying multiple responsibilities
- Avoid overly clever abstractions
- Prefer readable code over dense code

## Project structure

- `src/app`: app bootstrapping and providers (`App.tsx`, global styles)
- `src/pages`: route-level pages (`CanvasPage`)
- `src/features/canvas`: canvas interaction and node/edge behavior
- `src/features/document`: document-related UI and logic
- `src/features/image`: reference image handling
- `src/features/search`: search UI and interaction
- `src/features/inspector`: side panels / details
- `src/entities`: domain models and reusable typed building blocks
- `src/shared`: shared utilities, constants, common UI
  - `src/shared/styles/`: design token partials (`_tokens.scss`, `_typography.scss`, `_mixins.scss`)
  - `src/shared/stores/`: shared Zustand stores (`themeStore.ts`)

## Feature knowledge sharing

Use `docs/` so newly implemented features are discoverable to other agents.

- `docs/features/`: feature purpose, key files, state flow, UI behavior, constraints, coordinated update points
- `docs/decisions/`: non-obvious decisions, rationale, tradeoffs, follow-up implications
- `docs/features/_template.md`: required starting template for feature docs
- `docs/decisions/_template.md`: required starting template for decision records

### When to write docs

| Event | Required action |
|---|---|
| 새 feature 디렉터리 생성 | `docs/features/<name>.md` 신규 작성 |
| 기존 feature의 파일 구조·상태·UI 동작 변경 | 해당 `docs/features/*.md` 업데이트 |
| 라이브러리 선택, 아키텍처 경계, 상태 관리 전략 등 비자명한 결정 | `docs/decisions/NNN-*.md` 신규 작성 |
| 프로젝트 구조·스택·커맨드 변경 | `CLAUDE.md` + `STATUS.md` 업데이트 |

**작성 시점은 코드 변경 직후, 같은 커밋 또는 PR 내에서 처리한다.** 문서를 나중으로 미루지 않는다.

> **MANDATORY:** 코드 변경을 완료한 응답 안에서 문서화를 완료한다. 사용자에게 묻거나, 다음 단계로 미루거나, 별도로 안내하지 않는다.

## Styling

- Use SCSS Modules (`.module.scss`) for all component styles
- Do not use CSS-in-JS libraries (runtime overhead hurts canvas performance)
- Do not use plain `.css` files for new component styles
- File naming: `ComponentName.module.scss` co-located with the component file

### Design token system

All visual values are defined as CSS custom properties in `src/shared/styles/_tokens.scss`.

**Always use semantic tokens — never use palette tokens directly:**

```scss
// ✅ correct
color: var(--color-text-primary);
background: var(--color-bg-surface);
padding: var(--space-4);
border-radius: var(--radius-md);

// ❌ wrong — palette tokens are internal definitions
color: var(--palette-neutral-900);
```

**Available token categories:**

- `--color-bg-*` — surface backgrounds (app, surface, elevated, sunken, overlay)
- `--color-border-*` — borders (default, subtle, strong, focus)
- `--color-text-*` — text (primary, secondary, tertiary, disabled, inverse, link)
- `--color-interactive-*` — interactive element states (primary, secondary + hover/active variants)
- `--color-canvas-*` — canvas-specific tokens (bg, grid, node, edge, handle)
- `--color-success/warning/error/info` — status colors
- `--space-1` through `--space-24` — spacing scale (4px base unit)
- `--radius-sm/md/lg/xl/2xl/full` — border radius
- `--shadow-sm/md/lg/xl` — box shadows
- `--font-size-xs` through `--font-size-3xl` — type scale
- `--font-weight-regular/medium/semibold/bold` — font weights
- `--transition-fast/normal/slow` — transitions
- `--z-canvas-bg/edge/node/panel/modal/toast` — z-index layers

**Theme system:**

- Light/dark/system theme is managed by `useThemeStore` in `src/shared/stores/themeStore.ts`
- Theme is applied via `data-theme` attribute on `<html>`
- Semantic tokens automatically reflect the active theme — no manual theme branching needed

## State management

- Use Zustand for local/editor-like UI state
- Use TanStack Query for server state
- Do not mix remote fetching concerns directly into deeply presentational components
- Keep canvas models predictable and sync-friendly
- Separate canvas state from server state

## Performance guidance

- Canvas rendering is sensitive — minimize avoidable rerenders
- Memoize only when it solves a real issue
- Treat PDF and image rendering as heavy paths
- Prefer lazy and incremental strategies when possible
- Watch for expensive derived state in render

## Collaboration readiness

Real-time collaboration is planned through Yjs/Hocuspocus:

- use serializable data shapes where practical
- avoid update patterns that are hard to reconcile
- do not tightly couple UI behavior to purely local ephemeral assumptions
- keep node and edge models stable and well-typed
- prefer deterministic updates and stable identifiers

## Implementation guidance

- Keep feature boundaries clean
- Move reusable logic into hooks or shared utilities only when reuse is real
- Avoid premature abstraction
- Prefer explicit state transitions
- Avoid making page files too smart

## Preferred workflow

Before editing:
1. Identify the feature boundary
2. Inspect nearby files for patterns
3. Preserve current naming and layout conventions

When changing code:
1. Make small, focused edits
2. Explain tradeoffs if relevant
3. Avoid broad refactors unless requested

After changing code:
1. Ensure imports and exports stay consistent
2. Keep code formatted
3. Check type safety assumptions

## Avoid

- Large speculative refactors
- Adding libraries casually
- Mixing API concerns into canvas-only components
- Burying business logic inside JSX
- Introducing hidden coupling between unrelated features
- Adding mock data inside production components unless explicitly intended
