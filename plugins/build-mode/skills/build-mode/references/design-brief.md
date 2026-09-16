# The design brief

This step is the difference between an app that looks like the user built it on purpose and one that looks like it came out of a machine. It changes every screen in the project, so do not skip it, even for something internal.

It produces three things, not one:

1. **`docs/DESIGN.md`** with real values. Tokens, and also layout, hierarchy and a word budget. `#0A0A0A` gives Claude Code no room to guess; "modern and clean" gives it nothing to act on.
2. **A Screens section** in that file: one short spec per core screen, written once the roadmap says which screens exist.
3. **Mockups** in `docs/design/`: a picture of each core screen that the user has looked at and said yes to, before any of it is coded. A picture in the prompt beats a token table, because Claude Code reads images and the user can judge a picture in ten seconds where they cannot judge a spacing scale at all.

The order matters. Direction and tokens come before the roadmap because the stack call needs them. Screens and mockups come after, because until the roadmap exists you do not know what the screens are.

## Why AI-built apps look the same, twice over

There are two shared defaults, and a brief that only guards against the first lands squarely in the second. A real dashboard built from the old brief did exactly this: token-compliant, no gradients, no glass, and still flat, grey and wordy.

**The 2024 look.** Purple-to-blue gradients, glassmorphism, `shadow-lg` on everything, centred hero then three feature cards, emoji headings, Inter at default sizes, `rounded-2xl` on every surface, indigo-500 as the accent.

**The flat dev-tool clone.** Grey-on-grey with one accent colour, everything at 13px, monospace on every bit of metadata so it reads as terminal output, every card the same size and weight, stat tiles that are a number and a label with no picture, content in a narrow centred column beside a sidebar with 40 percent of the screen empty, no icons or imagery, a paragraph of helper text under every field, nothing that moves. It is what a model produces when told "dark and technical, no gradients, no shadows" and nothing else.

Both go in the Never list in `docs/DESIGN.md`, and `.claude/rules/design.md` references it so it loads whenever a UI file is touched. A general instruction to "look premium" does not override either default, because the defaults are what the model considers premium. Naming them does.

## Building the brief

### 1. Design interview, its own round, visual where possible

Do not fold design into the kickoff interview. It gets squeezed to one text question ("dark and technical") and Claude Code builds the most literal reading of the answer. Run a second `AskUserQuestion` round after the stack call, three questions max:

- **A reference.** "Paste one or two URLs of apps you like the look of." Then open them in the browser (`claude-in-chrome` or the built-in browser), screenshot them, and write down what actually makes them work: how much of the screen they use, what the eye lands on first, how many words are on a screen, where colour appears. That becomes the Direction paragraph. Taste they already have beats a name you guess.
- **Density.** Operator console (dense, lots visible at once, small type, keyboard-first) or one thing per screen (spacious, large type, one action). Most dashboards want the first; most consumer apps want the second. The wrong choice here is the single most common reason a screen feels off.
- **Mood, from pictures.** Draft three direction artboards with the `design` skill, each a rough version of the same core screen in a different direction, and let them pick. Three adjectives in a text question do not work; three pictures do. If the `design` skill is unavailable, fall back to named anchors: Linear (dense, neutral, precise), Stripe (editorial, generous whitespace, confident type), Notion (warm, soft, approachable), Vercel (stark, monochrome, high contrast). Pick one and commit; a design that hedges between two looks like neither.

Use `frontend-design` for the aesthetic thinking while you draft the artboards.

### 2. Values

Use `ui-ux-pro-max` for the palette, the type pairing, the spacing scale and the patterns for this product type. Invoke it and use what it returns rather than inventing values. It covers 192 product types; look up the one that matches rather than applying generic layout thinking.

### 3. Layout, by product type

This is the section the old brief did not have, and its absence is why an app dashboard ends up in a 900px column on a 2000px screen with both sides empty. Decide the shell and write it down:

| Product type | Shell | Content width | Grid |
|---|---|---|---|
| App with navigation (dashboard, admin, tool) | Sidebar 240px or top bar 56px, main area **fills the viewport** | No max width on the page. Max width only on reading content inside it (a settings form, a long article) at 720px | 12 columns, 24px gutters |
| Marketing or landing page | No shell | 1200px centred | 12 columns |
| Reading (docs, blog, guide) | Optional side nav | 720px centred for the text | Single column |
| Mobile-first consumer app | Bottom tab bar | Full width, 16px side padding | Single column, cards stack |

Also: where the page-level primary action lives (top right of the page header, consistently), sidebar item count and order, breakpoints (390, 768, 1024, 1440), and what collapses at each.

### 4. Hierarchy and pictures

A screen with no hierarchy is a list. Rules for every screen:

- **Three tiers of emphasis, no more.** Hero, supporting, detail. Type size and weight carry it: the hero is at least twice the body size. Sixteen things at 14px grey is zero tiers.
- **One hero element per screen.** The thing the eye lands on first. On a dashboard that is the metric that matters most, shown larger than the rest. On a list page it is the list. On a form it is the first field. If two things compete, one loses.
- **Numbers get a picture.** Any number with history gets a sparkline or bar. Any progress or ratio ("0 of 1", "3 of 5 steps") gets a ring or a progress bar. Any status gets colour, not just a word. Any money over time gets a small chart. `dataviz` in Cowork for the design, values written into DESIGN.md.
- **Cards are not all equal.** The hero stat is wider or taller. Secondary stats are smaller. A grid of identical boxes tells the reader nothing about what matters.
- **Name an icon set** (Lucide or Phosphor) and where icons appear: nav items, stat tile labels, empty states, status pills. Icons are the cheapest visual anchor there is and the flat clone has none.
- **Accent colour does one job.** Primary action and active state. If the accent is on more than three things on a screen, it is not an accent.
- **Something moves.** Hover states, a number counting in, a pill that fades in when status changes. Two or three motion rules, not zero. Zero is what makes a screen feel like a PDF.

### 5. The copy budget

"Write the real words" without a budget produces a manual inside the UI. One list page built from the old brief had around 180 words of labels and helper text before the list started. Budgets, per element:

| Element | Budget |
|---|---|
| Field label | 3 words or fewer |
| Helper text under a field | One line, and only where the field is genuinely ambiguous. Most fields get none. |
| Page subtitle | One sentence or none |
| Empty state | Heading up to 6 words, body up to 12, one button |
| Status line on a list item | One line. What it is, not why. |
| Explanation of how something works | Not on the screen. A tooltip on an info icon, or the Guide page, linked once. |
| Stat tile | Label of 1 to 3 words, the number, and one optional line of context under it |

Rule of thumb: if a stranger needs the sentence to use the screen, keep it. If it is there to reassure, cut it. A screen should be readable in five seconds.

### 6. Accessibility as a constraint, not a phase

Bake it into the values now and it costs nothing; retrofit it after every screen uses the tokens and it is a rewrite. `ux-designer` for the detail.

**Compute the contrast ratios. Do not estimate them.** About ten lines of Python implementing the WCAG relative-luminance formula, run over every foreground/background pair. The thresholds: **4.5:1** for body text, **3:1** for large text (18px and up, or 14px bold), and **3:1** for the boundary of any interactive component. That last one catches most palettes, because a soft neutral border chosen to look calm usually lands around 1.3:1. When it fails, add a `border-strong` token for interactive edges and keep the soft one for dividers.

Dark themes fail this differently: `text-muted` at `#71717A` on `#0A0A0A` passes, but the same grey on a `#1C1C1C` card does not. Compute against every surface, not just the page background.

### 7. Motion, briefly

Two or three rules. Durations, easing, what animates and what does not. `ui-ux-pro-max` has GSAP presets, and `vercel-react-view-transitions` covers page transitions in React.

### 8. Screens, after the roadmap

Once `docs/ROADMAP.md` exists you know which screens Phase 1 builds. Write a short spec for each core screen (three to five, never more) into the Screens section of DESIGN.md. Each one:

- **Job.** One sentence. What the user comes here to do.
- **Hero.** The one element the eye lands on first.
- **Layout.** Which shell, what the grid does on this screen, what is above the fold at 1440 and at 390.
- **Words.** A budget for the whole screen, usually under 60 outside the data itself.
- **Picture.** The mockup filename in `docs/design/`.

### 9. Mockups, before code

For each core screen, draft an artboard with the `design` skill at 1440 wide (and 390 for anything mobile-first), using the real tokens and real words, on the real layout. The user looks at it on the canvas, tweaks it if they want, and says yes. That yes is the design approval; nothing else in the loop asks them to judge design in the abstract.

Once approved, render each artboard to PNG and put it in the repo at `docs/design/<screen>.png`, alongside the artboard source as `docs/design/<screen>.html` so a later pass can re-render it. Render with Playwright in the Cowork workspace (Chromium is installed there), or ask the user to export from the canvas if that fails. Commit them with the rest of the doc pack.

Every UI prompt from then on says "Match `docs/design/dashboard.png`. Where the picture and the token table disagree, the picture wins on layout and the table wins on values."

If the `design` skill is unavailable, write an ASCII wireframe into the Screens section instead. Worse than a picture, far better than nothing.

## docs/DESIGN.md

```markdown
# Design

## Direction
<One paragraph. What it should feel like, one named reference, and what specifically
was borrowed from the reference URLs the user gave: screen usage, where the eye lands,
how many words per screen, where colour appears.>

Density: <operator console | one thing per screen>

## Layout
Shell: <sidebar 240px | top bar 56px | none | bottom tabs>
Main area: fills the viewport. No page-level max width.
Reading content (forms, articles): max 720px.
Grid: 12 columns, 24px gutters. Page padding 24px mobile, 32px desktop.
Page header: title left, primary action top right, always.
Breakpoints: 390 (stack everything), 768 (sidebar collapses to top bar), 1024, 1440.

## Colour
| Token | Hex | Use | Contrast vs bg | vs surface | Passes |
|---|---|---|---|---|---|
| bg | `#<hex>` | Page background | - | - | - |
| surface | `#<hex>` | Cards, panels | - | - | - |
| border | `#<hex>` | Decorative dividers only | <n>:1 | <n>:1 | n/a |
| border-strong | `#<hex>` | Input and button edges, focus rings | <n>:1 | <n>:1 | needs 3:1 |
| text | `#<hex>` | Body | <n>:1 | <n>:1 | needs 4.5:1 |
| text-muted | `#<hex>` | Secondary, captions | <n>:1 | <n>:1 | needs 4.5:1 |
| accent | `#<hex>` | The primary action and active nav. Nothing else. | <n>:1 | <n>:1 | needs 4.5:1 |
| accent-hover | `#<hex>` | | | | |
| success / warning / danger | `#<hex>` | Status pills, chart series | <n>:1 | <n>:1 | needs 4.5:1 |
| chart-1 .. chart-4 | `#<hex>` | Sparklines, bars, rings | | | needs 3:1 |

<!-- Ratios computed, not estimated, against every surface the token appears on. -->

Dark mode: <values, or "not in scope for v1">

## Type
Headings: `<font>`, weights <n>
Body: `<font>`, weights <n>
Mono: `<font>`, for code, IDs and SHAs only. Not for metadata, dates or money.

| Role | Size | Line height | Weight |
|---|---|---|---|
| display (hero numbers) | 40px | 1.1 | 600 |
| h1 | 28px | 1.2 | 600 |
| h2 | 20px | 1.3 | 600 |
| h3 | 16px | 1.4 | 600 |
| body | 15px | 1.6 | 400 |
| small | 13px | 1.5 | 400 |

The hero on any screen uses display or h1. Body is the floor for anything the user reads;
small is for captions only.

## Spacing
4px base. Use 4, 8, 12, 16, 24, 32, 48, 64. Nothing in between.

## Hierarchy
Three tiers per screen: hero, supporting, detail. One hero element per screen.
Numbers with history get a sparkline. Ratios and progress get a ring or bar.
Status gets colour. Stat tiles are not all the same size: the hero stat is wider.
Icons: `<Lucide | Phosphor>`, 20px in nav, 16px in tile labels and pills.

## Copy
Field labels 3 words or fewer. Helper text one line, and only where the field is
ambiguous. Empty states: heading up to 6 words, body up to 12, one button.
Explanations of how things work go in a tooltip or the guide page, not on the screen.
Under 60 words per screen outside the data itself.

## Components
**Buttons** - height 40px, radius 8px, 16px horizontal padding.
Primary: accent background. Secondary: border-strong, transparent. Ghost: no border.
Disabled: 40% opacity, no pointer.

**Cards** - surface background, 1px border, radius 12px, 24px padding, no shadow.
Hero card spans 2 columns of the stat grid; secondary cards span 1.

**Stat tile** - label (small, muted, with icon), number (display or h1), one optional
context line, and a sparkline or ring where the number has history or is a ratio.

**Inputs** - height 40px, radius 8px, 1px border-strong. Focus: 2px accent ring, 2px offset.
Error: danger border with the message below in 13px.

**Status pill** - 24px tall, coloured background at 15% and text at 100% of the state colour.

**Empty states** - heading, one line, one primary action. Never a bare blank page.

## Motion
Durations 150ms for hover, 250ms for entrances. Easing `cubic-bezier(0.4, 0, 0.2, 1)`.
Numbers count in on first paint. Pills fade when status changes. Rows lift 1px on hover.
Animate opacity and transform only. Respect `prefers-reduced-motion`.

## Screens
### <Screen name> - `docs/design/<screen>.png`
Job: <one sentence>
Hero: <the one element>
Layout: <shell, grid on this screen, what is above the fold at 1440 and 390>
Words: <budget>

<repeat for each core screen, three to five>

## Never
The 2024 look:
- Purple-to-blue gradients, glassmorphism, `shadow-lg` or heavier
- Centred hero with three feature cards, emoji in headings, `rounded-2xl` on everything

The flat dev-tool clone:
- Grey-on-grey with everything at 13px
- Monospace outside code, IDs and SHAs
- A grid of identical cards; stat tiles that are only a number and a label
- Page content in a centred column with the sides of the viewport empty
- No icons, no charts, no imagery anywhere
- A paragraph of helper text under every field
- Nothing that moves

And always:
- Lorem ipsum. Write the real words, within the copy budget.
- Colours or sizes not in this file

## Accessibility
Body text contrast 4.5:1 minimum against every surface it sits on. Visible focus rings
on every interactive element. Touch targets 44px or more. Every image gets alt text.
Forms label their inputs.
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

Read `docs/DESIGN.md` before building or changing any screen. If the screen has a
mockup in `docs/design/`, open the PNG and match it. The picture wins on layout,
the token table wins on values.

Use only the tokens defined there. No new colours, no font sizes outside the scale,
no spacing values off the 4px grid.

The main area fills the viewport. Do not centre app content in a narrow column.
One hero element per screen, at display or h1 size. Numbers with history get a
sparkline; ratios get a ring or bar; status gets colour. Icons from the named set.

Stay inside the copy budget in DESIGN.md. Labels 3 words. Helper text one line and
only where needed. Explanations go in a tooltip or the guide, not on the screen.

Never, either way: gradients, glassmorphism, `shadow-lg`, emoji in headings,
lorem ipsum, grey-on-grey at 13px, monospace outside code and IDs, identical
card grids, empty side margins, text-only stat tiles.

Every interactive element needs a visible focus state and a keyboard path to it.
Every list needs an empty state. Every async action needs a loading and an error state.
```

Those last two lines catch the most common gap in AI-built UI: the happy path is built and nothing else is.

## Figma

If the user has designs in Figma, `get_design_context` on the URL turns a frame into buildable spec, and `get_variable_defs` pulls the design tokens straight out. Going the other way, `use_figma` can push a built screen back into Figma. Read the `/figma-use` skill before calling `use_figma`; it is required. A Figma frame can stand in for a `design` skill mockup; export it to `docs/design/` the same way.

## Design passes later

When a screen looks generic, the answer is almost never "add more polish". `design-review.md` has the score and the diagnosis. The short version: the mockup was skipped, the Screens spec was too thin, the layout is the centred column, the hierarchy is flat, or the copy budget was ignored. Five different problems, five different fixes. Diagnose before writing a prompt.
