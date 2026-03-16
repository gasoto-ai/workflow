---
name: frontend-design
type: pattern
description: Create distinctive, production-grade frontend interfaces with high design quality. This is reference material — consult during implementation for UI design conventions and visual quality standards.
license: Complete terms in LICENSE.txt
---

This skill guides creation of production-grade frontend interfaces. It operates in two modes depending on whether the user provides a design reference.

## Mode Detection

**Mockup Mode** — The user provides a screenshot, mockup, Figma export, design comp, any visual reference, OR a detailed text description of a specific design/layout. If the user describes specific UI elements, layout structure, colors, or content — that's a mockup, not a creative brief. The goal is faithful implementation.

**Creative Mode** — The user gives a high-level request with no specific design details (e.g., "build me a dashboard" or "create a landing page for a coffee shop"). The goal is generating a distinctive design from scratch.

If unclear, ask: "Do you have a design/mockup to match, or should I design from scratch?"

---

## Mockup Mode: Faithful Implementation

When the user provides a design reference (visual or text), **accuracy is the primary goal**. The user or their designer already made the aesthetic decisions. Your job is to translate their design into working code, not to redesign it.

### The Golden Rule

**Do not add anything that isn't in the mockup.** No extra animations, no decorative elements, no texture overlays, no font substitutions, no grain effects, no shimmer effects, no staggered reveal animations. If the mockup doesn't show it, don't add it.

### Step 1: Analyze the Design

Before writing any code, extract these details from the reference:

- **Layout structure**: Column ratios (e.g., 47%/53%), flex vs grid, how sections relate. Use fixed widths (not responsive breakpoints) matching the mockup. If the mockup shows a desktop two-column layout, implement it as a two-column layout that stays side-by-side — don't add responsive stacking with `lg:` breakpoints unless the user asks for mobile support.
- **Spacing**: Padding, gaps, margins. Note the scale (4px, 8px based?).
- **Typography**: Use the fonts shown. If you can't identify the exact font, use a clean sans-serif system font stack — do NOT substitute fancy display fonts (no DM Serif Display, no Playfair, no editorial fonts) unless the mockup clearly uses them.
- **Colors**: Use the exact hex values specified or visible in the mockup. If the user says the accent is `#1a4a3a`, use `#1a4a3a` — don't substitute `#0B7A6F` or Tailwind's `teal-700` or any "close enough" value. Exact colors matter.
- **Components**: Button styles, input styles, border radius values, shadows. Note interactive patterns — tabs, radio card groups, payment method selectors, delivery option cards.
- **Content**: Use the exact text from the mockup. If the mockup says "Wholesale product pricing", write "Wholesale product pricing" — don't rephrase it. Every word matters.

### Step 2: Implement with Precision

- **Layout first**: Get the column structure right before anything else. Use fixed percentage widths (`w-[47%]` not `lg:w-[47%]`). The layout must match at the viewport the mockup targets.
- **Exact colors**: Use the hex values from the mockup. Don't approximate with Tailwind color classes unless they're exact matches.
- **Exact text content**: Copy every string verbatim. Don't rewrite, improve, or paraphrase any copy.
- **Match button styling**: If the mockup shows a solid dark teal button, implement `bg-[#1a4a3a]` — not a gradient, not a shadow-lift effect. Match what's shown.

### Step 3: Polish the Components

While the overall layout and structure must faithfully match the mockup, individual components should feel polished and production-ready. This means:

- **Interactive components get proper treatment**: Tab toggles should have clear active/inactive states. Radio card groups (like payment method or delivery options) should have visible selected states with borders or backgrounds. Dropdowns should feel clickable and responsive.
- **Brand icons and badges**: When the mockup shows card brand logos (Visa, Mastercard, etc.), payment provider icons (PayPal), or other recognizable badges, render them as recognizable styled elements — not empty placeholders.
- **Form inputs**: Inputs should have focus states, proper padding, and consistent border styling. Group related fields visually (e.g., expiration + security code side by side).
- **Hover/focus states**: Add sensible, subtle defaults — slight color shifts on hover, focus rings on inputs, pointer cursors on clickable elements.

The distinction: **component polish is implementing the mockup's components well.** It's making a tab toggle feel like a real tab toggle, a radio card feel like a real radio card. This is different from adding decorative extras (animations, textures, grain overlays, fancy fonts) which are banned.

### What NOT to Add

- Grain textures, noise overlays, decorative circles
- Shimmer effects, floating animations
- Font substitutions (DM Serif Display, Playfair, etc.)
- Gradient backgrounds on buttons (unless the mockup shows one)
- Decorative dividers, ornamental elements
- Elaborate hover animations (scale, translate, shadow-lift)

### Step 4: Flag Deviations

If you must deviate from the mockup, call it out explicitly:

> "The mockup appears to use [X font] — I used the system sans-serif stack as the closest available match."

Never silently "improve" on the mockup. If you think something could be better, mention it as a suggestion AFTER implementing it as-designed.

---

## Creative Mode: Design from Scratch

When no design reference is provided, generate a distinctive, memorable design.

### Design Thinking

Before coding, commit to a clear aesthetic direction:

- **Purpose**: What problem does this interface solve? Who uses it?
- **Tone**: Pick a direction — brutally minimal, maximalist, retro-futuristic, organic/natural, luxury/refined, playful, editorial/magazine, brutalist/raw, art deco, soft/pastel, industrial, or something else entirely.
- **Differentiation**: What makes this unforgettable?

### Aesthetics Guidelines

- **Typography**: Choose distinctive, characterful fonts. Avoid generic choices (Inter, Roboto, Arial). Pair a display font with a refined body font.
- **Color**: Commit to a cohesive palette with CSS variables. Dominant colors with sharp accents.
- **Motion**: Prioritize high-impact moments — staggered page load reveals over scattered micro-interactions.
- **Spatial Composition**: Unexpected layouts, asymmetry, overlap, grid-breaking elements.
- **Backgrounds & Texture**: Create atmosphere — gradient meshes, noise textures, geometric patterns, layered transparencies.

### Anti-patterns (Creative Mode Only)

Never produce generic "AI slop": overused fonts, predictable layouts, cookie-cutter component patterns. Each design should feel genuinely crafted for its context.

---

## Both Modes: Implementation Quality

All code should be:

- **Functional**: Working, production-grade code — not a static sketch.
- **Semantic**: Proper HTML elements, ARIA attributes where needed.
- **Performant**: Efficient CSS, minimal JS where possible.

When using a framework (React, Vue, etc.), follow its conventions. When the user's stack is known, use it.
