---
name: react-liquid-glass
description: Build Apple-style Liquid Glass or frosted-glass UI in React/Next.js. Use this skill whenever the user asks for glass surfaces, glassmorphism, liquid glass, refraction effects, frosted nav bars, transparent headers, or blurred floating UI — even if they don't name a specific library. Covers mobile header bars, bottom navs, tab bars, floating filters, and replacing homemade glass effects. Supports two paths: `simple-liquid-glass` (Apple-faithful refraction with lens/rim/shape modes) and React Bits `GlassSurface` (SVG displacement refraction with per-channel chromatic aberration). Both use SSR-safe portals, accessible controls, and browser fallbacks without Three.js/WebGL/canvas.
---

# React Liquid Glass

## Overview

Use this skill to implement a production React glass component that behaves like a real UI surface, not a decorative blur card.

Two paths, both supporting refraction and chromatic aberration:

| | Path A — `simple-liquid-glass` | Path B — React Bits `GlassSurface` |
| --- | --- | --- |
| **Refraction** | SVG filter pipeline, lens modes, rim light, shape adaptation | SVG `feDisplacementMap` per RGB channel |
| **Unique strengths** | 20+ props, `effectMode="auto"`, `quality` tiers, `iosMinBlur` | Per-channel color offsets, source copied in (no runtime dep), `mixBlendMode` |
| **Install** | `npm install simple-liquid-glass` | `npx shadcn@latest add @react-bits/GlassSurface-...` |
| **Browser** | Chrome/Edge real, Safari/Firefox frosted fallback | Chrome/Edge SVG, Safari/Firefox CSS fallback |

## Gated Workflow

Two gates. Do NOT skip either.

### Gate 1 — Path Selection

If the user has NOT named a path, ask:

> **Which glass path?**
>
> **A — `simple-liquid-glass`** (npm package)
> Apple-faithful. Lens modes (`rim`/`standard`), shape adaptation, quality tiers, 20+ tuning props. Best when maximum Apple fidelity or fine-grained control is needed.
>
> **B — React Bits `GlassSurface`** (shadcn copy)
> Self-contained SVG displacement refraction with per-channel chromatic aberration. Source copied into project, no runtime dependency. Best when the project already uses React Bits, or the team prefers inlined source.
>
> Key trade-off: Path A has more knobs (lens, shapeAdapt, quality). Path B gives per-channel color control (redOffset/greenOffset/blueOffset) and zero external dependencies.
>
> Which fits your project? (A / B)

Skip this gate if the user's message names the library (e.g. "use react-bits glass", "use simple-liquid-glass").

### Gate 2 — Prerequisites Check

Verify prerequisites BEFORE implementation. Report missing items; do not proceed until resolved.

**First: check the project is React.** Read `package.json`. If `react` is not a dependency, stop and tell the user this skill only supports React 18+ / Next.js 13+ projects.

#### Path A — `simple-liquid-glass`

| # | Prerequisite | How to verify | If missing |
| --- | --- | --- | --- |
| 1 | React 18+ / Next.js 13+ | `package.json` → `react >= 18` or `next >= 13` | Stop |
| 2 | Package manager | `package-lock.json` / `yarn.lock` / `pnpm-lock.yaml` / `bun.lockb` | Ask user which to use |
| 3 | No Three.js conflicts | `package.json` has no `three` / `postprocessing` / `@types/three` | Warn about bundle bloat, ask to confirm |
| 4 | Version to install | Check `package.json` and `node_modules` for existing `simple-liquid-glass`. If none, check npm for latest: `npm view simple-liquid-glass version`. Ask user to confirm version | — |
| 5 | Build tool | Has webpack / vite / turbopack / esbuild | Custom build: verify SVG filter assets not stripped |

#### Path B — React Bits `GlassSurface`

| # | Prerequisite | How to verify | If missing |
| --- | --- | --- | --- |
| 1 | React 18+ / Next.js 13+ | `package.json` → `react >= 18` or `next >= 13` | Stop |
| 2 | shadcn CLI | `components.json` exists in project root | Run `npx shadcn@latest init`, ask user to confirm |
| 3 | CSS approach | Check for `tailwind.config` / `postcss.config` | Tailwind → pick `TS-TW`/`JS-TW`. No Tailwind → pick `TS-CSS`/`JS-CSS` |
| 4 | Output directory | `components.json` → `aliases.ui` (e.g. `@/components/ui`) | Confirm with user |
| 5 | No duplicate | Check if `GlassSurface.tsx`/`.jsx` exists in UI dir | Ask: overwrite or use existing |

**Gate 2 output** — present to the user:

```
Path: [A / B]
Prerequisites: [N/N met]
Missing: [list, or "none"]
Ready: [Yes / No — resolve missing items first]
```

Proceed only after user confirms or all prerequisites are met.

## Implementation Workflow

After both gates pass:

1. **Install** — use the exact version/variant confirmed in Gate 2:
   - Path A: `npm install simple-liquid-glass@<confirmed-version>` (pin exact version, do not use `latest`).
   - Path B: `npx shadcn@latest add @react-bits/GlassSurface-<confirmed-variant>` (variant from Gate 2 prerequisite 3).
2. Read the relevant reference doc before coding:
   - Path A: [references/implementation-checklist.md](references/implementation-checklist.md)
   - Path B: [references/react-bits-glass-surface.md](references/react-bits-glass-surface.md)
3. Replace homemade glass wrappers only in the requested surface. Preserve existing filtering, routing, ARIA labels, and state transitions.
4. Render fixed mobile glass navs through a client-only portal to `document.body`.
5. Keep the portal host clean: no `transform`, `filter`, `backdrop-filter`, opacity animation, clip, mask, or layout containment on the element that owns the fixed glass surface.
6. Support the expected product shape. For mobile category headers, use a full-width row while the header is expanded, then switch to a rounded floating pill only after scroll collapse if that is the desired interaction.
7. Add focused tests for SSR safety, portal rendering, accessibility state, target size, and fixed-position layout stability.
8. Validate with lint, typecheck, unit tests, build, and Playwright screenshot/layout checks when a browser-rendered effect is involved.

## Path Migration

If the user wants to switch paths after initial implementation:

1. The portal host, accessibility markup, scroll-state logic, and tests are portable — keep them.
2. Replace only the glass component import and its props. Use the prop mapping table in the reference doc to translate values.
3. Re-run visual checks — the two libraries produce different visual results even with equivalent prop values, so expect tuning.
4. Remove the old library (uninstall npm package or delete copied component file).

## Guardrails

- Do not install `three`, `postprocessing`, `@types/three`, canvas engines, or screenshot-capture libraries to implement this effect.
- Do not make the glass surface a normal sticky child inside a transformed or filtered ancestor.
- Do not rely on scroll-sync screenshots or DOM cloning for refraction.
- Do not hide controls behind decorative glass. Buttons and links remain semantic and keyboard accessible.
- Do not leave a design mismatch untested. Capture the initial and scrolled states separately when the shape changes.
- Do not mix the two paths in the same glass surface. Pick one library per surface.
- Different surfaces in the same project CAN use different paths (e.g. header nav uses Path A, bottom bar uses Path B), but prefer consistency unless there's a specific reason to mix.
- Do not use Path B when the design requires rim lighting, shape adaptation, or lens modes — those are Path A only.

## Common Acceptance Checks

- The component is SSR-safe and returns nothing until mounted when it needs `document.body`.
- The fixed host stays visually stable during scroll.
- The active item exposes `aria-current` and, for buttons, `aria-pressed`.
- Touch targets are at least 44px high.
- Reduced motion disables non-essential transforms or transitions.
- **Path A**: Chrome/Edge get the real effect, Safari/iOS/Firefox get a frosted fallback via `effectMode="auto"`.
- **Path B**: Chrome/Edge get SVG displacement refraction, Safari/Firefox get CSS `backdrop-filter` fallback — both built-in, no config needed.

## Troubleshooting: "Effect Not Working"

When the user reports the glass effect looks wrong, flat, or not as expected — **ask these questions before changing code**:

1. **Which browser and OS?** — The effect degrades by environment. If they're on Safari/Firefox, they're seeing the CSS fallback by design, not a bug.
2. **Is `backdrop-filter` supported?** — In DevTools, check if the computed `backdrop-filter` is applied. If not, the browser hit the third degradation layer (semi-transparent background, no blur).
3. **Is the host inside a transformed/filtered ancestor?** — `position: fixed` breaks inside `transform`, `filter`, or `backdrop-filter` ancestors. The glass host moves with scroll instead of staying fixed.
4. **Is the browser zoom at 100%?** — Non-100% zoom can misalign SVG displacement maps on some browsers.
5. **Is hardware acceleration enabled?** — `backdrop-filter` often requires GPU compositing. Check `chrome://gpu` or equivalent.

If the user confirms they're on a fallback environment and wants the real effect, they need to switch to a supported browser (Chrome/Edge). If they're on Chrome/Edge and still see no effect, then investigate the code.
