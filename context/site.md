# CastCanvas Lab Site — Context

> 이 파일은 `cast-canvas-lab-site` 레포 작업 시 필요한 상세 컨텍스트다.
> 오케스트레이터가 관리하며, `cast-canvas-lab-site/CLAUDE.md`는 이 파일을 참조한다.

## About this repo

Public-facing website for CastCanvas Lab. **Not** the core application.

- **Goals:** Explain the product, present it visually, host docs, direct users to the app.
- **Stack:** Next.js, React, TypeScript, pnpm.

## Repository Boundaries

**Owns:** Landing pages, feature pages, public docs, screenshots/demo, app entry links.

**Does NOT Own:** Canvas logic, backend APIs, Yjs sync, collaboration transport.

## MVP Focus

- Support product introduction and credibility
- Clearly distinguish between current features and planned (MVP 제외) features
- Maintain terminology: workspace, canvas, document, reference image, node, edge, collaboration

## Implementation guidance

- **Clarity over Hype:** Use clear, specific language. Avoid exaggerated marketing fluff.
- **Maintainability:** Use reusable sections/components. Keep content easy to scan.
- **Consistency:** Align all public copy with the real system architecture in `ARCHITECTURE.md`.
- **Vanilla CSS:** Prefer Vanilla CSS for styling (unless Tailwind is explicitly requested).
- **Brand source of truth:** For the current redesign track, treat `tasks/SITE_FE_BRAND_GUIDE.md` as the shared tone guide and assume `site` is the first repository where the brand direction is finalized.

## Design System Rules

- **Token-first:** All colors, spacing, radius, shadow, font sizes must use CSS custom properties defined in `src/shared/styles/_tokens.scss`. Never use hardcoded values.
- **Typography:** Use `_typography.scss` tokens. Site base is 16px (`--font-size-lg`), not 14px like the app.
- **Mixins:** Use `@include container`, `@include section`, `@include sm/md/lg/xl` from `_mixins.scss`.
- **Component pattern:** Each component lives in `src/components/{Name}/` with `{Name}.tsx` + `{Name}.module.scss`.
- **Dark mode:** Supported via `data-theme` attribute + `prefers-color-scheme`. Both must be handled.
- **Alignment with FE repo:** Design tokens are derived from `cast-canvas-lab-fe`. If FE tokens change, sync `_tokens.scss` accordingly.

## Status Tracking

- **Always read `STATUS.md` before starting work.**
- **Always update `STATUS.md` after completing work.**
- Move items from Pending to Completed, add new findings to Pending, update Architecture Decisions if needed.
- When the task touches the redesign track, keep the result aligned with ORCH 에픽 `#8` and SITE 이슈 `#4`.

## Avoid

- Mixing core app logic into this repository
- Presenting planned features as already completed
- Broad speculative refactors without specific directives
