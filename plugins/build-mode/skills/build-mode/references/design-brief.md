# The design brief

This step is the difference between an app that looks like the user built it on purpose and one that looks like it came out of a machine. It takes about ten minutes and it changes every screen in the project, so do not skip it, even for something internal.

Output is `docs/DESIGN.md`, containing **real values**. That is the whole trick. "Modern and clean" gives Claude Code nothing to act on, so it falls back on defaults. `#0A0A0A` gives it no room to.

## Why AI-built apps look the same

There is a strong shared default across models, and you can name every part of it:

- Purple-to-blue gradients, usually on the hero and the primary button
- Glassmorphism: translucent cards with a backdrop blur
- `shadow-lg` on everything that is not nailed down
- Centred hero, then exactly three feature cards with icons
- Emoji as heading decoration
- Inter at default sizes with default spacing
- Rounded-2xl on every surface regardless of what it is
- Indigo-500 as the accent colour

A general instruction to "make it look premium" does not override these, because they are what the model considers premium. Naming them as things to avoid does. Put the list in `docs/DESIGN.md` and reference it in `.claude/rules/design.md` so it is loaded whenever a UI file is touched.

## Building the brief

**1. Direction.** From the user's answer in the interview. Use `frontend-design` for the aesthetic thinking. Pick one direction and commit - a design that hedges between two looks like neither. Useful anchors, because they are concrete and the user can picture them: Linear (dense, neutral, precise), Stripe (editorial, generous whitespace, confident typography), Notion (warm, soft, approachable), Vercel (stark, monochrome, high contrast).

**2. Values.** Use `ui-ux-pro-max` for the palette, the type pairing, the spacing scale and the patterns for this product type. Invoke the skill and use what it returns rather than inventing values - it holds a database of styles, palettes, font pairings and product types, and someone has already done the work of making those combinations coherent.

**3. Product-type patterns.** A dashboard, a landing page and a mobile app have different conventions. `ui-ux-pro-max` covers 192 product types. Look up the one that matches rather than applying generic layout thinking.

**4. Accessibility as a constraint, not a phase.** Bake it into the values now and it costs nothing; retrofit it after every screen uses the tokens and it is a rewrite. `ux-designer` for the detail.

**Compute the contrast ratios. Do not estimate them.** This is the one place in the whole skill worth writing a throwaway script for - about ten lines of Python implementing the WCAG relative-luminance formula, run over every foreground/background pair in the palette. Eyeballing it fails reliably, and the failure is invisible until someone with low vision cannot find the edge of an input box.

The thresholds: **4.5:1** for body text, **3:1** for large text (18px and up, or 14px bold), and **3:1** for the boundary of any interactive component - borders on inputs and buttons, focus rings. That last one catches most palettes, because a soft neutral border chosen to look calm usually lands around 1.3:1. When it fails, add a `border-strong` token for interactive edges and keep the soft one for decorative dividers.

**5. Motion, briefly.** Two or three rules is enough. Durations, easing, what animates and what does not. `ui-ux-pro-max` has GSAP presets, and `vercel-react-view-transitions` covers page transitions in React.

## docs/DESIGN.md

```markdown
# Design

## Direction
<One paragraph. What it should feel like, and one named reference.>

## Colour
| Token | Hex | Use | Contrast vs bg | Passes |
|---|---|---|---|---|
| bg | `#FFFFFF` | Page background | - | - |
| surface | `#FAFAFA` | Cards, panels | - | - |
| border | `#E4E4E7` | Decorative dividers only | 1.2:1 | n/a |
| border-strong | `#<hex>` | Input and button edges, focus rings | <n>:1 | needs 3:1 |
| text | `#18181B` | Body | <n>:1 | needs 4.5:1 |
| text-muted | `#71717A` | Secondary, captions | <n>:1 | needs 4.5:1 |
| accent | `#<hex>` | Primary buttons, links, active states | <n>:1 | needs 4.5:1 |
| accent-hover | `#<hex>` | | | |
| success / warning / danger | `#<hex>` | States | <n>:1 | needs 4.5:1 |

<!-- Ratios computed, not estimated. Every row marked "needs" must actually meet it. -->

Dark mode: <values, or "not in scope for v1">

## Type
Headings: `<font>`, weights <n>
Body: `<font>`, weights <n>

| Role | Size | Line height | Weight |
|---|---|---|---|
| h1 | 32px | 1.2 | 600 |
| h2 | 24px | 1.3 | 600 |
| h3 | 18px | 1.4 | 600 |
| body | 15px | 1.6 | 400 |
| small | 13px | 1.5 | 400 |

## Spacing
4px base. Use 4, 8, 12, 16, 24, 32, 48, 64. Nothing in between.
Page padding: 24px mobile, 32px desktop. Max content width: 1200px.

## Components
**Buttons** - height 40px, radius 8px, 16px horizontal padding.
Primary: accent background, white text. Secondary: border, transparent background.
Ghost: no border. Disabled: 40% opacity, no pointer.

**Cards** - surface background, 1px border, radius 12px, 24px padding, no shadow.

**Inputs** - height 40px, radius 8px, 1px border. Focus: 2px accent ring, 2px offset.
Error: danger border with the message below in 13px.

**Empty states** - heading, one line of body, one primary action. Never a bare blank page.

## Motion
Durations 150ms for hover, 250ms for entrances. Easing `cubic-bezier(0.4, 0, 0.2, 1)`.
Animate opacity and transform only. Respect `prefers-reduced-motion`.

## Never
- Purple-to-blue gradients
- Glassmorphism or backdrop blur
- `shadow-lg` or heavier
- Emoji in headings
- Lorem ipsum. Write the real words.
- Colours or sizes not in this file

## Accessibility
Body text contrast 4.5:1 minimum. Visible focus rings on every interactive element.
Touch targets 44px or more. Every image gets alt text. Forms label their inputs.
```

## The paired rule file

Generate `.claude/rules/design.md` alongside it, so the rules load automatically whenever Claude Code touches a UI file and cost nothing on backend work:

```markdown
---
paths:
  - "src/**/*.{tsx,jsx,css}"
  - "app/**/*.{tsx,jsx,css}"
  - "components/**/*.{tsx,jsx}"
---

# UI rules

Read `docs/DESIGN.md` before building or changing any screen.

Use only the tokens defined there. No new colours, no font sizes outside the scale,
no spacing values off the 4px grid.

Never: gradients, glassmorphism, `shadow-lg`, emoji in headings, lorem ipsum.

Every interactive element needs a visible focus state and a keyboard path to it.
Every list needs an empty state. Every async action needs a loading and an error state.
```

Those last two lines catch the most common gap in AI-built UI: the happy path is built and nothing else is.

## Figma

If the user has designs in Figma, `get_design_context` on the URL turns a frame into buildable spec, and `get_variable_defs` pulls the design tokens straight out - better than reading values off a screenshot. Going the other way, `use_figma` can push a built screen back into Figma. Read the `/figma-use` skill before calling `use_figma`; it is required.

## Design passes later

When the user says the UI looks generic, the answer is almost never "add more polish". It is usually one of:

- `docs/DESIGN.md` was too vague, so Claude Code filled the gaps with defaults. Fix the file, not the screen.
- The values are fine but the layout is the default centred-hero-three-cards. That is a structure problem, and `ui-ux-pro-max` product patterns are the fix.
- It is genuinely fine and missing only real content. Lorem ipsum makes everything look generic.

Diagnose which before writing a prompt. Three different problems, three different fixes.
