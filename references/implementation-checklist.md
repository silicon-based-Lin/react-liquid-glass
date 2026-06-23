# React Liquid Glass Implementation Checklist

> Path A reference. For Path B (React Bits `GlassSurface`), see [react-bits-glass-surface.md](react-bits-glass-surface.md).

## Package And API

- Prefer `simple-liquid-glass` for React surfaces that need Apple-style Liquid Glass.
- Install exactly when needed, for example `npm install simple-liquid-glass@4.1.0`.
- Read the installed package types or local docs before choosing props. In a Next app, useful props have included:
  - `mode="custom"`
  - `effectMode="auto"`
  - `lens="rim"`
  - `lensStrength`
  - `shapeAdapt`
  - `scale`
  - `radius`
  - `border`
  - `lightness`
  - `displace`
  - `alpha`
  - `blur`
  - `dispersion`
  - `saturation`
  - `aberrationIntensity`
  - `frost`
  - `glassColor`
  - `borderColor`
  - `quality`
  - `iosMinBlur`

## Component Shape

Use a small wrapper component instead of scattering `LiquidGlass` props through feature pages.

Required behavior for mobile nav/category bars:

- Mark it `"use client"`.
- Accept `items`, `value`, `onValueChange`, `ariaLabel`, `placement`, `offset`, `maxWidth`, `height`, `theme`, and optionally `appearance` and `radius`.
- Render nothing during SSR and before mount.
- Use `createPortal(content, document.body)` after mount.
- Render a real `<nav aria-label="...">`.
- Use `<button type="button">` for in-place filters and framework `<Link>` for routes.
- Use `aria-current="page"` for the selected item.
- Use `aria-pressed` on button filters.
- Disable links with `aria-disabled`, `tabIndex={-1}`, and `preventDefault`; disable buttons with `disabled`.
- Keep labels short and truncate safely when space is tight.

## Fixed Host CSS

The portal host should be the fixed positioning boundary:

```css
.host {
  position: fixed;
  left: max(10px, env(safe-area-inset-left));
  right: max(10px, env(safe-area-inset-right));
  z-index: 1000;
  max-width: var(--liquid-nav-max-width);
  margin-inline: auto;
  pointer-events: none;
}

.glass {
  pointer-events: auto;
  width: 100%;
}
```

Avoid these properties on `.host`: `transform`, `filter`, `backdrop-filter`, opacity transitions, `clip-path`, `mask`, `contain`, and `will-change`. Those can create new containing blocks, break fixed positioning, or interfere with SVG/CSS refraction sampling.

## Full Row To Floating Pill Pattern

For a mobile top category bar, model the two states explicitly:

```tsx
<FloatingLiquidGlassNav
  appearance={isHeaderCollapsed ? "floating" : "full"}
  maxWidth={isHeaderCollapsed ? "398px" : "430px"}
  offset={isHeaderCollapsed ? "0.65rem" : "4.65rem"}
  placement="top"
  radius={isHeaderCollapsed ? 28 : 0}
/>
```

CSS for the full-width state:

```css
.host[data-appearance="full"] {
  left: 0;
  right: 0;
  max-width: min(var(--liquid-nav-max-width, 430px), 100vw);
}
```

This prevents the initial unscrolled state from looking like a detached capsule while still allowing the scrolled state to become a rounded floating control.

## LiquidGlass Prop Starting Point

Use conservative values first, then tune visually:

```tsx
<LiquidGlass
  alpha={0.76}
  aberrationIntensity={0.35}
  blur={2}
  border={0.06}
  borderColor="rgba(255, 255, 255, 0.46)"
  displace={2}
  dispersion={14}
  effectMode="auto"
  frost={0.09}
  glassColor="rgba(255, 255, 255, 0.12)"
  iosMinBlur={10}
  lens="rim"
  lensStrength={0.9}
  lightness={58}
  mode="custom"
  quality="standard"
  radius={radius}
  saturation={140}
  scale={90}
  shapeAdapt
  style={{ width: "100%", height: "var(--liquid-nav-height)" }}
>
  {children}
</LiquidGlass>
```

## Testing

Unit tests should cover:

- server render returns an empty string or otherwise does not access `document`;
- the nav appears through a `document.body` portal after mount;
- links and buttons keep correct ARIA semantics;
- disabled items do not fire `onValueChange`;
- full-row and floating appearances expose stable attributes.

Playwright checks should cover:

- initial state: host is visible, `position: fixed`, full-row width matches the mobile viewport/content shell, and no forbidden host styles are present;
- scrolled state: host switches to floating, width becomes narrower than the viewport, and its rect remains stable across scroll;
- each visible category remains clickable and at least 44px tall;
- console errors stay empty while rendering the effect.

## Debugging

- If the effect moves oddly while scrolling, inspect ancestors and the host for `transform`, `filter`, `backdrop-filter`, `opacity`, `contain`, or `will-change`.
- If SSR fails, ensure `document` and `window` are only used after mount.
- If build fails after installing extra visual packages, remove accidental WebGL dependencies unless the user explicitly requested a WebGL effect.
- If the first viewport looks like a floating pill when the product wants a header row, add a separate `full` appearance and only switch to `floating` after scroll.
