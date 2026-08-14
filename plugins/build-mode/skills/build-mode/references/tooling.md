# Tooling - what is available and how to use it

Two failure modes this file exists to prevent. One: planning around a tool the user does not have, so the plan quietly does not work. Two: Claude Code hand-rolling something an installed skill already does properly, because nobody told it the skill was there. Both are cheap to avoid and expensive to discover late.

## The two sides

Cowork and Claude Code load skills from different places. Cowork uses the skills enabled on the user's claude.ai account. Claude Code uses `~/.claude/skills/` on their machine plus the project's `.claude/skills/`. Neither can see the other's, and you cannot copy one across.

**So the inventory is a thing you establish, not assume.** Do it once at kickoff, write it into `docs/TOOLING.md`, and never guess again in that project.

**Where both sides have a skill, name it directly in prompts.** "Use `ui-ux-pro-max` for the component patterns" is a stronger instruction than "if `ui-ux-pro-max` is installed, use it, otherwise work from DESIGN.md" - the hedge earns nothing once you have confirmed it is there, and a hedged prompt reliably produces "I'll use the skill" followed by improvisation.

**Where only Cowork has it, bake the output in instead.** This is the single most useful thing this skill does. Use the design skills here, in Cowork, and write their actual conclusions - the hex values, the type scale, the spacing steps - into `docs/DESIGN.md`. Use the database skills here and write the resulting rules into `docs/ARCHITECTURE.md`. Claude Code then reads a document with real numbers in it and does not need the skill at all.

Do this even when both sides have the skill. A document beats a skill call: it is already decided, it cannot drift between sessions, and it survives a compact. The skill on the Claude Code side becomes a second pair of hands rather than the only source.

**Where neither side has it, say so and give the fallback in the same breath.** A named gap with a workaround is fine. A silent gap is what wrecks a plan.

## The one dependency that changes what this skill can promise

**A browser-driving tool on the Claude Code side.** Either the Playwright MCP or the `webapp-testing` skill. This is what lets `/checkpoint` click through the work and prove a task is done, rather than assert it. Nothing in Cowork can do this: Cowork has no route to localhost on the user's machine, so the only side that can run the app is the side that lives on it.

Check for it at kickoff and record the answer.

- **Present.** Generate `/checkpoint` with the browser walk in, as written in `claude-code-setup.md`. This is the intended setup.
- **Absent.** Generate `/checkpoint` with build and typecheck only, and **tell the user plainly what that costs**: Claude Code can now prove the code compiles but not that the feature works, so every Done-when tick lands back on them to click, once, manually. Then offer the fix in the same message - installing the Playwright MCP is a few minutes and it is the highest-value thing they can add to this workflow. Do not quietly generate a weaker `/checkpoint` and say nothing.

Never write a check into `/checkpoint` that the machine cannot run. A command that errors every time trains the user to ignore the output, which is worse than not having the check.

## Running the scan

**What is connected here.** `ListSkills` for the skills on their account, `ListConnectors` for the MCP servers wired into this session. If a job needs a connector they do not have, `SearchMcpRegistry` then `SuggestConnectors` rather than working around the gap.

**What the project already has.** If the project folder is connected, list it:

```bash
ls -la /sessions/<session>/mnt/<project>/.claude/skills/ 2>/dev/null
cat  /sessions/<session>/mnt/<project>/.mcp.json 2>/dev/null
```

Two path traps worth remembering. `device_list_dir` and `device_stage_files` take the real path on the user's machine (`/Users/<name>/...` or equivalent). `device_bash` runs in a Linux VM where connected folders are mounted at `/sessions/<session>/mnt/<folder-name>` - run `ls mnt/` to get the exact name. And that VM has **no network**, so git reads, node and python work, but `npm install`, `git push` and anything that fetches do not.

**What is on their machine globally.** You cannot see it. `device_bash` only reaches connected folders, not their home directory, so `~/.claude/skills/` and `~/.claude.json` are out of reach. Ask once at kickoff, write the answer into `docs/TOOLING.md`, and never ask again in that project. That inventory - accounts, CLIs, which MCPs Claude Code has - is the most valuable thing in the file, precisely because it is the part you cannot look up.

## docs/TOOLING.md

Write what you found into the project so both sides can read it. Keep it short; it is a lookup, not an essay.

```markdown
# Tooling

## MCP servers
| Server | Available to | Use it for |
|---|---|---|
| Playwright | Claude Code | Driving a real browser. This is how `/checkpoint` proves a task is done. |
| Supabase | Both | Schema, migrations, RLS advisors, logs. Ask it rather than guessing at the database. |
| Vercel | Both | Deploys, build logs, runtime errors. Check here first when a deploy fails. |
| GitHub | Both | Commits, PRs, issues. |
| context7 | Claude Code | Current docs for any third-party library. |

## Plugins
Ask whether the user has any Claude Code plugins enabled, and check `/plugin` output if
they can paste it. **obra/superpowers** matters most: it is common, it loads itself on
every session, and it will re-open design questions this skill already settled unless
`CLAUDE.md` tells it not to. See `claude-code-setup.md` for the full handling.

## Skills
| Skill | Cowork | Claude Code |
|---|---|---|
| <e.g. ui-ux-pro-max> | yes | yes |
| <e.g. dataviz> | yes | no - bake values into DESIGN.md |
Name the both-sides ones directly in prompts. No hedging.

## Accounts
Supabase: project `<ref>`, free tier
Vercel: hobby, connected to the GitHub repo
Domain: <where it is registered>

## Not available
<Things neither side has, and what to do instead. Be specific: "no browser tool on the
Claude Code side, so /checkpoint builds and typechecks only and the user clicks the rest."
This section stops you re-proposing the same missing thing every session.>
```

The **Not available** section earns its place. It stops you re-proposing the same missing thing every session.

## Naming tools in prompts

Add one line to every prompt you hand over. This is the highest-leverage line in the whole skill. Claude Code will not go looking for a skill nobody reminded it of, so an install full of good skills sits unused while it hand-rolls a colour palette from scratch. If the user's Claude Code shows usage counts per skill, look: the ones that get named get used, and the rest read "never used".

```
**Use these:** `docs/DESIGN.md` for the palette and type scale - the values there are
already decided, do not invent new ones. `ui-ux-pro-max` for the component patterns
and `ux-designer` for the empty state and the error copy.
```

Point at the document first, since that always exists and is already decided. Then name the skills, plainly, no conditionals.

Rough mapping from task type to what to name:

| Task | Name these |
|---|---|
| Any screen or component | `ui-ux-pro-max`, `frontend-design`, plus `docs/DESIGN.md` |
| Forms, onboarding, empty states, error copy | `ux-designer` |
| Reviewing UI that already exists | `web-design-guidelines` |
| Database, auth, RLS, migrations | `supabase`, `supabase-postgres-best-practices`, Supabase MCP |
| React or Next.js structure and performance | `vercel-react-best-practices`, `vercel-composition-patterns` |
| Page transitions and animation | `vercel-react-view-transitions` |
| Native iOS or macOS | `apple-hig` |
| Checking it works before saying done | Playwright MCP, `webapp-testing` |
| Any third-party library or API | `context7` MCP for the current docs |
| Charts, dashboards, KPI tiles | `ui-ux-pro-max`, plus the chart values already in `docs/DESIGN.md` |

For MCP work, name the tool, not just the server: "use the Supabase MCP `get_advisors` to check the RLS policies" beats "use Supabase".

## Wiring MCP servers into Claude Code

A `.mcp.json` in the project root makes servers available to Claude Code the way they are to you here. Worth setting up at kickoff for Supabase and Vercel - it is the difference between Claude Code reading the real schema and Claude Code guessing at it.

The user has to add the credentials themselves. Give them the file and tell them which environment variables to fill in; never write a key into a file that gets committed. Check the current config shape at `/docs/en/mcp` before generating one, since this format has changed before.

## When the tool is missing

Say so plainly and give the fallback in the same breath. "There is no Stripe connector wired up, so I cannot check live payments. I will plan the integration from the docs and you can verify the first test payment yourself." A named gap with a workaround is fine. A silent gap is what wrecks a plan.
