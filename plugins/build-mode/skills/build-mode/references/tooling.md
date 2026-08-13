# Tooling - what is available and how to use it

Two failure modes this file exists to prevent. One: planning around a tool the user does not have, so the plan quietly does not work. Two: Claude Code hand-rolling something an installed skill already does properly, because nobody told it the skill was there. Both are cheap to avoid and expensive to discover late.

## The thing that confuses everyone

**Cowork and Claude Code do not see the same skills.**

- **Cowork** (here) loads the skills enabled on the user's claude.ai account, synced at session start. It does **not** read `~/.claude/skills/` on their own machine.
- **Claude Code** (the CLI on their Mac) loads `~/.claude/skills/` and the project's `.claude/skills/`. It does **not** see their claude.ai account skills.

So `ui-ux-pro-max` being available to you here says nothing about whether Claude Code can use it. If a skill matters to the build, it has to exist on the Claude Code side too.

Be careful not to paper over this. You cannot copy a skill across from here: `ui-ux-pro-max` lives on the user's claude.ai account, not on a filesystem you can reach, and `device_bash` cannot see their home directory. There is no one-line fix, so do not promise one.

**The reliable answer is to bake the output in rather than depend on the skill.** Use `ui-ux-pro-max` here, in Cowork, and write its actual conclusions - the hex values, the type scale, the spacing steps - into `docs/DESIGN.md`. Claude Code then reads a document with real values in it and does not need the skill at all. Same for architecture: use `supabase-postgres-best-practices` here, and write the resulting rules into `docs/ARCHITECTURE.md`. This is the single most useful thing this skill does, and it is why the design step happens in Cowork rather than being delegated.

Then, as a bonus rather than a dependency: ask the user once whether these skills are also installed on the Claude Code side, and record the answer in `docs/TOOLING.md`. If they are, name them in prompts. If they are not, they can add them from a plugin marketplace when they feel like it, and nothing breaks in the meantime.

The same split applies to MCP servers. Yours are connected to this Cowork session. Claude Code reads its own from `~/.claude.json` and the project's `.mcp.json`.

## Running the scan

**What is connected here.** `ListSkills` for the skills on their account, `ListConnectors` for the MCP servers wired into this session. If a job needs a connector they do not have, `SearchMcpRegistry` then `SuggestConnectors` rather than working around the gap.

**What the project already has.** If the project folder is connected, list it:

```bash
ls -la /sessions/<session>/mnt/<project>/.claude/skills/ 2>/dev/null
cat  /sessions/<session>/mnt/<project>/.mcp.json 2>/dev/null
```

Two path traps worth remembering. `device_list_dir` and `device_stage_files` take the real path on the user's own machine (`/Users/<name>/...` on a Mac, `/home/<name>/...` on Linux). `device_bash` runs in a Linux VM where connected folders are mounted at `/sessions/<session>/mnt/<folder-name>` - run `ls mnt/` to get the exact name. And that VM has **no network**, so git reads, node and python work, but `npm install`, `git push` and anything that fetches do not.

**What is on their Mac globally.** You cannot see it. `device_bash` only reaches connected folders, not their home directory, so `~/.claude/skills/` and `~/.claude.json` are out of reach. Ask once at kickoff, write the answer into `docs/TOOLING.md`, and never ask again in that project. That inventory - accounts, CLIs, which MCPs Claude Code has - is the most valuable thing in the file, precisely because it is the part you cannot look up.

## docs/TOOLING.md

Write what you found into the project so both sides can read it. Keep it short; it is a lookup, not an essay.

```markdown
# Tooling

## MCP servers
| Server | Available to | Use it for |
|---|---|---|
| Supabase | Cowork + Claude Code | Schema, migrations, RLS advisors, logs. Ask it rather than guessing at the database. |
| Vercel | Cowork | Deploys, build logs, runtime errors. Check here first when a deploy fails. |
| GitHub | Cowork | Commits, PRs, issues. |

## Claude Code skills in this project
| Skill | Use it for |
|---|---|
| `ui-ux-pro-max` | Colours, type pairing, spacing, motion, product patterns |
| `supabase` | Anything touching Supabase client or auth |

## Accounts
Supabase: project `<ref>`, free tier
Vercel: hobby, connected to the GitHub repo
Domain: <where it is registered>

## Not available
<Things deliberately not set up, and what to do instead.>
```

The **Not available** section earns its place. It stops you re-proposing the same missing thing every session.

## Naming tools in prompts

Add one line to every prompt you hand over. Claude Code will not go looking for a skill it has not been reminded of, and a skill's whole value is that someone already worked out the right answer.

```
**Use these:** `docs/DESIGN.md` for the palette and type scale - the values there are
already decided, do not invent new ones. If the `ui-ux-pro-max` skill is installed,
use it for the component patterns; if not, work from DESIGN.md alone.
```

Note the hedge. Name a skill Claude Code may not have and you get "I'll use the ui-ux-pro-max skill" followed by improvisation, which is worse than naming nothing. Point at the document first, since that always exists, and treat the skill as an upgrade. Only drop the hedge for a skill you have confirmed is installed.

Rough mapping from task type to what to name:

| Task | Name these |
|---|---|
| Any screen or component | `ui-ux-pro-max`, `frontend-design`, plus `docs/DESIGN.md` |
| Forms, onboarding, empty states, error copy | `ux-designer` |
| Reviewing UI that already exists | `web-design-guidelines` |
| Charts, dashboards, KPI tiles | `dataviz` |
| Database, auth, RLS, migrations | `supabase`, `supabase-postgres-best-practices`, Supabase MCP |
| React or Next.js structure and performance | `vercel-react-best-practices`, `vercel-composition-patterns` |
| Page transitions and animation | `vercel-react-view-transitions` |
| Native iOS or macOS | `apple-hig` |
| Checking it works before saying done | `webapp-testing` |

For MCP work, name the tool, not just the server: "use the Supabase MCP `get_advisors` to check the RLS policies" beats "use Supabase".

## Wiring MCP servers into Claude Code

A `.mcp.json` in the project root makes servers available to Claude Code the way they are to you here. Worth setting up at kickoff for Supabase and Vercel - it is the difference between Claude Code reading the real schema and Claude Code guessing at it.

The user has to add the credentials themselves. Give them the file and tell them which environment variables to fill in; never write a key into a file that gets committed. Check the current config shape at `/docs/en/mcp` before generating one, since this format has changed before.

## When the tool is missing

Say so plainly and give the fallback in the same breath. "There is no Stripe connector wired up, so I cannot check live payments. I will plan the integration from the docs and you can verify the first test payment yourself." A named gap with a workaround is fine. A silent gap is what wrecks a plan.
