# Build Mode

**A planning and prompting layer for people who build software in Claude Code but cannot read a diff.**

Claude Code is very good at writing code. It is less good at deciding what should get
built, in what order, to what standard, and whether the thing it just said was finished
actually is. Build Mode is the other half of that pairing.

You describe what you want. It interviews you once, picks the stack and tells you why,
writes a design brief with real values in it, generates a full doc pack into your repo,
then hands you one ready-to-paste prompt at a time. When Claude Code reports back, it
reads the repo and checks whether that report was true.

> **Where it runs:** Build Mode is a **Cowork** skill. It does the planning, designing and
> prompt writing in Cowork, and Claude Code does the building. The two are separate
> sessions and you move between them. Install it in Cowork.

---

## Install

### In Cowork

Paste this into a Cowork session:

```
Install this skill: https://github.com/Samcarr01/build-mode
```

That is it. Then connect the session to your project folder and start talking.

### In Claude Code (optional)

The repo is also a plugin marketplace, if you want the skill available on the Claude Code
side too:

```
/plugin marketplace add Samcarr01/build-mode
/plugin install build-mode@samcarr
```

If the install summary says `Run /reload-plugins to activate.`, run that. Update later
with `/plugin marketplace update samcarr`.

This is genuinely optional. The skill is written for Cowork and uses Cowork's tools, so
Cowork is where it earns its keep.

---

## Using it

Once installed, connect a Cowork session to your project folder and say something like:

> I want to build a habit tracker

or, mid-build:

> give me the next prompt

It triggers on its own. You do not need to type `/build-mode`.

**One session per project.** Everything the project knows lives in that repo, so a
session pointed at a different folder cannot see any of it.

---

## How it works

```
Your idea
   -> Cowork      : interview, stack call, design brief, roadmap, doc pack
   -> Cowork      : writes docs/next-prompt.md into the repo
   -> You         : in Claude Code, type  /next
   -> Claude Code : plans, builds, updates PROGRESS.md + ROADMAP.md
   -> You         : back in Cowork, "done" / "it broke" / "next"
   -> Cowork      : reads the repo, verifies, learns, writes the next prompt
```

Nothing in that loop depends on you remembering anything. The repo carries the state.

### Six modes

It reads what you said and picks one:

| You say | Mode |
|---|---|
| "I want to build X", "new project", "set up the repo" | **Kickoff** |
| "next", "what do I tell it now", "give me the prompt" | **Next task** |
| "done", "it built it", "check this", "where am I" | **Sync** |
| "it's stuck", "this error", "it built the wrong thing" | **Unblock** |
| "I want to add", "actually let's change", "drop that" | **Replan** |
| "make this look better", "the UI is generic" | **Design pass** |

---

## What it writes into your repo

| File | What it holds |
|---|---|
| `CLAUDE.md` | How this project works. Claude Code loads it every session |
| `docs/ROADMAP.md` | Milestones and tasks with permanent IDs |
| `docs/PROGRESS.md` | What actually got built, newest first |
| `docs/LEARNINGS.md` | Anything that cost more than ten minutes to figure out |
| `docs/ARCHITECTURE.md` | Stack call, schema, decisions and why |
| `docs/DESIGN.md` | Fonts, hex codes, spacing scale, component rules, motion |
| `docs/TOOLING.md` | What is actually available, so plans do not assume tools you lack |
| `docs/next-prompt.md` | The current prompt, overwritten each time |

It also sets up `.claude/skills/` in the repo so you get `/next`, `/checkpoint` and
`/blocked` inside Claude Code.

---

## What is actually in here

Eight files, about 85KB. The bulk of it is not prose, it is specifics:

- **`SKILL.md`** - the six modes, the memory rules, the folder discipline
- **`references/stack-picker.md`** - default stack and when to deviate, with costs
- **`references/design-brief.md`** - why AI-built apps all look the same, and the fix
- **`references/doc-pack.md`** - exact templates for all six documents
- **`references/prompt-recipes.md`** - the six-part prompt shape, plus worked recipes for
  a new screen, a third-party integration, an LLM feature and a bug report
- **`references/claude-code-setup.md`** - verified `CLAUDE.md`, `.claude/rules/` and skill
  frontmatter syntax, permission modes, the traps that fail silently
- **`references/tooling.md`** - the capability scan, and the skill-visibility gap below
- **`references/memory.md`** - what to write down and what to leave out

---

## Optional add-ons

Build Mode works on its own. These skills make its output better where they exist, and it
degrades gracefully where they do not. Install them in **Cowork**, alongside Build Mode.

**Design and UI**

| Skill | What it adds |
|---|---|
| `ui-ux-pro-max` | Colour palettes, type pairings, spacing scales, product-type patterns |
| `frontend-design` | Aesthetic direction, so the result does not read as templated |
| `ux-designer` | Flows, forms, onboarding, accessibility |
| `apple-hig` | Native Apple apps, exact measurements and platform conventions |
| `web-design-guidelines` | Auditing UI code that already exists |
| `dataviz` | Anything with a chart in it |

**Backend and framework**

| Skill | What it adds |
|---|---|
| `supabase` | Auth, database, edge functions, RLS |
| `supabase-postgres-best-practices` | Schema design, migrations, indexes, slow queries |
| `vercel-react-best-practices` | React and Next.js performance |
| `vercel-composition-patterns` | Component API design |
| `vercel-react-view-transitions` | Page and shared-element animation |
| `webapp-testing` | Driving a local app to check it actually works |

**MCP connectors worth wiring up:** Supabase (schema, migrations, logs), Vercel (deploys,
build logs, runtime errors), GitHub (commits, PRs, issues), Figma (design context both
directions).

### One thing worth knowing about add-ons

**Cowork and Claude Code do not share skills.** A skill enabled on your Claude account is
available in Cowork, but it is not automatically available inside Claude Code on your
machine, and there is no one-line fix.

Build Mode handles this by baking the output in rather than depending on the skill. It
uses the design skills in Cowork and writes the actual hex values, type scale and spacing
steps into `docs/DESIGN.md`. Claude Code then reads a document with real numbers in it
and does not need the skill at all. Same for architecture rules into
`docs/ARCHITECTURE.md`.

That is deliberate, and it is the single most useful thing the skill does.

---

## Who this is for

Someone who ships, understands systems, and has ideas worth building, but who will not
catch a bad architectural call by reading a diff. Build Mode assumes that by default:
it decides for you, explains in one line why, keeps prompts self-contained, and verifies
the work rather than trusting a "done" message.

Tell it you read code fluently and it dials back the explaining.

---

## Notes

- Verified against Claude Code as of August 2026. It moves fast, so check the live docs at
  [code.claude.com](https://code.claude.com/docs) if something behaves differently to the
  `claude-code-setup.md` reference.

## Licence

MIT. See [LICENSE](LICENSE).
