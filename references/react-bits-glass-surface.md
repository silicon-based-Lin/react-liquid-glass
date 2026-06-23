# React Bits Glass Surface

An alternative to `simple-liquid-glass` using React Bits' `GlassSurface` component. Supports SVG refraction, chromatic aberration, and browser-adaptive fallback — similar capabilities with a different API shape.

## Installation

React Bits components are installed individually via shadcn CLI:

```bash
npx shadcn@latest add @react-bits/GlassSurface-TS-TW
```

Variants available:
- `TS-TW` — TypeScript + Tailwind CSS
- `TS-CSS` — TypeScript + plain CSS
- `JS-TW` — JavaScript + Tailwind CSS
- `JS-CSS` — JavaScript + plain CSS

Pick the variant matching the project's stack. The component is copied into the project, not imported from `node_modules`.

## How It Works

The component has two rendering modes, chosen automatically:

1. **SVG mode** (Chrome/Edge): Uses an inline `<svg>` with `<filter>` containing `feDisplacementMap` per RGB channel. A dynamic displacement map is generated from SVG gradients and rendered as a `data:image/svg+xml` URI. This produces real refraction and chromatic aberration via `backdrop-filter: url(#filter-id)`.

2. **Fallback mode** (Safari/Firefox/unsupported): Uses standard CSS `backdrop-filter: blur() saturate() brightness()` with box-shadow highlights. No refraction — pure frosted glass.

Browser detection is via `supportsSVGFilters()`: tests if `backdrop-filter: url(#id)` is supported. WebKit and Firefox are explicitly excluded.

## Core Props

### Layout

| Prop | Type | Default | Description |
| --- | --- | --- | --- |
| `children` | `ReactNode` | — | Content inside the glass surface |
| `width` | `number \| string` | `200` | Surface width |
| `height` | `number \| string` | `80` | Surface height |
| `borderRadius` | `number` | `20` | Corner radius in px |

### Refraction & Displacement

| Prop | Type | Default | Description |
| --- | --- | --- | --- |
| `distortionScale` | `number` | `-180` | Base refraction strength (negative = inward) |
| `displace` | `number` | `0` | Gaussian blur stdDeviation on the final composited output |
| `borderWidth` | `number` | `0.07` | Edge width factor for the displacement map gradient |

### Chromatic Aberration

| Prop | Type | Default | Description |
| --- | --- | --- | --- |
| `redOffset` | `number` | `0` | Extra displacement scale for red channel |
| `greenOffset` | `number` | `10` | Extra displacement scale for green channel |
| `blueOffset` | `number` | `20` | Extra displacement scale for blue channel |
| `xChannel` | `string` | `'R'` | Which channel drives X displacement |
| `yChannel` | `string` | `'G'` | Which channel drives Y displacement |

### Surface Appearance

| Prop | Type | Default | Description |
| --- | --- | --- | --- |
| `brightness` | `number` | `50` | Inner surface brightness (HSL lightness) |
| `opacity` | `number` | `0.93` | Inner surface opacity |
| `blur` | `number` | `11` | Blur applied to the inner surface rect in the displacement map |
| `backgroundOpacity` | `number` | `0` | Frost background opacity (CSS custom property `--glass-frost`) |
| `saturation` | `number` | `1` | Backdrop saturation multiplier |
| `mixBlendMode` | `string` | `'difference'` | Blend mode for gradient layers in displacement map |

### Styling

| Prop | Type | Default | Description |
| --- | --- | --- | --- |
| `className` | `string` | `""` | Additional CSS classes |
| `style` | `CSSProperties` | `{}` | Additional inline styles |

## Prop Mapping: GlassSurface ↔ simple-liquid-glass

| GlassSurface | simple-liquid-glass | Notes |
| --- | --- | --- |
| `distortionScale` | `displace` + `lensStrength` | GlassSurface uses a single scale; SLG splits into displacement and lens |
| `redOffset` / `greenOffset` / `blueOffset` | `aberrationIntensity` + `dispersion` | GlassSurface per-channel control; SLG has two aggregate props |
| `displace` | `blur` | Both control Gaussian blur, but applied at different stages |
| `borderWidth` | `border` | Edge detection width |
| `brightness` | `lightness` | Inner surface lightness |
| `opacity` | `alpha` | Surface transparency |
| `saturation` | `saturation` | Same concept |
| `backgroundOpacity` | `frost` + `glassColor` | SLG separates frost amount and color |
| `mixBlendMode` | — | GlassSurface unique; controls gradient layer blending |
| — | `lens` | SLG unique; `"rim"` or `"standard"` lens mode |
| — | `shapeAdapt` | SLG unique; shape-adaptive refraction |
| — | `quality` | SLG unique; `"standard"` or `"high"` |
| — | `iosMinBlur` | SLG unique; iOS-specific minimum blur |
| — | `scale` | SLG unique; overall effect scale |

## Starting Values

Conservative starting point for a mobile nav:

```tsx
<GlassSurface
  width="100%"
  height={56}
  borderRadius={28}
  distortionScale={-180}
  redOffset={0}
  greenOffset={10}
  blueOffset={20}
  brightness={50}
  opacity={0.93}
  blur={11}
  displace={0.7}
  borderWidth={0.07}
  backgroundOpacity={0.12}
  saturation={1.4}
  style={{ pointerEvents: "auto" }}
>
  {children}
</GlassSurface>
```

For a subtle effect (less refraction, more frosted):

```tsx
<GlassSurface
  width="100%"
  height={56}
  borderRadius={28}
  distortionScale={-80}
  redOffset={0}
  greenOffset={5}
  blueOffset={10}
  brightness={60}
  opacity={0.9}
  blur={14}
  displace={0}
  backgroundOpacity={0.2}
  saturation={1.6}
>
  {children}
</GlassSurface>
```

## Integration Pattern

Because React Bits components are copied into the project, the Glass Surface file lives alongside other components. Wrap it in a thin adapter to keep the rest of the codebase decoupled:

```tsx
// components/GlassPanel.tsx
"use client";

import { GlassSurface } from "@/components/ui/GlassSurface";

interface GlassPanelProps {
  children: React.ReactNode;
  radius?: number;
  distortionScale?: number;
  redOffset?: number;
  greenOffset?: number;
  blueOffset?: number;
  backgroundOpacity?: number;
  saturation?: number;
  className?: string;
  style?: React.CSSProperties;
}

export function GlassPanel({
  children,
  radius = 20,
  distortionScale = -180,
  redOffset = 0,
  greenOffset = 10,
  blueOffset = 20,
  backgroundOpacity = 0.12,
  saturation = 1.4,
  className,
  style,
}: GlassPanelProps) {
  return (
    <GlassSurface
      borderRadius={radius}
      distortionScale={distortionScale}
      redOffset={redOffset}
      greenOffset={greenOffset}
      blueOffset={blueOffset}
      backgroundOpacity={backgroundOpacity}
      saturation={saturation}
      className={className}
      style={{ width: "100%", ...style }}
    >
      {children}
    </GlassSurface>
  );
}
```

## Mobile Nav Integration

For a floating mobile nav using React Bits Glass Surface, combine the adapter with the same portal and accessibility patterns from the `simple-liquid-glass` path:

```tsx
"use client";

import { useEffect, useRef, useState } from "react";
import { createPortal } from "react-dom";
import { GlassSurface } from "@/components/ui/GlassSurface";

interface MobileGlassNavProps {
  items: { label: string; value: string }[];
  value: string;
  onValueChange: (value: string) => void;
  ariaLabel: string;
}

export function MobileGlassNav({
  items,
  value,
  onValueChange,
  ariaLabel,
}: MobileGlassNavProps) {
  const [mounted, setMounted] = useState(false);
  const hostRef = useRef<HTMLDivElement>(null);

  useEffect(() => setMounted(true), []);

  if (!mounted) return null;

  return createPortal(
    <div
      ref={hostRef}
      style={{
        position: "fixed",
        bottom: "max(12px, env(safe-area-inset-bottom))",
        left: "max(12px, env(safe-area-inset-left))",
        right: "max(12px, env(safe-area-inset-right))",
        zIndex: 1000,
        maxWidth: 430,
        marginInline: "auto",
        pointerEvents: "none",
      }}
    >
      <GlassSurface
        width="100%"
        height={56}
        borderRadius={28}
        distortionScale={-180}
        redOffset={0}
        greenOffset={10}
        blueOffset={20}
        brightness={50}
        opacity={0.93}
        blur={11}
        displace={0.7}
        backgroundOpacity={0.12}
        saturation={1.4}
        style={{ pointerEvents: "auto" }}
      >
        <nav
          aria-label={ariaLabel}
          style={{
            display: "flex",
            alignItems: "center",
            justifyContent: "space-around",
            height: "100%",
          }}
        >
          {items.map((item) => (
            <button
              key={item.value}
              type="button"
              aria-pressed={item.value === value}
              onClick={() => onValueChange(item.value)}
              style={{
                minHeight: 44,
                minWidth: 44,
                padding: "0 12px",
                background: "none",
                border: "none",
                cursor: "pointer",
                fontWeight: item.value === value ? 600 : 400,
              }}
            >
              {item.label}
            </button>
          ))}
        </nav>
      </GlassSurface>
    </div>,
    document.body
  );
}
```

## Differences From simple-liquid-glass

| Dimension | simple-liquid-glass | React Bits GlassSurface |
| --- | --- | --- |
| Refraction | SVG filter pipeline | SVG `feDisplacementMap` per RGB channel |
| Chromatic aberration | Aggregate props (`aberrationIntensity`, `dispersion`) | Per-channel offset (`redOffset`, `greenOffset`, `blueOffset`) |
| Browser adaptation | `effectMode="auto"` prop | Auto-detect: SVG on Chrome/Edge, CSS fallback on Safari/Firefox |
| Install method | `npm install` runtime dependency | `shadcn add` — source copied into project |
| Bundle impact | Runtime package | Inlined source, no external dependency |
| Customizability | 20+ props (lens, shapeAdapt, quality, iosMinBlur, etc.) | 16 props (focused on displacement + color channels) |
| Lens modes | `lens="rim"` / `"standard"` | None — single displacement approach |
| Shape adaptation | `shapeAdapt` prop | None |
| Best for | Maximum Apple fidelity, fine-grained control | Self-contained refraction with per-channel color control |

## Testing

SSR safety, portal, and accessibility rules are the same as the `simple-liquid-glass` path.

Visual checks differ:
- **Chrome/Edge**: Verify SVG refraction and chromatic aberration render. The displacement map is generated dynamically — check that it updates on resize via `ResizeObserver`.
- **Safari/Firefox**: Verify the CSS fallback renders a clean frosted glass without refraction. This is expected behavior, not a bug.
- Both modes should produce stable layout during scroll.

Playwright checks:
- Host is visible, `position: fixed`, stable during scroll.
- Each nav item is at least 44px tall and clickable.
- No console errors during render.
- On Chrome: verify `backdrop-filter` contains `url(#glass-filter-...)`.
- On Safari: verify `backdrop-filter` contains `blur(`.

## Debugging

- If refraction looks too strong, reduce `distortionScale` (try -100 to -140).
- If chromatic aberration is too intense, reduce `greenOffset` and `blueOffset` (try 5/10 instead of 10/20).
- If the inner surface is too dark, increase `brightness` (try 60–70).
- If the frosted fallback is too transparent, increase `backgroundOpacity` (try 0.2–0.3).
- If the effect doesn't render on Chrome, check that the SVG `<filter>` element is in the DOM and the `filterId` matches.
- If the displacement map doesn't update on resize, verify `ResizeObserver` is working (check the container ref).
