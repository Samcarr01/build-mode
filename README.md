# Build Mode

**A planning and prompting layer for people who build software in Claude Code but cannot read a diff.**

Claude Code is very good at writing code. It is less good at deciding what should get
built, in what order, to what standard, and whether the thing it just said was finished
actually is. Build Mode is the other half of that pairing.

You describe what you want. It interviews you once, picks the stack and tells you why,
writes a design brief with real values in it, generates a full doc pack into your repo,
then hands you one ready-to-paste prompt at a time. When Claude Code reports back, it
checks the live deploy, opens the app in a browser and verifies the work rather than
taking the report on trust.

> **Where it runs:** Build Mode is a **Cowork** skill. It does the planning, designing and
> prompt writing in Cowork, and Claude Code does the building. The two are separate
> sessions and you move between them. Install it in Cowork.

---

## What is new in 2.0

Version 1 was a good planner with a weak evidence loop. It could tell Claude Code what to
build, but nothing in the system ever proved the result worked. Version 2 closes that.

- **`/checkpoint` now has to prove it.** Before a task can be ticked off it builds the
  project, typechecks it, walks the new flow in a real browser, then commits and pushes.
  If any of that fails the task is marked blocked, not done.
- **Sync looks at the running app**, not just the repo. It reads the deploy status, opens
  the live URL, screenshots it and audits what it sees against the design values it wrote.
- **Nothing was pushing.** In v1 only the first task pushed to git, so the deployed site
  quietly fell behind the code. `/checkpoint` now pushes every time.
- **Undo mode.** "Put it back", "it was working yesterday". Finds the last good commit,
  tells you in plain English what you lose, and refuses to strand your database.
- **Upgrade mode.** Refreshes the setup files in a project you already started, so
  improvements to this skill reach projects already under way.
- **Superpowers handling.** If you run [obra/superpowers](https://github.com/obra/superpowers),
  it will re-open design decisions Build Mode already settled. Build Mode now detects this
  and resolves it in `CLAUDE.md`. See [Plugins](#plugins-you-might-already-have).
- **First-prompt bug fixed.** v1 wrote `CLAUDE.md` into your folder and then told Claude
  Code to run `create-next-app` there, which refuses a non-empty directory. Session one,
  first prompt, stalled.

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

## What you need

Build Mode works with nothing else installed. But one thing changes what it can promise,
and the rest make its output better.

### The one that matters

| What | Where | Why |
|---|---|---|
| **A browser-driving tool** - the [Playwright MCP](https://github.com/microsoft/playwright-mcp) or the `webapp-testing` skill | **Claude Code** | This is what lets `/checkpoint` click through your app and prove a task works |

Without it, `/checkpoint` still builds and typechecks, but it cannot confirm the feature
actually does anything. Every check then lands back on you to click, once, manually. Build
Mode will say so rather than quietly generating a weaker setup.

Cowork cannot fill this gap. It has no route to localhost on your machine, so the only
side that can run your app is the side that lives on it.

### Accounts

You need somewhere to put the code and somewhere to run it. The defaults are GitHub,
Vercel and Supabase, all of which have free tiers that cover a project until it has real
users. Build Mode asks which you already have during the interview and plans around what
is missing.

### Skills that make it better

None are required. Build Mode degrades gracefully where they are absent and writes the
values into your docs instead. Install them in **Cowork**, alongside Build Mode.

**Design and UI**

| Skill | What it adds |
|---|---|
| `ui-ux-pro-max` | Colour palettes, type pairings, spacing scales, product-type patterns, chart types |
| `frontend-design` | Aesthetic direction, so the result does not read as templated |
| `ux-designer` | Flows, forms, onboarding, accessibility, microcopy |
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

### MCP connectors worth wiring up

| Connector | Where | What it unlocks |
|---|---|---|
| **Playwright** | Claude Code | The verification loop above. The highest-value thing on this page |
| **Vercel** | Cowork | Deploy status, build logs, runtime errors. Sync uses these first |
| **Supabase** | Both | Real schema, migrations, RLS advisors and logs, instead of guessing |
| **GitHub** | Both | Commits, PRs, issues |
| **context7** | Claude Code | Current docs for third-party libraries, instead of working from memory |
| **Figma** | Cowork | Turns a design into buildable spec, and pushes built screens back |

### Plugins you might already have

**[obra/superpowers](https://github.com/obra/superpowers)** is worth calling out, because
it and Build Mode overlap, and it wins by default if you do nothing.

Superpowers loads itself on every session through a startup hook, including after every
`/clear`. Its entry skill routes "let's build X" into a brainstorming skill that hard-gates
on you approving a design. So you type `/next`, and instead of building the task Build Mode
already scoped, Claude Code opens a design conversation about decisions made days ago.

That is not a bug in either one. Superpowers assumes the person driving Claude Code is a
developer who has not planned yet. Build Mode assumes the planning already happened,
somewhere else, with the person who could not do it alone.

Build Mode resolves it in the `CLAUDE.md` it writes: skip `brainstorming` and
`writing-plans`, keep `verification-before-completion`, `systematic-debugging` and
`requesting-code-review`. Those three are genuinely excellent for someone who cannot
review code, so this is a merge rather than a fight.

If you do not have superpowers, Build Mode leaves the block out entirely.

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
   -> Claude Code : plans, builds, then /checkpoint - builds it, clicks through it
                    in a browser, commits and pushes. Ticks it off only if that passed
   -> You         : back in Cowork, "done" / "it broke" / "next"
   -> Cowork      : checks the deploy, opens the app, verifies against DESIGN.md,
                    learns, writes the next prompt
```

Two places check, and they check different things. Claude Code proves the code runs,
because it is the only side that can execute anything. Cowork judges whether what runs is
what was asked for, because it wrote the spec. You are the last resort, not the first, and
you get one judgement question rather than a test plan.

Nothing in that loop depends on you remembering anything. The repo carries the state.

### Eight modes

It reads what you said and picks one:

| You say | Mode |
|---|---|
| "I want to build X", "new project", "set up the repo" | **Kickoff** |
| "next", "what do I tell it now", "give me the prompt" | **Next task** |
| "done", "it built it", "check this", "where am I" | **Sync** |
| "it's stuck", "this error", "it built the wrong thing" | **Unblock** |
| "put it back", "undo that", "it was working yesterday" | **Undo** |
| "I want to add", "actually let's change", "drop that" | **Replan** |
| "make this look better", "the UI is generic" | **Design pass** |
| "upgrade my project files", "refresh the setup" | **Upgrade** |

---

## What it writes into your repo

| File | What it holds |
|---|---|
| `CLAUDE.md` | How this project works. Claude Code loads it every session |
| `docs/ROADMAP.md` | Milestones and tasks with permanent IDs |
| `docs/PROGRESS.md` | What actually got built, with commit SHAs, newest first |
| `docs/LEARNINGS.md` | Anything that cost more than ten minutes to figure out |
| `docs/ARCHITECTURE.md` | Stack call, schema, decisions and why |
| `docs/DESIGN.md` | Fonts, hex codes, spacing scale, component rules, motion |
| `docs/TOOLING.md` | What is actually available, so plans do not assume tools you lack |
| `docs/next-prompt.md` | The current prompt, overwritten each time |

It also sets up `.claude/skills/` in the repo so you get `/next`, `/checkpoint` and
`/blocked` inside Claude Code, plus a `paths:`-scoped design rule that only loads when
Claude Code touches a UI file.

---

## What is actually in here

Eight files. The bulk of it is not prose, it is specifics:

- **`SKILL.md`** - the eight modes, the memory rules, the folder discipline
- **`references/stack-picker.md`** - default stack and when to deviate, with costs
- **`references/design-brief.md`** - why AI-built apps all look the same, and the fix
- **`references/doc-pack.md`** - exact templates for every document
- **`references/prompt-recipes.md`** - the six-part prompt shape, plus worked recipes for
  a new screen, a third-party integration, an LLM feature and a bug report
- **`references/claude-code-setup.md`** - verified `CLAUDE.md`, `.claude/rules/` and skill
  frontmatter syntax, permission modes, the superpowers handling, the traps that fail
  silently
- **`references/tooling.md`** - the capability scan and the skill-visibility gap
- **`references/memory.md`** - what to write down and what to leave out

---

## Who this is for

Someone who ships, understands systems, and has ideas worth building, but who will not
catch a bad architectural call by reading a diff. Build Mode assumes that by default:
it decides for you, explains in one line why, keeps prompts self-contained, and verifies
the work rather than trusting a "done" message.

Tell it you read code fluently and it dials back the explaining.

---

## Notes

- The prompt recipes and the generated `/checkpoint` assume Node and npm. Build Mode
  adapts the commands for a Python, Swift or other stack, but the templates read as
  JavaScript.
- Costs are quoted in pounds per month. Change that line in `SKILL.md` if you want
  another currency.
- Verified against Claude Code as of August 2026. It moves fast, so check the live docs at
  [code.claude.com](https://code.claude.com/docs) if something behaves differently to the
  `claude-code-setup.md` reference.

## Licence

MIT. See [LICENSE](LICENSE).
