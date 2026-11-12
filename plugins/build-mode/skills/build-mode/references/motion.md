# Motion

Motion is the cheapest polish there is and the easiest to overdo. Earlier versions of this skill asked for it ("something moves") without being able to check it, because the design review works from screenshots and a screenshot cannot see motion. This file is the fix: what to decide at kickoff, which tool does which job, the rules to paste into prompts, how Sync checks motion, and Showcase mode for sites whose job is to impress.

Drawn from Emil Kowalski's writing (animations.dev), LottieFiles' `motion-design` skill and iart.ai's `web-animation-skills` (both MIT).

**The one rule.** Motion confirms an action, shows that something changed, or points the eye at what matters. If it does none of those, cut it. Nothing the user needs waits hidden for an animation to reveal it.

## Decide it at kickoff

Pick one **personality** per project, then write the Motion section of `docs/DESIGN.md` in the shape below.

| Personality | For | Durations | Bounce |
|---|---|---|---|
| Corporate (default for apps) | Dashboards, CRMs, admin, tools | 150 to 300ms | None |
| Premium | Marketing sites, editorial | Up to 500ms, marketing pages only | None |
| Playful | Consumer apps, illustration | 150 to 300ms | Small spring |
| Energetic | Launches, games | 100 to 250ms | Spring |

```markdown
## Motion
Personality: Corporate
Curve: `cubic-bezier(0.16, 1, 0.3, 1)` for entering and leaving; `cubic-bezier(0.4, 0, 0.2, 1)` for things moving on screen.
Durations: fast 150ms (hover, press, toggles), base 200ms (pills, toasts, cards), slow 300ms (modals, drawers). Exits about a third faster than entrances.
Entrance: <one pattern, e.g. fade plus 8px rise>, first paint only, stagger 30 to 50ms, whole sequence under 400ms.
Moments, three at most: <e.g. 1. hero stat counts up once per session 2. status pills cross-fade 3. cards lift 2px on hover>
Reduced motion: remove parallax, large slides, zooms and loops. Turn other movement into a 150ms fade. Keep colour changes, focus rings and spinners.
Never: <the Never list below, plus anything project-specific>
```

The **moments** are chosen when the user approves the mockups and written beside the screen they belong to. Every UI prompt then names which moment it builds, or says "no new motion". When opening the user's reference URLs in the design interview, note what moves on them too: what, how fast, how often.

## Which tool

- **CSS through Tailwind first.** Hover, press, fades, loops, `@starting-style` for things appearing, and scroll-driven animation (in every major browser since October 2026; wrap it in `@supports (animation-timeline: view())`). Movement classes carry `motion-safe:`.
- **Motion** (`motion/react`) only for exits (`AnimatePresence`), layout changes (`layout`, `layoutId`) and springs. Added by the first task that needs it, not at scaffold. It lives in small leaf client components or is imported from `motion/react-client`; a page never gets `"use client"` just to animate. Wrap the app once in `<MotionConfig reducedMotion="user">`.
- **Page transitions:** `vercel-react-view-transitions` where installed (React's own ViewTransition). Not the "FrozenRouter" recipe, which leans on a Next.js internal import that breaks on upgrade.
- **GSAP:** only on a Showcase page (below), and never on the same page as Motion.
- **GSAP's own skills can try to take over.** If the user has GreenSock's `gsap-skills` plugin, it is active in every project, and `gsap-skills:gsap-react` recommends GSAP for any React animation unless a library has been chosen. So on every build that is not Showcase, any UI prompt that adds motion says in its Constraints: "Animation: CSS and Motion only, not GSAP."
- **Expo:** Reanimated.
- **ui-ux-pro-max:** leave its `--motion` dial unset. Every tier attaches a GSAP snippet, which fights the split above.
- **Animated component libraries:** none on app screens. On a marketing page, one at most, named in DESIGN.md, React Bits first because it handles reduced motion.

## Craft rules

Paste the ones that apply into the UI prompt's Constraints.

- Nothing on an app screen runs longer than 300ms. Exits are faster than entrances.
- Animate transform and opacity only. Never `transition-all`.
- Interactive things use CSS transitions, not keyframes, so they can reverse half-way.
- What people do many times a day stays quick, keyboard actions do not animate, and a count-up plays once per session. It has to feel right on the hundredth view.
- Press feedback is instant: scale to 0.97 on tap.
- Scale in from 0.95, never from 0. Popovers and menus grow from the thing that opened them.
- Staggers finish inside 500ms, and no more than a third of the screen moves at once.
- Anything that loops for more than five seconds can be paused or stops by itself (WCAG 2.2.2), and stops under reduced motion.

**Never:** fade-up on every section, glowing beams, shimmer borders, animated gradients, bouncing cards, parallax text, smooth-scroll or scroll-jacking libraries on app screens. These are the 2026 version of the purple gradient. A Showcase build lifts parts of this list; see below.

## Checking it: check 5 in Sync

A screenshot cannot see motion, so check 5 in `design-review.md` ("Picture and motion") needs the picture part plus all three of these:

1. **States.** Screenshot the hero element idle, hovered, and after its main action. Something visibly changes in each, and it matches the moments in DESIGN.md. Compare settled states rather than chasing frames.
2. **Probe.** With the browser's JavaScript tool, read `getComputedStyle(el).transitionProperty` and `transitionDuration` on what you hovered: no `all`, durations from DESIGN.md, nothing over 300ms on an app screen. Then reload, wait two seconds, and list anything in the first screen still at `opacity: 0`. A hit is a reveal that never fired, and it fails the check.
3. **Code.** `rg` the changed files for `transition-all`, for `"use client"` added to a `page.tsx`, and, if Motion is installed, for `MotionConfig` with `reducedMotion="user"`.

**Polish tasks add two things.** Claude Code runs Lighthouse on the live URL as a Done-when tick (`npx lighthouse <url> --only-categories=performance,seo --output=json --quiet`, mobile defaults): layout shift under 0.1, largest paint under 2.5s, blocking time under 200ms, and on public pages an SEO score of 90 or more. Screens behind a sign-in skip it and say so in the PROGRESS entry. And Cowork records a GIF of page load plus one hover with `claude-in-chrome` and sends it to the user: "does this feel right?" is the one judgement question for that Sync.

## Showcase mode

For sites whose job is to impress: an agency's own site, a launch page, a portfolio, a client who asks for "wow". Not app screens and not everyday client sites, where everything above holds. It is switched on in the design interview, or when the user says "make it flashy", "wow" or "award-winning", and written as `Mode: Showcase` at the top of `docs/DESIGN.md` so every later prompt reads it. It can cover one page, usually the home page, and not the rest.

Flashy and generic are not the same thing. AI-built "wow" sites look alike because they all reach for the same library effects. A showcase site is memorable because its big moment comes from the subject itself.

### What changes

- **One signature moment, built properly.** Drawn from the client's world, not a component library: current tracing a circuit as you scroll for an electrician, a slow camera move through the product for a software launch. It is the thing people screenshot. Up to four supporting effects from the menu below; everything else stays quiet, as `frontend-design` says.
- **Bigger and longer is allowed** for the hero and scroll scenes: up to about 800ms, or tied to scroll position. Buttons, menus and forms still follow the craft rules.
- **GSAP is the engine** on showcase pages (ScrollTrigger, SplitText, Flip, all free for commercial use since April 2025). Not Motion on the same page. Lenis smooth scroll is allowed on desktop.
- **Libraries:** up to two pieces from React Bits, Magic UI or Aceternity, named in DESIGN.md and restyled to the brand.

### The effect menu

Pick by what the brand is about, not by what looks impressive in a demo.

| Effect | Built with | Good for |
|---|---|---|
| Headline builds line by line or letter by letter | GSAP SplitText | Launches, bold type |
| Section pins while the visual changes through steps | GSAP ScrollTrigger | Explaining a service or process |
| Horizontal scroll through work | ScrollTrigger | Case studies, portfolios |
| A line or path draws itself as you scroll | SVG with ScrollTrigger | Circuits, journeys, timelines |
| 3D object or scene that reacts to mouse and scroll | React Three Fiber, model as a GLB | Products, hero scenes |
| Looping video hero | Muted MP4 or WebM with a poster image | Atmosphere, fast |
| Images unmask as they scroll in | CSS `clip-path` with scroll-driven animation, or GSAP | Photography, job photos |
| Marquee of logos or reviews | CSS | Social proof |
| Magnetic buttons, custom cursor | GSAP `quickTo` | Agency feel, desktop only |
| Animated transitions between case studies | `vercel-react-view-transitions` | Portfolios |

### Design it before code

- In the design interview, open two or three references from Awwwards, Godly or Codrops (Codrops demos come with code) alongside the user's own, and write down the one effect worth borrowing from each and why.
- The signature moment gets a storyboard on the artboard: three frames, start, middle and end, at 1440 and 390. The user approves it like any mockup.
- Art is half the effect. Write the image, video and 3D prompts (with a prompt-writing skill where installed), and have the user make or supply them as an `[@]` task before the UI task that uses them, files in `public/`.

### Guardrails that still hold

- **Reduced motion gets the calm version:** no smooth scroll, no pinning, the 3D scene swaps for its poster image, text appears whole.
- **Phones:** under 768px, 3D and pinned scenes become a still or a short video, and Lenis is off on touch.
- **Weight:** GSAP plugins, Three.js and video load only on the page that uses them, by dynamic import, never in the root layout. The Lighthouse gate still runs on mobile: largest paint under 2.5s, layout shift under 0.1; blocking time may go to 300ms.
- Every effect decorates words already in the server HTML. A 3D or video hero sits beside a real h1, never instead of one. See `references/findability.md`.
- Split text stays readable to screen readers, loops can be paused, and fade-up on every section is still banned.

### Tools and the check

GreenSock publishes official skills as a Claude Code plugin. Check for it in the capability scan (`/plugin` output). If it is missing, offer the install once, as an `[@]` task: `/plugin marketplace add greensock/gsap-skills`, then `/plugin install gsap-skills@gsap-skills`. It installs at user scope, so one install covers every project on that machine; cloud sessions do not load local plugins. Without it, Claude Code works from memory and `context7`.

Where it is installed, name the skills by their full names, because the skills that get named are the ones that get used:

- `gsap-skills:gsap-react` on every showcase task: the `useGSAP` hook and clean-up in Next.js
- `gsap-skills:gsap-scrolltrigger` for pinned and scroll-linked scenes
- `gsap-skills:gsap-plugins` for SplitText, Flip, the SVG plugins and CustomEase
- `gsap-skills:gsap-timeline` when the signature moment is a sequence
- `gsap-skills:gsap-performance` on the polish task

Also `frontend-design`, `vercel-react-view-transitions` for page transitions, and `context7` for React Three Fiber. When a project turns Showcase on, record the plugin on the Claude Code side of `docs/TOOLING.md`.

Check: the motion check above, plus the storyboard. On a scroll scene, scroll position is the clock, so scroll to the start, middle and end of the signature moment, screenshot each and compare them with the storyboard frames. Then record a GIF of one full scroll and send it to the user. In the code check, also `rg` for `gsap` imported in `app/layout.tsx` (it belongs only on the page that uses it) and for GSAP set up in a bare `useEffect` instead of `useGSAP`, which leaves animations running after the page changes.
